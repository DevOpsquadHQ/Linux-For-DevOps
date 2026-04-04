# Chapter 7 - Lesson 9: Samba/CIFS - Sharing with Windows

**Chapter 7 | Lesson 9 of 10**

## 🎯 এই Lesson-এ আমরা কী শিখব?

- Samba কী এবং কেন দরকার
- Linux থেকে Windows-এ folder share করা
- Windows থেকে Linux share access করা
- CIFS কী এবং কীভাবে Linux থেকে Windows share mount করে
- DevOps-এ Samba-র real-world use cases


## Samba কী?

মনে করেন আপনার office-এ কিছু Windows PC আছে, কিছু Linux server আছে। Windows users চায় Linux server-এর একটা folder access করতে যেন সেটা তাদের নিজের drive-এর মতো দেখাবে।

Linux-এ naturally Windows-এর SMB (Server Message Block) protocol বোঝার ক্ষমতা নেই। এখানেই আসে **Samba**।

> Samba হলো একজন translator-এর মতো। Linux কথা বলে একটা ভাষায়, Windows কথা বলে আরেক ভাষায়। Samba মাঝখানে দাঁড়িয়ে দুজনকে বুঝিয়ে দেয়।

## SMB vs CIFS vs Samba - পার্থক্য কী?

| Term | মানে কী |
|------|---------|
| **SMB** | Server Message Block - Microsoft-এর file sharing protocol |
| **CIFS** | Common Internet File System - SMB-এরই পুরনো version |
| **Samba** | Linux-এ SMB/CIFS implement করা open-source software |
| **NFS** | Linux-to-Linux sharing (আগের lesson) |

> সহজ কথা: Samba = Linux যখন Windows-এর ভাষায় কথা বলে


## PART 1: Linux থেকে Windows-এ Share করা (Samba Server)

### ধাপ 1: Samba Install করা

```bash
# Ubuntu/Debian
sudo apt update
sudo apt install samba -y

# RHEL/CentOS/Rocky Linux
sudo dnf install samba samba-common samba-client -y
```

Install হয়েছে কিনা check করুন:

```bash
samba --version
```

**Expected Output:**
```
Version 4.15.13-Ubuntu
```

### ধাপ 2: Share করার জন্য Directory তৈরি করা

```bash
# একটা folder তৈরি করুন যেটা share করবে
sudo mkdir -p /srv/samba/shared

# সবাই যেন ঢুকতে পারে (permissions দিন)
sudo chmod 0775 /srv/samba/shared

# Owner set করুন
sudo chown root:sambashare /srv/samba/shared
```

> **Note:** `sambashare` group automatically তৈরি হয় Samba install হলে।


### ধাপ 3: Samba Config File edit করা

Samba-র main config file হলো: **`/etc/samba/smb.conf`**

```bash
# আগে backup নিন
sudo cp /etc/samba/smb.conf /etc/samba/smb.conf.bak

# এখন edit করুন
sudo nano /etc/samba/smb.conf
```

File-এর একদম নিচে যান এবং এই block add করে দিন:

```ini
[shared]
    comment = Linux Shared Folder
    path = /srv/samba/shared
    browseable = yes
    read only = no
    writable = yes
    guest ok = no
    valid users = @sambashare
    create mask = 0664
    directory mask = 0775
```


### smb.conf-এর প্রতিটা line মানে কী?

| Line | মানে |
|------|------|
| `[shared]` | Share-এর নাম - Windows থেকে এই নামেই দেখাবে |
| `comment` | Description - optional |
| `path` | কোন directory share হবে |
| `browseable = yes` | Network browse করলে দেখা যাবে |
| `read only = no` | শুধু পড়া না, লেখাও যাবে |
| `writable = yes` | লেখার permission আছে |
| `guest ok = no` | Anonymous user ঢুকতে পারবে না |
| `valid users = @sambashare` | শুধু sambashare group-এর users ঢুকতে পারবে |
| `create mask` | নতুন file-এর default permission |
| `directory mask` | নতুন folder-এর default permission |


### ধাপ 4: Samba User তৈরি করা

Samba-র নিজস্ব password system আছে। Linux user থাকলেই হবে না, আলাদাভাবে Samba-তে add করতে হবে।

```bash
# প্রথমে Linux user তৈরি করুন (যদি না থাকে)
sudo useradd -M -s /sbin/nologin sambauser

# sambashare group-এ add করুন
sudo usermod -aG sambashare sambauser

# Samba password set করুন
sudo smbpasswd -a sambauser
```

**Output:**
```
New SMB password:
Retype new SMB password:
Added user sambauser.
```

> **Note:** `-M` মানে home directory তৈরি করবে না, `-s /sbin/nologin` মানে এই user দিয়ে shell login করা যাবে না। এটা শুধু Samba-র জন্য।


### ধাপ 5: Config Validate করা

```bash
testparm
```

**Expected Output:**
```
Load smb config files from /etc/samba/smb.conf
Loaded services file OK.
Server role: ROLE_STANDALONE

Press enter to see a dump of your service definitions
```

Enter চাপলে পুরো config দেখাবে।


### ধাপ 6: Samba Service Start করা

```bash
# Service start করুন
sudo systemctl start smbd nmbd

# Boot-এ auto start করুন
sudo systemctl enable smbd nmbd

# Status check করুন
sudo systemctl status smbd
```

**Output:**
```
● smbd.service - Samba SMB Daemon
     Loaded: loaded (/lib/systemd/system/smbd.service; enabled)
     Active: active (running) since ...
```

> **smbd** = actual file sharing daemon
> **nmbd** = NetBIOS name resolution (Windows network browsing-এর জন্য)


### ধাপ 7: Firewall Allow করা

```bash
# Ubuntu (ufw)
sudo ufw allow samba

# RHEL/CentOS (firewalld)
sudo firewall-cmd --permanent --add-service=samba
sudo firewall-cmd --reload
```


### ধাপ 8: Windows থেকে Connect করা

Windows PC-তে **Run** (Win+R প্রেস করুন) এবং type করুন:

```
\\<Linux-Server-IP>\shared
```

Example:
```
\\192.168.1.100\shared
```

Username: `sambauser`
Password: (যেটা smbpasswd দিয়ে set করেছিলে)


## PART 2: Linux থেকে Windows Share Access করা (CIFS Client)

এখন উল্টো দিক - Windows একটা folder share করেছে, Linux সেটা mount করবে।

### ধাপ 1: CIFS utilities install করা

```bash
# Ubuntu/Debian
sudo apt install cifs-utils -y

# RHEL/CentOS
sudo dnf install cifs-utils -y
```


### ধাপ 2: Mount point তৈরি করা

```bash
sudo mkdir -p /mnt/windows_share
```


### ধাপ 3: Windows Share mount করা

```bash
sudo mount -t cifs //192.168.1.200/SharedFolder /mnt/windows_share \
  -o username=WindowsUser,password=WinPassword,vers=3.0
```

**Syntax ব্যাখ্যা:**

| Part | মানে |
|------|------|
| `-t cifs` | filesystem type হলো CIFS |
| `//192.168.1.200/SharedFolder` | Windows PC-র IP ও share name |
| `/mnt/windows_share` | Linux-এ mount হবে এখানে |
| `-o` | options শুরু |
| `username=` | Windows username |
| `password=` | Windows password |
| `vers=3.0` | SMB version (3.0 recommended) |


### Password Securely দেওয়া (credentials file)

Password সরাসরি command-এ দেওয়া unsafe। তাই credentials file ব্যবহার করুন:

```bash
# File তৈরি করুন
sudo nano /etc/samba/credentials

# এই দুই line লিখুন:
username=WindowsUser
password=WinPassword

# Permissions restrict করুন
sudo chmod 600 /etc/samba/credentials
```

এখন mount করুন:

```bash
sudo mount -t cifs //192.168.1.200/SharedFolder /mnt/windows_share \
  -o credentials=/etc/samba/credentials,vers=3.0
```


### ধাপ 4: /etc/fstab-এ add করা (permanent mount)

```bash
sudo nano /etc/fstab
```

এই line add করুন:

```
//192.168.1.200/SharedFolder  /mnt/windows_share  cifs  credentials=/etc/samba/credentials,vers=3.0,_netdev  0  0
```

> `_netdev` মানে network ready হওয়ার পরে mount হবে। এটা network share-এর জন্য important।


## Samba - Useful Commands

```bash
# সব Samba shares list করুন
smbclient -L //192.168.1.100 -U sambauser

# Samba share-এ connect করুন (FTP-style)
smbclient //192.168.1.100/shared -U sambauser

# Connected Samba users দেখা
sudo smbstatus

# Samba user list দেখা
sudo pdbedit -L

# Samba user delete করুন
sudo smbpasswd -x sambauser

# Samba user disable করুন
sudo smbpasswd -d sambauser

# Samba user enable করুন
sudo smbpasswd -e sambauser
```


### `smbclient` দিয়ে interactive shell:

```bash
smbclient //192.168.1.100/shared -U sambauser
```

**Output:**
```
Enter WORKGROUP\sambauser's password:
Try "help" to get a list of possible commands.
smb: \>
```

এখন FTP-এর মতো commands চলবে:

```bash
smb: \> ls          # files দেখো
smb: \> get file.txt   # download করো
smb: \> put file.txt   # upload করো
smb: \> mkdir docs     # folder তৈরি করো
smb: \> exit           # বেরিয়ে যাও
```


## SELinux + Samba (RHEL/CentOS এ important!)

RHEL/CentOS-এ SELinux চালু থাকলে Samba block হতে পারে। Fix:

```bash
# Samba-কে home directory share করতে allow করুন
sudo setsebool -P samba_enable_home_dirs on

# Custom path share করতে
sudo semanage fcontext -a -t samba_share_t "/srv/samba/shared(/.*)?"
sudo restorecon -R /srv/samba/shared
```


## DevOps-এ Samba-র Real-World Use Cases

| Scenario | কী করা হয় |
|----------|------------|
| **Dev Environment** | Linux server-এ code থাকে, Windows developer VS Code দিয়ে সরাসরি edit করে |
| **File Server** | একটা Linux box পুরো office-এর Windows PC-কে shared drive দেয় |
| **CI/CD Artifacts** | Build artifacts Windows team-এর কাছে share করতে |
| **Legacy Systems** | পুরনো Windows application যেগুলো CIFS ছাড়া কাজ করে না |
| **Backup** | Windows machine-এর backup Linux Samba server-এ রাখা |


## NFS vs Samba - কোনটা কখন?

| বিষয় | NFS | Samba |
|-------|-----|-------|
| **Best for** | Linux ↔ Linux | Linux ↔ Windows |
| **Protocol** | NFS | SMB/CIFS |
| **Performance** | দ্রুততর | তুলনামূলক ধীর |
| **Security** | IP-based | Username/Password |
| **Windows Support** | না (natively) | হ্যাঁ |
| **Linux Support** | হ্যাঁ | হ্যাঁ |


## 📝 Quick Summary

- Samba হলো Linux-এ SMB/CIFS protocol implement করা software
- smbd = file sharing, nmbd = NetBIOS name resolution
- Config file: `/etc/samba/smb.conf`
- Samba-র আলাদা password system আছে → `smbpasswd` দিয়ে manage করতে হয়
- `testparm` দিয়ে config validate করতে পারেন
- Linux থেকে Windows share access করতে cifs-utils লাগে
- `mount -t cifs` দিয়ে Windows share mount করা যায়
- Credentials file ব্যবহার করুন - command-এ password দিও না
- RHEL/CentOS-এ SELinux context ঠিক রাখতে হবে


## 🏋️ Practice Tasks

**Task 1:**
`/srv/samba/devops` নামে একটা directory তৈরি করুন এবং `smb.conf`-এ `[devops]` নামে একটা share configure করুন।

**Task 2:**
`devopsuser` নামে একটা Samba user তৈরি করুন, তাকে `sambashare` group-এ add করুন এবং `smbclient` দিয়ে local machine থেকেই connect করে test করুন:
```bash
smbclient //localhost/devops -U devopsuser
```

**Task 3:**
`/etc/samba/credentials` file তৈরি করুন এবং সেটা ব্যবহার করে একটা CIFS share mount করুন। তারপর `/etc/fstab`-এ add করুন যেন reboot-এ automatically mount হয়।

---

## ⏭️ What's Next?

**Chapter 7 - Lesson 10: Disk Encryption (LUKS, cryptsetup)**

পরের lesson-এ শিখবো কীভাবে Linux disk/partition সম্পূর্ণ **encrypt** করা যায় - যাতে physical access পেলেও কেউ data চুরি করতে না পারে। DevOps-এ security-র জন্য অত্যন্ত গুরুত্বপূর্ণ।

> 🎉 আপনি দারুণ করছেন! Chapter 7-এর ৯টা lesson শেষ - মাত্র একটা বাকি! Storage chapter-এর শেষ lesson-টা খুবই exciting হবে। *Happy Learning* 🚀

<table width="100%">
  <tr>
    <td align="left">
      <a href="../08-NFS-Network-File-System">← NFS Network File System</a>
    </td>
    <td align="right">
      <a href="../10-Disk-Encryption">Disk Encryption →</a>
    </td>
  </tr>
</table>