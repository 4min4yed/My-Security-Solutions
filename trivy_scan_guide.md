# Trivy Scan Guide for LXC Containers

## 1. Install the latest Trivy version

```bash
curl -sfL https://raw.githubusercontent.com/aquasecurity/trivy/main/contrib/install.sh | sudo sh -s -- -b /usr/local/bin latest
```

- This installs the latest stable Trivy binary to `/usr/local/bin/trivy`.
- Check install:  
  ```bash
  trivy --version
  ```

---

## 2. Run a targeted filesystem scan

Example command for scanning `/etc`:

```bash
trivy fs /etc --format table --output trivy-report.txt
```

Common folders to scan individually:
- `/etc`
- `/usr/sbin`
- `/usr/bin`
- `/var/log/fail2ban`

Run a separate `trivy fs` command for each.

---

## 3. Transfer the report to your local machine

From your **local machine**, run:

```bash
scp user@container-ip:/path/to/trivy-report.txt /local/path/
```

---

## 4. Best way to view table output file

- **Visual Studio Code**
- **Terminal**
