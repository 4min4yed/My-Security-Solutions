# Nginx Hardening Issues and Fixes

## Problem I

Visiting `www.site/random`
![](images/random.png)

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
  ![](images/random.png)
- **After Configuration**
  ![](images/random_fix.png)

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
  ![](images/nmap_exposed.png)
- **After**
  ![](images/nmap_hidden.png)

## Problem III

By default, some files may be publicly accessible by default or upon installation (like README.md or .conf files...)
So to hide certain files via extension or files locations:

```nginx
server {
            ...

    location / {
…
 #DENY EXTENSIONS
       location ~* \.(md|git|env|json|lock|ts|js\.map)$ {deny all;}
	#DENY FOLDERS
       location ~ ^/(node_modules|e2e|src)/ {deny all;}}}
```
