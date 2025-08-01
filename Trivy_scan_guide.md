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

## 2. Run a targeted filesystem vulnerability scan

Example command for scanning `/etc`:

```bash
trivy fs /etc --format table --output trivy-report
```
*remove the last part if you want to view output on terminal*

Common folders to scan individually:
- `/etc`
- `/usr/sbin`
- `/usr/bin`
- `/var/log/fail2ban`
└─>Run a separate `trivy fs` command for each.

---

## 3. Scan for exposed secrets (private keys, passwords...)

preferably scan the entire filesystem `/`: 

```bash
trivy fs / --scanners secret
```
---

## 4. Best way to view table output file

- **Visual Studio Code**
- **Terminal**
