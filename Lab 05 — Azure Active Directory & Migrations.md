# Lab 05 — Azure Active Directory & Migrations

> **Difficulty:** 🟡 Intermediate
> **Estimated Duration:** ⏱️ 2–3 Hours

---

# 📖 Overview

This lab provides hands-on experience with **Microsoft Entra ID (formerly Azure Active Directory)** auditing, hybrid identity synchronization, secure DNS updates using **GSS-TSIG**, and DNS synchronization between **Azure Private DNS** and **Amazon Route 53**.

You will learn how to:

* Review Microsoft Entra ID audit and sign-in logs.
* Configure hybrid identity using Azure AD Connect.
* Secure dynamic DNS updates with GSS-TSIG.
* Synchronize DNS zones between Azure and AWS.

---

# 🎯 Objective

Perform:

* Azure AD auditing
* Simple migrations using Azure AD Connect
* GSS-TSIG configuration for secure DNS updates
* Route 53 Amazon Sync configurations

---

# Exercise 1 — Audit Microsoft Entra ID (Azure AD)

## Step 1: Review Activity Logs

Use the Azure CLI to review activity logs.

```bash id="f9r0ka"
az monitor activity-log list \
--start-time 2024-01-01T00:00:00Z \
--output table
```

---

## Step 2: Review Audit Logs in the Azure Portal

Navigate to:

```text id="5br6g0"
Azure Active Directory → Audit Logs
```

Filter the logs by:

```text id="6jwwzo"
Category: UserManagement
```

---

## Step 3: Export Logs to Log Analytics

Configure diagnostic settings to export Microsoft Entra ID logs.

```bash id="aqoqsv"
az monitor diagnostic-settings create \
--resource /subscriptions/.../Microsoft.AAD \
--workspace /subscriptions/.../workspaces/myLAW \
--name aad-diag
```

> [!NOTE]
> Exporting audit logs to Log Analytics enables centralized monitoring, reporting, and long-term retention.

---

# Exercise 2 — Migrate On-Premises Active Directory to Microsoft Entra ID

## Step 1: Install Azure AD Connect

1. Download Azure AD Connect from the Microsoft Download Center.
2. Run the installer on the on-premises Domain Controller.
3. Select **Express Settings** for a simple single-forest synchronization.
4. Enter **Global Administrator** credentials for Microsoft Entra ID.
5. Start synchronization.

---

## Step 2: Start a Delta Synchronization

```powershell id="68ghn8"
Start-ADSyncSyncCycle -PolicyType Delta
```

---

## Step 3: Verify User Synchronization

In the Azure portal, navigate to:

```text id="nnv7mj"
Azure Portal → Azure AD → Users
```

Verify that users display:

```text id="7nsg7y"
Source: Windows Server AD
```

> [!IMPORTANT]
> Verify successful synchronization before modifying authentication or identity management settings.

---

# Exercise 3 — Configure GSS-TSIG for Secure Dynamic DNS Updates

## Overview

GSS-TSIG uses **Kerberos authentication** to securely authorize dynamic DNS updates between Windows DHCP and BIND or Infoblox DNS servers.

---

## Step 1: Configure Windows DHCP Credentials

Run the following PowerShell command on the Windows DHCP Server.

```powershell id="jlwm1r"
Set-DhcpServerDnsCredential -Credential (Get-Credential)
```

---

## Step 2: Configure Infoblox

Navigate to:

```text id="8krmv4"
Data Management → DNS → Grid DNS Properties → Dynamic Updates → GSS-TSIG
```

Add the Windows DHCP Server's **Kerberos Principal** as an allowed updater.

> [!NOTE]
> GSS-TSIG ensures authenticated and secure DNS updates without exposing shared credentials.

---

# Exercise 4 — Configure Amazon Route 53 Synchronization

## Step 1: Create a Private Hosted Zone

Create a Route 53 Private Hosted Zone.

```bash id="clc3an"
aws route53 create-hosted-zone \
--name cloud.lab.local \
--caller-reference unique-ref-001 \
--hosted-zone-config PrivateZone=true \
--vpc VPCRegion=ap-south-1,VPCId=vpc-XXXXX
```

---

## Step 2: Synchronize Azure Private DNS with Route 53

Use **AWS Lambda** together with **Amazon EventBridge** to monitor Azure DNS changes and synchronize them with Route 53.

```bash id="qarftr"
aws lambda create-function \
--function-name dns-sync \
--runtime python3.11 \
--handler lambda_function.handler \
--zip-file fileb://dns_sync.zip \
--role arn:aws:iam::ACCOUNT:role/lambda-dns-role
```

> [!IMPORTANT]
> Ensure the Lambda execution role has the required permissions to manage Route 53 hosted zones.

---

# ✅ Best Practices

> [!TIP]
> Always deploy **Azure AD Connect** in **Staging Mode** first to validate synchronization results before enabling production synchronization.

---

> [!TIP]
> Use **Conditional Access Policies** in Microsoft Entra ID to enforce **Multi-Factor Authentication (MFA)** for all administrative accounts without exception.

---

> [!TIP]
> Rotate **TSIG keys** every **90 days** and store them securely in a secrets management solution instead of configuration files.

---

# ✔️ Lab Summary

In this lab, you completed the following tasks:

* Reviewed Microsoft Entra ID activity and audit logs.
* Exported audit logs to Azure Log Analytics.
* Configured hybrid identity using Azure AD Connect.
* Performed an on-demand synchronization with Microsoft Entra ID.
* Verified synchronized users.
* Configured GSS-TSIG for secure dynamic DNS updates.
* Integrated Windows DHCP with Infoblox DNS.
* Created an Amazon Route 53 Private Hosted Zone.
* Configured DNS synchronization using AWS Lambda and EventBridge.
* Reviewed identity, security, and DNS integration best practices.
