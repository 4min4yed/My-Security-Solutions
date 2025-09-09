# Solution to Stop SSH Port Discovery

### Problem
SSH port is discoverable after running a scan tool like `nmap`.

### Solution
After implementing **Port Knocking**:

### Steps

*Assuming SSH is already running and listening on port `5212`*

#### 1. Add a firewall rule using iptables to block all incoming connections (including scans) on the SSH port:
```bash
sudo iptables -A INPUT -p tcp --dport 5212 -j DROP
sudo iptables -L   # to check firewall rule is added
```

#### 2. Install and enable knockd:
```bash
sudo apt install knockd
sudo systemctl start knockd
sudo systemctl status knockd   # to check if it’s running
```

#### 3. Configure knockd (interface, port, knocking sequence, command after knock):
```bash
sudo vim /etc/knockd.conf
```

- Default knock sequence: `7000, 8000, 9000`  
- Default interface: `eth0`  
- Default command: `iptables -A INPUT -s …`

Change it to:
- Sequence: `1233, 4688, 8756`  
- Port: `5212`  
- Command:  
```bash
sudo iptables -I INPUT 1 -s %IP% -p tcp --dport 5212 -j ACCEPT
```
> **Note:** Use `-I` (insert) instead of `-A` (append) to avoid being overshadowed by the first firewall rule.

---

### Usage

#### Without knock
On another machine:
```bash
ssh -p 5212 root@<ip>
```
➡️ Rejected.

#### With knock sequence
```bash
knock <ip> 1233 4688 8756
# Wait ~2 seconds
ssh -p 5212 root@<ip>
```
➡️ Access granted ✅

Check new iptables rule:
```bash
sudo iptables -L
```
Now only the knocking IP can access port `5212`.

#### Closing sequence
```bash
knock <ip> 8756 4688 1233
```
➡️ SSH port is closed again until the opening sequence is repeated.  
➡️ Scanning tools won’t detect the port as open ✔️

---

### Persisting configuration

Run knockd at boot:
```bash
sudo vim /etc/default/knockd
# set
START_KNOCKD=1

sudo systemctl enable knockd
```

Save iptables rules after reboot:
```bash
sudo apt install iptables-persistent
sudo netfilter-persistent save
iptables-save > /etc/iptables/rules.v4
```

---

