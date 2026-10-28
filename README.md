# Wazuh_v4_14_9
🛡️ Wazuh Home Lab – SIEM & XDR Practice
📌 Overview
This repository documents my Wazuh home lab setup, built for hands‑on practice in SIEM (Security Information and Event Management) and XDR (Extended Detection and Response). The lab demonstrates how to deploy, register, and troubleshoot Wazuh agents across Windows Server, Windows 11, and Kali Linux environments, with detailed commands and configuration notes.

⚙️ Lab Architecture
Manager: Ubuntu Server (LAN IP 192.168.1.xx) running Wazuh Manager, Indexer, and Dashboard.

Agents:

Windows Server 2022 (10.10.10.xxx)

Windows 11 Pro (192.168.1.x)

Windows Server 2022 (10.10.10.xx)

Kali Linux (10.10.10.xxx)

All agents are registered under the default group and monitored via the Wazuh dashboard.

🛠 Deployment Steps
1. Manager Configuration
bash
sudo /var/ossec/bin/manage_agents
# A → Add agent
# E → Extract key
# L → List agents
# R → Remove agent
Enable <auth> in /var/ossec/etc/ossec.conf:

xml
<auth>
  <disabled>no</disabled>
  <port>1515</port>
  <use_source_ip>yes</use_source_ip>
</auth>
Firewall rules:

bash
sudo ufw allow 1514/tcp
sudo ufw allow 1515/tcp
sudo ufw reload
2. Windows Agent
Install MSI package.

Configure via GUI:

Manager IP: 192.168.1.xx

Authentication Key: paste from manager

Restart service:

powershell
net stop wazuh-agent
net start wazuh-agent
3. Kali/Linux Agent
Install .deb:

bash
wget https://packages.wazuh.com/4.x/apt/pool/main/w/wazuh-agent/wazuh-agent_4.14.9-1_amd64.deb
sudo WAZUH_MANAGER='192.168.1.xx' WAZUH_AGENT_NAME='kali2025' dpkg -i ./wazuh-agent_4.14.9-1_amd64.deb
Add key to /var/ossec/etc/client.keys:

Code
004 kali2025 10.10.10.xxx <auth_key>
Restart agent:

bash
sudo systemctl enable wazuh-agent
sudo systemctl restart wazuh-agent
📊 Verification
On Manager:

bash
sudo /var/ossec/bin/agent_control -l
Active → Connected successfully

Pending → Registered but no heartbeat

Never connected → Agent not reaching manager

On Agent (Linux):

bash
systemctl status wazuh-agent
nc -vz 192.168.1.xx 1514
⚠️ Common Issues
Agent Running but Dashboard Disconnected → Wrong Manager IP or firewall block.

Never Connected → Key mismatch or <auth> misconfigured.

ID Gaps (001, 002, 004, 005) → IDs auto‑increment; reset client.keys to renumber.

📚 Lessons Learned
Always use LAN/Bridged IP for Manager IP in agent configs.

Document agent IDs, names, and IPs for tracking.

Restart both manager and agent after key changes.

IDs are auto‑incrementing; reset client.keys if sequential numbering is required.

“Running but disconnected” almost always means wrong IP or firewall block.
