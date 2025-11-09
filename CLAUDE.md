# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Repository Overview

This is a documentation repository for managing a multi-server self-hosted infrastructure using Ubuntu Server, Docker, Portainer, and Traefik. It contains configuration files, deployment guides, and operational procedures for running containerized services across two servers.

**Architecture:**
- **svc-01 (Primary Server)**: Runs Traefik reverse proxy, n8n workflow automation, Homepage dashboard, Uptime Kuma monitoring, and Gitea
- **svc-02 (Database Server)**: Hosts PostgreSQL, pgAdmin, Redis, Redis Commander, and Qdrant vector database

**Key Technologies:**
- Docker Compose for container orchestration
- Traefik for SSL termination and reverse proxy (using Cloudflare DNS challenge)
- Portainer for container management
- Cloudflare for DNS and wildcard SSL certificates

## Directory Structure

```
/
├── svc-01/          # Docker Compose files for primary server services
│   ├── traefik.yml
│   ├── n8n.yml
│   ├── homepage-.yml
│   ├── uptime-kuma.yml
│   └── gitea.yaml
├── svc-02/          # Docker Compose files for database server
│   └── databases.yml
└── *.md             # Documentation files
```

## Common Operations

### Deploying Services via Portainer

Services are deployed through Portainer's web interface, not via command line:

1. Access Portainer at `https://portainer-01.local.yourdomain.com` (svc-01) or `https://portainer-02.local.yourdomain.com` (svc-02)
2. Navigate to "Stacks" → "Add stack"
3. Upload the appropriate YAML file from `svc-01/` or `svc-02/`
4. Configure environment variables in Portainer's "Environment variables" section
5. Deploy the stack

**Important**: Service management is done through Portainer UI, not `docker-compose` CLI commands.

### Docker Commands

```bash
# Monitor all containers
docker stats

# Check container status with health
docker ps --format "table {{.Names}}\t{{.Status}}\t{{.Ports}}"

# View container logs (last 100 lines)
docker logs --tail 100 <container_name>

# Restart a service
docker restart <container_name>

# Execute command in container
docker exec -it <container_name> <command>
```

### Database Operations

**PostgreSQL:**
```bash
# Connect to PostgreSQL
docker exec -it postgres psql -U <username> -d <database>

# Backup database
docker exec postgres pg_dumpall -U postgres > backup.sql

# Restore database
docker exec -i postgres psql -U postgres postgres < backup.sql

# Test connectivity
docker exec postgres pg_isready -U <username> -d <database>
```

**Redis:**
```bash
# Test Redis connectivity
docker exec redis redis-cli ping

# Access Redis CLI
docker exec -it redis redis-cli

# Monitor Redis commands
docker exec redis redis-cli monitor
```

**Qdrant:**
```bash
# Check Qdrant health
curl http://192.168.1.19:6333/health

# Create snapshot
curl -X POST http://192.168.1.19:6333/snapshots
```

### Backup and Restore

**Creating backups:**
```bash
# Database backup
docker exec postgres pg_dumpall -U postgres > ~/backups/$(date +%Y-%m-%d)/postgres_backup.sql
gzip ~/backups/$(date +%Y-%m-%d)/postgres_backup.sql

# n8n workflows export
docker exec n8n n8n export:workflows --output=/tmp/workflows.json
docker cp n8n:/tmp/workflows.json ~/backups/$(date +%Y-%m-%d)/

# Encrypt backup
gpg -c backup_file.tar.gz
```

**Restoring backups:**
```bash
# Decrypt backup
gpg --decrypt backup.tar.gz.gpg > backup.tar.gz

# Restore PostgreSQL
docker exec -i postgres psql -U postgres postgres < postgres_backup.sql

# Import n8n workflows
docker cp workflows.json n8n:/tmp/
docker exec n8n n8n import:workflows --input=/tmp/workflows.json
```

### Network and Traefik

**Important Network Details:**
- Traefik network: `traefik-public` (external network, must be created before deploying Traefik)
- Database network: `app_network` (internal bridge network)
- DNS pattern: `*.local.yourdomain.com` points to server IPs (configured in Cloudflare)

**Create Traefik network:**
```bash
docker network create traefik-public
```

**Generate Traefik dashboard credentials:**
```bash
echo $(htpasswd -nb <USERNAME> <PASSWORD>) | sed -e s/\\$/\\$\\$/g
```

**Check Traefik logs:**
```bash
docker logs traefik
```

### System Maintenance

**Update packages:**
```bash
sudo apt update && sudo apt upgrade -y
sudo apt autoremove -y && sudo apt clean
```

**Docker cleanup:**
```bash
# Remove unused images
docker image prune -a

# Remove unused volumes (CAUTION: may delete data)
docker volume prune

# Remove unused networks
docker network prune

# System-wide cleanup
docker system prune -a --volumes
```

**Monitor disk usage:**
```bash
df -h                          # Overall disk usage
docker system df               # Docker disk usage
ncdu /var/lib/docker          # Interactive disk analyzer
```

## Important Configuration Details

### Traefik Configuration (svc-01/traefik.yml)

- Uses Cloudflare DNS challenge for SSL certificates
- Required environment variables: `CF_API_EMAIL`, `CF_DNS_API_TOKEN`, `TRAEFIK_HOST`, `TRAEFIK_AUTH_USER`
- Automatically redirects HTTP → HTTPS
- Dynamic configuration loaded from `/home/charizard10/traefik/dynamic.yml`
- Exposes ports 80 and 443

### Database Stack (svc-02/databases.yml)

- PostgreSQL exposed on port 5432 (accessible from svc-01)
- pgAdmin on port 5050
- Redis on port 6379
- Redis Commander on port 8081
- Qdrant on ports 6333 (HTTP) and 6334 (gRPC)
- All services include resource limits (memory/CPU)

### n8n Configuration (svc-01/n8n.yml)

- Connects to PostgreSQL on svc-02 via IP address (192.168.1.19)
- Required env vars: `POSTGRES_HOST`, `POSTGRES_DB`, `POSTGRES_NON_ROOT_USER`, `POSTGRES_NON_ROOT_PASSWORD`, `N8N_ENCRYPTION_KEY`, `N8N_HOST`
- Uses separate bridge network `n8n-bridge` for cross-server database access

### Services Accessible via Traefik

Services on the same server as Traefik are configured with Docker labels. Remote services (on svc-02) are configured in Traefik's `dynamic.yml` file with direct IP:port mappings.

## Security Considerations

- All services use SSL/TLS via Traefik with Let's Encrypt certificates
- Environment variables stored in Portainer (never committed to Git)
- Traefik dashboard protected with basic authentication
- Database servers only accessible internally or through Traefik
- Firewall (ufw) restricts external access
- GPG encryption used for backup files

## Useful Advanced Tools

The repository documents several modern CLI tools in `useful-commands.md`:
- `ripgrep` (rg) - faster grep alternative
- `fd` - user-friendly find alternative
- `bat` - cat with syntax highlighting
- `lazydocker` - interactive Docker UI
- `ncdu` - interactive disk usage analyzer
- `glances` - all-in-one monitoring dashboard

Install via: `sudo apt install <tool-name>`

## Troubleshooting

**Container won't start:**
```bash
docker logs <container_name>     # Check error messages
docker-compose config            # Validate YAML syntax
docker system df                 # Check available resources
```

**Service connection issues:**
```bash
docker exec <container> ping <other_container>    # Test network connectivity
docker exec <container> env | grep VAR_NAME       # Check environment variables
docker port <container>                            # Verify port bindings
```

**SSL certificate problems:**
- Verify Cloudflare API token has DNS edit permissions
- Check DNS records point to correct IPs (gray cloud, not proxied)
- Review Traefik logs: `docker logs traefik`

**Database connection problems:**
```bash
docker exec postgres pg_isready -U user -d database    # Test PostgreSQL
docker exec redis redis-cli ping                        # Test Redis
curl http://192.168.1.19:6333/health                   # Test Qdrant
```

## Documentation Structure

- `README.md` - Project overview and table of contents
- `server-setup.md` - Ubuntu Server installation and initial configuration
- `network-configuration.md` - Traefik, SSL, DNS, and Portainer setup
- `services-deployment.md` - Service deployment via Portainer with detailed configurations
- `backup-restoration.md` - Backup creation, encryption, and restoration procedures
- `maintenance.md` - Ongoing maintenance tasks and monitoring
- `useful-commands.md` - Modern CLI tools reference
- `ubuntu-server-setup.md` - Detailed Ubuntu Server setup guide
- `ubuntu-desktop-setup.md` - Ubuntu desktop environment setup

## Key Points for AI Assistants

1. **Never suggest `docker-compose` CLI commands for deployment** - all deployments are done through Portainer's web UI
2. **Services span two physical servers** - svc-01 and svc-02 communicate over local network (192.168.1.x)
3. **Environment variables are sensitive** - they're stored in Portainer or encrypted files, never in Git
4. **Traefik is the single entry point** - all HTTPS traffic goes through Traefik on svc-01
5. **Backups must be encrypted** - always use GPG encryption before storing backups
6. **Domain pattern is `*.local.yourdomain.com`** - this is the standard for all services
7. **Inter-server communication** - n8n on svc-01 connects to PostgreSQL on svc-02 via IP address
