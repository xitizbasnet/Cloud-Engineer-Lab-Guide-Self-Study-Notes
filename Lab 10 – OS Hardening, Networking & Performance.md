# Lab 10 – OS Hardening, Networking & Performance

> [!IMPORTANT]
> This lab focuses on operating system hardening, performance optimization, networking, firewall configuration, hardware management, and technical documentation. You will secure Linux systems, perform compliance audits, configure router and firewall features, manage applications across multiple operating systems, and create a professional Knowledge Base (KB) article.

---

# 📋 Lab Information

| Property | Value |
|----------|-------|
| **Lab ID** | LAB 10 |
| **Title** | OS Hardening, Networking & Performance |
| **Estimated Duration** | 2–3 Hours |
| **Difficulty** | Advanced |

---

# 🎯 Objective

Perform operating system performance tuning and hardening across **Linux**, **Windows**, and **macOS**. Configure routers, firewalls, NAT, port forwarding, DMZ, and hardware management. Create and validate professional Knowledge Base (KB) documentation.

At the end of this lab, you will:

- Harden a Linux operating system.
- Perform compliance auditing using Lynis.
- Configure routing, NAT, port forwarding, and DMZ.
- Configure port triggering concepts.
- Manage hardware and applications across Windows, Linux, and macOS.
- Produce a professional Knowledge Base article.

---

# 📚 Prerequisites

Before starting this lab, ensure that you have:

- A Linux server with sudo privileges.
- Windows and macOS systems (or virtual machines) for application management tasks.
- Administrative access to a Linux router, pfSense, or OPNsense firewall.
- Internet connectivity for installing required packages.
- Access to an internal documentation platform such as Confluence, SharePoint, or GitHub Wiki.

---

# 🛠️ Lab Tasks

---

# Step 1 — Linux Performance Tuning & Hardening

Configure Linux kernel parameters and harden SSH.

---

## Configure Kernel Parameters (`sysctl`)

Edit:

```text
/etc/sysctl.conf
```

Add or update the following parameters.

```text
net.ipv4.tcp_syncookies = 1
# SYN flood protection

net.ipv4.conf.all.rp_filter = 1
# Reverse path filtering

net.ipv4.conf.all.accept_redirects = 0
# Disable ICMP redirects

kernel.dmesg_restrict = 1
# Restrict dmesg access

vm.swappiness = 10
# Reduce swapping
```

Apply the configuration.

```bash
sudo sysctl -p
```

---

## Harden SSH Configuration

Edit:

```text
/etc/ssh/sshd_config
```

Configure the following settings.

```text
PermitRootLogin no

PasswordAuthentication no

MaxAuthTries 3

AllowUsers cloudadmin
```

Restart the SSH service.

```bash
sudo systemctl restart sshd
```

> [!IMPORTANT]
> Disabling password authentication requires SSH key-based authentication to be configured before reconnecting.

---

# Step 2 — Machine Compliance Check with Lynis

Install and run the Lynis security auditing tool.

---

## Install Lynis

```bash
sudo apt install lynis -y
```

---

## Run a System Audit

```bash
sudo lynis audit system
```

---

## Review the Audit Report

Audit report location:

```text
/var/log/lynis.log
```

Focus on:

- Hardening index score (aim for **80+**).
- Warnings about open ports.
- Weak SSH configuration.
- World-writable files.
- Remediate findings and re-run the audit.

> [!TIP]
> Re-running Lynis after remediation validates that security improvements have been successfully applied.

---

# Step 3 — Router, Firewall & DMZ Configuration

Simulate routing functionality using a Linux router or physical router CLI.

---

## Enable IP Forwarding

```bash
echo 'net.ipv4.ip_forward=1' >> /etc/sysctl.conf

sudo sysctl -p
```

---

## Configure NAT Masquerading

```bash
sudo iptables -t nat \
-A POSTROUTING \
-o eth0 \
-j MASQUERADE
```

---

## Configure Port Forwarding

Forward TCP port **8080** to an internal web server.

```bash
sudo iptables -t nat \
-A PREROUTING \
-p tcp \
--dport 8080 \
-j DNAT \
--to-destination 192.168.10.20:80
```

---

## Configure a DMZ

Route traffic for the DMZ subnet.

```bash
sudo iptables \
-A FORWARD \
-d 192.168.20.0/24 \
-j ACCEPT
```

```bash
sudo iptables \
-A FORWARD \
-s 192.168.20.0/24 \
-j ACCEPT
```

> [!NOTE]
> A DMZ isolates publicly accessible services from the internal production network, improving overall security.

---

# Step 4 — Port Triggering Configuration

Port triggering dynamically opens inbound ports when outbound traffic matches a defined trigger.

On a **pfSense** or **OPNsense** firewall:

1. Navigate to:

```text
Firewall
    └── NAT
            └── Port Forward
```

2. Create a trigger rule:

- Internal host connects to port **5060 (SIP)**.
- Automatically open inbound ports **10000–20000 (RTP Audio)**.

3. This configuration is commonly used for VoIP applications.

---

## Verify NAT Rules on Linux

```bash
sudo iptables -t nat -L -v --line-numbers
```

> [!NOTE]
> Port triggering differs from static port forwarding because ports are opened only when outbound traffic initiates the connection.

---

# Step 5 — Hardware, Peripherals & Application Management

Manage hardware inventory and software across Linux, Windows, and macOS.

---

## Linux Hardware Detection

Full hardware inventory.

```bash
lshw -short
```

PCI devices.

```bash
lspci
```

USB devices.

```bash
lsusb
```

Memory information.

```bash
dmidecode -t memory
```

---

## Windows Application Management (PowerShell)

List installed applications.

```powershell
Get-WmiObject -Class Win32_Product |
Select Name, Version
```

Silent installation example.

```powershell
Start-Process 'setup.exe' -ArgumentList '/S' -Wait
```

---

## macOS Application Management

Install applications using Homebrew.

```bash
brew install wget curl vim
```

Manage installed packages.

```bash
brew list

brew update

brew upgrade
```

> [!TIP]
> Homebrew simplifies package management and software updates on macOS.

---

# Step 6 — Knowledge Base Writing & Content Publishing

Create a Knowledge Base article using the following structure.

---

## Knowledge Base Template

**Title**

```text
How to Configure DNS Forwarding on BIND9
```

**Affected Systems**

```text
Ubuntu 20.04+

RHEL 8+
```

**Problem Statement**

```text
Internal DNS not resolving external domains.
```

**Root Cause**

```text
forwarders not configured in named.conf.options.
```

**Solution Steps**

- Numbered instructions.
- Precise commands.
- Expected output.

**Verification**

- Describe how to confirm the issue has been resolved.

**Rollback Plan**

- Explain how to undo the changes if issues occur.

---

## Publish the Article

Publish the completed Knowledge Base article to your internal documentation platform.

Examples include:

- Confluence
- SharePoint
- GitHub Wiki

---

# 🌐 Networking Overview

```text
Internet
     │
     ▼
 Linux Router
     │
     ├── NAT
     ├── Port Forwarding
     ├── Port Triggering
     └── DMZ
             │
             ▼
      Internal Servers

Linux
│
├── Performance Tuning
├── Hardening
└── Compliance (Lynis)

Windows
└── Application Management

macOS
└── Homebrew Package Management
```

---

# ✅ Verification Checklist

After completing this lab, verify that:

- [ ] Linux kernel parameters have been configured.
- [ ] SSH hardening settings have been applied.
- [ ] Lynis audit completes successfully.
- [ ] Hardening score meets or exceeds the target.
- [ ] IP forwarding is enabled.
- [ ] NAT masquerading functions correctly.
- [ ] Port forwarding rules are operational.
- [ ] DMZ routing has been configured.
- [ ] NAT rules can be verified.
- [ ] Hardware inventory commands execute successfully.
- [ ] Windows applications can be managed with PowerShell.
- [ ] Homebrew successfully manages macOS applications.
- [ ] Knowledge Base article is completed.
- [ ] KB article is published to the internal documentation platform.

---

# 💡 Best Practice Tips

> [!TIP]
> Automate compliance checks using **Lynis** or **OpenSCAP** within your CI/CD pipeline to detect security drift before deployment.

---

> [!TIP]
> Document every firewall rule change with a ticket number, implementation date, and engineer name in your configuration management system.

---

> [!TIP]
> Create Knowledge Base articles immediately after resolving incidents while implementation details are still fresh. Well-written documentation significantly reduces future support effort.

---

# 📖 Summary

In this lab, you completed the following tasks:

- Hardened Linux kernel and SSH configurations.
- Performed a compliance audit using Lynis.
- Configured Linux routing, NAT, port forwarding, and DMZ.
- Reviewed port triggering concepts using pfSense/OPNsense.
- Managed hardware and software across Linux, Windows, and macOS.
- Created and published a professional Knowledge Base article.
- Applied operating system hardening and enterprise networking best practices.

---

# 📚 References

- Linux Kernel (`sysctl`)
- OpenSSH
- Lynis
- IPTables
- Network Address Translation (NAT)
- Port Forwarding
- Port Triggering
- DMZ
- pfSense
- OPNsense
- Homebrew
- Windows PowerShell
- BIND9
- Knowledge Base Documentation
