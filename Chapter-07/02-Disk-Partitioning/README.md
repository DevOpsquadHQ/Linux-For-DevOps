# Chapter 7 - Lesson 2: Partitioning Disks

**Chapter 7 | Lesson 2 of 10**


## 🎯 এই Lesson-এ আমরা যা শিখবো

- Disk partition কী এবং কেন দরকার
- MBR vs GPT partition table
- `fdisk` - MBR disk partition tool
- `gdisk` - GPT disk partition tool
- `parted` - উভয় support করে, interactive ও non-interactive
- Real DevOps scenario তে কীভাবে ব্যবহার করবেন


## Partition কী?

মনে করেন আপনার কাছে একটা বড় খালি জমি আছে। আপনি সেই জমিকে ভাগ করতে চান - একটা অংশে বাড়ি, একটায় বাগান, একটায় গ্যারেজ। Disk partitioning ঠিক এটাই করে।

একটা physical disk (যেমন `/dev/sda`) কে আপনি multiple logical sections-এ ভাগ করতে পারেন। প্রতিটা partition আলাদাভাবে use হয়:

```
/dev/sda      ← পুরো disk (জমি)
├── /dev/sda1 ← partition 1 (বাড়ি - /boot)
├── /dev/sda2 ← partition 2 (বাগান - /)
└── /dev/sda3 ← partition 3 (গ্যারেজ - /home)
```

## Partition Table কী? MBR vs GPT

Partition table হলো একটা "map" যা disk-এ লেখা থাকে - কতগুলো partition আছে, কোথায় শুরু, কোথায় শেষ।

দুই ধরনের partition table আছে:

| Feature | **MBR** (Master Boot Record) | **GPT** (GUID Partition Table) |
|---|---|---|
| বয়স | পুরনো (1983) | নতুন (2000s) |
| Max Disk Size | 2 TB | 9.4 ZB (বিশাল!) |
| Max Partitions | 4 primary | 128 partitions |
| BIOS/UEFI | BIOS | UEFI (modern) |
| Tool | `fdisk` | `gdisk` বা `parted` |
| DevOps Use | পুরনো VM/server | নতুন server, cloud |

> **DevOps Tip:** আজকাল cloud servers (AWS EC2, GCP VM) সব GPT use করে। তবে `fdisk` জানাটাও জরুরি কারণ অনেক legacy system এখনো MBR ব্যবহার করে।


## Practice এর জন্য Dummy Disk তৈরি করা

Real disk ছাড়াও আমরা একটা `virtual disk file` তৈরি করে practice করতে পারি। এটা DevOps-এ lab setup-এ খুবই common।

```bash
# একটা 1GB virtual disk file তৈরি করুন
dd if=/dev/zero of=/tmp/practice-disk.img bs=1M count=1024
```

**Output:**
```
1024+0 records in
1024+0 records out
1073741824 bytes (1.1 GB, 1.0 GiB) copied, 2.5 s, 429 MB/s
```

```bash
# এটাকে একটা loop device হিসেবে attach করুন
sudo losetup -fP /tmp/practice-disk.img

# কোন loop device হলো দেখুন
sudo losetup -l
```

**Output:**
```
NAME       SIZELIMIT OFFSET AUTOCLEAR RO BACK-FILE                  DIO LOG-SEC
/dev/loop0         0      0         0  0 /tmp/practice-disk.img       0     512
```

এখন `/dev/loop0` হলো আমাদের practice disk! Real `/dev/sda` এর মতোই কাজ করবে।


## Tool 1: fdisk (MBR Partitioning)

### fdisk কী?

`fdisk` হলো সবচেয়ে classic disk partitioning tool। এটা interactive - আপনি commands টাইপ করেন, আর সে কাজ করে।

### fdisk দিয়ে একটা disk দেখা

```bash
# Disk-এর বর্তমান partition information দেখুন
sudo fdisk -l /dev/loop0
```

**Output (নতুন disk, কোনো partition নেই):**
```
Disk /dev/loop0: 1 GiB, 1073741824 bytes, 2097152 sectors
Units: sectors of 1 * 512 = 512 bytes
Sector size (logical/physical): 512 bytes / 512 bytes
I/O size (minimum/optimal): 512 bytes / 512 bytes
```

### fdisk Interactive Mode-এ ঢোকা

```bash
sudo fdisk /dev/loop0
```

আপনি এই prompt দেখবেন:
```
Welcome to fdisk (util-linux 2.37.2).
Changes will remain in memory only, until you decide to write them.
Be careful before using the write command.

Command (m for help):
```

### fdisk-এর গুরুত্বপূর্ণ Commands (ভেতরে):

| Command | কাজ |
|---|---|
| `m` | Help menu দেখাও |
| `p` | Current partition table দেখাও |
| `n` | নতুন partition তৈরি করো |
| `d` | Partition delete করো |
| `t` | Partition type পরিবর্তন করো |
| `l` | সব partition types দেখাও |
| `w` | পরিবর্তন save করো (disk-এ লেখা) |
| `q` | Save ছাড়া বের হও |

### Step-by-Step: নতুন Partition তৈরি

**Step 1:** `fdisk` রান করুন এবং `p` দিয়ে current state দেখুন:
```
Command (m for help): p

Disk /dev/loop0: 1 GiB, 1073741824 bytes, 2097152 sectors
Units: sectors of 1 * 512 = 512 bytes
Sector size (logical/physical): 512 bytes / 512 bytes
I/O size (minimum/optimal): 512 bytes / 512 bytes
Disklabel type: dos
Disk identifier: 0x00000000
```

**Step 2:** `n` দিয়ে নতুন partition তৈরি করুন:
```
Command (m for help): n
Partition type
   p   primary (0 primary, 0 extended, 4 free)
   e   extended (container for logical partitions)
Select (default p): p
```
> `p` = primary partition (Enter চাপুন)

**Step 3:** Partition number দাও:
```
Partition number (1-4, default 1): 1
```
> শুধু Enter চাপুন - default 1 সিলেক্ট করুন

**Step 4:** First sector (শুরু):
```
First sector (2048-2097151, default 2048):
```
> শুধু Enter চাপুন - default সিলেক্ট করুন

**Step 5:** Last sector (শেষ - আকার নির্ধারণ):
```
Last sector, +/-sectors or +/-size{K,M,G,T,P} (2048-2097151, default 2097151): +500M
```
> `+500M` দিন - মানে 500MB এর একটা partition

**Output:**
```
Created a new partition 1 of type 'Linux' and of size 500 MiB.
```

**Step 6:** আবার `p` দিয়ে দেখুন partition তৈরি হয়েছে কিনা:
```
Command (m for help): p
Device       Boot Start     End Sectors  Size Id Type
/dev/loop0p1       2048 1026047 1024000  500M 83 Linux
```

**Step 7:** আরেকটা partition তৈরি করুন (বাকি space):
```
Command (m for help): n
Select (default p): p
Partition number (2-4, default 2): 2
First sector (1026048-2097151, default 1026048):   ← Enter
Last sector (1026048-2097151, default 2097151):     ← Enter (বাকি সব)

Created a new partition 2 of type 'Linux' and of size 523 MiB.
```

**Step 8:** `w` দিয়ে save করুন:
```
Command (m for help): w
The partition table has been altered.
Calling ioctl() to re-read partition table.
```

**Verify করুন:**
```bash
sudo fdisk -l /dev/loop0
```

**Output:**
```
Device       Boot   Start     End Sectors  Size Id Type
/dev/loop0p1         2048 1026047 1024000  500M 83 Linux
/dev/loop0p2      1026048 2097151 1071104  523M 83 Linux
```

দুটো partition তৈরি হয়ে গেছে!


## Tool 2: gdisk (GPT Partitioning)

### gdisk কী?

`gdisk` হলো `fdisk`-এর GPT version। Interface প্রায় একই, কিন্তু GPT partition table use করে।

```bash
# আগের disk পরিষ্কার করুন নতুন practice এর জন্য
sudo losetup -d /dev/loop0
dd if=/dev/zero of=/tmp/practice-disk2.img bs=1M count=1024
sudo losetup -fP /tmp/practice-disk2.img
sudo losetup -l  # নতুন loop device নাম দেখুন (হয়তো /dev/loop1)
```

```bash
sudo gdisk /dev/loop1
```

**Output:**
```
GPT fdisk (gdisk) version 1.0.8

Partition table scan:
  MBR: not present
  BSD: not present
  APM: not present
  GPT: not present

Creating new GPT entries in memory.

Command (? for help):
```

### gdisk Commands:

| Command | কাজ |
|---|---|
| `?` | Help দেখাও |
| `p` | Partition table দেখাও |
| `n` | নতুন partition তৈরি করো |
| `d` | Delete করো |
| `t` | Partition type পরিবর্তন করো (GUID) |
| `l` | Partition type list দেখাও |
| `w` | Save করো |
| `q` | বের হও |

### gdisk দিয়ে Partition তৈরি:

```
Command (? for help): n
Partition number (1-128, default 1): 1
First sector (34-2097118, default = 2048):           ← Enter
Last sector (2048-2097118, default = 2097118): +300M
Current type is 8300 (Linux filesystem)
Hex code or GUID (L to show codes, Enter = 8300):    ← Enter

Added partition 1 of type 'Linux filesystem' and of size 300 MiB
```

> **MBR vs GPT** পার্থক্য এখানে: GPT-তে partition type GUID দিয়ে চেনা যায়। Common types:
> - `8300` = Linux filesystem
> - `8200` = Linux swap
> - `EF00` = EFI System
> - `8E00` = Linux LVM ← DevOps-এ অনেক ব্যবহার হয়!

```bash
# Save করুন
Command (? for help): w
# "Y" দিয়ে confirm করুন
```

## Tool 3: parted (সবচেয়ে Powerful)

### parted কী?

`parted` হলো সবচেয়ে flexible tool:
- MBR এবং GPT উভয়ই support করে
- **Interactive mode** এবং **non-interactive (scripting) mode** উভয়ই আছে
- DevOps automation-এ script-এ সরাসরি ব্যবহার করা যায়

### parted দিয়ে Disk দেখা:

```bash
sudo parted /dev/loop0 print
```

**Output:**
```
Model: Loopback device (loopback)
Disk /dev/loop0: 1074MB
Sector size (logical/physical): 512B/512B
Partition Table: msdos
Disk Flags:

Number  Start   End     Size    Type     File system  Flags
 1      1049kB  525MB   524MB   primary
 2      525MB   1074MB  548MB   primary
```

### parted Interactive Mode:

```bash
sudo parted /dev/loop1
```

```
GNU Parted 3.4
Using /dev/loop1
Welcome to GNU Parted! Type 'help' to view a list of commands.
(parted)
```

### parted এর গুরুত্বপূর্ণ Commands:

| Command | কাজ |
|---|---|
| `help` | সব commands দেখাও |
| `print` | Partition table দেখাও |
| `mklabel gpt` | GPT table তৈরি করো |
| `mklabel msdos` | MBR table তৈরি করো |
| `mkpart` | নতুন partition তৈরি করো |
| `rm 1` | Partition 1 delete করো |
| `resizepart` | Partition resize করো |
| `quit` | বের হও |

### parted দিয়ে GPT Disk Setup:

```
(parted) mklabel gpt
(parted) mkpart primary ext4 1MiB 400MiB
(parted) mkpart primary ext4 400MiB 100%
(parted) print
```

**Output:**
```
Number  Start   End     Size    File system  Name     Flags
 1      1049kB  419MB   418MB   ext4         primary
 2      419MB   1074MB  655MB   ext4         primary
```

```
(parted) quit
```

### parted - Non-Interactive Mode (DevOps Scripting!):

এটাই সবচেয়ে গুরুত্বপূর্ণ - automation script-এ এভাবে ব্যবহার হয়:

```bash
# একটা নতুন disk-এ সব কাজ এক লাইনে!
sudo parted -s /dev/loop1 mklabel gpt
sudo parted -s /dev/loop1 mkpart primary ext4 1MiB 50%
sudo parted -s /dev/loop1 mkpart primary ext4 50% 100%
sudo parted -s /dev/loop1 print
```

> `-s` flag মানে "script mode" - কোনো interactive prompt নেই। **Terraform, Ansible, shell script** - সব জায়গায় এভাবে ব্যবহার হয়।


## Real-World DevOps Scenarios

### Scenario 1: নতুন Server-এ Extra Disk যোগ হয়েছে

```bash
# নতুন disk খুঁজে বের করুন
lsblk
sudo fdisk -l

# Output-এ দেখবেন /dev/sdb নতুন যোগ হয়েছে, কোনো partition নেই
# GPT partition তৈরি করুন
sudo parted -s /dev/sdb mklabel gpt
sudo parted -s /dev/sdb mkpart primary 1MiB 100%
```

### Scenario 2: LVM এর জন্য Partition তৈরি (পরের lesson-এ কাজে লাগবে)

```bash
# gdisk দিয়ে LVM type partition
sudo gdisk /dev/sdb
# n → Enter → Enter → +100G → 8E00 → w
# 8E00 = Linux LVM
```

### Scenario 3: Boot Disk Setup (EFI + Root + Swap)

```bash
sudo parted -s /dev/sda mklabel gpt
sudo parted -s /dev/sda mkpart ESP fat32 1MiB 512MiB          # EFI partition
sudo parted -s /dev/sda set 1 esp on                          # EFI flag
sudo parted -s /dev/sda mkpart primary ext4 512MiB 50GiB      # Root
sudo parted -s /dev/sda mkpart primary linux-swap 50GiB 100%  # Swap
```


## তিনটি Tool-এর তুলনা

| | **fdisk** | **gdisk** | **parted** |
|---|---|---|---|
| Partition Table | MBR only | GPT only | MBR + GPT |
| Mode | Interactive | Interactive | Interactive + Script |
| Scripting | কঠিন | কঠিন | সহজ (-s flag) |
| Disk Size | 2TB max | Unlimited | Unlimited |
| DevOps Use | Legacy | Modern GPT | Automation |
| Availability | সব distro | আলাদা install | সব distro |

## Cleanup (Practice শেষে)

```bash
# Loop devices detach করুন
sudo losetup -d /dev/loop0
sudo losetup -d /dev/loop1

# Practice files মুছে ফেলুন
rm /tmp/practice-disk.img /tmp/practice-disk2.img
```


## 📝 Quick Summary

- **Partition** = disk-কে logical ভাগে ভাগ করা
- **MBR** = পুরনো, max 4 partition, max 2TB → `fdisk` ব্যবহার করুন
- **GPT** = নতুন, 128 partition, unlimited size → `gdisk` বা `parted` ব্যবহার করুন
- **fdisk** = interactive, MBR, সব জায়গায় আছে
- **gdisk** = interactive, GPT, modern server
- **parted** = সবচেয়ে flexible, scripting-এ best, DevOps automation-এ ideal
- Partition তৈরির পর `w` দিয়ে save করতে হবে (fdisk/gdisk)
- `lsblk` এবং `fdisk -l` দিয়ে সবসময় verify করুন


## Practice Tasks

**Task 1:**
একটা 500MB virtual disk file তৈরি করুন এবং `fdisk` দিয়ে ২টা partition করুন:
- Partition 1: 200MB
- Partition 2: বাকি সব

**Task 2:**
আরেকটা 800MB virtual disk তৈরি করুন এবং `parted` non-interactive mode (`-s` flag) ব্যবহার করে GPT table তৈরি করে ৩টা equal partition তৈরি করুন।

**Task 3:**
`lsblk` এবং `sudo fdisk -l` run করুন এবং output-এ পার্থক্য খুঁজে বের করুন - কোনটা বেশি তথ্য দেয়?

---

## ⏭️ What's Next?

**Lesson 3: File System Types (ext4, xfs, btrfs - differences & use cases)**

পরের lesson-এ আমরা শিখবো - partition তৈরির পর সেটাকে usable করতে হলে কী করতে হয়! কোন filesystem কোথায় ব্যবহার করবেন এবং DevOps-এ কোনটা বেশি popular - সব কিছু। *Happy Learning* 🚀

<table width="100%">
  <tr>
    <td align="left">
      <a href="../01-Disk-Basics">← Disk Basics</a>
    </td>
    <td align="right">
      <a href="../03-File-System-Types">File System Types →</a>
    </td>
  </tr>
</table>