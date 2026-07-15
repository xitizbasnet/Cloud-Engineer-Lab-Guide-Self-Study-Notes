# Lab 02 — DNS, DHCP & IPAM Configuration

> **Difficulty:** 🟡 Intermediate

> **Estimated Duration:** ⏱️ 2–3 Hours

---

# 📖 Overview

This lab provides hands-on experience configuring **DNS (BIND)**, **DHCP**, and **IP Address Management (IPAM)** services on Linux.

You will also learn how to:

* Configure BIND DNS.
* Deploy and manage a DHCP server.
* Implement DNS Round-Robin load balancing.
* Manage IP allocations using phpIPAM.
* Integrate Linux DNS with Microsoft Active Directory Domain Services (AD DS).

---

# 🎯 Objective

Configure **DNS (BIND)**, **DHCP**, and **IPAM** services on Linux.

Understand:

* DNS Load Balancing
* Zone Transfers
* Integration with AD DS in a Windows environment

---

# Exercise 1 — Install and Configure BIND DNS on Ubuntu

## Step 1: Install BIND

Install the required BIND packages.

```bash
sudo apt install bind9 bind9utils -y
```

---

## Step 2: Configure the Forward Zone

Edit the following configuration file:

```text
/etc/bind/named.conf.local
```

Add the following zone configuration:

```conf
zone "cloud.lab.local" {
    type master;
    file "/etc/bind/db.cloud.lab.local";
};
```

---

## Step 3: Create the Zone File

Create the following file:

```text
/etc/bind/db.cloud.lab.local
```

Configure the zone file with:

* A records
* NS records

---

## Step 4: Validate the Configuration

Verify both the BIND configuration and the DNS zone before restarting the service.

```bash
sudo named-checkconf && sudo named-checkzone cloud.lab.local /etc/bind/db.cloud.lab.local
```

---

## Step 5: Restart the DNS Service

```bash
sudo systemctl restart bind9 && sudo systemctl enable bind9
```

> [!NOTE]
> Always validate the configuration before restarting the BIND service.

---

# Exercise 2 — Configure the DHCP Server (isc-dhcp-server)

## Step 1: Install the DHCP Server

```bash
sudo apt install isc-dhcp-server -y
```

---

## Step 2: Configure DHCP

Edit the following configuration file:

```text
/etc/dhcp/dhcpd.conf
```

Add the following subnet configuration:

```conf
subnet 192.168.10.0 netmask 255.255.255.0 {

    range 192.168.10.100 192.168.10.200;

    option domain-name-servers 192.168.10.10;

    option routers 192.168.10.1;

    default-lease-time 600;

    max-lease-time 7200;

}
```

---

## Step 3: Restart the DHCP Service

```bash
sudo systemctl restart isc-dhcp-server
```

---

# Exercise 3 — Configure DNS Load Balancing (Round-Robin)

## Step 1: Add Multiple A Records

Edit the DNS zone file and add multiple A records for the same hostname.

```dns
web IN A 192.168.10.20
web IN A 192.168.10.21
web IN A 192.168.10.22
```

---

## Step 2: Verify Round-Robin Resolution

Run the following command multiple times to verify that DNS rotates the returned IP address.

```bash
for i in {1..6}; do dig web.cloud.lab.local +short; done
```

> [!TIP]
> Round-Robin DNS provides basic load distribution by rotating responses among multiple A records.

---

# Exercise 4 — IPAM: Track IP Allocations

## Step 1: Install phpIPAM Prerequisites

```bash
sudo apt install apache2 php php-mysql -y
```

---

## Step 2: Download phpIPAM

```bash
git clone https://github.com/phpipam/phpipam.git /var/www/html/phpipam
```

---

## Step 3: Import the Subnet

Import the following subnet into phpIPAM:

```text
192.168.10.0/24
```

---

## Step 4: Discover Active Hosts

Scan the subnet for live hosts.

```bash
php /var/www/html/phpipam/functions/scripts/discoveryCheck.php
```

---

# Exercise 5 — Active Directory DNS Integration

## Step 1: Verify AD-Integrated DNS Zones

On the Windows Server Domain Controller, run:

```powershell
Get-DnsServerZone | Select ZoneName, ZoneType, IsDsIntegrated
```

---

## Step 2: Configure a Conditional Forwarder

Forward DNS requests for the Linux zone to the BIND server.

```powershell
Add-DnsServerConditionalForwarderZone -Name 'cloud.lab.local' -MasterServers 192.168.10.10
```

> [!IMPORTANT]
> Conditional forwarders enable Windows DNS servers to resolve names hosted on external DNS servers such as Linux BIND.

---

# ✅ Best Practices

> [!TIP]
> Always run the following validation commands before restarting the BIND service to identify syntax errors:
>
> ```bash
> named-checkconf
> named-checkzone
> ```

---

> [!TIP]
> Configure DNS TTL values based on the environment:
>
> * **300 seconds** during migrations
> * **3600 seconds or higher** in stable production environments

---

> [!TIP]
> Document every DHCP reservation and static IP address in your IPAM solution. Maintaining accurate IP allocation records is essential for effective troubleshooting and infrastructure management.

---

# ✔️ Lab Summary

In this lab, you completed the following tasks:

* Installed and configured BIND DNS on Ubuntu.
* Created and validated DNS forward zones.
* Configured an ISC DHCP Server.
* Implemented DNS Round-Robin load balancing.
* Installed and configured phpIPAM for IP address management.
* Scanned network subnets for active hosts.
* Integrated Linux BIND with Active Directory DNS using a conditional forwarder.
* Reviewed enterprise DNS, DHCP, and IPAM best practices.
