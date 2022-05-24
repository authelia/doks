---
title: "NGINX"
description: "An integration guide for Authelia and the NGINX reverse proxy"
lead: "A guide on integrating Authelia with the nginx reverse proxy."
date: 2022-03-19T04:53:05+00:00
lastmod: 2022-03-19T04:53:05+00:00
draft: false
images: []
menu:
  integration:
    parent: "proxies"
weight: 250
toc: true
---

[NGINX] is a reverse proxy supported by **Authelia**.

_**Important:** When using these guides it's important to recognize that we cannot provide a guide for every possible
method of deploying a proxy. These are guides showing a suggested setup only and you need to understand the proxy
configuration and customize it to your needs. To-that-end we include links to the official proxy documentation
throughout this documentation and in the [See Also](#see-also) section._

## Requirements

You need the following to run **Authelia** with [NGINX]:

- [NGINX] must be built with the `http_auth_request` module which is relatively common
- [NGINX] must be built with the `http_realip` module which is relatively common

## Trusted Proxies

_**Important:** You should read the [Forwarded Headers] section and this section as part of any proxy configuration.
Especially if you have never read it before._

_**Important:** The included example is **NOT** meant for production use. It's used expressly as an example to showcase
how you can configure multiple IP ranges. You should customize this example to fit your specific architecture and needs.
You should only include the specific IP address ranges of the trusted proxies within your architecture and should not
trust entire subnets unless that subnet only has trusted proxies and no other services._

[NGINX]'s `http_realip` module is used to configure the trusted proxies' configuration. In our examples this is
configured in the `proxy.conf` file. Each `set_realip_from` directive adds a trusted proxy address range to the trusted
proxies list. Any request that comes from a source IP not in one of the configured ranges results in the header being
replaced with the source IP of the client.

## Configuration

Below you will find commented examples of the following configuration:

* [Authelia Portal](#authelia-portal)
* [Protected Endpoint (Nextcloud)](#protected-endpoint)
* [Supporting Configuration Snippets](#supporting-configuration-snippets)

With the below configuration you can add `authelia-location.conf` internal location snippet to virtual hosts to support
protection with Authelia. The `authelia-authrequest.conf` snippet is used to trigger the authentication flow in specific
location blocks.

### Standard Example

This example is for using the **Authelia** portal redirection flow on a specific endpoint. It requires you to have the
[authelia-location.conf](#authelia-locationconf) and
[authelia-authrequest.conf](#authelia-authrequestconf) snippets. In the example these files exist in the
`/config/nginx/` directory.

#### Authelia Portal

##### auth.example.com.conf

```nginx
server {
    listen 80;

    server_name auth.example.com;

    return 301 https://$server_name$request_uri;
}

server {
    listen 443 ssl http2;

    server_name auth.example.com;

    include /config/nginx/ssl.conf;

    location / {
        set $upstream_authelia http://authelia:9091; # This example assumes a Docker deployment
        proxy_pass $upstream_authelia;
        include /config/nginx/proxy.conf;
    }
}
```

#### Protected Endpoint

##### nextcloud.example.com.conf

```nginx
server {
    listen 80;

    server_name nextcloud.example.com;

    return 301 https://$server_name$request_uri;
}

server {
    listen 443 ssl http2;

    server_name nextcloud.example.com;

    include /config/nginx/ssl.conf;

    include /config/nginx/authelia-location.conf; # Virtual endpoint to forward auth requests

    location / {
        set $upstream_nextcloud https://nextcloud;
        proxy_pass $upstream_nextcloud;
        include /config/nginx/authelia-authrequest.conf; # Activates Authelia for specified route/location, please ensure you have setup the domain in your configuration.yml
        include /config/nginx/proxy.conf; # Reverse proxy configuration
    }
}
```

### HTTP Basic Authentication Example

This example is for using HTTP basic auth on a specific endpoint. It is based on the full example above. It requires you
to have the [authelia-location-basic.conf](#authelia-location-basicconf) and
[authelia-authrequest-basic.conf](#authelia-authrequest-basicconf) snippets. In the example these files exist in the
`/config/nginx/` directory.

#### HTTP Basic Authentication Protected Endpoint

```nginx
server {
    listen 80;

    server_name nextcloud.example.com;

    return 301 https://$server_name$request_uri;
}

server {
    listen 443 ssl http2;

    server_name nextcloud.example.com;

    include /config/nginx/ssl.conf;

    include /config/nginx/authelia-location-basic.conf; # Use the "basic" endpoint

    location / {
        set $upstream_nextcloud https://nextcloud;
        proxy_pass $upstream_nextcloud;
        include /config/nginx/authelia-authrequest-basic.conf; # Activate authelia with basic auth
        include /config/nginx/proxy.conf; # this file is the exact same as above
    }
}
```

### Supporting Configuration Snippets

The following configuration files are snippets that are used as includes in other files. The includes in the other files
match the headings, so if you wish to put them in a specific location or rename them, then make sure to update the
includes appropriately.

#### proxy.conf

The following is an example `proxy.conf`. The important directives include the `real_ip` directives which you should read
[Trusted Proxies](#trusted-proxies) section to understand, or set the `X-Forwarded-Proto`, `X-Forwarded-Host`,
`X-Forwarded-Uri`, and `X-Forwarded-For` headers.

```nginx
## Headers
proxy_set_header Host $host;
proxy_set_header X-Forwarded-Proto $scheme;
proxy_set_header X-Forwarded-Host $http_host;
proxy_set_header X-Forwarded-Uri $request_uri;
proxy_set_header X-Forwarded-Ssl on;
proxy_set_header X-Forwarded-For $remote_addr;
proxy_set_header X-Real-IP $remote_addr;
proxy_set_header Connection "";

## Basic Proxy Configuration
client_body_buffer_size 128k;
proxy_next_upstream error timeout invalid_header http_500 http_502 http_503; ## Timeout if the real server is dead.
proxy_redirect  http://  $scheme://;
proxy_http_version 1.1;
proxy_cache_bypass $cookie_session;
proxy_no_cache $cookie_session;
proxy_buffers 64 256k;

## Trusted Proxies Configuration
## Please read the following documentation before configuring this:
##     https://www.authelia.com/integration/proxies/nginx/#trusted-proxies
# set_real_ip_from 10.0.0.0/8;
# set_real_ip_from 172.16.0.0/12;
# set_real_ip_from 192.168.0.0/16;
# set_real_ip_from fc00::/7;
real_ip_header X-Forwarded-For;
real_ip_recursive on;

## Advanced Proxy Config
send_timeout 5m;
proxy_read_timeout 360;
proxy_send_timeout 360;
proxy_connect_timeout 360;
```

#### authelia-location.conf

_The following snippet is used within the `server` block of a virtual host as a supporting endpoint used by
`auth_request` and is paired with [authelia-authrequest.conf](#authelia-authrequestconf)._

```nginx
set $upstream_authelia http://authelia:9091/api/verify;

# Virtual endpoint created by nginx to forward auth requests.
location /authelia {
    # Essential Proxy Config.
    internal; # Mark this as an internal only location.
    proxy_pass $upstream_authelia;

    # [REQUIRED] Needed by Authelia to check authorizations of the resource.
    # Provide either X-Original-URL and X-Forwarded-Proto or
    # X-Forwarded-Proto, X-Forwarded-Host and X-Forwarded-Uri or both.
    # Those headers will be used by Authelia to deduce the target url of the user.

    # Header Proxy Config.
    proxy_set_header X-Original-URL $scheme://$http_host$request_uri;
    proxy_set_header X-Forwarded-Method $request_method;
    proxy_set_header X-Forwarded-Proto $scheme;
    proxy_set_header X-Forwarded-Host $http_host;
    proxy_set_header X-Forwarded-Uri $request_uri;
    proxy_set_header X-Forwarded-For $remote_addr;
    proxy_set_header Content-Length "";
    proxy_set_header Connection "";

    # Basic Proxy Config.
    proxy_pass_request_body off;
    proxy_next_upstream error timeout invalid_header http_500 http_502 http_503; # Timeout if the real server is dead
    proxy_redirect http:// $scheme://;
    proxy_http_version 1.1;
    proxy_cache_bypass $cookie_session;
    proxy_no_cache $cookie_session;
    proxy_buffers 4 32k;
    client_body_buffer_size 128k;

    # Advanced Proxy Config.
    send_timeout 5m;
    proxy_read_timeout 240;
    proxy_send_timeout 240;
    proxy_connect_timeout 240;
}
```

#### authelia-authrequest.conf

_The following snippet is used within a `location` block of a virtual host which uses the appropriate location block
and is paired with [authelia-location.conf](#authelia-locationconf)._

```nginx
## Send a subrequest to Authelia to verify if the user is authenticated and has permission to access the resource.
auth_request /authelia;

## Set the $target_url variable based on the original request.
auth_request_set $target_url $scheme://$http_host$request_uri;

## Save the upstream response headers from Authelia to variables.
auth_request_set $user $upstream_http_remote_user;
auth_request_set $groups $upstream_http_remote_groups;
auth_request_set $name $upstream_http_remote_name;
auth_request_set $email $upstream_http_remote_email;

## Inject the response headers from the variables into the request made to the backend.
proxy_set_header Remote-User $user;
proxy_set_header Remote-Groups $groups;
proxy_set_header Remote-Name $name;
proxy_set_header Remote-Email $email;

## If the subreqest returns 200 pass to the backend, if the subrequest returns 401 redirect to the portal.
error_page 401 =302 https://auth.example.com/?rd=$target_url;
```

#### authelia-location-basic.conf

_The following snippet is used within the `server` block of a virtual host as a supporting endpoint used by
`auth_request` and is paired with [authelia-authrequest-basic.conf](#authelia-authrequest-basicconf). This particular
snippet is rarely required. It's only used if you want to only allow
[HTTP Basic Authentication](https://developer.mozilla.org/en-US/docs/Web/HTTP/Authentication) for a particular
endpoint. It's recommended to use [authelia-location.conf](#authelia-locationconf) instead._

```nginx
set $upstream_authelia http://authelia:9091/api/verify?auth=basic;

# Virtual endpoint created by nginx to forward auth requests.
location /authelia-basic {
    # Essential Proxy Config.
    internal; # Mark this as an internal only location.
    proxy_pass $upstream_authelia;

    # [REQUIRED] Needed by Authelia to check authorizations of the resource.
    # Provide either X-Original-URL and X-Forwarded-Proto or
    # X-Forwarded-Proto, X-Forwarded-Host and X-Forwarded-Uri or both.
    # Those headers will be used by Authelia to deduce the target url of the user.

    # Header Proxy Config.
    proxy_set_header X-Original-URL $scheme://$http_host$request_uri;
    proxy_set_header X-Forwarded-Method $request_method;
    proxy_set_header X-Forwarded-Proto $scheme;
    proxy_set_header X-Forwarded-Host $http_host;
    proxy_set_header X-Forwarded-Uri $request_uri;
    proxy_set_header X-Forwarded-For $remote_addr;
    proxy_set_header Content-Length "";
    proxy_set_header Connection "";

    # Basic Proxy Config.
    proxy_pass_request_body off;
    proxy_next_upstream error timeout invalid_header http_500 http_502 http_503; # Timeout if the real server is dead
    proxy_redirect http:// $scheme://;
    proxy_http_version 1.1;
    proxy_cache_bypass $cookie_session;
    proxy_no_cache $cookie_session;
    proxy_buffers 4 32k;
    client_body_buffer_size 128k;

    # Advanced Proxy Config.
    send_timeout 5m;
    proxy_read_timeout 240;
    proxy_send_timeout 240;
    proxy_connect_timeout 240;
}
```

#### authelia-authrequest-basic.conf

_The following snippet is used within a `location` block of a virtual host which uses the appropriate location block
and is paired with [authelia-location-basic.conf](#authelia-location-basicconf). This particular snippet is rarely
required. It's only used if you want to only allow
[HTTP Basic Authentication](https://developer.mozilla.org/en-US/docs/Web/HTTP/Authentication) for a particular
endpoint. It's recommended to use [authelia-authrequest.conf](#authelia-authrequestconf) instead._

```nginx
## Send a subrequest to Authelia to verify if the user is authenticated and has permission to access the resource.
auth_request /authelia-basic;

## Set the $target_url variable based on the original request.
auth_request_set $target_url $scheme://$http_host$request_uri;

## Save the upstream response headers from Authelia to variables.
auth_request_set $user $upstream_http_remote_user;
auth_request_set $groups $upstream_http_remote_groups;
auth_request_set $name $upstream_http_remote_name;
auth_request_set $email $upstream_http_remote_email;

## Inject the response headers from the variables into the request made to the backend.
proxy_set_header Remote-User $user;
proxy_set_header Remote-Groups $groups;
proxy_set_header Remote-Name $name;
proxy_set_header Remote-Email $email;
```

#### authelia-location-detect.conf

_The following snippet is used within the `server` block of a virtual host as a supporting endpoint used by
`auth_request` and is paired with [authelia-authrequest-detect.conf](#authelia-authrequest-detectconf). This particular
snippet is rarely required. It's only used if you want to conditionally require
[HTTP Basic Authentication](https://developer.mozilla.org/en-US/docs/Web/HTTP/Authentication) for a particular
endpoint. It's recommended to use [authelia-location.conf](#authelia-locationconf) instead._

```nginx
include /config/nginx/authelia-location.conf;

set $is_basic_auth ""; # false value

## Detect the client you want to force basic auth for here
## For the example we just match a path on the original request
if ($request_uri = "/force-basic") {
    set $is_basic_auth "true";
    set $upstream_authelia "$upstream_authelia?auth=basic";
}

## A new virtual endpoint to used if the auth_request failed
location  /authelia-detect {
    internal;

    if ($is_basic_auth) {
        ## This is a request where we decided to use basic auth, return a 401.
        ## Nginx will also proxy back the WWW-Authenticate header from Authelia's
        ## response. This is what informs the client we're expecting basic auth.
        return 401;
    }

    ## The original request didn't target /force-basic, redirect to the pretty login page
    ## This is what `error_page 401 =302 https://auth.example.com/?rd=$target_url;` did.
    return 302 https://auth.example.com/$is_args$args;
}
```

#### authelia-authrequest-detect.conf

_The following snippet is used within a `location` block of a virtual host which uses the appropriate location block
and is paired with [authelia-location-detect.conf](#authelia-location-detectconf). This particular snippet is rarely
required. It's only used if you want to conditionally require
[HTTP Basic Authentication](https://developer.mozilla.org/en-US/docs/Web/HTTP/Authentication) for a particular
endpoint. It's recommended to use [authelia-authrequest.conf](#authelia-authrequestconf) instead._

```nginx
## Basic Authelia Config
## Send a subsequent request to Authelia to verify if the user is authenticated
## and has the right permissions to access the resource.
auth_request /authelia;

## Set the `target_url` variable based on the request. It will be used to build the portal
## URL with the correct redirection parameter.
auth_request_set $target_url $scheme://$http_host$request_uri;

## Set the X-Forwarded-User and X-Forwarded-Groups with the headers
## returned by Authelia for the backends which can consume them.
## This is not safe, as the backend must make sure that they come from the
## proxy. In the future, it's gonna be safe to just use OAuth.
auth_request_set $user $upstream_http_remote_user;
auth_request_set $groups $upstream_http_remote_groups;
auth_request_set $name $upstream_http_remote_name;
auth_request_set $email $upstream_http_remote_email;

proxy_set_header Remote-User $user;
proxy_set_header Remote-Groups $groups;
proxy_set_header Remote-Name $name;
proxy_set_header Remote-Email $email;

## If Authelia returns 401, then nginx passes it to the user.
## If it returns 200, then the request pass through to the backend.
error_page 401 /authelia-detect?rd=$target_url;
```

## See Also

- [NGINX ngx_http_auth_request_module Module Documentation](https://nginx.org/en/docs/http/ngx_http_auth_request_module.html)
- [NGINX ngx_http_realip_module Module Documentation](https://nginx.org/en/docs/http/ngx_http_realip_module.html)
- [Forwarded Headers]

[NGINX]: https://www.nginx.com/
[Forwarded Headers]: fowarded-headers
