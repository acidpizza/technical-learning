---
title: "Kong"
weight: 2
---

## Overview

- Components
  - **Service:** the API or microservice managed by Kong (URI that Kong will call)
  - **Route:** specify how requests reach services. One service can have many routes. (How user called the URI)
  - **Consumer:** end user of services. Can configure access control and audit flows.
  - **Admin API:** RESTful API for administration.
  - **Plugins:** extend Kong's functionality.
		
- Enterprise Components
  - **Kong Manager:** Web UI for monitoring and managing Kong Enterprise.
  - **Developer Portal:** provides a single source of truth for developers to access all services.
	
- Listening ports: 
  - Proxy Listen: 8000, 8443
  - Admin API: 8001


## Configuration

```bash
# Create service (example_service)
curl -i -X POST http://<admin-hostname>:8001/services \
  --data name=example_service \
  --data url='http://mockbin.org'
	
# Create route (mocking route at /mock to example_service)
curl -i -X POST http://<admin-hostname>:8001/services/example_service/routes \
  --data 'paths[]=/mock' \
  --data 'name=mocking'
	
# Test configured route (at /mock)
curl -i -X GET http://<admin-hostname>:8000/mock
```

```bash
# Rate limiting plugin: protects overuse of admin API and starving other consumers.
curl -i -X POST http://<admin-hostname>:8001/plugins \
--data "name=rate-limiting" \
--data "config.minute=5" \
--data "config.policy=local"
```

```bash	
# Proxy caching plugin: Improve performance.
curl -i -X POST http://<admin-hostname>:8001/plugins \
--data name=proxy-cache \
--data config.content_type="application/json" \
--data config.cache_ttl=30 \
--data config.strategy=memory
```

```bash
# Key-Auth plugin: protects "mocking" route with API key
curl -X POST http://<admin-hostname>:8001/routes/mocking/plugins \
  --data name=key-auth
	
## Create consumer
curl -i -X POST -d "username=myconsumer&custom_id=myconsumer" http://<admin-hostname>:8001/consumers/
	
## Create API key for consumer
curl -i -X POST http://<admin-hostname>:8001/consumers/myconsumer/key-auth -d 'key=myapikey'
	
## Use API key
curl -i http://<admin-hostname>:8000/mock/request -H 'apikey:myapikey'
```

- ACL plugin: Controls consumer access to services and routes.
	
- Load Balancing:
  - Setup an upstream 
  - Configure upstream to point to multiple targets
  - Setup a service and point to upstream.


## Declarative Configuration 

### Native, DB-less mode

- Entire config must fit in memory (mem_cache_size)
- No central database. 
  - No cluster propagation.
  - Multiple Kong configurations must be loaded independently.
- Read-only Admin API 
- Not all plugins are compatible

```bash
# Create template config file (creates kong.yml in current directory)
kong config -c kong.conf init
	
# Check template config file 
kong config -c kong.conf parse kong.yml
	
# Start Kong without database and with declarative config
export KONG_DATABASE=off
export KONG_DECLARATIVE_CONFIG=kong.yml
kong start -c kong.conf
	
# Replace config in entirety (via cli)
http :8001/config config=@kong.yml
# Replace config in entirety (via curl)
curl -X POST http://<admin-hostname>:8001/config -d 'config=@kong.yml'
```

### 3rd-party, DB mode

```bash
# Use hbagdi/deck project for declarative with DB
deck --kong-addr http://<admin-hostname>:8001 sync -s /home/deckuser/kong.yml
```


