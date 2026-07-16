# Lab 08 – Automation: Bash, PowerShell & APIs

> [!IMPORTANT]
> This lab demonstrates how to automate common cloud administration tasks using **Bash**, **PowerShell**, **Azure CLI**, **Azure REST APIs**, and **Infoblox WAPI**. You will also configure scheduled automation using **cron** and query Azure Cost Management for usage reporting.

---

# 📋 Lab Information

| Property | Value |
|----------|-------|
| **Lab ID** | LAB 08 |
| **Title** | Automation – Bash, PowerShell & APIs |
| **Estimated Duration** | 2–3 Hours |
| **Difficulty** | Intermediate |

---

# 🎯 Objective

Write **Bash** and **PowerShell** scripts for server automation. Use the **Azure Cost Management API** and **Azure REST APIs** to build tools for cost reporting and resource management.

At the end of this lab, you will:

- Create a Bash server health monitoring script.
- Automate DNS management using the Infoblox REST API.
- Automate Azure virtual machine management with PowerShell.
- Query Azure Cost Management using the Azure REST API.
- Schedule recurring automation tasks using cron.

---

# 📚 Prerequisites

Before starting this lab, ensure that you have:

- An active Azure subscription.
- Azure CLI installed and authenticated.
- Azure PowerShell module installed.
- Bash shell available on Linux.
- Access to an Infoblox appliance with WAPI enabled.
- Appropriate permissions to manage Azure resources.
- Administrative access to the Linux server.

---

# 🛠️ Lab Tasks

---

# Step 1 — Bash Script: Server Health Check Automation

Create a Bash script to monitor server health and generate alerts.

## Health Check Script

```bash
#!/bin/bash

# Server Health Check Script

LOGFILE=/var/log/healthcheck.log
DATE=$(date '+%Y-%m-%d %H:%M:%S')

echo "[$DATE] --- Health Check Start ---" >> $LOGFILE

# CPU Usage
CPU=$(top -bn1 | grep 'Cpu(s)' | awk '{print $2}')
echo "CPU Usage: $CPU%" >> $LOGFILE

# Memory
MEM=$(free -h | awk '/^Mem:/ {print $3"/"$2}')
echo "Memory: $MEM" >> $LOGFILE

# Disk
DISK=$(df -h / | awk 'NR==2 {print $5}')
echo "Disk Usage: $DISK" >> $LOGFILE

# Alert if disk > 80%
USAGE=$(echo $DISK | sed 's/%//')

if [ $USAGE -gt 80 ]; then
echo "ALERT: Disk usage critical!" | mail -s 'Disk Alert' admin@cloud.lab
fi
```

> [!NOTE]
> This script records CPU, memory, and disk utilization in a log file and sends an email alert if disk usage exceeds **80%**.

---

# Step 2 — Bash: Automate Infoblox API Calls

Infoblox exposes a REST API (WAPI). Use `curl` to create DNS records.

## Configure API Variables

```bash
INFOBLOX_IP='192.168.10.5'
CREDS='admin:infoblox'
```

---

## Create an A Record

```bash
curl -k -u $CREDS -H 'Content-Type: application/json' \
-X POST "https://$INFOBLOX_IP/wapi/v2.12/record:a" \
-d '{"name":"test.cloud.lab.local","ipv4addr":"192.168.10.50"}'
```

---

## List All A Records

```bash
curl -k -u $CREDS \
"https://$INFOBLOX_IP/wapi/v2.12/record:a" \
| python3 -m json.tool
```

> [!TIP]
> Formatting JSON output with `python3 -m json.tool` makes API responses easier to read during troubleshooting.

---

# Step 3 — PowerShell: Azure Automation

Automate Azure virtual machine management using Azure PowerShell.

---

## Connect to Azure

```powershell
Connect-AzAccount
```

---

## List All Virtual Machines

```powershell
Get-AzVM |
Select-Object Name,
ResourceGroupName,
Location,
@{N='Size';E={$_.HardwareProfile.VmSize}}
```

---

## Stop All Non-Production Virtual Machines

```powershell
$vms = Get-AzVM |
Where-Object {$_.Tags['env'] -ne 'prod'}

$vms |
ForEach-Object {
    Stop-AzVM `
        -ResourceGroupName $_.ResourceGroupName `
        -Name $_.Name `
        -Force
}
```

---

## Start a Virtual Machine

```powershell
Start-AzVM `
-ResourceGroupName cloud-lab-rg `
-Name cloud-linux-vm
```

> [!NOTE]
> Tag-based automation helps reduce Azure costs by shutting down non-production workloads outside business hours.

---

# Step 4 — Azure Cost Management API

Retrieve Azure cost information using the Azure REST API.

---

## Obtain an OAuth Access Token

```bash
TOKEN=$(az account get-access-token --query accessToken -o tsv)
SUBSCRIPTION_ID=$(az account show --query id -o tsv)
```

---

## Query Cost for the Last 30 Days

```bash
curl -X POST \
-H "Authorization: Bearer $TOKEN" \
-H 'Content-Type: application/json' \
"https://management.azure.com/subscriptions/$SUBSCRIPTION_ID/providers/Microsoft.CostManagement/query?api-version=2023-03-01" \
-d '{
"type":"Usage",
"timeframe":"MonthToDate",
"dataset":{
"granularity":"Daily",
"aggregation":{
"totalCost":{
"name":"Cost",
"function":"Sum"
}
}
}
}'
```

> [!IMPORTANT]
> The Azure Cost Management API provides programmatic access to subscription usage and spending data for reporting and automation.

---

# Step 5 — Cron Job Automation

Schedule recurring automation tasks using cron.

---

## Edit the Crontab

```bash
crontab -e
```

---

## Schedule the Health Check Every 15 Minutes

```cron
*/15 * * * * /home/cloudadmin/scripts/healthcheck.sh
```

---

## Schedule Daily Backup Rotation

```cron
0 2 * * * /home/cloudadmin/scripts/backup_rotate.sh
```

---

## List All User Cron Jobs

```bash
crontab -l
```

---

## View System Cron Jobs

```bash
cat /etc/cron.d/*
```

> [!NOTE]
> Cron provides reliable scheduling for recurring maintenance, monitoring, backup, and automation tasks.

---

# 🔄 Automation Workflow

```text
Bash Scripts
        │
        ├── Health Monitoring
        ├── Logging
        └── Email Alerts
                │
                ▼
            Cron Scheduler
                │
                ▼
Azure CLI ─────────────── Azure REST API
        │                       │
        ▼                       ▼
 VM Automation         Cost Management API

                │
                ▼

          Infoblox WAPI
                │
                ▼
        DNS Record Automation
```

---

# ✅ Verification Checklist

After completing this lab, verify that:

- [ ] Bash health check script executes successfully.
- [ ] Server health information is written to the log file.
- [ ] Disk usage alert triggers when utilization exceeds 80%.
- [ ] Infoblox API successfully creates an A record.
- [ ] DNS records can be listed through WAPI.
- [ ] Azure PowerShell connects successfully.
- [ ] Non-production VMs stop successfully.
- [ ] Target VM starts successfully.
- [ ] Azure Cost Management API returns usage data.
- [ ] Cron jobs are configured correctly.
- [ ] Scheduled automation executes successfully.

---

# 💡 Best Practice Tips

> [!TIP]
> Always add error handling in Bash scripts by including `set -e` (exit on error) and `set -o pipefail` at the beginning of your scripts.

---

> [!TIP]
> Store API tokens and credentials in **Azure Key Vault** or **AWS Secrets Manager**. Never hardcode sensitive information directly in scripts.

---

> [!TIP]
> Use `az account set --subscription` in automation scripts to ensure that all commands target the correct Azure subscription.

---

# 📖 Summary

In this lab, you completed the following tasks:

- Created a Bash server health monitoring script.
- Logged CPU, memory, and disk utilization.
- Automated DNS record management using the Infoblox WAPI.
- Managed Azure virtual machines with PowerShell.
- Queried Azure Cost Management using the Azure REST API.
- Configured scheduled automation using cron.
- Applied automation best practices for scripting and cloud resource management.

---

# 📚 References

- Bash
- PowerShell
- Azure CLI
- Azure PowerShell
- Azure REST API
- Azure Cost Management API
- Infoblox WAPI
- Cron
- Azure Key Vault
- AWS Secrets Manager
