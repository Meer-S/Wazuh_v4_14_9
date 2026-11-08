# Wazuh SIEM Installation Guide on Ubuntu Server 24.04

## Project Overview
This document provides a complete step-by-step guide for deploying Wazuh on Ubuntu Server 24.04 in a home lab or proof-of-concept environment.

---

# 1. Lab Requirements

## Hardware
- CPU: 4 vCPU
- Memory: 8 GB RAM (Minimum 4 GB)
- Disk: 80 GB+

## Networking
- Adapter 1: NAT (Internet access)
- Adapter 2: Bridged (LAN access)

## Operating System
- Ubuntu Server 24.04 LTS

---

# 2. Install Ubuntu Server 24.04

## Update System

```bash
sudo apt update && sudo apt upgrade -y
```

## Set Hostname

```bash
sudo hostnamectl set-hostname wazuh
```

Verify:

```bash
hostname
hostname -f
```

---

# 3. Configure Static IP

Check interfaces:

```bash
ip a
```

Edit netplan:

```bash
sudo micro /etc/netplan/50-cloud-init.yaml
```

Example:

```yaml
network:
  version: 2
  ethernets:
    ens33:
      dhcp4: true
    ens37:
      dhcp4: true
      optional: true
```

Apply:

```bash
sudo netplan generate
sudo netplan apply
```

Verify:

```bash
ip a
```

---

# 4. Prevent Ubuntu From Sleeping

```bash
sudo systemctl mask sleep.target suspend.target hibernate.target hybrid-sleep.target
```

Verify:

```bash
systemctl status sleep.target
```

---

# 5. Download Wazuh Installer

```bash
curl -sO https://packages.wazuh.com/4.7/wazuh-install.sh
```

Grant execute permission:

```bash
chmod +x wazuh-install.sh
```

---

# 6. Install Wazuh All-in-One

Ubuntu 24.04 is not officially listed for Wazuh 4.7 installer validation.

Use:

```bash
sudo bash ./wazuh-install.sh -a --ignore-check
```

Components Installed:

- Wazuh Manager
- Wazuh Indexer
- Wazuh Dashboard
- Filebeat

---

# 7. Verify Services

```bash
sudo systemctl status wazuh-manager
sudo systemctl status wazuh-indexer
sudo systemctl status wazuh-dashboard
```

Enable at boot:

```bash
sudo systemctl enable wazuh-manager
sudo systemctl enable wazuh-indexer
sudo systemctl enable wazuh-dashboard
```

---

# 8. Check Wazuh Daemon Status

```bash
sudo /var/ossec/bin/wazuh-control status
```

Expected:

```text
wazuh-modulesd is running
wazuh-analysisd is running
wazuh-remoted is running
wazuh-apid is running
```

---

# 9. Dashboard Access

Identify server IP:

```bash
hostname -I
```

Verify dashboard listener:

```bash
sudo ss -tulpn | grep 443
```

Expected:

```text
0.0.0.0:443
```

Access:

```text
https://SERVER-IP
```

---

# 10. Firewall Verification

Check:

```bash
sudo ufw status
```

If enabled:

```bash
sudo ufw allow 443/tcp
sudo ufw allow 1514/tcp
sudo ufw allow 1515/tcp
sudo ufw reload
```

---

# 11. Verify API

Check API Port:

```bash
sudo ss -tulpn | grep 55000
```

Test API:

```bash
curl -k https://127.0.0.1:55000
```

Expected:

```json
{"title":"Unauthorized","detail":"No authorization token provided"}
```

---

# 12. Locate Generated Passwords

```bash
cat wazuh-passwords.txt
```

Important Accounts:

- admin
- kibanaserver
- wazuh
- wazuh-wui

---

# 13. Dashboard API Authentication Issue Resolution

Problem:

```text
API connection: No API available to connect
```

Root Cause:

Incorrect API password in:

```bash
/usr/share/wazuh-dashboard/data/wazuh/config/wazuh.yml
```

Update:

```yaml
hosts:
  - default:
      url: https://127.0.0.1
      port: 55000
      username: wazuh-wui
      password: "<PASSWORD_FROM_wazuh-passwords.txt>"
      run_as: true
```

Restart:

```bash
sudo systemctl restart wazuh-dashboard
```

Validate:

```bash
curl -k -u "wazuh-wui:PASSWORD" https://127.0.0.1:55000/security/user/authenticate?raw=true
```

---

# 14. Service Recovery Procedure

If dashboard reports:

```text
No API available to connect
```

Check:

```bash
sudo /var/ossec/bin/wazuh-control status
```

Recovery:

```bash
sudo /var/ossec/bin/wazuh-control restart
sleep 20
sudo /var/ossec/bin/wazuh-control status
```

---

# 15. Certificate Deployment

Extract certificates:

```bash
sudo mkdir -p /etc/wazuh-certificates
sudo tar -xvf /home/ubuntu/wazuh-certificates.tar -C /etc/wazuh-certificates/
```

Permissions:

```bash
sudo chown wazuh:wazuh /etc/wazuh-certificates/*
sudo chmod 640 /etc/wazuh-certificates/*
```

---

# 16. Health Check Commands

```bash
sudo systemctl status wazuh-manager
sudo systemctl status wazuh-indexer
sudo systemctl status wazuh-dashboard
```

```bash
sudo /var/ossec/bin/wazuh-control status
```

```bash
sudo ss -tulpn | grep 443
```

```bash
sudo ss -tulpn | grep 55000
```

```bash
curl -k https://127.0.0.1:55000
```

---

# Lessons Learned

- Use Bridged networking for external dashboard access.
- Dashboard API failures are often credential related.
- Validate wazuh-wui credentials using wazuh-passwords.txt.
- Always check daemon status using wazuh-control status.
- Restarting all Wazuh daemons resolves many startup synchronization issues.
- Ubuntu 24.04 installations require --ignore-check during installation.
