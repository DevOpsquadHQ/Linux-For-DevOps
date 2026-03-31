# Chapter 7 - Lesson 1: Disk Basics (Block Devices, /dev/sda, Partitions, MBR vs GPT)

**Chapter 7 | Lesson 1 of 10**

## 🎯 এই Lesson-এ আমরা যা শিখবো

- Disk বা Storage আসলে Linux-এ কীভাবে দেখা যায়
- Block Device কী?
- `/dev/sda`, `/dev/sdb` এগুলো মানে কী?
- Partition কী এবং কেন দরকার?
- MBR vs GPT - পার্থক্য এবং কোনটা কখন ব্যবহার করতে হয়


কল্পনা করেন আপনার কাছে একটা বড় জমি আছে। সেই জমিটা হলো আপনার **Hard Disk**।

এখন আপনি সেই জমিকে ভাগ করলেন যেমন এক অংশে বাড়ি, এক অংশে বাগান, এক অংশে গ্যারেজ।
এই ভাগগুলোই হলো **Partition**।

আর Linux সেই জমির ম্যাপ রাখে দুটো পদ্ধতিতে **MBR** অথবা **GPT**।


## Block Device কী?

Linux-এ সব কিছু একটা file। Disk-ও তার ব্যতিক্রম না।

**Block Device** মানে এমন একটা device যেটা data-কে fixed-size "block"-এ read/write করে।
যেমন: Hard Disk, SSD, USB Drive, Virtual Disk।

Character Device (যেমন keyboard, terminal) data একটা একটা character করে পাঠায়।
কিন্তু Block Device একসাথে অনেক data (block) পাঠাতে পারে তাই এটা storage-এর জন্য perfect।


## `/dev` Directory - Disk-এর ঠিকানা

Linux-এ সব hardware device থাকে `/dev` directory-তে।

```bash
ls /dev/sd*
```

**Expected Output:**
```
/dev/sda    /dev/sda1    /dev/sda2
/dev/sdb    /dev/sdb1
```

এখন এটা decode করি

## `/dev/sda` মানে কী?

| অংশ | মানে |
|-----|------|
| `/dev` | Device files থাকার জায়গা |
| `s` | SCSI/SATA/SAS disk (আজকের প্রায় সব disk) |
| `d` | Disk |
| `a` | প্রথম disk (a=1st, b=2nd, c=3rd...) |
| `1`, `2`... | Partition নম্বর |

তাহলে:
- `/dev/sda` → প্রথম disk (পুরো disk)
- `/dev/sda1` → প্রথম disk-এর প্রথম partition
- `/dev/sda2` → প্রথম disk-এর দ্বিতীয় partition
- `/dev/sdb` → দ্বিতীয় disk

> **NVMe SSD হলে** নাম হবে `/dev/nvme0n1`, `/dev/nvme0n1p1` একটু আলাদা format।


## Disk দেখার Commands

### ১. `lsblk` - সব Block Device দেখো

```bash
lsblk
```

**Expected Output:**
```
NAME   MAJ:MIN RM  SIZE RO TYPE MOUNTPOINT
sda      8:0    0   50G  0 disk
├─sda1   8:1    0   48G  0 part /
└─sda2   8:2    0    2G  0 part [SWAP]
sdb      8:16   0   20G  0 disk
```

এখানে:
- `NAME` → device-এর নাম
- `SIZE` → কত বড়
- `TYPE` → disk নাকি partition
- `MOUNTPOINT` → কোথায় mount করা আছে

### ২. `fdisk -l` - Disk-এর বিস্তারিত তথ্য

```bash
sudo fdisk -l
```

**Expected Output:**
```
Disk /dev/sda: 50 GiB, 53687091200 bytes, 104857600 sectors
Disk model: Virtual disk
Units: sectors of 1 * 512 = 512 bytes
Sector size (logical/physical): 512 bytes / 512 bytes
I/O size (minimum/optimal): 512 bytes / 512 bytes
Disklabel type: gpt

Device       Start       End   Sectors  Size Type
/dev/sda1     2048    999423   997376  487M EFI System
/dev/sda2   999424 104857566 103858143 49.5G Linux filesystem
```


### ৩. `df -h` - Mounted Disk কতটুকু ব্যবহার হচ্ছে

```bash
df -h
```

**Expected Output:**
```
Filesystem      Size  Used Avail Use% Mounted on
/dev/sda1        48G   15G   31G  33% /
tmpfs           1.9G     0  1.9G   0% /dev/shm
```

### ৪. `blkid` - Disk/Partition-এর UUID দেখা

```bash
sudo blkid
```

**Expected Output:**
```
/dev/sda1: UUID="a1b2c3d4-..." TYPE="ext4" PARTUUID="..."
/dev/sda2: UUID="e5f6g7h8-..." TYPE="swap"
```

UUID হলো প্রতিটা partition-এর unique ID - `/etc/fstab`-এ এটা ব্যবহার হয়।


## Partition কী এবং কেন দরকার?

একটা disk-কে আলাদা আলাদা logical অংশে ভাগ করাই হলো **Partitioning**।

### কেন Partition করি?

| কারণ | ব্যাখ্যা |
|------|----------|
| OS আলাদা রাখা | `/` (root), `/home`, `/var` আলাদা partition-এ |
| Data সুরক্ষা | OS crash করলেও data partition ঠিক থাকে |
| Performance | `/var/log` আলাদা রাখলে log flood root disk নষ্ট করবে না |
| Dual Boot | Windows ও Linux পাশাপাশি রাখা |
| Swap Space | RAM শেষ হলে disk ব্যবহারের জায়গা |

### DevOps-এ Common Partition Layout:

```
/dev/sda1  →  /boot      (500MB)  - Bootloader
/dev/sda2  →  /          (20GB)   - Root filesystem
/dev/sda3  →  /var       (10GB)   - Logs, databases
/dev/sda4  →  /home      (15GB)   - User files
/dev/sda5  →  swap       (2GB)    - Virtual memory
```

## MBR vs GPT - দুটো Partition Table

Disk-এর শুরুতে একটা "ম্যাপ" থাকে - কতটা partition আছে, কোথায় শুরু-শেষ।
এই ম্যাপ রাখার পদ্ধতি দুটো: **MBR** এবং **GPT**।

### MBR (Master Boot Record)

- ১৯৮৩ সালে তৈরি - অনেক পুরনো পদ্ধতি
- Disk-এর একদম প্রথম sector-এ (512 bytes) সব তথ্য থাকে
- **সীমাবদ্ধতা:**
  - সর্বোচ্চ **4টা Primary Partition**
  - সর্বোচ্চ **2TB** disk support করে
  - Backup নেই - সেই 512 bytes নষ্ট হলে সব শেষ!

```
MBR Disk Layout:
┌──────────────────────────────────────────────┐
│  512 bytes MBR  │  Part1  │  Part2  │  Part3 │
│ (bootcode+table)│         │         │        │
└──────────────────────────────────────────────┘
```

### GPT (GUID Partition Table)

- আধুনিক পদ্ধতি - UEFI-এর সাথে আসে
- **সুবিধা:**
  - সর্বোচ্চ **128টা Partition** (Linux-এ)
  - **9.4 ZB** পর্যন্ত disk support করে (practically unlimited!)
  - Disk-এর শুরু ও শেষে দুটো backup রাখে
  - প্রতিটা partition-এর নিজস্ব **GUID** (unique ID) থাকে

```
GPT Disk Layout:
┌─────┬────────────────────────────────────────┬─────┐
│ MBR │  GPT Header + 128 Partition Entries    │Backup│
│Prot │  Part1 │ Part2 │ Part3 │ ... │ Part128 │ GPT │
└─────┴────────────────────────────────────────┴─────┘
```


### MBR vs GPT তুলনা

| বিষয় | MBR | GPT |
|-------|-----|-----|
| বয়স | ১৯৮৩ (পুরনো) | ২০০০ (আধুনিক) |
| Max Disk Size | **2 TB** | **9.4 ZB** |
| Max Partitions | **4** Primary | **128** |
| Backup | নেই | আছে (disk-এর শেষে) |
| Boot System | BIOS | UEFI (MBR-ও সাপোর্ট করে) |
| Recovery | কঠিন | সহজ |
| DevOps Use | Legacy system | নতুন সব server |

> আজকের দিনে সবসময় GPT ব্যবহার করা উচিৎ। শুধু পুরনো machine বা 2TB-এর কম disk এ BIOS থাকলে MBR লাগতে পারে।


## আপনার Disk MBR না GPT - কীভাবে বুঝবেন?

```bash
sudo fdisk -l /dev/sda | grep "Disklabel type"
```

**Output:**
```
Disklabel type: gpt
```
অথবা
```
Disklabel type: dos    ← এটাই MBR
```

অথবা `parted` দিয়ে:

```bash
sudo parted /dev/sda print | grep "Partition Table"
```

**Output:**
```
Partition Table: gpt
```

## Sector কী? (Bonus Concept)

Disk-কে ছোট ছোট "Sector"-এ ভাগ করা থাকে।
প্রতিটা Sector সাধারণত **512 bytes** বা **4096 bytes (4K)**।

Partition মানে হলো নির্দিষ্ট sector থেকে নির্দিষ্ট sector পর্যন্ত একটা অংশ বরাদ্দ করা।

```bash
sudo fdisk -l /dev/sda
```

Output-এ দেখবেন:
```
Units: sectors of 1 * 512 = 512 bytes
Device     Start      End  Sectors  Size
/dev/sda1   2048   999423   997376  487M
```

মানে: `/dev/sda1` শুরু হয় sector 2048 থেকে, শেষ হয় sector 999423-এ।


## 📝 Lesson Summary

- Block Device হলো storage hardware যা block-এ data read/write করে
- Linux-এ সব disk থাকে `/dev/` directory-তে (`/dev/sda`, `/dev/nvme0n1`)
- `a`, `b`, `c` মানে প্রথম, দ্বিতীয়, তৃতীয় disk; `1`, `2` মানে partition নম্বর
- Partition করা মানে disk-কে logical অংশে ভাগ করা - OS, data, swap আলাদা রাখতে
- MBR পুরনো - max 4 partition, max 2TB
- GPT আধুনিক - max 128 partition, practically unlimited size, backup সহ
- DevOps-এ সবসময় GPT prefer করুন
- Key commands: `lsblk`, `fdisk -l`, `df -h`, `blkid`, `parted`

## Practice Tasks

**Task 1:**
নিচের command গুলো run করুন এবং output দেখুন:
```bash
lsblk
sudo fdisk -l
sudo blkid
```

**Task 2:**
আপনার system-এ কতটা disk আছে এবং প্রতিটা disk MBR না GPT - সেটা বের করুন:
```bash
sudo fdisk -l | grep -E "Disk /dev|Disklabel type"
```

**Task 3:**
আপনার root partition (`/`) কোন device-এ আছে এবং কতটুকু ব্যবহার হচ্ছে বের করুন:
```bash
df -h /
```

---

## ⏭️ What's Next?

**Chapter 7 - Lesson 2: Partitioning Disks (`fdisk`, `gdisk`, `parted`)**
পরের lesson-এ আমরা শিখবো কীভাবে একটা disk partition করতে হয় - নতুন partition তৈরি করা, delete করা, এবং type set করা। এটা DevOps-এ server setup করার সময় প্রায়ই লাগে! *Happy Learning* 🚀

<table width="100%">
  <tr>
    <td align="left">
      <a href="../../Chapter-06/09-Assessment">← Chapter 06 - Assessment</a>
    </td>
    <td align="right">
      <a href="../02-Disk-Partitioning">Disk Partitioning →</a>
    </td>
  </tr>
</table>