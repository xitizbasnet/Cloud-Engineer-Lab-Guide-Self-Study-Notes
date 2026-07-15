# Lab 01 — Linux OS & Multi-Distro Administration

> **Difficulty:** 🟢 Beginner

> **Estimated Duration:** ⏱️ 2–3 Hours

---

## 📖 Overview

This lab provides hands-on experience administering multiple Linux distributions (**RHEL, CentOS, Fedora, and Ubuntu**) along with **Microsoft Windows Server**.

By completing this lab, you will learn how to:

* Install Linux operating systems in a virtual environment.
* Perform package management across multiple distributions.
* Create and manage users and groups.
* Control and monitor system services.
* Install and configure core Windows Server roles.

---

## 🎯 Objective

Gain hands-on experience administering multiple Linux distributions (**RHEL, CentOS, Fedora, Ubuntu**) and Microsoft Windows Server.

Learn:

* Linux installation
* User management
* Package management
* Service control

---

## ✅ Prerequisites

Before starting this lab, ensure the following requirements are met:

* VirtualBox or VMware Workstation installed
* Minimum **8 GB RAM**
* ISO images for:

  * RHEL
  * Ubuntu
  * Windows Server

> [!NOTE]
> Additional memory and CPU resources may improve VM performance during the lab.

---

# Exercise 1 — Install RHEL / CentOS on a Virtual Machine

## Step 1: Create the Virtual Machine

Create a new virtual machine with the following configuration:

| Setting | Value |
| ------- | ----- |
| vCPU    | 2     |
| Memory  | 4 GB  |
| Disk    | 30 GB |

---

## Step 2: Install the Operating System

1. Attach the **RHEL 9 ISO**.
2. Boot the virtual machine.
3. Select **Server with GUI** during installation.
4. Configure the hostname:

```text
cloud-lab-rhel
```

---

## Step 3: Register the RHEL System

After installation, register the server with Red Hat.

```bash
subscription-manager register --username=<user> --password=<pass>
subscription-manager attach --auto
```

> [!IMPORTANT]
> Registration is required to receive official Red Hat updates and security patches.

---

# Exercise 2 — Package Management Across Linux Distributions

## RHEL / CentOS (DNF/YUM)

Update the system and install required packages.

```bash
sudo dnf update -y
sudo dnf install httpd net-tools vim -y
```

---

## Ubuntu / Debian (APT)

Update package repositories and install packages.

```bash
sudo apt update && sudo apt upgrade -y
sudo apt install apache2 net-tools vim -y
```

---

## Fedora

Install NGINX.

```bash
sudo dnf install nginx -y
```

---

# Exercise 3 — User & Group Management

Create users, assign administrative privileges, and manage groups.

```bash
useradd -m -s /bin/bash cloudadmin

passwd cloudadmin

usermod -aG sudo cloudadmin      # Ubuntu

usermod -aG wheel cloudadmin     # RHEL/CentOS

groupadd devops && usermod -aG devops cloudadmin
```

### Tasks Performed

* Create a local user.
* Set a password.
* Assign administrative privileges.
* Create a DevOps group.
* Add the user to the DevOps group.

---

# Exercise 4 — Service Management with systemctl

Manage the Apache HTTP service.

```bash
systemctl start httpd && systemctl enable httpd

systemctl status httpd

systemctl restart httpd

journalctl -u httpd --since '1 hour ago'
```

### Commands Explained

| Command      | Description                                  |
| ------------ | -------------------------------------------- |
| `start`      | Starts the service                           |
| `enable`     | Starts the service automatically during boot |
| `status`     | Displays service status                      |
| `restart`    | Restarts the service                         |
| `journalctl` | Displays service logs                        |

---

# Exercise 5 — Windows Server Role Installation

## Install Server Roles

Open:

**Server Manager → Add Roles and Features**

Install the following roles:

* Active Directory Domain Services (AD DS)
* DNS
* DHCP

---

## Promote the Server to a Domain Controller

Run the following PowerShell commands:

```powershell
Install-WindowsFeature AD-Domain-Services -IncludeManagementTools

Install-ADDSForest -DomainName 'cloud.lab.local'
```

> [!IMPORTANT]
> Promoting a server to a Domain Controller creates a new Active Directory forest named **cloud.lab.local**.

---

# ✅ Best Practices

> [!TIP]
> **Always keep RHEL systems registered and subscribed.**
> Unregistered systems won't receive security patches.

---

> [!TIP]
> Use the following commands regularly to audit installed packages:
>
> **RHEL**
>
> ```bash
> dnf history
> ```
>
> **Ubuntu**
>
> ```bash
> apt list --installed
> ```

---

> [!TIP]
> Name your virtual machines using environment prefixes such as:
>
> * `dev-`
> * `prod-`
> * `lab-`
>
> Using a consistent naming convention from day one helps avoid confusion as environments grow.

---

## ✔️ Lab Summary

In this lab, you completed the following tasks:

* Installed RHEL/CentOS in a virtual machine.
* Registered a Red Hat system.
* Managed software packages across multiple Linux distributions.
* Created Linux users and groups.
* Managed Linux services using `systemctl`.
* Installed Windows Server roles.
* Promoted a Windows Server to an Active Directory Domain Controller.
* Reviewed recommended administration best practices.
