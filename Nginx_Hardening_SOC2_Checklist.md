# Nginx Hardening and SOC 2 Audit Checklist

## New Configurations

### 1. Harden NGINX Configuration

**Prevent HTTP Request Smuggling / CRLF Injection**  
Avoid unsafe use of `$uri` and `$document_uri`; use `$request_uri` instead.

```nginx
location / {
  return 302 https://example.com$request_uri;
}
```

Validate and reject malformed URLs containing `%0d`, `%0a`, or encoded CRLF:

```nginx
if ($request_uri ~* "%0D|%0A") {
  return 400 "Bad Request";
}
```

**Avoid Missing Root Location**  
Ensure the `/` location is explicitly defined to prevent default access to sensitive folders:

```nginx
server {
    root /var/www/html;

    location / {
        index index.html;
    }

    location /hello.txt {
        try_files $uri $uri/ =404;
        proxy_pass http://127.0.0.1:8080/;
    }
}
```

**Disable Unused/Misleading Headers**  
Avoid backend data leaks:

```nginx
proxy_intercept_errors on;
proxy_hide_header Secret-Header;
```

Use `add_header` cautiously, and never echo client inputs.

---

### 2. Sanitize Regex Locations
Avoid vague regex that allows unexpected characters:

```nginx
# BAD
location ~ /docs/([^/])? { ... $1 ... }

# GOOD
location ~ /docs/([^/\s])? { ... $1 ... }
```

---

### 3. Mitigate merge_slashes and Path Confusion
Disable `merge_slashes` to prevent bypassing URL restrictions:

```nginx
merge_slashes off;
```

---

### 4. Restrict Raw Backend Response Exposure
- Strictly validate requests.  
- Ensure malformed requests don’t reach the backend unprocessed.  
- Backend should not include sensitive info in headers.  

---

### 5. Secure Proxy Usage
Avoid direct use of variables like `$uri` in `proxy_pass`:

```nginx
# VULNERABLE
proxy_pass https://backend$uri;

# SAFE
proxy_pass https://backend$request_uri;
```

---

## 1. Configuration Hardening (nginx.conf and Site Configs)

- Run as **Non-Root** inside the container.  
- **Disable Version Disclosure**: `server_tokens off;`  
- **Remove Default Pages and Info** (replace “Welcome to nginx!” and error pages).  
- **Disable Unneeded Modules** (autoindex, charset, userdir).  
- **Listen Only on Approved Ports** (`listen 443 ssl http2;`).  
- **HTTPS-Only Enforcement** with redirects.  
- **TLS Protocols and Ciphers**:  

```nginx
ssl_protocols TLSv1.2 TLSv1.3;
ssl_ciphers HIGH:!aNULL:!MD5;
ssl_prefer_server_ciphers on;
```

- **Certificate and Key Security**: restrict key file permissions.  
- **HSTS**:  

```nginx
add_header Strict-Transport-Security "max-age=31536000; includeSubDomains" always;
```

- **OCSP Stapling**: enable stapling and configure resolver.  
- **Security Headers**: add recommended headers:  
  - Strict-Transport-Security  
  - Content-Security-Policy  
  - X-Frame-Options  
  - X-Content-Type-Options  
  - Referrer-Policy  
  - Permissions-Policy  

- **HTTP Method Restrictions**:  

```nginx
location / {
    limit_except GET POST { deny all; }
    try_files $uri $uri/ =404;
}
```

- **Request and Connection Limits**:  

```nginx
limit_req_zone $binary_remote_addr zone=mylimit:10m rate=10r/s;
```

- **Timeouts**:  

```nginx
client_header_timeout 5s;
client_body_timeout 5s;
keepalive_timeout 10s;
send_timeout 10s;
```

- **Buffer and Body Limits**:  

```nginx
client_max_body_size 10m;
large_client_header_buffers 4 16k;
```

- **Access Control / Host Checks**:  

```nginx
server {
    listen 443 ssl default_server;
    server_name _;
    return 444;
}
```

- **Logging Format**:  

```nginx
log_format main '$remote_addr - $remote_user [$time_local] "$request" $status $body_bytes_sent "$http_referer" "$http_user_agent"';
access_log /var/log/nginx/access.log main;
error_log /var/log/nginx/error.log info;
```

- Enable log rotation and secure file permissions.  

---

## 2. Container Image and Runtime Hardening

- Use minimal, trusted base image (`nginx:1.27.5-alpine`).  
- Use fixed tags and digests.  
- Run as **non-root** (`nginxinc/nginx-unprivileged:1.27.5`).  
- Drop unnecessary Linux capabilities.  
- Use **read-only filesystem** and avoid privileged mode.  
- Enforce resource limits.  
- Automate updates with CI/CD pipeline and image scanning.  
- Verify integrity with GPG keys and Docker Content Trust.  
- Keep CI/CD audit trails.  

---

## 3. Operational Controls (Logging, Monitoring, Throttling, Updates)

- Centralized logging (ELK, Splunk, CloudWatch).  
- Retain logs per SOC 2 requirements (6–12 months).  
- Monitor metrics (Prometheus, Datadog, Amplify).  
- Implement rate limiting & fail2ban.  
- Run regular vulnerability scans (Nessus, Qualys).  
- Test TLS with SSL Labs.  
- Secure update process with changelogs.  
- Use health checks and readiness probes.  
- Enable `stub_status` for metrics:  

```nginx
location /nginx_status {
    stub_status;
    allow 127.0.0.1;
    deny all;
}
```

- Define incident response procedures.  

---

## 4. Testing and Validation

- Automated vulnerability scanning (Trivy, OpenVAS, Nessus).  
- TLS/SSL validation with `openssl`, `testssl.sh`.  
- Security header checks with curl / Mozilla Observatory.  
- Functional testing to avoid breaking apps.  
- Penetration checks (SQLi, brute-force, port scans).  
- Configuration audits (CIS benchmarks).  
- Failover and availability testing.  
- Version verification (`nginx -v`, `nginx -V`).  

---

## 5. Audit Evidence and Documentation

- Retain logs and snapshots.  
- Document change history with Git/Jira.  
- Maintain certificate/key inventory.  
- Record patch/update logs.  
- Save test reports (SSL scans, nmap, curl).  
- Record ownership and permission checks.  
- Provide monitoring configuration evidence.  
- Keep access control records (IAM, RBAC).  
- Provide encryption configuration (TLS1.2/1.3).  
- Keep incident and audit logs.  
- Save container security scan results.  

---

**Each item must be implemented and documented before SOC 2 audit.**  
These measures ensure Nginx is hardened and evidence is available to prove compliance.  
