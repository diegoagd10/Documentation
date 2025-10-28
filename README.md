# Self-Hosting Management Documentation

This documentation provides comprehensive guidance for setting up and managing a self-hosted infrastructure using Ubuntu Server, Docker, and various services for home automation, monitoring, and data management.

## Overview

This project covers the deployment of a multi-server self-hosting environment with:
- **Server 1 (svc-01)**: Main services including Traefik reverse proxy, n8n workflow automation, Homepage dashboard, and Uptime Kuma monitoring
- **Server 2 (svc-02)**: Database services including PostgreSQL, pgAdmin, Redis, Redis Commander, and Qdrant vector database

The infrastructure uses Docker Compose for container orchestration, Portainer for management, and Traefik for SSL termination and routing with Cloudflare DNS challenge.

## Table of Contents

1. **[Server Setup](server-setup.md)** - Ubuntu Server installation, initial configuration, and security hardening
2. **[Network Configuration](network-configuration.md)** - Private network setup, DNS configuration, and Traefik SSL proxy
3. **[Services Deployment](services-deployment.md)** - Docker stacks, Portainer usage, and service configurations
4. **[Backup and Restoration](backup-restoration.md)** - Backup procedures, encryption, and restoration processes
5. **[Maintenance](maintenance.md)** - Monitoring, updates, troubleshooting, and system management

## Prerequisites

- Ubuntu Server 24.04 LTS ISO
- USB drive (8GB minimum) for installation
- Domain name registered with Cloudflare
- Two Ubuntu servers (physical or virtual)
- Basic Linux command line knowledge
- Docker and Docker Compose installed

## High-Level Workflow

1. **Setup Phase**
   - Install Ubuntu Server on both machines
   - Configure basic networking and security
   - Install Docker and required tools

2. **Network Phase**
   - Configure Cloudflare DNS for wildcard subdomains
   - Deploy Traefik reverse proxy on primary server
   - Set up SSL certificates with Let's Encrypt

3. **Services Phase**
   - Deploy database services on secondary server
   - Deploy application services on primary server
   - Configure Portainer for management
   - Set up Homepage dashboard

4. **Configuration Phase**
   - Configure service interconnections
   - Set up monitoring and alerting
   - Implement backup procedures

5. **Maintenance Phase**
   - Regular system updates
   - Monitor service health
   - Perform backups and test restorations

## Important Notes

- **Security**: All services are configured with SSL/TLS encryption
- **Networking**: Uses internal Docker networks for service communication
- **Backups**: Encrypted backups stored off-site (Google Drive)
- **Monitoring**: Uptime Kuma provides 24/7 service monitoring
- **Updates**: Automatic security updates enabled with manual major version upgrades

## Quick Start

1. Follow the [Server Setup](server-setup.md) guide for both servers
2. Configure your [Network](network-configuration.md) with Cloudflare and Traefik
3. Deploy [Services](services-deployment.md) using the provided Docker Compose files
4. Set up [Backups](backup-restoration.md) and test restoration procedures
5. Implement [Maintenance](maintenance.md) routines

## Support

- Check the troubleshooting sections in each guide
- Review service logs using `docker logs <container_name>`
- Use Portainer dashboard for container management
- Monitor system health with Uptime Kuma

---

**Last Updated**: October 2025
**Ubuntu Version**: 24.04 LTS
**Docker Version**: Latest stable