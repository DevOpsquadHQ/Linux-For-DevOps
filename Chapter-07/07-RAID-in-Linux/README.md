# Chapter 7 — Lesson 7: RAID in Linux (mdadm, RAID 0/1/5/10)

✅ **Chapter 7 | Lesson 7 of 10**

---

## 🎯 এই Lesson-এ তুমি কী শিখবে?

- RAID কী এবং কেন ব্যবহার করা হয়
- RAID এর বিভিন্ন levels (0, 1, 5, 10)
- `mdadm` দিয়ে Software RAID তৈরি করা
- RAID monitor করা এবং failure handle করা
- DevOps-এ RAID এর real-world use cases

---

## 🧠 RAID কী? — সহজ ভাষায়

**RAID** মানে হলো **Redundant Array of Independent Disks।**

সহজ ভাষায় বলতে গেলে — একাধিক physical disk কে একসাথে combine করে **একটি logical disk** হিসেবে ব্যবহার করার technique।

### 🏗️ Real-World Analogy:

> মনে করো তুমি একটি বড় সেতু তৈরি করছো। একটি মাত্র pillar দিলে সেটা ভেঙে পড়তে পারে। কিন্তু ৪টা pillar দিলে — একটা ভাঙলেও সেতু দাঁড়িয়ে থাকে। RAID অনেকটা এরকমই।

### RAID কেন ব্যবহার করা হয়?

RAID মূলত দুটো কারণে ব্যবহার হয়:

| উদ্দেশ্য | মানে কী |
|---|---|
| **Performance** | একসাথে multiple disk থেকে data read/write করো — দ্রুত হয় |
| **Redundancy** | একটা disk নষ্ট হলেও data হারাবে না |

---

## 🔢 RAID Levels — বিস্তারিত

### ⚡ RAID 0 — Striping (Speed, No Safety)

```
Disk 1:  [A1][A3][A5]
Disk 2:  [A2][A4][A6]
```

**কীভাবে কাজ করে:**
Data কে ছোট ছোট টুকরো (strips) করে পাশাপাশি সব disk-এ লেখে।

**Analogy:**
> দুটো লেন দিয়ে একসাথে গাড়ি চলছে — traffic দ্বিগুণ হয়, কিন্তু একটা লেন বন্ধ হলে সব বন্ধ।

| বিষয় | তথ্য |
|---|---|
| Minimum Disks | 2 |
| Speed | ✅ খুব দ্রুত (read/write দুটোই) |
| Redundancy | ❌ একদমই নেই |
| Usable Space | 100% (2×1TB = 2TB usable) |
| একটা Disk নষ্ট হলে | 💥 সব data হারিয়ে যাবে |

**DevOps Use Case:** Temporary scratch disk, video editing cache, CI build artifacts (যেখানে data হারালে সমস্যা নেই)

---

### 🛡️ RAID 1 — Mirroring (Safety, No Extra Speed)

```
Disk 1:  [A1][A2][A3]
Disk 2:  [A1][A2][A3]  ← exact copy
```

**কীভাবে কাজ করে:**
প্রতিটা data দুটো disk-এ হুবহু একই লেখা হয় (mirror)।

**Analogy:**
> তুমি একটা গুরুত্বপূর্ণ document-এর দুটো photocopy রাখলে। একটা নষ্ট হলে অন্যটা থেকে কাজ চলবে।

| বিষয় | তথ্য |
|---|---|
| Minimum Disks | 2 |
| Speed | Read দ্রুত, Write স্বাভাবিক |
| Redundancy | ✅ সম্পূর্ণ (১টা disk নষ্ট হলেও চলবে) |
| Usable Space | 50% (2×1TB = 1TB usable) |
| একটা Disk নষ্ট হলে | ✅ কোনো সমস্যা নেই |

**DevOps Use Case:** OS disk, database disk, যেখানে data হারানো একদমই চলবে না

---

### 🔄 RAID 5 — Striping with Parity (Balance of Speed + Safety)

```
Disk 1:  [A1][B1][C1][P4]
Disk 2:  [A2][B2][P3][C2]
Disk 3:  [A3][P2][B3][C3]
Disk 4:  [P1][A4][B4][C4]
              ↑
         Parity block (XOR calculation)
```

**কীভাবে কাজ করে:**
Data stripe করে লেখে এবং সাথে একটা **parity** block রাখে। Parity হলো XOR calculation — যদি একটা disk নষ্ট হয়, বাকি disks আর parity দিয়ে সেই data reconstruct করা যায়।

**Analogy:**
> তুমি 4 বন্ধুর সাথে একটা equation শেয়ার করলে যেখানে যেকোনো ১ জনের অংশ হারিয়ে গেলে বাকি ৩ জনের তথ্য দিয়ে সেটা বের করা যাবে।

| বিষয় | তথ্য |
|---|---|
| Minimum Disks | 3 |
| Speed | Read দ্রুত, Write একটু ধীর (parity calculate করতে হয়) |
| Redundancy | ✅ ১টা disk নষ্ট হলে চলবে |
| Usable Space | (N-1)/N × total (3×1TB = 2TB usable) |
| ২টা Disk নষ্ট হলে | 💥 data হারাবে |

**DevOps Use Case:** File servers, NAS storage, general-purpose server storage

---

### 🏆 RAID 10 — Stripe of Mirrors (Best of Both Worlds)

```
       Mirror Pair 1        Mirror Pair 2
       ↙          ↘        ↙          ↘
Disk 1: [A1][A2]  Disk 2: [A1][A2]  Disk 3: [B1][B2]  Disk 4: [B1][B2]
         ↑ stripe across mirror pairs ↑
```

**কীভাবে কাজ করে:**
প্রথমে disk গুলোকে mirror pair (RAID 1) বানানো হয়, তারপর সেই pair গুলোকে stripe (RAID 0) করা হয়।

**Analogy:**
> দুটো করে twins বানালে (mirror), তারপর সেই twins দের টিমকে একসাথে কাজে লাগালে (stripe)। একজন sick হলে তার twin কাজ করবে।

| বিষয় | তথ্য |
|---|---|
| Minimum Disks | 4 (even number) |
| Speed | ✅ খুব দ্রুত (RAID 0 এর মতো) |
| Redundancy | ✅ প্রতি pair থেকে ১টা করে নষ্ট হলে চলবে |
| Usable Space | 50% (4×1TB = 2TB usable) |
| সর্বোচ্চ disk failure tolerance | প্রতি mirror pair থেকে ১টা |

**DevOps Use Case:** High-traffic databases (MySQL, PostgreSQL), production application servers

---

## 📊 RAID Levels Quick Comparison

| RAID | Min Disks | Speed | Redundancy | Usable Space | Best For |
|---|---|---|---|---|---|
| **0** | 2 | ⚡⚡⚡ | ❌ | 100% | Temp/cache storage |
| **1** | 2 | ⚡⚡ | ✅✅ | 50% | OS/critical data |
| **5** | 3 | ⚡⚡ | ✅ | (N-1)/N | File/web servers |
| **10** | 4 | ⚡⚡⚡ | ✅✅ | 50% | Databases |

---

## 🔧 Software RAID vs Hardware RAID

Linux-এ দুই ধরনের RAID আছে:

| ধরন | কী | সুবিধা | অসুবিধা |
|---|---|---|---|
| **Hardware RAID** | Dedicated RAID controller card | খুব দ্রুত, CPU load নেই | দামি, vendor lock-in |
| **Software RAID** | OS নিজেই manage করে (`mdadm`) | সস্তা, flexible | সামান্য CPU usage |

**DevOps-এ Software RAID (mdadm) বেশি ব্যবহার হয়** কারণ cloud/VM environment-এ hardware RAID controller থাকে না।

---

## 🛠️ mdadm — Linux Software RAID Tool

`mdadm` মানে **Multiple Device Admin।** এটাই Linux-এর primary software RAID management tool।

### Installation:

```bash
# Ubuntu/Debian
sudo apt install mdadm -y

# RHEL/CentOS
sudo yum install mdadm -y
```

---

## 💻 Practical: RAID তৈরি করা

### 🔍 Step 1: Available Disks দেখো

```bash
lsblk
```

**Example Output:**
```
NAME   MAJ:MIN RM  SIZE RO TYPE MOUNTPOINT
sda      8:0    0   20G  0 disk
├─sda1   8:1    0   19G  0 part /
└─sda2   8:2    0    1G  0 part [SWAP]
sdb      8:16   0   10G  0 disk
sdc      8:32   0   10G  0 disk
sdd      8:48   0   10G  0 disk
sde      8:64   0   10G  0 disk
```

এখানে `sdb`, `sdc`, `sdd`, `sde` — এই ৪টা disk আমরা RAID-এ ব্যবহার করবো।

---

### 🔨 Step 2: RAID 1 তৈরি করা (২টা Disk)

```bash
sudo mdadm --create /dev/md0 \
  --level=1 \
  --raid-devices=2 \
  /dev/sdb /dev/sdc
```

**প্রতিটা অংশের মানে:**

| অংশ | মানে |
|---|---|
| `--create /dev/md0` | নতুন RAID device তৈরি করো, নাম হবে `md0` |
| `--level=1` | RAID level 1 (mirror) |
| `--raid-devices=2` | মোট ২টা disk ব্যবহার হবে |
| `/dev/sdb /dev/sdc` | এই দুটো disk ব্যবহার করো |

**Expected Output:**
```
mdadm: Note: this array has metadata at the start and
    may not be suitable as a boot device.  If you plan to
    store '/boot' on this device please ensure that
    your boot-loader understands md/v1.x metadata, or use
    --metadata=0.90
Continue creating array? y
mdadm: Metadata version 1.2 applied to /dev/sdb
mdadm: Metadata version 1.2 applied to /dev/sdc
mdadm: array /dev/md0 started.
```

---

### 🔨 Step 3: RAID 5 তৈরি করা (৩টা Disk)

```bash
sudo mdadm --create /dev/md1 \
  --level=5 \
  --raid-devices=3 \
  /dev/sdb /dev/sdc /dev/sdd
```

---

### 🔨 Step 4: RAID 10 তৈরি করা (৪টা Disk)

```bash
sudo mdadm --create /dev/md2 \
  --level=10 \
  --raid-devices=4 \
  /dev/sdb /dev/sdc /dev/sdd /dev/sde
```

---

## 📊 RAID Status Monitor করা

### RAID এর status দেখো:

```bash
cat /proc/mdstat
```

**Example Output (RAID সবে তৈরি হচ্ছে):**
```
Personalities : [raid1] [raid6] [raid5] [raid4]
md0 : active raid1 sdc[1] sdb[0]
      10477568 blocks super 1.2 [2/2] [UU]
      [=>...................]  resync =  8.5% (892416/10477568) finish=0.8min speed=198427K/sec

unused devices: <none>
```

**Output বোঝো:**

| অংশ | মানে |
|---|---|
| `md0 : active raid1` | md0 হলো active RAID 1 array |
| `sdc[1] sdb[0]` | sdb disk 0, sdc disk 1 হিসেবে আছে |
| `[UU]` | দুটো disk-ই Up ও healthy |
| `[_U]` বা `[U_]` | একটা disk down/failed |
| `resync = 8.5%` | RAID sync হচ্ছে, 8.5% সম্পন্ন |

---

### Detailed RAID info দেখো:

```bash
sudo mdadm --detail /dev/md0
```

**Example Output:**
```
/dev/md0:
           Version : 1.2
     Creation Time : Sun Mar 15 10:30:00 2026
        Raid Level : raid1
        Array Size : 10477568 (9.99 GiB 10.73 GB)
     Used Dev Size : 10477568 (9.99 GiB 10.73 GB)
      Raid Devices : 2
     Total Devices : 2
       Persistence : Superblock is persistent

       Update Time : Sun Mar 15 10:31:00 2026
             State : clean
    Active Devices : 2
   Working Devices : 2
    Failed Devices : 0
     Spare Devices : 0

Consistency Policy : resync

              Name : server1:0  (local to host server1)
              UUID : a7d8e9f0:1234abcd:5678efgh:9012ijkl
            Events : 17

    Number   Major   Minor   RaidDevice State
       0       8       16        0      active sync   /dev/sdb
       1       8       32        1      active sync   /dev/sdc
```

**গুরুত্বপূর্ণ fields:**

| Field | মানে |
|---|---|
| `State: clean` | RAID সুস্থ আছে ✅ |
| `State: degraded` | একটা disk নষ্ট, কিন্তু চলছে ⚠️ |
| `Failed Devices: 0` | কোনো disk fail হয়নি |
| `Spare Devices: 0` | কোনো hot spare নেই |

---

## 🗂️ RAID কে Format ও Mount করা

RAID array তৈরির পর এটাকে regular disk এর মতোই ব্যবহার করো:

```bash
# Step 1: Format করো
sudo mkfs.ext4 /dev/md0

# Step 2: Mount point তৈরি করো
sudo mkdir /mnt/raid1

# Step 3: Mount করো
sudo mount /dev/md0 /mnt/raid1

# Step 4: Verify করো
df -h /mnt/raid1
```

**Output:**
```
Filesystem      Size  Used Avail Use% Mounted on
/dev/md0        9.8G   24M  9.3G   1% /mnt/raid1
```

---

## 💾 RAID Configuration Save করা (Reboot এর পরেও কাজ করুক)

```bash
sudo mdadm --detail --scan | sudo tee -a /etc/mdadm/mdadm.conf
```

তারপর `/etc/fstab`-এ add করো:

```bash
echo '/dev/md0 /mnt/raid1 ext4 defaults 0 0' | sudo tee -a /etc/fstab
```

---

## 🚨 RAID Failure Simulation ও Recovery

### Scenario: একটা Disk Fail হলো — কী করবে?

**Step 1: Disk টাকে manually fail করো (test এর জন্য):**

```bash
sudo mdadm --manage /dev/md0 --fail /dev/sdb
```

**Output:**
```
mdadm: set /dev/sdb faulty in /dev/md0
```

**Step 2: Status দেখো:**

```bash
cat /proc/mdstat
```

**Output:**
```
md0 : active raid1 sdc[1] sdb[0](F)
      10477568 blocks super 1.2 [2/1] [_U]
```

`(F)` মানে **Faulty**, `[_U]` মানে একটা disk down।

**Step 3: Failed disk remove করো:**

```bash
sudo mdadm --manage /dev/md0 --remove /dev/sdb
```

**Step 4: নতুন disk add করো (replacement):**

```bash
sudo mdadm --manage /dev/md0 --add /dev/sdb
```

**Step 5: Rebuild হচ্ছে কিনা দেখো:**

```bash
watch cat /proc/mdstat
```

**Output (rebuilding চলছে):**
```
md0 : active raid1 sdb[2] sdc[1]
      10477568 blocks super 1.2 [2/1] [_U]
      [==>.................]  recovery = 12.5% (1310720/10477568) finish=1.2min
```

Rebuild শেষ হলে `[UU]` দেখাবে — array আবার healthy! ✅

---

## 🔥 Hot Spare — Auto-Rebuild

Hot spare মানে হলো একটা **extra disk** যেটা idle থাকে। যখনই কোনো disk fail হয়, hot spare automatically সেই জায়গা নেয় এবং rebuild শুরু হয়।

```bash
# RAID 5 তৈরি করো + ১টা hot spare যোগ করো
sudo mdadm --create /dev/md1 \
  --level=5 \
  --raid-devices=3 \
  --spare-devices=1 \
  /dev/sdb /dev/sdc /dev/sdd \
  /dev/sde   # ← এটা hot spare
```

Status দেখো:
```bash
sudo mdadm --detail /dev/md1
```

```
    Number   Major   Minor   RaidDevice State
       0       8       16        0      active sync   /dev/sdb
       1       8       32        1      active sync   /dev/sdc
       2       8       48        2      active sync   /dev/sdd
       3       8       64        -      spare         /dev/sde   ← hot spare
```

---

## 📧 RAID Failure Notification Setup

Production server-এ disk failure হলে email পাওয়া উচিত:

```bash
# /etc/mdadm/mdadm.conf এ add করো
MAILADDR admin@yourcompany.com
```

তারপর:
```bash
sudo systemctl restart mdmonitor
```

এখন কোনো disk fail হলে তুমি email পাবে।

---

## 🔍 Useful mdadm Commands — Cheat Sheet

```bash
# RAID তৈরি করা
sudo mdadm --create /dev/md0 --level=1 --raid-devices=2 /dev/sdb /dev/sdc

# RAID detail দেখা
sudo mdadm --detail /dev/md0

# সব RAID arrays দেখা
cat /proc/mdstat

# Disk manually fail করা (testing)
sudo mdadm --manage /dev/md0 --fail /dev/sdb

# Failed disk remove করা
sudo mdadm --manage /dev/md0 --remove /dev/sdb

# নতুন disk add করা
sudo mdadm --manage /dev/md0 --add /dev/sdb

# Hot spare add করা
sudo mdadm --manage /dev/md0 --add-spare /dev/sde

# RAID config save করা
sudo mdadm --detail --scan >> /etc/mdadm/mdadm.conf

# RAID stop করা
sudo mdadm --stop /dev/md0

# RAID সম্পূর্ণ মুছে ফেলা
sudo mdadm --zero-superblock /dev/sdb /dev/sdc
```

---

## 🌍 Real DevOps Scenarios

| Scenario | RAID Choice | কারণ |
|---|---|---|
| Web server static files | RAID 5 | balanced speed + safety, cost effective |
| MySQL production database | RAID 10 | maximum speed + redundancy |
| Build server temp storage | RAID 0 | max speed, data loss acceptable |
| OS boot disk | RAID 1 | simple mirroring, OS must survive disk failure |
| NAS/File Server | RAID 5 বা RAID 6 | efficient storage use with fault tolerance |

---

## 🔔 RAID ব্যবহারের সময় মনে রাখো

> ⚠️ **RAID is NOT a backup!**
> RAID disk failure থেকে protect করে, কিন্তু accidental deletion, ransomware, বা corruption থেকে রক্ষা করে না। সবসময় আলাদা backup রাখো।

---

## 📝 Quick Summary

- ✅ **RAID 0** — Speed only, no redundancy, data loss on any disk failure
- ✅ **RAID 1** — Mirror copy, 50% space, survives 1 disk failure
- ✅ **RAID 5** — Stripe + parity, (N-1)/N space, survives 1 disk failure, min 3 disks
- ✅ **RAID 10** — Stripe of mirrors, 50% space, best performance + redundancy, min 4 disks
- ✅ **mdadm** — Linux software RAID tool
- ✅ **`/proc/mdstat`** — RAID status real-time দেখার জায়গা
- ✅ **Hot spare** — Auto-rebuild এর জন্য idle extra disk
- ✅ **RAID ≠ Backup** — সবসময় আলাদা backup রাখো

---

## 🏋️ Practice Tasks

1. **`/proc/mdstat` output পড়া practice করো।** একটা virtual machine-এ `mdadm` install করো এবং `lsblk` দিয়ে available disks দেখো।

2. **RAID simulation:** VirtualBox বা VMware-এ ২টা extra virtual disk add করো এবং RAID 1 তৈরি করার command গুলো practice করো — `mdadm --create`, `mdadm --detail`।

3. **Failure scenario simulate করো:** RAID 1 তৈরির পর `--fail` দিয়ে একটা disk fail করো, তারপর `--remove` করো এবং নতুন disk `--add` করে rebuild হতে দেখো।

---

## ⏭️ What's Next?

**Chapter 7 — Lesson 8: NFS — Network File System**
> Server থেকে network এর মাধ্যমে filesystem share করবে — NFS server setup, client mount, `/etc/exports` configuration, এবং DevOps-এ shared storage কীভাবে কাজ করে তা শিখবে।

তুমি এগিয়ে যাচ্ছো দারুণভাবে! RAID একটু complex topic — এটা বুঝতে পারলে storage architecture নিয়ে confident হওয়া যায়। 💪