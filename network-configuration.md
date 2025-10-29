# Network Configuration Guide

This guide covers setting up a private network infrastructure with SSL termination, DNS routing, and secure access to self-hosted services.

## Prerequisites

- Domain name registered with Cloudflare
- Two Ubuntu servers configured and accessible
- Cloudflare account with API access
- Basic understanding of DNS and SSL certificates

## Cloudflare DNS Configuration

### Obtain Cloudflare API Token

1. Go to [Cloudflare API Tokens](https://dash.cloudflare.com/profile/api-tokens)
2. Click "Create Token"
3. Use the "Edit zone DNS" template
4. Configure permissions:
   - Zone: DNS → Edit
   - Zone Resources: Include → Specific zone → yourdomain.com
5. Click "Continue to summary" → "Create Token"
6. **Copy the token immediately** (it won't be shown again)

### Create DNS Records

1. Go to [Cloudflare Dashboard](https://dash.cloudflare.com)
2. Select your domain
3. Click "DNS" → "Records"
4. Click "Add record"
5. Configure wildcard subdomain:
   - **Type**: A
   - **Name**: *.local (exactly as shown)
   - **IPv4 address**: Your primary server's local IP (e.g., 192.168.1.100)
   - **Proxy status**: Disabled (gray cloud)
   - **TTL**: Auto
6. Click "Save"

**Result**: `*.local.yourdomain.com` now points to your servers.

## Traefik Reverse Proxy Setup

### Create Traefik Network

On the server designated to host Traefik (typically svc-01):

```bash
docker network create traefik-public
```

### Generate Authentication Credentials

Create username and password for Traefik dashboard access:

```bash
echo $(htpasswd -nb <USERNAME> <PASSWORD>) | sed -e s/\\$/\\$\\$/g
```

Save this hash for the `TRAEFIK_AUTH_USER` environment variable.

### Deploy Traefik Stack

1. Create a new stack in Portainer named "traefik"
2. Upload or paste the contents of `svc-01/traefik.yml`
3. Set required environment variables:
   - `CF_API_EMAIL`: Your Cloudflare email
   - `CF_DNS_API_TOKEN`: The API token from step 1
   - `TRAEFIK_HOST`: traefik.local.yourdomain.com
   - `TRAEFIK_AUTH_USER`: The hash from step above
4. Deploy the stack

### Verify Traefik Dashboard

1. Open browser to: `https://traefik.local.yourdomain.com`
2. Enter the username and password from earlier
3. Verify SSL certificate (green lock icon)
4. Explore the dashboard to see configured routes

## Configure Local Services with Traefik

### Services on Traefik Server (svc-01)

For services running on the same server as Traefik, add these labels to your Docker Compose:

```yaml
networks:
  - traefik-public
labels:
  - "traefik.enable=true"
  - "traefik.http.routers.service.rule=Host(`service.local.yourdomain.com`)"
  - "traefik.http.routers.service.entrypoints=websecure"
  - "traefik.http.routers.service.tls.certresolver=cloudflare"
  - "traefik.http.services.service.loadbalancer.server.port=PORT"
  - "traefik.docker.network=traefik-public"

networks:
  traefik-public:
    external: true
```

Example configurations:

#### Homepage Dashboard
```yaml
services:
  homepage:
    image: ghcr.io/gethomepage/homepage:latest
    container_name: homepage
    restart: unless-stopped
    ports:
      - "3000:3000"
    volumes:
      - homepage-config:/app/config
      - /var/run/docker.sock:/var/run/docker.sock:ro
    environment:
      - TZ=America/New_York
      - PUID=1000
      - PGID=1000
      - HOMEPAGE_ALLOWED_HOSTS=home.local.yourdomain.com
    networks:
      - traefik-public
    labels:
      - "traefik.enable=true"
      - "traefik.http.routers.homepage.rule=Host(`home.local.yourdomain.com`)"
      - "traefik.http.routers.homepage.entrypoints=websecure"
      - "traefik.http.routers.homepage.tls.certresolver=cloudflare"
      - "traefik.http.services.homepage.loadbalancer.server.port=3000"
      - "traefik.docker.network=traefik-public"

volumes:
  homepage-config:
    driver: local

networks:
  traefik-public:
    external: true
```

#### Uptime Kuma
```yaml
services:
  uptime-kuma:
    image: louislam/uptime-kuma:1
    container_name: uptime-kuma
    restart: unless-stopped
    volumes:
      - uptime-kuma-data:/app/data
    environment:
      - TZ=America/New_York
    networks:
      - traefik-public
    labels:
      - "traefik.enable=true"
      - "traefik.http.routers.kuma.rule=Host(`kuma.local.yourdomain.com`)"
      - "traefik.http.routers.kuma.entrypoints=websecure"
      - "traefik.http.routers.kuma.tls.certresolver=cloudflare"
      - "traefik.http.services.kuma.loadbalancer.server.port=3001"
      - "traefik.docker.network=traefik-public"

volumes:
  uptime-kuma-data:
    driver: local

networks:
  traefik-public:
    external: true
```

### Services on Remote Servers

For services running on different servers (like databases on svc-02), configure Traefik's file provider.

#### Create Dynamic Configuration File

On the Traefik server, create the dynamic configuration file:

```bash
mkdir -p ~/traefik
nano ~/traefik/dynamic.yml
```

Add the following Traefik dynamic configuration:

```yaml
http:
  routers:
    portainer-svc01:
      rule: "Host(`portainer-01.local.dagdappshub.com`)"
      entryPoints:
        - websecure
      service: portainer-svc-01
      tls:
        certResolver: cloudflare

    portainer-svc02:
      rule: "Host(`portainer-02.local.dagdappshub.com`)"
      entryPoints:
        - websecure
      service: portainer-svc-02
      tls:
        certResolver: cloudflare

    pgadmin:
      rule: "Host(`pgadmin.local.dagdappshub.com`)"
      entryPoints:
        - websecure
      service: pgadmin-svc
      tls:
        certResolver: cloudflare

    redis:
      rule: "Host(`redis.local.dagdappshub.com`)"
      entryPoints:
        - websecure
      service: redis-svc
      tls:
        certResolver: cloudflare

    qdrant:
      rule: "Host(`qdrant.local.dagdappshub.com`)"
      entryPoints:
        - websecure
      service: qdrant-svc
      tls:
        certResolver: cloudflare

  services:
    portainer-svc-01:
      loadBalancer:
        servers:
          - url: "http://192.168.1.254:9000"

    portainer-svc-02:
      loadBalancer:
        servers:
          - url: "http://192.168.1.19:9000"

    pgadmin-svc:
      loadBalancer:
        servers:
          - url: "http://192.168.1.19:5050"

    redis-svc:
      loadBalancer:
        servers:
          - url: "http://192.168.1.19:8081"

    qdrant-svc:
      loadBalancer:
        servers:
          - url: "http://192.168.1.19:6333"
```

**Note**: This configuration includes specific IP addresses and domain names for your setup. Update as needed for your environment.

#### Update Traefik Configuration

Ensure the Traefik stack includes the dynamic configuration volume:

```yaml
volumes:
  - /var/run/docker.sock:/var/run/docker.sock:ro
  - traefik-certs:/letsencrypt
  - /home/username/traefik/dynamic.yml:/etc/traefik/dynamic.yml:ro
```

## Portainer Configuration

### Install Portainer

Deploy Portainer using the following Docker command:

```bash
docker run -d --name portainer --restart always -p 9000:9000 -p 9443:9443 -v /var/run/docker.sock:/var/run/docker.sock -v portainer_data:/data portainer/portainer-ce:latest
```

**Note**: This command deploys Portainer with default settings. For production environments, consider additional security configurations and network isolation.

Access Portainer at `https://server-ip:9443`


## Testing Configuration

### Test SSL Certificates

1. Visit each service URL
2. Verify green lock icon (valid SSL)
3. Check certificate details (issued by Let's Encrypt)

### Test Service Accessibility

1. Access Portainer to verify services are running
2. Test individual service URLs directly
3. Verify services load correctly
4. Test functionality of each service

### Network Troubleshooting

```bash
# Check Traefik logs
docker logs traefik

# Test DNS resolution
nslookup service.local.yourdomain.com

# Test connectivity to remote services
curl -I http://remote-server-ip:port

# Check firewall rules
sudo ufw status
```

## Security Considerations

- SSL certificates auto-renew via Let's Encrypt
- All traffic encrypted with TLS 1.2+
- Basic authentication on Traefik dashboard
- Firewall configured to restrict access
- DNS challenge prevents port 80/443 exposure

## Common Issues

### SSL Certificate Problems
- Check Cloudflare API token permissions
- Verify DNS records are correct
- Ensure domain is not proxied in Cloudflare

### Service Not Accessible
- Check if service is running: `docker ps`
- Verify Traefik labels in compose file
- Check service logs: `docker logs <service>`
- Confirm network connectivity between servers

### Homepage Not Updating
- Restart homepage container after config changes
- Check file permissions on config volume
- Verify YAML syntax is correct

---

**Domain**: Replace `yourdomain.com` with your actual domain
**IPs**: Update all IP addresses to match your server configuration
**Ports**: Verify service ports match your deployments

**Created**: October 2025