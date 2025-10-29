# Homepage Dashboard Configuration

This guide covers configuring the Homepage dashboard for monitoring your self-hosted services.

## Prerequisites

- Homepage service deployed and running
- Access to Docker volume management
- Basic YAML configuration knowledge

## Steps

1. **Find homepage volume**:
   ```bash
   docker volume ls
   ```

2. **Find directory path** for the homepage configuration from the volume:
   ```bash
   docker volume inspect homepage_homepage-config
   ```

3. **Update services.yml**:
   ```bash
   sudo vim /PATH_TO_HOME_PAGE_CONFIG_FROM_VOLUME/services.yaml
   ```

   Add the following configuration:
   ```yaml
   ---
   # For configuration options and examples, please see:
   # https://gethomepage.dev/configs/services/
   
   - Container Management:
       - Portainer 01:
           icon: portainer.png
           href: https://portainer-01.local.dagdappshub.com
           description: Manages n8n, traefik, homepage, uptime-kuma and backends
   
       - Portainer 02:
           icon: portainer.png
           href: https://portainer-02.local.dagdappshub.com
           description: Manages databases
   
   - Monitoring:
       - Uptime Kuma:
           icon: uptime-kuma.png
           href: https://kuma.local.dagdappshub.com
           description: Verifies that services and apps are still up and running
   
   - Network:
       - Traefik:
           icon: traefik.png
           href: https://traefik.local.dagdappshub.com
           description: Simple reverse proxy server
   
   - AI:
       - Replicate:
           icon: https://d31rfu1d3w8e4q.cloudfront.net/static/favicon.fdf5ef8729cc.png
           href: https://replicate.com/account/billing
           description: Allows to host your own trained models (We have IBM Granite 4 here :))
   
       - OpenCode:
           icon: https://opencode.ai/_build/assets/opencode-poster-CbUiDHgA.png
           href: https://opencode.ai/workspace/wrk_01K7WM94FGX6MMTY063KE7D2GY/billing
           description: This is used for coding
   
       - Antropic:
           icon: https://cdn.jsdelivr.net/gh/selfhst/icons/png/claude.png
           href: https://console.anthropic.com/dashboard
   
       - Claude:
           icon: https://cdn.jsdelivr.net/gh/selfhst/icons/png/claude.png
           href: https://claude.ai/new
   
       - OpenAI:
           icon: https://cdn.jsdelivr.net/gh/selfhst/icons/png/chatgpt.png
           href: https://platform.openai.com/docs/api-reference/introduction
   
   - Databases:
       - PG Admin 4:
           icon: postgres.png
           href: https://pgadmin.local.dagdappshub.com
           description: PostgreSQL administrator
   
       - Redis:
           icon: redis.png
           href: https://redis.local.dagdappshub.com
           description: Caching administrator
   
       - Qdrant:
           icon: qdrant.png
           href: https://qdrant.local.dagdappshub.com/dashboard#/welcome
           description: Vector database administrator
   
   - Learning:
       - Try Hack Me:
           icon: https://simpleicons.org/icons/tryhackme.svg
           href: https://tryhackme.com/room/defensivesecurityintro
   
       - Cmd Challenge:
           icon: https://simpleicons.org/icons/gnometerminal.svg
           href: https://cmdchallenge.com/
   ```

4. **Update widgets**:
   ```bash
   sudo vim /var/snap/docker/common/var-lib/docker/volumes/homepage_homepage-config/_data/widgets.yaml
   ```

   Add the following configuration:
   ```yaml
   ---
   # For configuration options and examples, please see:
   # https://gethomepage.dev/configs/info-widgets/

   - resources:
       cpu: true
       cputemp: true
       memory: true
       disk: /
       uptime: true

   - search:
       provider: duckduckgo
       target: _blank
   ```

## Post-Configuration

After updating the configuration files, restart the Homepage container:

```bash
docker restart homepage
```

## Troubleshooting

- Ensure YAML syntax is correct
- Check file permissions on the volume
- Verify service URLs are accessible
- Restart container after configuration changes

---

**Note**: Replace `dagdappshub.com` with your actual domain name.

**Created**: October 2025
