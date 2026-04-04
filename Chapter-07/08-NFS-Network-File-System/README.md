# Chapter 7 - Lesson 8: NFS - Network File System

**Chapter 7 | Lesson 8 of 10**


## 🎯 এই Lesson-এ আমরা কী শিখবো?

- NFS কী এবং কেন DevOps-এ দরকার?
- NFS Server setup করা
- NFS Client-এ mount করা
- `/etc/fstab` দিয়ে auto-mount
- Permissions ও Security
- Real DevOps use cases


## NFS কী?

কল্পনা করেন আপনার অফিসে একটা **shared drive** আছে। সবাই সেই drive-এ ফাইল রাখতে পারে, পড়তে পারে। আপনি আপনার computer থেকেই সেই drive access করতে পারেন, মনে হয় যেন সেটা আপনার নিজের computer-এরই একটা folder।

**NFS (Network File System)** ঠিক এই কাজটাই করে। একটা Linux server-এর directory বা folder কে network-এর মাধ্যমে অন্য Linux machines থেকে access করা যায়, মনে হয় যেন সেটা local disk।

```
[ NFS Server ]                    [ NFS Client ]
/data/shared  ←── Network ──→  /mnt/nfs (same files!)
```


## NFS Architecture

```
┌─────────────────────────────────────────────────────┐
│                    NFS SERVER                       │
│  /etc/exports ──→ defines what to share             │
│  nfs-kernel-server ──→ the NFS daemon               │
│  /data/shared ──→ actual directory being shared     │
└─────────────────────────────────────────────────────┘
              ↕  TCP/UDP Port 2049
┌─────────────────────────────────────────────────────┐
│                    NFS CLIENT                       │
│  mount command ──→ connects to server               │
│  /mnt/nfs ──→ local mount point                     │
│  /etc/fstab ──→ auto-mount at boot                  │
└─────────────────────────────────────────────────────┘
```


## Lab Setup

এই lesson-এ আমরা assume করবো:

| Role | IP Address | Hostname |
|------|-----------|----------|
| NFS Server | `192.168.1.100` | `nfs-server` |
| NFS Client | `192.168.1.101` | `nfs-client` |

> আপনার কাছে দুইটা VM বা machine না থাকলেও সমস্যা নেই - concepts বুঝতে পারবেন। Practice-এ localhost দিয়েও test করা যায়।


## PART 1: NFS SERVER SETUP

### Step 1 - NFS Package Install করা

```bash
# Ubuntu/Debian
sudo apt update
sudo apt install nfs-kernel-server -y

# RHEL/CentOS/Rocky
sudo yum install nfs-utils -y
# অথবা
sudo dnf install nfs-utils -y
```


### Step 2 - Share করার জন্য Directory তৈরি করা

```bash
sudo mkdir -p /data/shared
sudo chown nobody:nogroup /data/shared    # Ubuntu
# অথবা
sudo chown nfsnobody:nfsnobody /data/shared  # RHEL

sudo chmod 755 /data/shared
```

> `nobody:nogroup` এটা একটা special user যার কোনো real privilege নেই। NFS-এ anonymous access-এর জন্য ব্যবহার হয়।


### Step 3 - `/etc/exports` File Configure করা

`/etc/exports` হলো NFS-এর main configuration file। এখানে define করা হয় কোন directory কাকে share করবে।

```bash
sudo nano /etc/exports
```

```bash
# Format:
# /path/to/share   client_ip(options)

/data/shared   192.168.1.101(rw,sync,no_subtree_check)
```

#### `/etc/exports` Options - বিস্তারিত:

| Option | মানে কী? |
|--------|---------|
| `rw` | Read + Write access দেবে |
| `ro` | Read Only access |
| `sync` | Data disk-এ লেখার পরেই confirm করবে (safe) |
| `async` | আগে confirm করে, পরে লেখে (fast কিন্তু risky) |
| `no_subtree_check` | Performance বাড়ায়, subtree checking বন্ধ করে |
| `no_root_squash` | Client-এর root user কে server-এও root হিসেবে দেখাবে (risky) |
| `root_squash` | Client-এর root কে `nobody` হিসেবে treat করে (default, safe) |
| `all_squash` | সব user কে `nobody` হিসেবে treat করে |

#### আরো Examples:

```bash
# শুধু একটা specific IP কে access দেওয়া
/data/shared   192.168.1.101(rw,sync,no_subtree_check)

# পুরো subnet কে access দেওয়া
/data/shared   192.168.1.0/24(rw,sync,no_subtree_check)

# সবাইকে access দেওয়া (production-এ avoid করবেন)
/data/shared   *(rw,sync,no_subtree_check)

# Multiple clients
/data/shared   192.168.1.101(rw,sync) 192.168.1.102(ro,sync)

# Read-only সবার জন্য
/data/backups  192.168.1.0/24(ro,sync,no_subtree_check)
```


### Step 4 - Export Apply করা

```bash
# /etc/exports reload করুন (নতুন share activate করুন)
sudo exportfs -a

# Verify - কোন কোন directory export হয়েছে দেখো
sudo exportfs -v
```

**Expected Output:**
```
/data/shared    192.168.1.101(rw,wdelay,root_squash,no_subtree_check,sec=sys,rw,root_squash,no_all_squash)
```

```bash
# exportfs flags:
# -a  = /etc/exports থেকে সব export করুন
# -v  = verbose, বিস্তারিত দেখাও
# -r  = re-export সব (exports reload)
# -u  = un-export করুন
```


### Step 5 - NFS Service Start করা

```bash
# Ubuntu/Debian
sudo systemctl start nfs-kernel-server
sudo systemctl enable nfs-kernel-server
sudo systemctl status nfs-kernel-server

# RHEL/CentOS
sudo systemctl start nfs-server
sudo systemctl enable nfs-server
sudo systemctl status nfs-server
```


### Step 6 - Firewall Allow করা

```bash
# UFW (Ubuntu)
sudo ufw allow from 192.168.1.0/24 to any port nfs
sudo ufw reload

# firewalld (RHEL)
sudo firewall-cmd --permanent --add-service=nfs
sudo firewall-cmd --permanent --add-service=rpcbind
sudo firewall-cmd --permanent --add-service=mountd
sudo firewall-cmd --reload
```


## PART 2: NFS CLIENT SETUP

এবার **Client machine**-এ যান।

### Step 1 - NFS Client Package Install করা

```bash
# Ubuntu/Debian
sudo apt install nfs-common -y

# RHEL/CentOS
sudo yum install nfs-utils -y
```


### Step 2 - Server-এর Exports দেখুন (Optional but Useful)

```bash
# Server কী কী share করছে সেটা দেখুন
showmount -e 192.168.1.100
```

**Expected Output:**
```
Export list for 192.168.1.100:
/data/shared 192.168.1.101
```


### Step 3 - Mount Point তৈরি করা

```bash
sudo mkdir -p /mnt/nfs/shared
```

---

### Step 4 - NFS Share Mount করা

```bash
sudo mount -t nfs 192.168.1.100:/data/shared /mnt/nfs/shared
```

#### Syntax Breakdown:

```
sudo mount  -t nfs    192.168.1.100:/data/shared   /mnt/nfs/shared
             ↑              ↑                            ↑
         file system    server_ip:/remote_path      local_mount_point
           type
```

### Step 5 - Mount হয়েছে কিনা Verify করা

```bash
# Mount হয়েছে কিনা দেখুন
df -h | grep nfs
```

**Expected Output:**
```
192.168.1.100:/data/shared  20G  1.2G   19G   6% /mnt/nfs/shared
```

```bash
# আরেকটা উপায়
mount | grep nfs
```

### Step 6 - Test করা

```bash
# Server-এ file তৈরি করুন
echo "Hello from NFS Server" | sudo tee /data/shared/test.txt

# Client-এ সেই file দেখুন
cat /mnt/nfs/shared/test.txt
```

**Expected Output (Client-এ):**
```
Hello from NFS Server
```

কাজ করছে! Client থেকে Server-এর file দেখা যাচ্ছে।


## PART 3: `/etc/fstab` দিয়ে Auto-Mount

প্রতিবার reboot-এর পর manually `mount` করতে হবে এটা avoid করতে `/etc/fstab`-এ entry দিতে হবে।

```bash
sudo nano /etc/fstab
```

এই line যোগ করুন:

```
# NFS Mount
192.168.1.100:/data/shared   /mnt/nfs/shared   nfs   defaults,_netdev   0 0
```

#### fstab Fields Breakdown:

```
192.168.1.100:/data/shared  /mnt/nfs/shared  nfs  defaults,_netdev  0  0
        ↑                        ↑             ↑          ↑           ↑  ↑
    remote share            local path     fs type    options      dump pass
```

| Option | মানে |
|--------|------|
| `defaults` | Standard mount options |
| `_netdev` | Network available হওয়ার পরেই mount করো |
| `0` (dump) | backup tool এটা skip করবে |
| `0` (pass) | fsck check skip করবে |

#### Test fstab Entry:

```bash
# প্রথমে Unmount করুন
sudo umount /mnt/nfs/shared

# fstab থেকে mount করুন (test)
sudo mount -a

# Verify
df -h | grep nfs
```


## PART 4: NFS Permissions & Security

### Root Squashing বোঝা

```
Client-এর root user ──→ Server-এ "nobody" হয়ে যায়
```

```bash
# /etc/exports-এ root_squash (default - safe)
/data/shared 192.168.1.101(rw,sync,root_squash)

# no_root_squash - ⚠️ DANGEROUS, avoid in production
/data/shared 192.168.1.101(rw,sync,no_root_squash)
```

### NFS Security Best Practices

```bash
# Specific IP দেন, wildcard (*) avoid করুন
/data/shared 192.168.1.101(rw,sync)   # Good

# এটা avoid করুন
/data/shared *(rw,sync)   # সবাই access পাবে

# Read-only যেখানে write দরকার নেই
/data/reports 192.168.1.0/24(ro,sync)

# root_squash সবসময় রাখুন (default-এ থাকে)
```


## PART 5: NFS Troubleshooting

### Common Commands:

```bash
# NFS service status দেখুন
sudo systemctl status nfs-kernel-server   # Ubuntu
sudo systemctl status nfs-server          # RHEL

# Active exports দেখুন
sudo exportfs -v

# Client থেকে server exports দেখুন
showmount -e 192.168.1.100

# Mount হওয়া NFS shares দেখুন
mount -t nfs

# NFS statistics
nfsstat

# Connection test
telnet 192.168.1.100 2049
```


### Common Errors & Solutions:

| Error | কারণ | সমাধান |
|-------|------|--------|
| `mount.nfs: Connection timed out` | Firewall block করছে | Server-এ firewall rule দেন |
| `mount.nfs: access denied` | `/etc/exports`-এ IP নেই | Client IP যোগ করুন, `exportfs -r` রান করুন |
| `mount.nfs: No route to host` | Network issue | `ping server_ip` দিয়ে check করুন |
| `Stale file handle` | Server restart হয়েছে | Client-এ remount করুন |
| `Permission denied` | Ownership/permission সমস্যা | `chown nobody:nogroup /data/shared` |


## Real DevOps Use Cases

### Use Case 1 - Shared Log Storage

```bash
# Log server-এ
/var/log/apps   10.0.0.0/8(rw,sync,no_subtree_check)

# সব app server-এ mount করুন
# এখন সব app server-এর logs এক জায়গায়
```


### Use Case 2 - Kubernetes Persistent Volumes

```yaml
# Kubernetes PersistentVolume with NFS
apiVersion: v1
kind: PersistentVolume
metadata:
  name: nfs-pv
spec:
  capacity:
    storage: 10Gi
  nfs:
    server: 192.168.1.100
    path: /data/shared
```


### Use Case 3 - CI/CD Artifact Sharing

```bash
# Build server NFS share
/artifacts   192.168.1.0/24(rw,sync,no_subtree_check)

# সব deploy server mount করে artifacts নেয়
```

## NFS Quick Reference

```bash
# ===== SERVER SIDE =====
sudo apt install nfs-kernel-server        # Install
sudo nano /etc/exports                    # Configure shares
sudo exportfs -a                          # Apply exports
sudo exportfs -v                          # Verify exports
sudo exportfs -r                          # Reload exports
sudo systemctl restart nfs-kernel-server  # Restart service

# ===== CLIENT SIDE =====
sudo apt install nfs-common              # Install
showmount -e SERVER_IP                   # See server shares
sudo mount -t nfs SERVER:/path /mnt/pt   # Manual mount
sudo umount /mnt/pt                      # Unmount
mount | grep nfs                         # Verify mounts
df -h | grep nfs                         # Check mounted shares
```


## 📝 Quick Summary

- NFS = Network File System - Linux-এ network-এর মাধ্যমে directory share করার উপায়
- Server-এ `nfs-kernel-server` install করে `/etc/exports` configure করতে হয়
- Client-এ `nfs-common` install করে `mount` command দিয়ে connect করতে হয়
- `/etc/fstab` দিয়ে boot-এ auto-mount হয়
- `root_squash` সবসময় enable রাখা - এটা security-র জন্য জরুরি
- Port 2049 - NFS-এর main port
- `exportfs -a` exports apply করার command
- `showmount -e` server কী share করছে দেখার command


## 🏋️ Practice Tasks

1. একটা NFS server setup করুন (localhost বা VM-এ), `/tmp/nfs-test` directory share করুন এবং নিজেই client হিসেবে mount করুন।

2. `/etc/exports`-এ দুইটা different option দিয়ে দেখুন। একটা `rw` আরেকটা `ro` তারপর client থেকে write করার চেষ্টা করুন, দেখুন কী হয়।

3. `/etc/fstab`-এ entry দিয়ে auto-mount setup করুন, তারপর manually unmount করে `mount -a` দিয়ে verify করুন।

---

## ⏭️ What's Next?

**Chapter 7 - Lesson 9: Samba/CIFS - Sharing with Windows**
> Linux server থেকে Windows machine-এ file share করার উপায়। অফিস environment-এ এটা খুব common! *Happy Learning* 🚀

<table width="100%">
  <tr>
    <td align="left">
      <a href="../07-RAID-in-Linux">← RAID in Linux</a>
    </td>
    <td align="right">
      <a href="../09-Samba-CIFS-Sharing-with-Windows">Samba CIFS Sharing with Windows →</a>
    </td>
  </tr>
</table>


