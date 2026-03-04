# Chapter 1 - Lesson 2: Linux File System Hierarchy

✅ **Chapter 1 | Lesson 2 of 10**

---

## 🗂️ Linux File System কী?

Windows-এ আমরা দেখেছি `C:\`, `D:\` এইরকম drive। কিন্তু Linux-এ সবকিছু শুরু হয় একটা জায়গা থেকে, যেটাকে বলে **root** ( `/` )।

> কল্পনা করুন একটা বিশাল গাছ। গাছের গোড়া হলো `/` (root)। সেখান থেকে ডালপালা বের হয়েছে যেমন `/etc`, `/home`, `/var` ইত্যাদি। প্রতিটা ডাল আলাদা আলাদা কাজের জন্য।

Linux-এ **সবকিছুই একটা file** এমনকি hardware device-ও!

---

## 🌳 File System Hierarchy - The Big Picture

```
/
├── etc/
├── home/
├── var/
├── proc/
├── sys/
├── tmp/
├── bin/
├── sbin/
├── usr/
├── lib/
├── dev/
├── boot/
├── root/
├── opt/
└── mnt/
```

এখন একটা একটা করে দেখি প্রতিটা directory কী করে।


## 📁 প্রতিটি Directory-র বিস্তারিত


### 1. `/` - Root Directory

- এটা **সবকিছুর শুরু**। পুরো Linux file system এখান থেকে শুরু।
- Windows-এর `C:\` এর মতো, কিন্তু এটা **একমাত্র starting point**।
- শুধুমাত্র `root` user এখানে সরাসরি কাজ করতে পারে।

```bash
ls /
```
**Expected Output:**
```
bin  boot  dev  etc  home  lib  media  mnt  opt  proc  root  run  sbin  srv  sys  tmp  usr  var
```

### 2. `/etc` - Configuration Files

> এটা আপনার অফিসের **সেটিংস ফাইল রাখার ক্যাবিনেট**। সব software-এর configuration এখানে থাকে।

- Network config, user info, service config ইত্যাদি সব এখানে।
- DevOps-এ আপনি এখানে সবচেয়ে বেশি কাজ করবেন।

| File/Folder | কাজ |
|---|---|
| `/etc/passwd` | সব user-এর তালিকা |
| `/etc/hostname` | Server-এর নাম |
| `/etc/hosts` | Local DNS mapping |
| `/etc/ssh/sshd_config` | SSH server configuration |
| `/etc/nginx/` | Nginx web server config |
| `/etc/crontab` | Scheduled tasks |

```bash
cat /etc/hostname
```
**Expected Output:**
```
ubuntu-server
```

### 3. `/home` - User Home Directories

> এটা আপনার **ব্যক্তিগত ঘর**। প্রতিটা user-এর আলাদা ঘর আছে।

- প্রতিটা user-এর জন্য একটা করে folder থাকে।
- যেমন: user `munir` হলে তার home হবে `/home/munir`
- আপনার নিজের files, documents সব এখানে থাকবে।

```bash
ls /home
```
**Expected Output:**
```
munir   deploy   ubuntu
```

- `~` এই চিহ্নকে `tilda` বলে। এটা আপনার home directory-কে represent করে।
- `cd ~` মানে সরাসরি আপনার home-এ চলে যাবে।

### 4. `/var` - Variable Data

> এটা আপনার **"চলমান ডায়েরি"**। যেসব data সময়ের সাথে সাথে বাড়তে থাকে সেগুলো এখানে থাকে।

- **Logs, databases, cache, mail** সব এখানে জমা হয়।
- DevOps-এ এটিই সবচেয়ে গুরুত্বপূর্ণ জায়গা কারণ **server logs এখানে থাকে।**

| Path | কাজ |
|---|---|
| `/var/log/` | সব system & app logs |
| `/var/log/syslog` | General system log |
| `/var/log/auth.log` | Login/authentication log |
| `/var/www/` | Web server files |
| `/var/lib/` | Application data (databases) |

```bash
ls /var/log/
```
**Expected Output:**
```
auth.log  syslog  kern.log  nginx/  apt/  dpkg.log
```

### 5. `/proc` - Process & Kernel Information

> এটা Linux-এর **"লাইভ ড্যাশবোর্ড"**। এখানে কোনো real file নেই। সবকিছু RAM থেকে live দেখায়।

- এটা একটা **virtual file system** disk-এ কিছু নেই, সব memory-তে।
- চলমান সব process-এর info এখানে থাকে।

| Path | কাজ |
|---|---|
| `/proc/cpuinfo` | CPU-র বিস্তারিত তথ্য |
| `/proc/meminfo` | RAM-এর তথ্য |
| `/proc/uptime` | Server কতক্ষণ চলছে |
| `/proc/1234/` | PID 1234 process-এর তথ্য |

```bash
cat /proc/uptime
```
**Expected Output:**
```
3600.45 7189.23
```
*(মানে: server ৩৬০০ সেকেন্ড = ১ ঘণ্টা চলছে)*

```bash
cat /proc/cpuinfo | grep "model name"
```
**Expected Output:**
```
model name : Intel(R) Core(TM) i7-9750H CPU @ 2.60GHz
```

### 6. `/sys` - System & Hardware Information

> `/proc` এর ছোট ভাই। এটা **hardware device-এর তথ্য** দেখায়।

- Kernel এবং hardware-এর মধ্যে bridge।
- `/proc` এর মতোই virtual disk-এ কিছু নেই।
- DevOps-এ network interface, block device info এখান থেকে পাওয়া যায়।

```bash
ls /sys/class/net/
```
**Expected Output:**
```
eth0   lo   docker0
```
*(আপনার server-এর সব network interface এখানে দেখাবে)*

---

### 7. `/tmp` - Temporary Files

> এটা আপনার **scratch paper** যেকোনো কিছু লিখতে পারেন, reboot হলেই মুছে সব যাবে।

- যেকোনো user এখানে file রাখতে পারে।
- Reboot করলে সব মুছে যায়।
- Scripts বা programs temporary file এখানে রাখে।

```bash
ls /tmp/
touch /tmp/mytest.txt
ls /tmp/
```

### 8. `/bin` এবং `/sbin` - Essential Commands

| Directory | কাজ |
|---|---|
| `/bin` | সব user-এর জন্য basic commands (ls, cp, mv, cat) |
| `/sbin` | শুধু root/admin-এর জন্য system commands (fdisk, reboot, ifconfig) |

```bash
ls /bin | head -10
```
**Expected Output:**
```
bash  cat  chmod  chown  cp  date  df  echo  false  grep
```

### 9. `/usr` - User Programs & Utilities

> এটা আপনার **"software installation folder"**। Windows-এর `Program Files`-এর মতো।

- `/usr/bin` - installed programs (python3, git, vim)
- `/usr/lib` - program libraries
- `/usr/local` - আপনি যেগুলো নিজে manually install করেছেন সেইসব software

```bash
ls /usr/bin | grep python
```
**Expected Output:**
```
python3  python3.10
```

### 10. `/dev` - Device Files

> এটা আপনার **"hardware-এর দরজা"**। প্রতিটা hardware device এখানে একটা file হিসেবে আছে।

| Device | মানে |
|---|---|
| `/dev/sda` | প্রথম hard disk |
| `/dev/sda1` | সেই disk-এর প্রথম partition |
| `/dev/null` | "ব্লাক হোল" এখানে পাঠানো data মুছে যায় |
| `/dev/tty` | Terminal |

```bash
ls /dev/sd*
```
**Expected Output:**
```
/dev/sda  /dev/sda1  /dev/sda2
```

এখন বোঝার চেস্টা করি `/dev/sda` কোথা থেকে আসে?

Linux-এ প্রতিটা storage device-কে একটা **নাম দেওয়া হয়** তার **type** অনুযায়ী।

> **Analogy:** ঠিক যেমন বাসে সিটের নম্বর থাকে — A1, A2, B1 ইত্যাদি। Linux-ও disk-এর নাম দেয় তার **ধরন** আর **ক্রম** অনুযায়ী।

## 📦 Storage Device-এর তিন ধরনের নাম

| Device Name | কোন ধরনের Disk | Example |
|---|---|---|
| `/dev/sda` | SATA/SCSI/USB HDD বা SSD | পুরোনো laptop, desktop HDD |
| `/dev/hda` | পুরোনো IDE disk (আজকাল নেই) | খুব পুরোনো PC |
| `/dev/nvme0n1` | NVMe SSD (আধুনিক, অনেক fast) | যেমন আমার machine! |


## ⚡ NVMe কী? কেন আলাদা নাম?

**NVMe = Non-Volatile Memory Express**

> **Analogy:** পুরোনো SATA disk হলো **সাইকেল**, আর NVMe হলো **বিমান**। দুটোই গন্তব্যে যায়, কিন্তু speed আকাশ-পাতাল তফাৎ।

- NVMe disk সরাসরি **PCIe lane**-এ connect থাকে (motherboard-এর সাথে directly)
- SATA disk connect থাকে **SATA cable** দিয়ে, তুলনামুলক অনেক slow।
- এই কারণে Linux এদের **আলাদা নামে** চেনে

| বিষয় | SATA SSD/HDD | NVMe SSD |
|---|---|---|
| Speed | ~500 MB/s | ~3500 MB/s+ |
| Connection | SATA port | PCIe/M.2 slot |
| Linux নাম | `/dev/sda` | `/dev/nvme0n1` |

## 🔤 NVMe Naming - কিভাবে নাম তৈরি হয়?

```
/dev/nvme0n1
       │  │
       │  └── n1 = Namespace 1 (প্রায় সবসময় n1 থাকে)
       └───── 0  = প্রথম NVMe controller (0 থেকে শুরু)

/dev/nvme0n1p1
              │
              └── p1 = প্রথম Partition
```

### NVMe VS sda:

| NVMe | মানে | SATA equivalent |
|---|---|---|
| `/dev/nvme0n1` | প্রথম NVMe disk | `/dev/sda` |
| `/dev/nvme0n1p1` | প্রথম NVMe disk-এর partition 1 | `/dev/sda1` |
| `/dev/nvme0n1p2` | প্রথম NVMe disk-এর partition 2 | `/dev/sda2` |
| `/dev/nvme1n1` | দ্বিতীয় NVMe disk | `/dev/sdb` |

আপনার machine-এ এই commands run করে দেখেন:

**১. সব disk দেখার জন্য:**
```bash
lsblk
```
**Expected Output (যদি machine-এ SSD ইউজ করা থাকে):**
```
NAME        MAJ:MIN RM   SIZE RO TYPE MOUNTPOINT
nvme0n1     259:0    0   512G  0 disk
├─nvme0n1p1 259:1    0   512M  0 part /boot/efi
├─nvme0n1p2 259:2    0     1G  0 part /boot
└─nvme0n1p3 259:3    0 510.5G  0 part /
```

**২. Disk-এর type ও details:**
```bash
lsblk -d -o NAME,ROTA,TYPE,SIZE,MODEL
```
```
NAME    ROTA TYPE  SIZE  MODEL
nvme0n1    0 disk  512G  Samsung SSD 970 EVO
```
> `ROTA = 0` মানে **Rotational নয়** = SSD/NVMe ✅
> `ROTA = 1` মানে **Rotational** = HDD

**৩. `/dev` তে দেখো:**
```bash
ls /dev/nvme*
```
**Expected Output:**
```
/dev/nvme0  /dev/nvme0n1  /dev/nvme0n1p1  /dev/nvme0n1p2
```

## 💡 DevOps-এ কেন এটা জানা দরকার?

কারণ যখন আপনি:
- **Disk partition** করবে → সঠিক device name দিতে হবে
- **Disk mount** করবে → `/dev/nvme0n1p1` এইভাবে reference করতে হবে
- **Disk performance** monitor করবে → device name জানা লাগবে
- **Server provision** করবে → cloud server-এ `/dev/xvda` বা `/dev/nvme0n1` দুটোই দেখতে পাবে


## 🌐 Cloud Server-এ কী দেখবে?

| Cloud Provider | Device Name |
|---|---|
| AWS EC2 (নতুন) | `/dev/nvme0n1` |
| AWS EC2 (পুরোনো) | `/dev/xvda` |
| Google Cloud | `/dev/sda` |
| DigitalOcean | `/dev/vda` বা `/dev/sda` |

তাই **device name hardcode না করে** `lsblk` দিয়ে আগে check করা best practice! ✅


## ✅ Key Takeaway

- যদি আপনার machine-এ **NVMe SSD** হয়, তাহলে নাম `/dev/nvme0n1` এটা সম্পূর্ণ স্বাভাবিক
- `/dev/sda` = SATA disk, `/dev/nvme0n1` = NVMe disk শুধু **hardware type-এর পার্থক্য**
- Concept একই, শুধু **নামের format আলাদা**
- সবসময় `lsblk` দিয়ে আগে disk check করে দেখা উচিৎ।


### 11. `/boot` - Boot Files

> এখানে Linux চালু হওয়ার জন্য দরকারি files থাকে।

- **GRUB bootloader**, Linux **kernel** এখানে আছে।
- এই folder-এ ভুল করে কিছু delete করলে system boot নাও হতে পারে তাই সাবধান!

```bash
ls /boot/
```
**Expected Output:**
```
grub  initrd.img  vmlinuz  config-5.15.0
```

### 12. `/root` - Root User-এর Home

- এটা `root` (admin) user-এর personal home directory।
- `/home/root` **নয়**, বরং `/root` এটা আলাদা।
- সাধারণ user এখানে access পায় না।

### 13. `/opt` - Optional/Third-party Software

> এটা আপনার **"extra software রাখার আলাদা shelf"**।

- যেসব software Linux package manager দিয়ে আসে না, সেগুলো এখানে install হয়।
- যেমন: custom DevOps tools, third-party apps।

### 14. `/mnt` এবং `/media` - Mount Points

| Directory | কাজ |
|---|---|
| `/mnt` | Manually mount করা disk/network share |
| `/media` | Auto-mount হওয়া USB, CD-ROM |

```bash
ls /mnt/
```

## 🗺️ পুরো Structure একসাথে - Quick Reference Table

| Directory | সহজ কথায় | DevOps-এ কেন দরকার |
|---|---|---|
| `/` | সবকিছুর গোড়া | Starting point |
| `/etc` | Configuration files | Service config করতে |
| `/home` | User-এর ঘর | Personal files |
| `/var` | Variable/changing data | **Log দেখতে সবচেয়ে বেশি** |
| `/proc` | Live system info | CPU/RAM/Process monitor করতে |
| `/sys` | Hardware info | Network/device info |
| `/tmp` | Temporary files | Temp script রাখতে |
| `/bin` | Basic commands | ls, cp, cat সব এখানে |
| `/sbin` | Admin commands | System tools |
| `/usr` | Installed software | Programs রাখার জায়গা |
| `/dev` | Device files | Disk, hardware access |
| `/boot` | Boot files | Kernel, GRUB |
| `/root` | Root user home | Admin-এর ঘর |
| `/opt` | Extra software | Custom tools |
| `/mnt` | Mount points | External storage |

## 📝 Quick Summary

- ✅ Linux-এ সব কিছু শুরু হয় `/` (root) থেকে, কোনো C: D: নেই
- ✅ `/etc` সব configuration এখানে, DevOps-এ সবচেয়ে বেশি ব্যবহার
- ✅ `/var/log` সব logs এখানে, troubleshooting-এর প্রথম জায়গা
- ✅ `/proc` এবং `/sys` virtual file system, live system information
- ✅ `/tmp` reboot হলে সব মুছে যায়
- ✅ `/home` প্রতিটা user-এর নিজের জায়গা
- ✅ `/dev` hardware device-গুলো file হিসেবে থাকে।

## 🛠️ Practice Tasks

এখন আপনি নিজে নিজে এই কাজগুলো করেন:

**Task 1:** নিচের command গুলো run করেন এবং output দেখেন:
```bash
ls /
ls /etc
ls /var/log
```

**Task 2:** তোমার server-এর CPU তথ্য দেখেন:
```bash
cat /proc/cpuinfo | grep "model name"
cat /proc/meminfo | grep MemTotal
```

**Task 3:** `/tmp`-তে একটা file তৈরি করুন, তারপর দেখুন সেটা আছে কিনা:
```bash
touch /tmp/hello_linux.txt
ls /tmp/
cat /tmp/hello_linux.txt
```

## ⏭️ What's Next

**Chapter 1 - Lesson 3: Navigating the File System**
পরের lesson-এ আমরা শিখবো কিভাবে এই file system-এ navigate করতে হয় `pwd`, `ls`, `cd`, `tree` commands দিয়ে। মানে, এই বিশাল গাছের মধ্যে কিভাবে চলাফেরা করতে হয়!
