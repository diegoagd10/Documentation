# Services Deployment Guide

This guide covers deploying and configuring Docker-based services using Portainer, including database services, application services, and monitoring tools.

## Prerequisites

- Ubuntu servers configured with Docker and Docker Compose
- Portainer installed and accessible
- Traefik reverse proxy configured
- Network connectivity between servers

## Portainer Overview

Portainer provides a web interface for Docker container management. Use it to:

- Deploy Docker Compose stacks
- Monitor container status and logs
- Manage images and volumes
- Configure networks and secrets

### Accessing Portainer

- **Primary Server (svc-01)**: `https://portainer-01.local.yourdomain.com`
- **Secondary Server (svc-02)**: `https://portainer-02.local.yourdomain.com` or `https://server-ip:9443`

## Database Services Deployment (svc-02)

Deploy database services on the secondary server for better resource isolation.

### PostgreSQL, pgAdmin, Redis, and Qdrant Stack

1. In Portainer on svc-02, navigate to "Stacks"
2. Click "Add stack"
3. Name: `databases`
4. Upload or paste the contents of `svc-02/databases.yml`
5. Configure environment variables:

```bash
POSTGRES_USER=your_db_user
POSTGRES_PASSWORD=your_secure_password
POSTGRES_DB=your_database_name
PGADMIN_EMAIL=your_email@example.com
PGADMIN_PASSWORD=your_pgadmin_password
```

6. Deploy the stack

### Service Configuration Details

#### PostgreSQL
- **Port**: 5432 (internal only)
- **Health Check**: Enabled with `pg_isready` test
- **Resource Limits**: 1GB RAM, 0.5 CPU cores
- **Persistence**: `postgres_data` volume

#### pgAdmin
- **Port**: 80 (internal), routed through Traefik
- **Access**: `https://pgadmin.local.yourdomain.com`
- **Resource Limits**: 512MB RAM, 0.25 CPU cores
- **Depends on**: PostgreSQL container

#### Redis
- **Port**: 6379 (internal only)
- **Append-only File**: Enabled for persistence
- **Resource Limits**: 512MB RAM, 0.25 CPU cores
- **Persistence**: `redis_data` volume

#### Redis Commander
- **Port**: 8081 (internal), routed through Traefik
- **Access**: `https://redis.local.yourdomain.com`
- **Resource Limits**: 256MB RAM, 0.25 CPU cores
- **Depends on**: Redis container

#### Qdrant
- **Ports**: 6333 (HTTP), 6334 (gRPC)
- **Access**: `https://qdrant.local.yourdomain.com`
- **Resource Limits**: 1GB RAM, 0.5 CPU cores
- **Persistence**: `qdrant_storage` volume

## Application Services Deployment (svc-01)

Deploy application services on the primary server.

### n8n Workflow Automation

1. In Portainer on svc-01, create a new stack named `n8n`
2. Upload or paste the contents of `svc-01/n8n.yml`
3. Configure environment variables:

```bash
# PostgreSQL Configuration
POSTGRES_HOST=192.168.1.19  # IP of svc-02
POSTGRES_DB=your_database_name
POSTGRES_NON_ROOT_USER=your_db_user
POSTGRES_NON_ROOT_PASSWORD=your_secure_password

# Security
N8N_ENCRYPTION_KEY=your_32_character_encryption_key

# Network
N8N_HOST=n8n.local.yourdomain.com
N8N_PORT=5678

# Additional
GENERIC_TIMEZONE=America/New_York
```

4. Deploy the stack

**Note**: n8n uses a separate bridge network (`n8n-bridge`) and connects directly to the PostgreSQL server on svc-02.

### Homepage Dashboard

1. Create a stack named `homepage`
2. Upload or paste the contents of `svc-01/homepage-.yml`
3. Configure environment variables:

```bash
TZ=America/New_York
PUID=1000
PGID=1000
HOMEPAGE_ALLOWED_HOSTS=home.local.yourdomain.com
```

4. Deploy the stack

### Uptime Kuma Monitoring

1. Create a stack named `uptime-kuma`
2. Upload or paste the contents of `svc-01/uptime-kuma.yml`
3. Configure environment variables:

```bash
TZ=America/New_York
```

4. Deploy the stack

## Service Configuration and Management

### Environment Variables Management

Store sensitive environment variables securely:

1. Use Portainer's "Environment variables" section in stack configuration
2. For encrypted values, use a password manager
3. Document all required variables for each service

### Volume Management

Monitor and manage Docker volumes:

```bash
# List all volumes
docker volume ls

# Inspect volume details
docker volume inspect volume_name

# Prune unused volumes (CAUTION)
docker volume prune
```

### Network Configuration

Services use different network types:

- **Traefik Public**: External network for web-accessible services
- **Bridge Networks**: For service-to-service communication
- **Host Network**: When needed for specific port requirements

## Service-Specific Configuration

### PostgreSQL Database Setup

After deployment, connect to PostgreSQL:

```bash
# From svc-02 or via pgAdmin
docker exec -it postgres psql -U your_user -d your_database

# Create additional databases/users as needed
CREATE DATABASE n8n_db;
CREATE USER n8n_user WITH PASSWORD 'secure_password';
GRANT ALL PRIVILEGES ON DATABASE n8n_db TO n8n_user;
```

### pgAdmin Configuration

1. Access `https://pgadmin.local.yourdomain.com`
2. Login with configured email and password
3. Add server connection:
   - **Host**: postgres (container name)
   - **Port**: 5432
   - **Username/Password**: From environment variables

### Redis Configuration

Redis is configured with append-only file for persistence. Access via Redis Commander at `https://redis.local.yourdomain.com`

### Qdrant Configuration

Access Qdrant dashboard at `https://qdrant.local.yourdomain.com/dashboard#/welcome`

### n8n Configuration

1. Access n8n at the configured port (default: 5678)
2. Complete initial setup wizard
3. Configure database connection using environment variables
4. Import workflows from backups if available

### Homepage Configuration

Configure Homepage through its web interface or by editing the config files in the Docker volume.

## Monitoring and Health Checks

### Container Health Monitoring

```bash
# Check container status
docker ps

# View container logs
docker logs container_name

# Monitor resource usage
docker stats
```

### Service Health Checks

Most services include built-in health checks:

- PostgreSQL: `pg_isready` test every 30 seconds
- Other services: Basic connectivity tests

### Portainer Monitoring

Use Portainer dashboard to:
- Monitor container CPU/memory usage
- View real-time logs
- Check container health status
- Restart failed containers

## Troubleshooting Common Issues

### Container Won't Start

```bash
# Check container logs
docker logs container_name

# Validate compose syntax
docker-compose config

# Check resource availability
docker system df
```

### Service Connection Issues

```bash
# Test network connectivity
docker exec container_name ping other_container

# Check environment variables
docker exec container_name env | grep VAR_NAME

# Verify port bindings
docker port container_name
```

### Database Connection Problems

```bash
# Test PostgreSQL connectivity
docker exec postgres pg_isready -U user -d database

# Check Redis connectivity
docker exec redis redis-cli ping

# Verify Qdrant status
curl http://qdrant:6333/health
```

### Performance Issues

```bash
# Monitor container resources
docker stats

# Check disk space
df -h

# Review system logs
journalctl -u docker
```

## Backup Considerations

Before making changes:

1. Backup environment variables
2. Export Portainer stack configurations
3. Backup database volumes
4. Document current configuration

## Scaling and Updates

### Updating Services

```bash
# Pull latest images
docker-compose pull

# Restart services
docker-compose up -d

# Or update via Portainer
```

### Resource Scaling

Adjust resource limits in compose files based on usage:

```yaml
deploy:
  resources:
    limits:
      memory: 2g
      cpus: '1.0'
```

## Security Best Practices

- Use strong passwords for all services
- Regularly update Docker images
- Limit container privileges
- Use secrets for sensitive data
- Monitor access logs

---

**Note**: Replace `yourdomain.com` with your actual domain and update IP addresses to match your server configuration.

**Created**: October 2025