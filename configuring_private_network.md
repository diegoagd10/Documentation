# Private Network Configuration

This guide covers setting up a private network infrastructure with SSL termination, DNS routing, and secure access to self-hosted services.

## Prerequisites

- Domain name registered with Cloudflare
- Two Ubuntu servers configured and accessible
- Cloudflare account with API access
- Basic understanding of DNS and SSL certificates

## Steps

1. **Get a domain** from a registrar and configure it with Cloudflare

2. **PASO 1: Configurar Cloudflare**

   **1.1 Obtener API Token**
   - Go to: https://dash.cloudflare.com/profile/api-tokens
   - Click "Create Token"
   - Use the template "Edit zone DNS"
   - Configure permissions:
     - Zone → DNS → Edit
     - Zone Resources: Include → Specific zone → <YOUR_DOMAIN>
   - Click "Continue to summary" → "Create Token"
   - **COPY THE TOKEN** (shown only once)

   **1.2 Crear registros DNS**
   - Go to: https://dash.cloudflare.com
   - Select your domain <YOUR_DOMAIN>.com
   - Click "DNS" → "Records"
   - Click "Add record"
   - Configure:
     - Type: A
     - Name: *.local (exactly as shown)
     - IPv4 address: 192.168.1.100 (your server's local IP)
     - Proxy status: ⚪ DISABLED (gray cloud, not orange)
     - TTL: Auto
   - Click "Save"

   ✅ Now *.local.<YOUR_DOMAIN>.com points to your local server.

   Repeat the above for each local server you have.

3. **Connect to server** which will host Traefik and execute:
   ```bash
   docker network create traefik-public
   ```

4. **Create a user and password** with the following bash snippet:
   ```bash
   echo $(htpasswd -nb <USER> <PASSWORD>) | sed -e s/\\$/\\$\\$/g
   ```

   You will need this to configure the TRAEFIK_AUTH_USER environment variable.

5. **Create a stack on Portainer** for Traefik and use `traefik.yml` to configure it and set all environment variables.

6. **PASO 6: Probar Traefik Dashboard**
   - Open your browser
   - Go to: https://traefik.local.<YOUR_DOMAIN>
   - Username: <USER>
   - Password: <PASSWORD>
   - ✅ You should see the Traefik dashboard with valid SSL certificate (green lock)

7. **Configuring local apps** on server where Traefik got installed with:
   ```yaml
   networks:
     - traefik-public
   labels:
     - "traefik.enable=true"
     - "traefik.http.routers.kuma.rule=Host(`kuma.local.<YOUR_DOMAIN>`)"
     - "traefik.http.routers.kuma.entrypoints=websecure"
     - "traefik.http.routers.kuma.tls.certresolver=cloudflare"
     - "traefik.http.services.kuma.loadbalancer.server.port=3001"
     - "traefik.docker.network=traefik-public"

   networks:
     traefik-public:
       external: true
   ```

8. **Configuring apps external** to server where Traefik got installed.

   **8.1 Update** `~/traefik/dynamic.yml` and create the file if it doesn't exist:
   ```yaml
   http:
     routers:
       portainer-svc02:
         rule: "Host(`portainer-02.local.<YOUR_DOMAIN>`)"
         entryPoints:
           - websecure
         service: portainer-svc
         tls:
           certResolver: cloudflare

       pgadmin:
         rule: "Host(`pgadmin.local.<YOUR_DOMAIN>`)"
         entryPoints:
           - websecure
         service: pgadmin-svc
         tls:
           certResolver: cloudflare

       redis:
         rule: "Host(`redis.local.<YOUR_DOMAIN>`)"
         entryPoints:
           - websecure
         service: redis-svc
         tls:
           certResolver: cloudflare

       qdrant:
         rule: "Host(`qdrant.local.<YOUR_DOMAIN>`)"
         entryPoints:
           - websecure
         service: qdrant-svc
         tls:
           certResolver: cloudflare

     services:
       portainer-svc:
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

## Security Considerations

- SSL certificates auto-renew via Let's Encrypt
- All traffic encrypted with TLS 1.2+
- Basic authentication on Traefik dashboard
- Firewall configured to restrict access
- DNS challenge prevents port 80/443 exposure

## Troubleshooting

### SSL Certificate Problems
- Check Cloudflare API token permissions
- Verify DNS records are correct
- Ensure domain is not proxied in Cloudflare

### Service Not Accessible
- Check if service is running: `docker ps`
- Verify Traefik labels in compose file
- Check service logs: `docker logs <service>`
- Confirm network connectivity between servers

---

**Note**: Replace `<YOUR_DOMAIN>` with your actual domain name and update IP addresses to match your server configuration.

**Created**: October 2025