1. Find homepage volume:

```docker volume ls```

2. Find directory if the homepage configuration from the volume using:

```
docker volume inspect homepage_homepage-config
```

3. Updte services.yml  

```
sudo vim /PATH_TO_HOME_PAGE_CONFIG_FROM_VOLUME/services.yaml
```
 
```
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
```

4. Update Widgets:

```
sudo cat /var/snap/docker/common/var-lib-docker/volumes/homepage_homepage-config/_data/widgets.yaml
```

```
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
````