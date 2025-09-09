# Nginx Hardening Issues and Fixes

## Problem I

Visiting `www.site/random`

### The Fix
Edit `nginx.conf` of the Nginx server (not the reverse proxy) inside the Angular app located at  
`…/app/nginx.conf` if you used `-v` when building it, or inside the container (temporary):

```nginx
server {
    ...
    try_files $uri $uri/ /index.html;  # if not accessible serve home

    error_page 404 /custom_404.html;
    location = /custom_404.html { 
        root /path/to/error/pages; # Directory where custom_404.html is
        internal;                  # Prevents direct access
    }
}
```

### PoC
- **Before Configuration**
- **After Configuration**

---

## Problem II

By default, **NGINX** sends out version info in HTTP headers and even in your error pages. 

### The Fix
```nginx
server_tokens off;                  # wipe out nginx version info
proxy_hide_header X-Powered-By;     # hide headers
add_header X-Powered-By "";         # remove any accidental X-Powered-By headers
```

### PoC
- **Before**
- **After**
