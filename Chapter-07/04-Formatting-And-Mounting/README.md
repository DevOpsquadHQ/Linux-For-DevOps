# Chapter 7 - Lesson 4: Formatting & Mounting

**Chapter 7 | Lesson 4 of 10**


## 🎯 এই Lesson-এ আমরা কী শিখবো?

- `mkfs` দিয়ে partition format করা
- `mount` ও `umount` দিয়ে filesystem attach/detach করা
- `/etc/fstab` ফাইল বোঝা ও permanent mount করা
- Real DevOps scenarios


মনে করেন আপনি একটা নতুন USB drive কিনলে।

| বাস্তব জীবন | Linux-এ |
|---|---|
| USB drive কেনা | নতুন disk/partition তৈরি করা |
| USB drive **format** করা (FAT32/NTFS) | `mkfs` দিয়ে filesystem তৈরি করা |
| USB drive কম্পিউটারে **plug in** করা | `mount` করা |
| USB drive **safely eject** করা | `umount` করা |
| কম্পিউটার চালু হলে **auto-connect** হওয়া | `/etc/fstab`-এ entry দেওয়া |


## Part 1: mkfs - Make File System

### mkfs কী?

`mkfs` মানে **Make File System**। একটা partition-কে ব্যবহারযোগ্য করতে হলে তাকে একটা নির্দিষ্ট filesystem দিয়ে format করতে হয়। এটা ঠিক যেভাবে একটা খালি জমিতে বাড়ি বানানোর আগে **foundation** তৈরি করতে হয়।

### Common mkfs Commands

```bash
mkfs.ext4    # Linux-এর সবচেয়ে popular filesystem
mkfs.xfs     # High-performance, production servers-এ বেশি ব্যবহার হয়
mkfs.btrfs   # Modern, snapshot support আছে
mkfs.vfat    # FAT32 - USB/Windows compatibility-র জন্য
```

### Syntax

```bash
mkfs.<type> [options] <device>
```

### Example 1: ext4 দিয়ে format করা

```bash
sudo mkfs.ext4 /dev/sdb1
```

**Expected Output:**
```
mke2fs 1.46.5 (30-Dec-2021)
Creating filesystem with 5242880 4k blocks and 1310720 inodes
Filesystem UUID: a1b2c3d4-e5f6-7890-abcd-ef1234567890
Superblock backups stored on blocks:
        32768, 98304, 163840, 229376, 294912

Allocating group tables: done
Writing inode tables: done
Creating journal (32768 blocks): done
Writing superblocks and filesystem accounting information: done
```

### Example 2: Label সহ format করা

Label মানে disk-এর একটা নাম দেওয়া যাতে পরে খুব সহজে চিনতে পারি।

```bash
sudo mkfs.ext4 -L "mydata" /dev/sdb1
```

`-L "mydata"` → এই partition-এর নাম হবে "mydata"


### Example 3: xfs দিয়ে format করা

```bash
sudo mkfs.xfs /dev/sdb2
```

**Expected Output:**
```
meta-data=/dev/sdb2       isize=512    agcount=4, agsize=655360 blks
data     =                bsize=4096   blocks=2621440, imaxpct=25
naming   =version 2       bsize=4096   ascii-ci=0, ftype=1
log      =internal log    bsize=4096   blocks=2560, version=2
realtime =none            extsz=4096   blocks=0, rtextents=0
```


### ⚠️ সতর্কতা!

> `mkfs` একটা destructive command। এটা রান করালে সেই partition-এর সব data মুছে যাবে। সবসময় double-check করে নিন যে কোন device-এ কমান্ডটি রান করছেন।


## Part 2: mount - Filesystem Attach করা

### mount কী?

Linux-এ কোনো storage use করতে হলে সেটাকে filesystem tree-র একটা directory-তে attach করতে হয়। এই কাজটাই `mount` করে।

ধরুন আপনার বাড়িতে একটা নতুন room আছে কিন্তু দরজা নেই। `mount` হলো সেই room-এর সাথে একটা দরজা লাগানো যাতে আপনি ঢুকতে পারেন।

### Syntax

```bash
mount [options] <device> <mount_point>
```

| অংশ | মানে |
|---|---|
| `<device>` | কোন disk/partition - যেমন `/dev/sdb1` |
| `<mount_point>` | কোন directory-তে attach হবে - যেমন `/mnt/mydata` |


### Step-by-step: Disk Mount করা

**Step 1: Mount point directory তৈরি করুন**

```bash
sudo mkdir -p /mnt/mydata
```

**Step 2: Disk mount করুন**

```bash
sudo mount /dev/sdb1 /mnt/mydata
```

**Step 3: Verify করুন**

```bash
mount | grep sdb1
```

**Expected Output:**
```
/dev/sdb1 on /mnt/mydata type ext4 (rw,relatime)
```


### Example: Mount হয়েছে কিনা দেখুন

```bash
df -h
```

**Expected Output:**
```
Filesystem      Size  Used Avail Use% Mounted on
/dev/sda1        20G  5.2G   14G  28% /
/dev/sdb1        10G   24K  9.5G   1% /mnt/mydata
tmpfs           1.9G     0  1.9G   0% /dev/shm
```

এখন `/mnt/mydata`-তে গেলেই `/dev/sdb1`-এর content দেখা যাবে।


### Specific filesystem type দিয়ে mount

```bash
sudo mount -t ext4 /dev/sdb1 /mnt/mydata
```

`-t ext4` → filesystem type explicitly বলে দেওয়া (না দিলেও Linux নিজে detect করে)


### Read-only mount করা

```bash
sudo mount -o ro /dev/sdb1 /mnt/mydata
```

`-o ro` → read-only mode। কেউ এই disk-এ লিখতে পারবে না।

**DevOps Use Case:** Production backup disk - read-only mount করা উচিত। যাতে ভুলেও data overwrite না হয়।


### Currently mounted সব filesystem দেখা

```bash
mount
# অথবা আরো সুন্দর output-এর জন্য:
findmnt
```

**`findmnt` Output:**
```
TARGET                                SOURCE     FSTYPE     OPTIONS
/                                     /dev/sda1  ext4       rw,relatime
├─/mnt/mydata                         /dev/sdb1  ext4       rw,relatime
├─/proc                               proc       proc       rw,nosuid
└─/sys                                sysfs      sysfs      rw,nosuid
```


## Part 3: umount - Filesystem Detach করা

### umount কী?

`umount` দিয়ে একটা mounted filesystem-কে safely detach করা হয়। USB drive eject করার মতো।

### Syntax

```bash
sudo umount <device অথবা mount_point>
```


### Example 1: mount point দিয়ে umount

```bash
sudo umount /mnt/mydata
```


### Example 2: device দিয়ে umount

```bash
sudo umount /dev/sdb1
```


### Common Error: Device is busy

```bash
sudo umount /mnt/mydata
# Output:
umount: /mnt/mydata: target is busy.
```

**মানে:** কেউ (কোনো process) এই directory-তে আছে বা এর কোনো file use করছে।

**Solution:**
```bash
# কে ব্যবহার করছে দেখুন
sudo lsof /mnt/mydata
# অথবা
sudo fuser -m /mnt/mydata

# Force unmount (সাবধানে!)
sudo umount -f /mnt/mydata

# Lazy unmount - সব process শেষ হলে auto-unmount হবে
sudo umount -l /mnt/mydata
```


## Part 4: /etc/fstab - Permanent Mount Configuration

### /etc/fstab কী?

প্রতিবার system restart হলে manually `mount` করা অনেক ঝামেলা। `/etc/fstab` ফাইলে একবার লিখে দিলে system boot হওয়ার সময় automatically mount হয়ে যাবে।

এটা যেন আপনার বাসার চাবির তালিকা - কোন তালায় কোন চাবি লাগবে সেটা লেখা আছে। বাড়িতে ঢুকতে হলে এই তালিকা দেখেই কাজ হয়।


### /etc/fstab এর Structure

```bash
cat /etc/fstab
```

**Output:**
```
# <file system>  <mount point>  <type>  <options>  <dump>  <pass>
UUID=abc123...   /              ext4    defaults    0       1
UUID=def456...   /boot          ext4    defaults    0       2
/dev/sdb1        /mnt/mydata    ext4    defaults    0       2
```

প্রতিটা column-এর মানে:

| Column | নাম | মানে |
|---|---|---|
| 1 | **File System** | Device path বা UUID |
| 2 | **Mount Point** | কোথায় mount হবে |
| 3 | **Type** | Filesystem type (ext4, xfs, swap...) |
| 4 | **Options** | Mount options (defaults, ro, noexec...) |
| 5 | **Dump** | Backup করবে কিনা (0 = না, 1 = হ্যাঁ) - সাধারণত 0 |
| 6 | **Pass** | Boot-এ fsck চেক order (0 = skip, 1 = root, 2 = others) |


### UUID কেন ব্যবহার করবো?

`/dev/sdb1` এর সমস্যা হলো disk add/remove করলে এই নাম পরিবর্তন হয়ে যেতে পারে। কিন্তু UUID সবসময় একই থাকে।

```bash
# Disk-এর UUID বের করুন
sudo blkid /dev/sdb1
```

**Output:**
```
/dev/sdb1: UUID="a1b2c3d4-e5f6-7890-abcd-ef1234567890" TYPE="ext4" LABEL="mydata"
```


### /etc/fstab-এ Entry যোগ করা - Step by Step

**Step 1: UUID বের করুন**
```bash
sudo blkid /dev/sdb1
# UUID="a1b2c3d4-e5f6-7890-abcd-ef1234567890"
```

**Step 2: fstab ফাইল edit করুন**
```bash
sudo nano /etc/fstab
```

**Step 3: নিচে এই line যোগ করুন**
```
UUID=a1b2c3d4-e5f6-7890-abcd-ef1234567890   /mnt/mydata   ext4   defaults   0   2
```

**Step 4: Save করুন** (`Ctrl+O`, `Enter`, `Ctrl+X`)

**Step 5: fstab test করুন (reboot ছাড়াই!)**
```bash
sudo mount -a
```

`-a` মানে fstab-এর সব entry mount করুন। যদি কোনো error না আসে - সব ঠিক আছে!

**Step 6: Verify করুন**
```bash
df -h | grep mydata
```


### Common fstab Options

| Option | মানে |
|---|---|
| `defaults` | rw, suid, dev, exec, auto, nouser, async - সব default চালু |
| `ro` | Read-only |
| `rw` | Read-write |
| `noexec` | এই filesystem-এ কোনো program execute করা যাবে না |
| `nosuid` | SUID/SGID bits ignore করবে (security) |
| `noauto` | Boot-এ auto mount হবে না |
| `user` | Normal user mount করতে পারবে |
| `nofail` | Device না থাকলেও boot fail হবে না |


### Real DevOps Example: External Data Disk

মনে করেন production server-এ একটা আলাদা `/data` নামে একটা ডিরেক্টরি আছে:

```bash
# /etc/fstab entry:
UUID=xxxx-yyyy   /data   xfs   defaults,nofail   0   2
```

`nofail` দেওয়া হলো কারণ যদি কোনো কারণে disk না থাকে বা /data নামে কোন ডিরেকটরি না থাকে, তাহলে যাতে server boot হওয়া বন্ধ না হয়। Production-এ এটা অনেক important!


## Part 5: Swap Space - Bonus Topic

### Swap কী?

RAM পূর্ণ হয়ে গেলে Linux disk-এর একটা অংশকে temporary RAM হিসেবে ব্যবহার করে - এটাই swap।

```bash
# Swap partition fstab-এ:
UUID=swap-uuid   none   swap   sw   0   0

# Swap দেখুন:
swapon --show
free -h
```


## পুরো Flow একবার দেখি

```
নতুন Disk
    │
    ▼
fdisk/gdisk → Partition তৈরি (/dev/sdb1)
    │
    ▼
mkfs.ext4 → Filesystem Format করা
    │
    ▼
mkdir /mnt/mydata → Mount Point তৈরি
    │
    ▼
mount /dev/sdb1 /mnt/mydata → Temporarily Mount
    │
    ▼
/etc/fstab → Permanently Mount (UUID ব্যবহার করে)
    │
    ▼
mount -a → Test করা
```


## 📝 Quick Summary

- `mkfs` - partition-কে format করে filesystem তৈরি করে (`mkfs.ext4`, `mkfs.xfs`)
- `mount` - filesystem-কে directory-তে attach করে, use করার সুযোগ দেয়
- `umount` - filesystem safely detach করে
- `/etc/fstab` - boot-এ automatic mount-এর configuration ফাইল
- UUID ব্যবহার করা উচিত device name (`/dev/sdb1`) এর বদলে কারণ UUID কখনো বদলায় না
- `mount -a` দিয়ে reboot ছাড়াই fstab test করা যায়
- Production-এ `nofail` option ব্যবহার করা উচিত


## Practice Tasks

> এগুলো নিজে নিজে try করুন। Virtual machine বা lab environment-এ করুন।

**Task 1:**
একটা নতুন disk বা loop device তৈরি করুন এবং `mkfs.ext4` দিয়ে format করুন। তারপর `/mnt/test` directory-তে mount করুন এবং `df -h` দিয়ে verify করুন।

```bash
# Hint: এভাবে Loop device তৈরি করতে পারেন (real disk না থাকলে):
dd if=/dev/zero of=/tmp/testdisk.img bs=1M count=100
losetup /dev/loop0 /tmp/testdisk.img
mkfs.ext4 /dev/loop0
```

**Task 2:**
`blkid` দিয়ে সেই disk-এর UUID বের করুন এবং `/etc/fstab`-এ entry যোগ করুন। তারপর `mount -a` দিয়ে test করুন।

**Task 3:**
Mount করা disk-টা `umount` করার চেষ্টা করুন। তারপর সেই mount point-এ `cd` করে ভিতরে প্রবেশ করুন এবং আবার umount করার চেষ্টা করুন - কী error আসে দেখুন। `lsof` দিয়ে কারণ খুঁজে বের করুন।

---

## ⏭️ What's Next?

**Chapter 7 - Lesson 5: LVM - Logical Volume Manager**
> `pvcreate`, `vgcreate`, `lvcreate` disk management-এর সবচেয়ে powerful tool! Production server-এ disk extend করা, multiple disk একসাথে manage করা - সব শিখবো পরের lesson-এ। *Happy Learning* 🚀

<table width="100%">
  <tr>
    <td align="left">
      <a href="../03-File-System-Types">← File System Types</a>
    </td>
    <td align="right">
      <a href="../05-LVM-Logical-Volume-Manager">LVM - Logical Volume Manager →</a>
    </td>
  </tr>
</table>