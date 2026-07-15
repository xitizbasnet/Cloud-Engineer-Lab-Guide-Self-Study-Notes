# Lab 03 — Infoblox: On-Prem to Azure & AWS Migration

> **Difficulty:** 🔴 Advanced

> **Estimated Duration:** ⏱️ 3–4 Hours

---

# 📖 Overview

This lab focuses on migrating an on-premises **Infoblox DDI** infrastructure to **Microsoft Azure** and **Amazon Web Services (AWS)** using enterprise deployment models such as **Anycast Site Router (ASR)**, **Grid Master Candidate (GMC)**, and **Multi-Master Zone (MMZ)**.

You will also configure integrations with:

* Active Directory (AD)
* LDAP
* RADIUS
* Azure Site Recovery (ASR)
* DNS Traffic Control (ATC)
* Response Policy Zones (RPZ)

---

# 🎯 Objective

Migrate on-premises Infoblox DDI servers to Azure and AWS using:

* ASR (Anycast Site Router)
* GMC (Grid Master Candidate)
* MMZ (Multi-Master Zone)

Configure Infoblox integrations with:

* Active Directory
* LDAP
* RADIUS

---

# Exercise 1 — Understand the Infoblox Grid Architecture

## Infoblox Grid Components

The Infoblox Grid consists of the following components:

| Component                       | Description                                         |
| ------------------------------- | --------------------------------------------------- |
| **Grid Master (GM)**            | Central management node                             |
| **Grid Master Candidate (GMC)** | High Availability (HA) failover for the Grid Master |
| **Grid Members**                | DNS/DHCP service nodes                              |
| **MMZ**                         | Multi-Master Zone for distributed authoritative DNS |

---

## Review the Existing Grid

Open the Infoblox GUI and review the current on-premises Grid topology.

**Navigation**

```text
Grid → Grid Manager
```

---

# Exercise 2 — Deploy Infoblox vNIOS on Microsoft Azure

## Step 1: Obtain the vNIOS Image

1. Download the **vNIOS OVA** from the Infoblox Support Portal.
2. Deploy using:

   * Azure Marketplace, **or**
   * Upload the VHD to an Azure Storage Account.

---

## Step 2: Create an Azure Resource Group

```bash id="x9f3kk"
# Create resource group
az group create --name infoblox-rg --location eastus
```

---

## Step 3: Deploy the Virtual Machine

```bash id="u6krwd"
# Create VM from custom image
az vm create \
--resource-group infoblox-rg \
--name infoblox-gm \
--image /subscriptions/.../customImage \
--admin-username admin \
--vnet-name infoblox-vnet \
--subnet infoblox-subnet
```

---

## Step 4: Configure Required Network Ports

Open the following ports:

| Port  | Protocol | Purpose            |
| ----- | -------- | ------------------ |
| 443   | HTTPS    | Web Management     |
| 2114  | TCP      | Grid Communication |
| 53    | TCP/UDP  | DNS                |
| 67/68 | UDP      | DHCP               |

> [!IMPORTANT]
> Ensure Azure Network Security Groups (NSGs) permit communication on all required ports before joining the Grid.

---

# Exercise 3 — Deploy Infoblox vNIOS on AWS

## Launch the NIOS AMI

Deploy the Infoblox virtual appliance from the AWS Marketplace.

```bash id="qg57g6"
# Launch from AWS Marketplace (NIOS AMI)
aws ec2 run-instances \
--image-id ami-XXXXXXXXX \
--instance-type r5.large \
--subnet-id subnet-XXXXX \
--security-group-ids sg-XXXXX \
--region ap-south-1
```

---

## Configure the Security Group

Allow communication on the following ports between the **Grid Master** and **Grid Members**.

| Port | Protocol | Purpose            |
| ---- | -------- | ------------------ |
| 53   | UDP      | DNS                |
| 53   | TCP      | DNS                |
| 443  | TCP      | HTTPS              |
| 2114 | TCP      | Grid Communication |

---

# Exercise 4 — Join Cloud vNIOS Members to the Grid

## Configure Grid Membership

On the Azure or AWS vNIOS member, open the NIOS CLI or Web UI and configure Grid membership.

```text
set gridinfo

set membership grid-master-ip=<grid-master-ip> shared-secret=<shared-secret>
```

---

## Approve the Join Request

On the Grid Master, approve the pending Grid member request.

**Navigation**

```text
Grid → Pending Join Requests
```

> [!NOTE]
> A Grid Member does not become operational until the join request is approved by the Grid Master.

---

# Exercise 5 — Configure Active Directory, LDAP, and RADIUS Integration

## Active Directory Integration

Navigate to:

```text
Grid → Grid Properties → Authentication → Add AD Domain
```

Configure authentication.

```text
set ad_auth domain=cloud.lab.local
```

---

## LDAP Integration

Navigate to:

```text
Security → Authentication Policy → Add LDAP Server
```

Supported ports:

* 389 (LDAP)
* 636 (LDAPS)

---

## RADIUS Integration

Navigate to:

```text
Grid → Grid Properties → RADIUS
```

Configure:

* RADIUS Server IP
* Shared Secret

---

# Exercise 6 — Configure DNS Traffic Control (ATC) and Response Policy Zones (RPZ)

## Enable Active Trust Cloud (ATC)

Active Trust Cloud provides real-time DNS threat intelligence.

Navigate to:

```text
Security → DNS Firewall → Active Trust Cloud → Enable
```

---

## Configure Response Policy Zones (RPZ)

Create an RPZ to block malicious domains.

Navigation:

```text
Data Management → DNS → Response Policy Zones → Add Zone
```

Configure rule actions such as:

* PASSTHRU
* NXDOMAIN

---

## Enable Threat Analytics (TA)

Enable behavioral DNS monitoring.

Navigation:

```text
Security → Threat Analytics
```

---

# Exercise 7 — Configure Azure Site Recovery (BCDR)

## Enable Replication

Enable Azure Site Recovery protection for the Infoblox virtual machine.

```bash id="nkm3u1"
# Enable replication for Infoblox VM
az site-recovery protected-item create \
--resource-group infoblox-rg \
--vault-name infoblox-vault \
--fabric-name PrimaryFabric \
--protection-container PrimaryContainer
```

---

## Configure Disaster Recovery

Set:

| Setting                        | Value      |
| ------------------------------ | ---------- |
| Recovery Point Objective (RPO) | 15 Minutes |
| Test Failover                  | Quarterly  |

> [!IMPORTANT]
> Regular failover testing validates disaster recovery readiness and helps identify configuration issues before a real outage occurs.

---

# ✅ Best Practices

> [!TIP]
> Always deploy a **Grid Master Candidate (GMC)** alongside the **Grid Master (GM)**. Without a GMC, failure of the Grid Master can impact the availability of the entire Infoblox Grid.

---

> [!TIP]
> Use separate **Virtual Networks (VNets)** or **subnets** for:
>
> * Infoblox management traffic
> * DNS service traffic
>
> Network segmentation improves security, scalability, and operational management.

---

> [!TIP]
> Store the Grid shared secret securely in:
>
> * Azure Key Vault
> * AWS Secrets Manager
>
> Never store shared secrets in plaintext configuration files or documentation.

---

# ✔️ Lab Summary

In this lab, you completed the following tasks:

* Reviewed the Infoblox Grid architecture.
* Deployed Infoblox vNIOS on Microsoft Azure.
* Deployed Infoblox vNIOS on AWS.
* Joined cloud Grid Members to the existing Infoblox Grid.
* Integrated Infoblox with Active Directory, LDAP, and RADIUS.
* Configured DNS Traffic Control (ATC).
* Created Response Policy Zones (RPZ).
* Enabled Threat Analytics (TA).
* Configured Azure Site Recovery for business continuity and disaster recovery.
* Reviewed enterprise deployment and security best practices.
