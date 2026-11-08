📘 Wazuh Agent Deployment & Troubleshooting Guide
Windows, Linux, Kali, Ubuntu Agent Onboarding
Project Objective

This document provides a complete guide for:

Windows Server 2022 Agent Deployment
Windows 11 Agent Deployment
Ubuntu Agent Deployment
Kali Linux Agent Deployment
Agent Registration
Agent Key Management
Troubleshooting Pending Status
Never Connected Issues
API Connectivity Validation
Dashboard Validation

Lab Environment

        Component	              IP Address	        Purpose
        Wazuh Manager	          192.168.1.11	    Manager
        Dashboard	              192.168.1.11	    Web Interface
        Indexer	                  192.168.1.11	    OpenSearch
        Windows Server 2022	      10.10.10.163	        Agent
        Windows 11	              10.10.10.x	        Agent
        Kali Linux	              10.10.10.140	        Agent
        Ubuntu Client	          10.10.10.x	        Agent

      
Step 1 – Open Required Ports

On Wazuh Manager:

Shell
sudo ufw allow 1514/tcp
sudo ufw allow 1515/tcp
sudo ufw reload

Verify:

Shell
sudo ufw status
Step 2 – Agent Registration
Manager Side

Start Agent Manager:

Shell
sudo /var/ossec/bin/manage_agents
Add Agent

Select:

Plain Text
A

Example:

Plain Text
Agent Name: kali2025
IP Address: 10.10.10.140
Extract Key

Select:

Plain Text
E

Example:

Plain Text
ID: 004

Generated Key Example:

Plain Text
MDA0IGthbGkyMDI1IGFueSAxNmI0...

Copy the entire key.

Windows Agent Deployment
Download and Install

Download:

PowerShell
Invoke-WebRequest `
-Uri https://packages.wazuh.com/4.x/windows/wazuh-agent-4.14.9-1.msi `
-OutFile $env:tmp\wazuh-agent.msi

Install:

PowerShell
msiexec.exe /i `
$env:tmp\wazuh-agent.msi `
/q `
WAZUH_MANAGER='192.168.1.11' `
WAZUH_AGENT_NAME='Win2022'
Open Agent Manager UI
Plain Text
Start Menu
→ Wazuh Agent Manager

Configure:

Plain Text
Manager IP: 192.168.1.11

Import Key:

Paste the key extracted from manage_agents.

Save.

Restart Agent
On Wazuh Manager:

sudo ufw allow 1514/tcp
sudo ufw allow 1515/tcp
sudo ufw reload


Verify:

sudo ufw status

Step 2 – Agent Registration
Manager Side

Start Agent Manager:

sudo /var/ossec/bin/manage_agents

Add Agent

Select:

A


Example:

Agent Name: kali2025
IP Address: 10.10.10.140

Extract Key

Select:

E


Example:

ID: 004


Generated Key Example:

MDA0IGthbGkyMDI1IGFueSAxNmI0...


Copy the entire key.

Windows Agent Deployment
Download and Install

Download:

Invoke-WebRequest `
-Uri https://packages.wazuh.com/4.x/windows/wazuh-agent-4.14.9-1.msi `
-OutFile $env:tmp\wazuh-agent.msi


Install:

msiexec.exe /i `
$env:tmp\wazuh-agent.msi `
/q `
WAZUH_MANAGER='192.168.1.11' `
WAZUH_AGENT_NAME='Win2022'

Open Agent Manager UI
Start Menu
→ Wazuh Agent Manager


Configure:

Manager IP: 192.168.1.11


Import Key:

Paste the key extracted from manage_agents.

Save.

Restart Agent
net stop wazuh-agent
net start wazuh-agent

Kali Linux Agent Deployment
Download Package
wget https://packages.wazuh.com/4.x/apt/pool/main/w/wazuh-agent/wazuh-agent_4.14.9-1_amd64.deb

Install
sudo WAZUH_MANAGER='192.168.1.11' \
WAZUH_AGENT_NAME='kali2025' \
dpkg -i wazuh-agent_4.14.9-1_amd64.deb

Configure Manager

Edit:

sudo micro /var/ossec/etc/ossec.conf


Update:

<client>
  <server>
    <address>192.168.1.11</address>
    <port>1514</port>
    <protocol>tcp</protocol>
  </server>
</client>

Import Key

Create:

sudo micro /var/ossec/etc/client.keys


Example:

004 kali2025 10.10.10.140 MDA0IGthbGkyMDI1IGFueSAxNmI0...

Enable and Start
sudo systemctl enable wazuh-agent

sudo systemctl start wazuh-agent

Verify
sudo systemctl status wazuh-agent

Ubuntu Agent Deployment

Install:

wget https://packages.wazuh.com/4.x/apt/pool/main/w/wazuh-agent/wazuh-agent_4.14.9-1_amd64.deb

sudo WAZUH_MANAGER='192.168.1.11' \
WAZUH_AGENT_NAME='ubuntu-agent' \
dpkg -i wazuh-agent*.deb


Configure and import key using same procedure as Kali.

Agent Verification

Manager:

sudo /var/ossec/bin/agent_control -l


Expected:

001 Win2022 Active
002 Win11 Active
004 kali2025 Active
005 ubuntu-agent Active

Troubleshooting
Agent Shows "Never Connected"
Verify Service

Linux:

sudo systemctl status wazuh-agent


Windows:

Get-Service wazuh-agent

Verify Manager Connectivity

Linux:

nc -vz 192.168.1.11 1514


Windows:

Test-NetConnection 192.168.1.11 -Port 1514

Verify Firewall

Manager:

sudo ufw allow 1514/tcp
sudo ufw allow 1515/tcp

Verify Manager
sudo systemctl status wazuh-manager

Agent "Pending" Status

Agent successfully registered but has not completed handshake.

Verify:

sudo /var/ossec/bin/agent_control -l


Common Causes:

Incorrect key
Firewall
Wrong Manager IP
Agent Service Stopped
Dashboard Showing Agent Disconnected

Agent may be running but dashboard reports:

Disconnected
Never Connected


Verify:

sudo /var/ossec/bin/agent_control -l


and

sudo tail -100 /var/ossec/logs/ossec.log

Agent ID Management

Remove Agent:

sudo /var/ossec/bin/manage_agents


Choose:

R


Add Agent:

A


Important:

Wazuh IDs are not reused.

Example:

001
002
004 deleted


Next agent becomes:

005

Dashboard API Troubleshooting

Verify:

curl -k https://127.0.0.1:55000


Expected:

{
 "title":"Unauthorized"
}

Dashboard API Authentication

Verify:

curl -k -u "wazuh-wui:<password>" \
https://127.0.0.1:55000/security/user/authenticate?raw=true


Expected:

JWT_TOKEN

Dashboard Shows "No API Available To Connect"

Check:

sudo /var/ossec/bin/wazuh-control status


Problem encountered:

wazuh-modulesd not running


Resolution:

sudo /var/ossec/bin/wazuh-control restart

sleep 20

sudo /var/ossec/bin/wazuh-control status


Result:

Dashboard Online
API Connected

Lessons Learned

✅ Always use Bridged IP for Manager

✅ Agent service running does not mean connected

✅ Validate ports 1514 & 1515

✅ Dashboard Offline may be due to daemon failures

✅ Verify with:

sudo /var/ossec/bin/wazuh-control status


✅ Restarting all Wazuh daemons often resolves module synchronization issues

✅ IDs are auto-increment and not reused

References

Manager Status:

sudo /var/ossec/bin/wazuh-control status


Agent Status:

sudo /var/ossec/bin/agent_control -l


API Test:

curl -k https://127.0.0.1:55000


Dashboard:

https://192.168.1.11


