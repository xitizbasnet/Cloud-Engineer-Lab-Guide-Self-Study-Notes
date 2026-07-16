# LAB 06 – Azure Business Continuity & Disaster Recovery (BCDR)
## Azure Site Recovery (ASR) & Azure Backup

> [!IMPORTANT]
> This lab demonstrates how to configure **Azure Backup** and **Azure Site Recovery (ASR)** to protect Azure Virtual Machines (VMs), validate disaster recovery capabilities, and measure recovery objectives (RTO/RPO).

---

## 📋 Lab Information

| Property | Value |
|----------|-------|
| **Lab ID** | LAB 06 |
| **Title** | Azure BCDR – ASR & Azure Backup |
| **Estimated Duration** | 2–3 Hours |
| **Difficulty** | Intermediate |

---

# 🎯 Objective

Configure **Azure Site Recovery (ASR)** for virtual machine replication and **Azure Backup** for data protection.

At the end of this lab, you will:

- Configure an Azure Recovery Services Vault.
- Enable Azure Backup for Azure Virtual Machines.
- Configure Azure Site Recovery (ASR) replication.
- Perform a test failover.
- Validate the configured **Recovery Time Objective (RTO)** and **Recovery Point Objective (RPO)**.

---

# 📚 Prerequisites

Before beginning this lab, ensure that:

- You have an active Azure subscription.
- The Azure CLI is installed and authenticated.
- A resource group named:

```text
cloud-lab-rg
```

already exists.

- A virtual machine named:

```text
cloud-linux-vm
```

is deployed.

---

# 🛠️ Lab Tasks

---

# Step 1 — Create a Recovery Services Vault

Create an Azure Recovery Services Vault using Azure CLI.

## Azure CLI

```bash
az backup vault create \
--resource-group cloud-lab-rg \
--name cloud-lab-vault \
--location eastus
```

## Configure Backup Storage Replication

After the vault has been created:

1. Open **Recovery Services Vault**.
2. Navigate to:

```text
Properties
    └── Backup Configuration
```

3. Set the storage replication type to:

```text
Geo-Redundant Storage (GRS)
```

> [!NOTE]
> Geo-Redundant Storage (GRS) provides protection against regional outages by replicating backup data to a paired Azure region.

---

# Step 2 — Enable Azure Backup for Virtual Machines

Enable Azure Backup protection for the virtual machine.

## Azure CLI

```bash
az backup protection enable-for-vm \
--resource-group cloud-lab-rg \
--vault-name cloud-lab-vault \
--vm cloud-linux-vm \
--policy-name DefaultPolicy
```

---

## Trigger an On-Demand Backup

Run an immediate backup after enabling protection.

```bash
az backup protection backup-now \
--resource-group cloud-lab-rg \
--vault-name cloud-lab-vault \
--container-name cloud-linux-vm \
--item-name cloud-linux-vm \
--retain-until 30-01-2025
```

> [!TIP]
> Performing an on-demand backup verifies that backup protection is functioning correctly before configuring disaster recovery.

---

# Step 3 — Configure Azure Site Recovery (ASR)

Configure Azure Site Recovery replication for the virtual machine.

## Azure Portal

Navigate to:

```text
Recovery Services Vault
    └── Site Recovery
            └── Replicate Application
```

Configure the following settings:

| Setting | Value |
|----------|-------|
| **Source** | Azure |
| **Source Location** | East US |
| **Target Region** | West US |
| **Replicated VM** | cloud-linux-vm |
| **Recovery Point Objective (RPO)** | 15 Minutes |
| **Recovery Point Retention** | 24 Hours |

---

## Create a Replication Policy

Use Azure CLI to create the replication policy.

```bash
az site-recovery replication-policy create \
--resource-group cloud-lab-rg \
--vault-name cloud-lab-vault \
--name 15min-policy \
--recovery-point-threshold-in-minutes 15
```

---

# Step 4 — Test Failover & Validate Recovery Time Objective (RTO)

Perform a test failover to verify disaster recovery readiness.

## Azure Portal

Navigate to:

```text
Recovery Services Vault
    └── Replicated Items
            └── cloud-linux-vm
                    └── Test Failover
```

Select:

- Recovery Point
- **OK**

---

## Validate the Test Failover

Verify that:

- ✅ The replicated virtual machine boots successfully.
- ✅ The application is reachable in the secondary region.
- ✅ Networking functions as expected.
- ✅ The VM is operating normally.

---

## Clean Up the Test Failover

After validation is complete:

Navigate to:

```text
Recovery Services Vault
    └── Replicated Items
            └── cloud-linux-vm
                    └── Cleanup Test Failover
```

---

# 📈 Validate Recovery Objectives

Record the following metrics after completing the failover.

| Recovery Objective | Description |
|-------------------|-------------|
| **RTO (Recovery Time Objective)** | Time from initiating failover until the VM is fully accessible in the target region. |
| **RPO (Recovery Point Objective)** | Maximum acceptable amount of data loss measured by the latest recovery point. |

> [!IMPORTANT]
> Document your **RTO** by measuring the elapsed time from triggering the failover until the virtual machine becomes accessible in the target region.

---

# ✅ Verification Checklist

After completing this lab, verify that:

- [ ] Recovery Services Vault has been created.
- [ ] Backup storage replication is configured as Geo-Redundant Storage (GRS).
- [ ] Azure Backup is enabled for **cloud-linux-vm**.
- [ ] An on-demand backup completed successfully.
- [ ] Azure Site Recovery replication is configured.
- [ ] Replication policy uses a 15-minute RPO.
- [ ] Test failover completed successfully.
- [ ] Test failover cleanup completed.
- [ ] Recovery Time Objective (RTO) has been documented.

---

# 💡 Best Practice Tips

> [!TIP]
> **Run a test failover at least once per quarter.** Real disasters are not the time to discover that your Business Continuity and Disaster Recovery (BCDR) plan does not work.

---

> [!TIP]
> Use **Geo-Redundant Storage (GRS)** for your Recovery Services Vault to provide protection against regional Azure outages.

---

> [!TIP]
> Configure backup retention policies according to your organization's compliance requirements. Many regulatory standards require a **minimum retention period of 90 days**.

---

# 📖 Summary

In this lab, you completed the following tasks:

- Created an Azure Recovery Services Vault.
- Configured Geo-Redundant Storage (GRS).
- Enabled Azure Backup for an Azure Virtual Machine.
- Performed an on-demand backup.
- Configured Azure Site Recovery replication.
- Created a replication policy with a 15-minute RPO.
- Executed and validated a test failover.
- Recorded the Recovery Time Objective (RTO).
- Applied Azure BCDR best practices.

---

## 📚 References

- Azure Recovery Services Vault
- Azure Backup
- Azure Site Recovery (ASR)
- Business Continuity and Disaster Recovery (BCDR)
- Recovery Time Objective (RTO)
- Recovery Point Objective (RPO)
