---
title: "Traefik"
weight: 1
---

## Overview

- `--api.insecure=true` enables web UI at `localhost:8080/dashboard`
- Restrict auto service discovery with `exposedByDefault=False`
- Components
  - **Providers** discover the services that live on your infrastructure (their IP, health, ...)
  - **Entrypoints** listen for incoming traffic (ports, ...)
  - **Routers** analyse the requests (host, path, headers, SSL, ...)
  - **Services** forward the request to your services (load balancing, ...)
  - **Middlewares** may update the request or make decisions based on the request (authentication, rate limiting, ...)


## Config

Config Type
- **Static:** Providers, Entrypoints
- **Dynamic:** Routers, Services, Middleware, Certificates

		
Priority (evaluated in order)
- Config File
- Command line arguments
- Environment variable


Config File
- Name: traefik.toml / traefik.yml / traefik.yaml
- Location:
  - /etc/traefik/
  - $XDG_CONFIG_HOME/
  - $HOME/.config/
  - . (the working directory)
- Command line argument: `traefik --configFile=foo/bar/myconfigfile.toml`


## Docker Provider

- Configure with labels
- Docker API access
  - Volume mount: `/var/run/docker.sock:/var/run/docker.sock:ro`
  - Better to expose docker socket via TCP or SSH instead of default unix socket file.
- Rules
  - **HTTP:** ```traefik.http.routers.<service_name>.rule=Host(`example.com`)```
  - **Port:** `traefik.http.services.<service_name>.loadbalancer.server.port=8000`
  - **Conditionals:** ```rule = "Host(`example.com`) || (Host(`example.org`) && Path(`/traefik`))"```
  - **HTTPS:** 
    - ```traefik.tcp.routers.<service_name>.rule=HostSNI(`example.com`)```
    - `traefik.tcp.routers.<service_name>.entrypoints=websecure`
    - `traefik.tcp.routers.<service_name>.tls.passthrough=true (don't MITM)`
    - `traefik.tcp.services.<service_name>.loadbalancer.server.port=8443`

```yaml
services:
  blog:
    image: wordpress:4.9.8-apache
    environment:
      WORDPRESS_DB_PASSWORD:
    labels:
    - traefik.http.routers.blog.rule=Host(`blog.your_domain`)
    - traefik.http.routers.blog.tls=true
    - traefik.http.routers.blog.tls.certresolver=lets-encrypt
    - traefik.port=80
    networks:
    - internal
    - web
    depends_on:
    - mysql
```