# My home Gateway setup

The setup is based on [Traefik](https://traefik.io/traefik/)

## Features

- 🔐 Automatically generates TLS certificates using [LetsEncrypt](https://letsencrypt.org/) 🔐
- 🪄 Automatically adds labeled docker containers🪄

## How to use

### Shared network

In order to route traffic, the gateway must be in the same docker network as the target services.

For this I use external network:

```bash
docker network create mynetwork
```

Then add the network to the gateway service with the following configuration:

```yaml
services:
  traefik:
    ...
    networks:
      - mynetwork


networks:
  mynetwork:
    external: true
```

### Docker labels

```yaml
...
services:
  <service name>:
    ...
    labels:
      - traefik.enable=true

      - traefik.http.middlewares.frontend.basicAuth.users=${HASHED_ADMIN_USER_PASS}
      - traefik.http.routers.<service name>.tls=true
      - traefik.http.routers.<service name>.tls.certresolver=le
      - traefik.http.routers.<service name>.tls.domains[0].main=${DOMAIN}
      - traefik.http.routers.<service name>.rule=Host(`${DOMAIN}`)
      - traefik.http.routers.<service name>.service=<service name>
      - traefik.http.services.<service name>loadbalancer.server.port=5080
      - traefik.http.routers.<service name>.middlewares=frontend
```

### Basic http auth

Generate `HASHED_ADMIN_USER_PASS` using below command

```bash
htpasswd -B -C 10 -c .htpasswd user1
cat .htpasswd | sed -e s/\\$/\\$\\$/g
```

## DDNS

For DDNS I use [this](https://github.com/8tomat8/ddns)

```yaml
version: "3.9"

include:
  - path: ../ddns/docker-compose.yml

services: ...
```
