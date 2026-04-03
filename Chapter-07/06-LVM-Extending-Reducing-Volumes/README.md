# Chapter 7 - Lesson 6: LVM - Extending & Reducing Volumes

**Chapter 7 | Lesson 6 of 10**


## 🎯 এই Lesson-এ আমরা কী শিখব?

- LVM Volume কিভাবে extend (বাড়ানো) করতে হয়
- File System কিভাবে resize করতে হয়
- LVM Volume কিভাবে reduce (কমানো) করতে হয়
- Real-world DevOps scenario তে এটা কখন দরকার হয়


আগের Lesson-এ আমরা শিখেছিলাম:

```
Physical Volume (PV) → Volume Group (VG) → Logical Volume (LV)
```

মনে করেন আপনি একটা **LV** তৈরি করেছেন `/dev/myvg/mylv` এবং সেটা `/mnt/data` ডিরেকটরিতে mount করা আছে।

এখন সেই disk full হয়ে যাচ্ছে এখন আপনাকে সেটা extend করতে হবে।

## Real-world Analogy

> আপনার কাছে একটা আলমারি আছে (Logical Volume)।
> আলমারি ভরে গেছে।
>
> আপনি পাশে একটা নতুন তাক (Physical Volume) লাগিয়ে আলমারি বড় করলেন।
> এটাই LVM Extend!
>
> কিন্তু যদি কমাতে চান তাহলে আগে জিনিসপত্র সরাতে হবে, তারপর তাক কমাতে হবে।
> এটাই LVM Reduce এটা বিপজ্জনক, সতর্ক থাকতে হবে!

## পুরো Process-এর Map

```
EXTEND করতে:
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
[নতুন Disk/Partition]
        ↓
  pvcreate  →  নতুন PV তৈরি
        ↓
  vgextend  →  VG-তে নতুন PV যোগ
        ↓
  lvextend  →  LV-এর size বাড়ানো
        ↓
  resize2fs / xfs_growfs  →  File System কে জানানো

REDUCE করতে (ext4 only, XFS পারে না):
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
  umount    →  আগে unmount করেন
        ↓
  e2fsck    →  File System check
        ↓
  resize2fs →  File System ছোট করেন
        ↓
  lvreduce  →  LV ছোট করেন
        ↓
  mount     →  আবার mount করেন
```


## PART 1: LV Extend করা

### ধাপ ১ - আগে VG-তে Free Space আছে কিনা দেখেন

```bash
vgdisplay myvg
```

**Output:**
```
  --- Volume group ---
  VG Name               myvg
  VG Size               20.00 GiB
  PE Size               4.00 MiB
  Total PE              5119
  Alloc PE / Size       2560 / 10.00 GiB
  Free  PE / Size       2559 / 10.00 GiB   ← এখানে Free Space আছে
```

> `Free PE / Size` দেখুন এখানে **10 GiB** free Space আছে মানে extend করা যাবে।


### ধাপ ২ - যদি VG-তে Free Space না থাকে → নতুন PV যোগ করেন

```bash
# নতুন disk আছে ধরো /dev/sdc
pvcreate /dev/sdc
```

```
  Physical volume "/dev/sdc" successfully created.
```

```bash
# এখন VG-তে নতুন PV যোগ করেন
vgextend myvg /dev/sdc
```

```
  Volume group "myvg" successfully extended
```

এখন VG-তে আরও space যুক্ত হয়েছে। এখন extend করা যাবে।


### ধাপ ৩ - `lvextend` দিয়ে LV বাড়ান

এটা সবচেয়ে গুরুত্বপূর্ণ command।

#### Syntax:
```bash
lvextend [option] <LV_path>
```


#### Option ১: নির্দিষ্ট size যোগ করেন (+)

```bash
lvextend -L +5G /dev/myvg/mylv
```

| Part | মানে |
|---|---|
| `-L` | Size specify করতে |
| `+5G` | **আরও** 5 GiB যোগ করেন (+ মানে add) |
| `/dev/myvg/mylv` | কোন LV-তে? |

**Output:**
```
  Size of logical volume myvg/mylv changed from 10.00 GiB to 15.00 GiB.
  Logical volume myvg/mylv successfully resized.
```

#### Option ২: নির্দিষ্ট size-এ নিয়ে যান (absolute)

```bash
lvextend -L 20G /dev/myvg/mylv
```

> এটা LV-কে ঠিক **20 GiB** করে দেবে, যা-ই আগে ছিল।


#### Option ৩: VG-এর সব Free Space ব্যবহার করেন

```bash
lvextend -l +100%FREE /dev/myvg/mylv
```

| Part | মানে |
|---|---|
| `-l` | Extents দিয়ে specify করেন (lowercase L) |
| `+100%FREE` | VG-তে যা free আছে সব নাও |

> DevOps-এ এটা সবচেয়ে বেশি ব্যবহার হয়। disk এ্যাড করলে সব space LV-তে দিয়ে দেন।


### ধাপ ৪ - File System Resize করা - সবচেয়ে গুরুত্বপূর্ণ ধাপ!

> LV বাড়ালেই হয় না! File System-কেও জানাতে হবে যে তার জায়গা বেড়েছে।
>
> এটা না করলে `df -h` এ পুরনো size-ই দেখাবে!


#### ext4 File System-এর জন্য → `resize2fs`

```bash
resize2fs /dev/myvg/mylv
```

**Output:**
```
resize2fs 1.46.5
Filesystem at /dev/myvg/mylv is mounted on /mnt/data; on-line resizing required
old_desc_blocks = 2, new_desc_blocks = 2
The filesystem on /dev/myvg/mylv is now 3932160 (4k) blocks long.
```

> `resize2fs` online resize করতে পারে মানে unmount না করেই বাড়ানো যায়!


#### xfs File System-এর জন্য → `xfs_growfs`

```bash
xfs_growfs /mnt/data
```

> XFS-এ **mount point** দিতে হয়, device path না।


#### Shortcut: একসাথে extend + resize করেন!

```bash
lvextend -L +5G -r /dev/myvg/mylv
```

| Part | মানে |
|---|---|
| `-r` | `--resizefs` LV extend করার পরপরই file system resize করেন |

> এটা সবচেয়ে সহজ একটা command-এই সব হয়ে যায়!


### ধাপ ৫ - Verify করেন

```bash
df -h /mnt/data
```

**Output:**
```
Filesystem               Size  Used Avail Use% Mounted on
/dev/mapper/myvg-mylv    15G   2.1G   13G  14% /mnt/data
```

Size বেড়ে গেছে!


## PART 2: LV Reduce করা বিপজ্জনক!

> **Warning:** Reduce করলে data loss হতে পারে যদি সঠিকভাবে না করতে না পারেন।
> XFS reduce করা যায় না শুধু ext4 পারে।
> সবসময় আগে backup নিয়ে রাখতে হবে।


### ধাপ ১ - Unmount করা

```bash
umount /mnt/data
```

> Reduce করতে হলে অবশ্যই unmount করতে হবে। Online reduce সম্ভব না।


### ধাপ ২ - File System Check করেন

```bash
e2fsck -f /dev/myvg/mylv
```

| Part | মানে |
|---|---|
| `e2fsck` | ext2/3/4 file system checker |
| `-f` | Force check - mounted না থাকলেও check করেন |

**Output:**
```
e2fsck 1.46.5
Pass 1: Checking inodes, blocks, and sizes
Pass 2: Checking directory structure
Pass 3: Checking directory connectivity
Pass 4: Checking reference counts
Pass 5: Checking group summary information
/dev/myvg/mylv: 11/655360 files (0.0% non-contiguous), 66780/2621440 blocks
```

> কোনো error না থাকলেই পরের ধাপে যান।


### ধাপ ৩ - প্রথমে File System ছোট করে ফেলেন!

```bash
resize2fs /dev/myvg/mylv 8G
```

> আগে file system ছোট করতে হবে, তারপর LV। ছোট না করলে data নষ্ট হবে!

**Output:**
```
resize2fs 1.46.5
Resizing the filesystem on /dev/myvg/mylv to 2097152 (4k) blocks.
The filesystem on /dev/myvg/mylv is now 2097152 (4k) blocks long.
```


### ধাপ ৪ - LV ছোট করা

```bash
lvreduce -L 8G /dev/myvg/mylv
```

**Output দেখাবে warning:**
```
  WARNING: Reducing active logical volume to 8.00 GiB.
  THIS MAY DESTROY YOUR DATA (filesystem etc.)
  Do you really want to reduce myvg/mylv? [y/n]: y
  Size of logical volume myvg/mylv changed from 15.00 GiB to 8.00 GiB.
```

> `y` দিয়ে confirm করুন।


#### Shortcut - একসাথে করা (safer):

```bash
lvreduce -L -2G -r /dev/myvg/mylv
```

> `-r` flag দিলে সে নিজেই `e2fsck` + `resize2fs` + `lvreduce` সব করবে।
> কিন্তু তবুও আগে unmount করতে হবে।


### ধাপ ৫ - আবার Mount করেন

```bash
mount /dev/myvg/mylv /mnt/data
df -h /mnt/data
```

**Output:**
```
Filesystem               Size  Used Avail Use% Mounted on
/dev/mapper/myvg-mylv    8.0G  2.1G  5.9G  27% /mnt/data
```

Size কমে গেছে!


## সব Command এক জায়গায়

| Command | কাজ |
|---|---|
| `vgdisplay <vg>` | VG-তে free space দেখা |
| `pvcreate /dev/sdX` | নতুন PV তৈরি |
| `vgextend <vg> /dev/sdX` | VG-তে নতুন PV যোগ |
| `lvextend -L +5G <lv>` | LV-তে 5G যোগ করা |
| `lvextend -l +100%FREE <lv>` | সব free space নাও |
| `lvextend -L +5G -r <lv>` | LV extend + FS resize একসাথে |
| `resize2fs <lv>` | ext4 FS resize (extend/reduce) |
| `xfs_growfs <mountpoint>` | XFS FS resize (শুধু extend) |
| `umount <mountpoint>` | Reduce-এর আগে unmount |
| `e2fsck -f <lv>` | FS check করা |
| `lvreduce -L 8G <lv>` | LV ছোট করা |
| `lvreduce -L -2G -r <lv>` | LV ছোট করা + FS resize একসাথে |


## Real-world DevOps Scenario

### Scenario 1: Production Server-এর `/var` full হয়ে গেছে!

```bash
# /var ডিরেকটরি 95% full
df -h /var

# LV দেখুন
lvdisplay /dev/myvg/var_lv

# VG-তে free space আছে কিনা দেখুন
vgs

# একটা command-এ extend + resize করুন (no downtime!)
lvextend -L +10G -r /dev/myvg/var_lv

# confirm করুন
df -h /var
```

> Production server-এ downtime ছাড়াই disk বাড়ানো - LVM-এর সবচেয়ে বড় power!


### Scenario 2: নতুন disk যোগ করে space বাড়ানো

```bash
# নতুন disk /dev/sdd দেওয়া হয়েছে
pvcreate /dev/sdd
vgextend datavg /dev/sdd
lvextend -l +100%FREE -r /dev/datavg/datalv
df -h /data
```


## 📝 Quick Summary

- `lvextend` দিয়ে LV বড় করেন, তারপর `resize2fs` বা `xfs_growfs` দিয়ে FS resize করেন
- `-r` flag ব্যবহার করলে FS resize automatically হয়
- ext4 extend করা যায় online (unmount ছাড়াই)
- XFS শুধু grow করা যায়, reduce করা যায় না
- Reduce করতে হলে: unmount → e2fsck → resize2fs → lvreduce → mount
- Reduce-এর আগে সবসময় backup নিয়ে রাখতে হবে
- `vgdisplay` বা `vgs` দিয়ে সবসময় free space আছে কিনা প্রথমে চেক করে নেয়া


## 🏋️ Practice Tasks

1. একটা 10G LV তৈরি করেন, ext4 ফাইল সিস্টেম দিয়ে format করেন, mount করেন। তারপর `lvextend -L +3G -r` দিয়ে extend করেন এবং `df -h` দিয়ে confirm করেন।

2. `vgdisplay` এবং `lvdisplay` রান করেন এবং output-এর প্রতিটি line বোঝার চেষ্টা করেন। কোনটা size, কোনটা free space ইত্যাদি।

3. একটা LV extend করার পর `resize2fs` না চালিয়ে `df -h` দেখুন - তারপর `resize2fs` চালিয়ে আবার দেখুন। পার্থক্য বোঝার চেস্টা করুন।

---

## ⏭️ What's Next?

**Chapter 7 - Lesson 7: RAID in Linux**
> `mdadm` দিয়ে RAID 0, 1, 5, 10 কিভাবে তৈরি করতে হয়, কোনটা কখন ব্যবহার করবো, এবং DevOps-এ RAID-এর real-world use case। *Happy Learning* 🚀

<table width="100%">
  <tr>
    <td align="left">
      <a href="../05-LVM">← LVM - Logical Volume Manager</a>
    </td>
    <td align="right">
      <a href="../07-RAID-in-Linux">RAID in Linux →</a>
    </td>
  </tr>
</table>