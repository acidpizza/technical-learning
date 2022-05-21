---
title: "Nginx"
weight: 3
---

## References

- https://www.digitalocean.com/community/tutorials/understanding-nginx-server-and-location-block-selection-algorithms


## Block Configurations

**Server blocks** handle requests based on the requested domain name, port and IP address.

**Location blocks** within server blocks handle how requests are routed to resources and URIs for the parent server.

```nginx
server {
  listen 80;
  server_name *.example.com;
}

server {
  listen 80;
  server_name host1.example.com;
}
```


## Server Block

Server is selected via `listen` or `server_name` directive.

### `listen` directive

The `listen` directive defines which IP address and port that the server block will respond to. Nginx will choose the server with the **most specific** `listen` directive to serve the request.

By default, the server will listen to `0.0.0.0:80` (or `0.0.0.0:8080` if run by a non-root user).

```bash
listen 1.1.1.1:443;
listen 1.1.1.1;              # default port 80
listen 443;                  # default all interfaces on that port
listen /Path/to/unix/socket;
```
			
### `server_name` directive

The `server_name` directive is only evaluated if there are multiple server blocks with the same level of specificty matching on the `listen` directive. Nginx will choose the server with the **most specific** `server_name` directive.

`server_name` is matched against the HTTP `Host` header. 

Fuzzy matches:
- Wildcard *
- Regular expressions (start with ~)

Priority (highest to lowest):
- exact match
- leading wildcard match
- trailing wildcard match
- regular expression match
- first matching server block or the server with `default_server` option

```nginx
server_name *.example.com;
server_name ~^(www|host1).*\.example\.com$;
```

## Location Block

### Syntax

```nginx
location optional_modifier location_match {
  ...
}
```

`location_match` matches against URL path.

`optional_modifier`
- `None`: prefix match of location_match against URL (match the beginning)
- `=`: Full match
- `~`: Case sensitive regex match
- `~*`: Case insensitive regex match
- `^~`: Don't use regex match

```nginx
# Match /site, /site/page1/index.html
location /site {}

# Match /site only
location = /site {}

# Match /tortise.jpg but not /FLOWER.PNG
location ~ \.(jpe?g|png|gif|ico)$ {}

# No regex matching
location ^~ /costumes {}
```

### Matching Algorithm

Priority (highest to lowest):
- exact match using `=`
- longest matching non-regex prefix with `^~`
- **first** regular expression that matches with `~` or `~*`
- prefix match with no modifier


### Location Jump

Location block evaluation can jump to other location blocks when triggered by certain directives.

**index**

```nginx
# request of /exact is handled by the 1st block.
# the index directive inherited by the 1st block redirects to /index.html which is handled by the 2nd block
index index.html

location = /exact {}

location / {}
```

**try_files**

```nginx
# request of /blahblah is handled by the 1st block.
# after failing to find blahblah and blahblah.html in /var/www/html
# it will redirect to /fallback/index.html which is handled by the 2nd block
root /var/www/main;

location / {
    try_files $uri $uri.html $uri/ /fallback/index.html;
}

location /fallback {
    root /var/www/another;
}
```

**rewrite**

```nginx
# request for /rewriteme/fallback/hello is handled by the 1st block
# and is rewritten to /fallback/hello which is handled by the 2nd block
root /var/www/main;

location / {
    rewrite ^/rewriteme/(.*)$ /$1 last;
}

location /fallback {
    root /var/www/another;
}
```

**error_page**

```nginx
# request for /not_found is handled by the 1st block 
# and will redirect to /another/whoops.html if the resource is not found in /var/www/main
# which is handled by the 2nd block
root /var/www/main;

location / {
    error_page 404 /another/whoops.html;
}

location /another {
    root /var/www;
}
```


## LUA Module (Open Resty)

Debug Lua Modules in Nginx: `ngx.log(ngx.STDERR, "Hello World")`

Headers: `ngx.var.http_<header name with - replaced with _>`
- Eg. `ngx.var.http_X_ID_Token` to get value of X-ID-Token

