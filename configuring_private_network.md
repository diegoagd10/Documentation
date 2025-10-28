1. Get a domain
2. PASO 1: Configurar Cloudflare
1.1 Obtener API Token

Ve a: https://dash.cloudflare.com/profile/api-tokens
Click "Create Token"
Usa la plantilla "Edit zone DNS"
Configura:

Permissions: Zone → DNS → Edit
Zone Resources: Include → Specific zone → <YOUR_DOMAIN>


Click "Continue to summary" → "Create Token"
COPIA EL TOKEN (solo se muestra una vez)

1.2 Crear registros DNS

Ve a: https://dash.cloudflare.com
Selecciona tu dominio <YOUR_DOMAIN>.com
Click en "DNS" → "Records"
Click "Add record"
Configura:

Type: A
Name: *.local (exactamente así)
IPv4 address: 192.168.1.100 (tu IP local del servidor)
Proxy status: ⚪ DESACTIVADO (nube GRIS, no naranja)
TTL: Auto


Click "Save"

✅ Ahora *.local.<YOUR_DOMAIN>.com apunta a tu servidor local.

Repeat the above for each local server you have.

3. Connect to server which will host traefik and execute the following command
```
docker network create traefik-public
```

4. Create a user and password with the following bash snippet:

echo $(htpasswd -nb <USER> <PASSWORD>) | sed -e s/\\$/\\$\\$/g^C

You will need this to configure the TRAEFIK_AUTH_USER environment variable

5. Create a stack on portainer for traefik and use @traefik.yml to configure it and set all environment variables.

PASO 6: Probar Traefik Dashboard

Abre tu navegador
Ve a: https://traefik.local.<YOUR_DOMAIN>
Usuario: <USER>
Password: <PASSWORD>
✅ Deberías ver el dashboard de Traefik con certificado SSL válido (candado verde)

7. Configuring local apps on server where trefik got installed with:

 ```
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

 8. Configuring apps external to serevr where traefik got installed.

 8.1 Update vim ~/traefik/dynamic.yml and create the file if doesn't exist

 ```
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