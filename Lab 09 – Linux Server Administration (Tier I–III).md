# Lab 09 – Linux Server Administration (Tier I–III)

> [!IMPORTANT]
> This lab demonstrates Linux server administration across **Tier I**, **Tier II**, and **Tier III** operational support levels. You will configure storage, logical volume management (LVM), RAID, file-sharing services, virtualization, web services, proxy services, and firewall security.

---

# 📋 Lab Information

| Property | Value |
|----------|-------|
| **Lab ID** | LAB 09 |
| **Title** | Linux Server Administration (Tier I–III) |
| **Estimated Duration** | 3–4 Hours |
| **Difficulty** | Intermediate |

---

# 🎯 Objective

Administer Linux servers across all operational support tiers:

- **Tier I** – Basic server administration, storage, and filesystem management.
- **Tier II** – Storage management, Apache, Samba, NFS, KVM virtualization, and RAID.
- **Tier III** – Advanced storage, firewall management, load balancing concepts, and kernel-level administration.

At the end of this lab, you will:

- Configure Linux storage and persistent mounts.
- Create Logical Volume Manager (LVM) storage.
- Configure Software RAID using `mdadm`.
- Deploy Samba and NFS file-sharing services.
- Manage KVM virtual machines with `virsh`.
- Configure Apache Virtual Hosts.
- Configure a Squid Proxy server.
- Implement firewall rules using IPTables.

---

# 📚 Prerequisites

Before starting this lab, ensure that you have:

- A Linux (Ubuntu) server.
- Root or sudo privileges.
- Multiple virtual disks attached for LVM and RAID exercises.
- Internet connectivity for package installation.
- Administrative access to modify firewall rules.

---

# 🛠️ Lab Tasks

---

# Step 1 — Tier I: Storage & File Mount Management

Manage Linux storage devices and configure persistent mounts.

---

## List Available Block Devices

```bash
lsblk && fdisk -l
```

---

## Create a Partition and Format the Filesystem

```bash
sudo fdisk /dev/sdb
```

> Create a new partition.

```bash
sudo mkfs.ext4 /dev/sdb1
```

---

## Mount the Filesystem

```bash
sudo mkdir /mnt/data

sudo mount /dev/sdb1 /mnt/data
```

---

## Configure Persistent Mount

```bash
echo '/dev/sdb1 /mnt/data ext4 defaults 0 2' | sudo tee -a /etc/fstab
```

Reload all filesystem mounts.

```bash
sudo mount -a && df -h
```

> [!NOTE]
> Entries in `/etc/fstab` ensure filesystems are mounted automatically after system reboot.

---

# Step 2 — Tier II: LVM & RAID Storage

Configure Logical Volume Manager (LVM) and Software RAID.

---

## LVM Setup

Create Physical Volumes.

```bash
sudo pvcreate /dev/sdc /dev/sdd
```

Create a Volume Group.

```bash
sudo vgcreate data-vg /dev/sdc /dev/sdd
```

Create a Logical Volume.

```bash
sudo lvcreate -L 20G -n data-lv data-vg
```

Format the logical volume.

```bash
sudo mkfs.xfs /dev/data-vg/data-lv
```

Mount the logical volume.

```bash
sudo mount /dev/data-vg/data-lv /mnt/lvm-data
```

---

## Configure Software RAID-1 with mdadm

Install `mdadm`.

```bash
sudo apt install mdadm -y
```

Create a RAID-1 array.

```bash
sudo mdadm --create /dev/md0 \
--level=1 \
--raid-devices=2 \
/dev/sde \
/dev/sdf
```

Format the RAID device.

```bash
sudo mkfs.ext4 /dev/md0
```

Mount the RAID array.

```bash
sudo mount /dev/md0 /mnt/raid
```

Verify RAID status.

```bash
cat /proc/mdstat
```

> [!TIP]
> RAID-1 mirrors data across disks, providing redundancy in the event of a drive failure.

---

# Step 3 — SMB/Samba & NFS Server Configuration

Configure Linux file-sharing services.

---

## Samba (Windows File Sharing)

Install Samba.

```bash
sudo apt install samba -y
```

Create the shared directory.

```bash
sudo mkdir /srv/samba/shared

sudo chmod 777 /srv/samba/shared
```

Edit the Samba configuration file.

```text
/etc/samba/smb.conf
```

Add the following configuration.

```ini
[shared]
path = /srv/samba/shared
browsable = yes
writable = yes
guest ok = yes
```

Restart the Samba service.

```bash
sudo systemctl restart smbd
```

---

## NFS Server

Install the NFS server.

```bash
sudo apt install nfs-kernel-server -y
```

Configure exports.

```bash
echo '/srv/nfs 192.168.10.0/24(rw,sync,no_subtree_check)' >> /etc/exports
```

Apply the export configuration.

```bash
sudo exportfs -a

sudo systemctl restart nfs-kernel-server
```

> [!NOTE]
> Samba provides Windows-compatible file sharing, while NFS is optimized for Linux and UNIX environments.

---

# Step 4 — KVM Management with virsh

Manage Kernel-based Virtual Machines (KVM).

---

## Install KVM

```bash
sudo apt install qemu-kvm libvirt-daemon-system -y
```

Grant the current user access.

```bash
sudo usermod -aG libvirt $USER
```

---

## List Virtual Machines

```bash
sudo virsh list --all
```

---

## Create a Virtual Machine

```bash
sudo virt-install \
--name testvm \
--ram 2048 \
--vcpus 2 \
--disk path=/var/lib/libvirt/images/testvm.qcow2,size=20 \
--cdrom /tmp/ubuntu-22.04.iso \
--os-variant ubuntu22.04
```

---

## Start the Virtual Machine

```bash
sudo virsh start testvm
```

---

## Connect to the Console

```bash
sudo virsh console testvm
```

---

# Step 5 — Apache Virtual Hosts & Squid Proxy

Configure Apache Virtual Hosts and a Squid proxy server.

---

## Apache Virtual Host

Install Apache.

```bash
sudo apt install apache2 -y
```

Create a new virtual host configuration.

```bash
sudo nano /etc/apache2/sites-available/app1.conf
```

Example configuration.

```apache
ServerName app1.cloud.lab.local
DocumentRoot /var/www/app1
```

Enable the virtual host.

```bash
sudo a2ensite app1

sudo systemctl reload apache2
```

---

## Squid Proxy

Install Squid.

```bash
sudo apt install squid -y
```

Edit the Squid configuration.

```text
/etc/squid/squid.conf
```

Add the following ACL.

```text
acl localnet src 192.168.10.0/24
```

Restart Squid.

```bash
sudo systemctl restart squid
```

> [!NOTE]
> Squid can be used for web caching, proxy services, and internet access control.

---

# Step 6 — Tier III: IPTables Firewall Rules

Configure Linux firewall rules using IPTables.

---

## Flush Existing Rules

```bash
sudo iptables -F
```

---

## Allow Established Connections

```bash
sudo iptables -A INPUT \
-m state \
--state ESTABLISHED,RELATED \
-j ACCEPT
```

---

## Allow SSH from a Specific Subnet

```bash
sudo iptables -A INPUT \
-s 192.168.10.0/24 \
-p tcp \
--dport 22 \
-j ACCEPT
```

---

## Allow HTTP

```bash
sudo iptables -A INPUT \
-p tcp \
--dport 80 \
-j ACCEPT
```

---

## Allow HTTPS

```bash
sudo iptables -A INPUT \
-p tcp \
--dport 443 \
-j ACCEPT
```

---

## Drop All Other Incoming Traffic

```bash
sudo iptables -A INPUT -j DROP
```

---

## Save Firewall Rules

```bash
sudo iptables-save > /etc/iptables/rules.v4
```

> [!WARNING]
> Incorrect IPTables rules can block SSH access to the server. Always verify firewall changes before closing your terminal session.

---

# 🖥️ Linux Administration Overview

```text
Tier I
│
├── Disk Management
├── Partitions
└── Persistent Mounts

Tier II
│
├── LVM
├── RAID
├── Samba
├── NFS
├── Apache
├── Squid
└── KVM

Tier III
│
├── IPTables
├── Storage Management
├── Advanced Administration
├── Security
└── Virtualization
```

---

# ✅ Verification Checklist

After completing this lab, verify that:

- [ ] Storage partition is created successfully.
- [ ] Filesystem is mounted correctly.
- [ ] Persistent mount is configured in `/etc/fstab`.
- [ ] LVM Physical Volumes, Volume Group, and Logical Volume are created.
- [ ] RAID-1 array is healthy.
- [ ] RAID status is visible in `/proc/mdstat`.
- [ ] Samba service is operational.
- [ ] NFS exports are configured correctly.
- [ ] KVM virtualization is installed.
- [ ] Virtual machine is created and running.
- [ ] Apache Virtual Host is enabled.
- [ ] Squid proxy is configured.
- [ ] IPTables firewall rules are active.
- [ ] Firewall configuration has been saved.

---

# 💡 Best Practice Tips

> [!TIP]
> Always test IPTables rules inside a **screen** or **tmux** session. A misconfigured firewall rule can lock you out of SSH.

---

> [!TIP]
> Use **LVM** for production storage. Logical Volume Manager allows storage to be extended online with minimal disruption.

---

> [!TIP]
> Monitor RAID array health using `cat /proc/mdstat` and configure email alerts through `mdadm.conf` to detect drive failures promptly.

---

# 📖 Summary

In this lab, you completed the following tasks:

- Configured Linux storage devices and persistent mounts.
- Created Logical Volume Manager (LVM) storage.
- Configured Software RAID-1 using `mdadm`.
- Deployed Samba and NFS file-sharing services.
- Managed KVM virtual machines using `virsh`.
- Configured Apache Virtual Hosts.
- Installed and configured a Squid proxy server.
- Implemented IPTables firewall rules for secure network access.
- Applied Linux administration best practices across Tier I, Tier II, and Tier III operations.

---

# 📚 References

- Linux Storage Management
- Logical Volume Manager (LVM)
- mdadm
- Samba
- Network File System (NFS)
- KVM
- libvirt
- virsh
- Apache HTTP Server
- Squid Proxy
- IPTables
