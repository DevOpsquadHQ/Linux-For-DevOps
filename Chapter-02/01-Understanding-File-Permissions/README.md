# Chapter 2 - Lesson 1: Understanding File Permissions (rwx, Numeric & Symbolic Mode)

**Chapter 2 | Lesson 1 of 10**


## আজকের Lesson-এ কী শিখবো?

আজকে আমরা Linux-এর একটা **খুবই গুরুত্বপূর্ণ** বিষয় শিখবো **File Permissions**।

Linux-এ প্রতিটা file এবং directory-র একটা **permission system** আছে। এটা নির্ধারণ করে **কে কী করতে পারবে**।


Permissions বিষয়টা কি বাড়ির চাবির উদাহরণের মাধ্যমে বুঝার চেস্টা করি। একটা বাড়ি চিন্তা করুন। বাড়িতে তিন ধরনের মানুষ থাকতে পারে:

| মানুষ | Linux-এ কে? |
|---|---|
| বাড়ির মালিক | Owner (User) |
| মালিকের পরিবার | Group |
| বাইরের লোক | Others |

এখন বাড়ির মালিক ঠিক করে দেয় কে দরজা খুলতে পারবে, কে রান্নাঘরে ঢুকতে পারবে, কে কিছু পরিবর্তন করতে পারবে।

Linux-এও ঠিক একইভাবে প্রতিটা file-এ তিনটা গ্রুপের জন্য আলাদা আলাদা permission থাকে।

## Permission দেখার Command

```bash
ls -l
```

### Example Output:

```
-rwxr-xr-- 1 rahim developers 4096 Mar 4 10:00 script.sh
```

এই একটা লাইনেই অনেক তথ্য আছে। চলুন একটা একটা করে ভাঙি।

---

## Permission String বিশ্লেষণ

```
-rwxr-xr--
```

এই **10টা character**-কে এভাবে ভাগ করুন:

```
-   rwx   r-x   r--
↑    ↑     ↑     ↑
|   Owner  Group  Others
|
File Type
```

### প্রথম Character - File Type:

| Character | মানে |
|---|---|
| `-` | Regular file (সাধারণ file) |
| `d` | Directory (ফোল্ডার) |
| `l` | Symbolic Link (shortcut) |
| `b` | Block Device (যেমন hard disk) |
| `c` | Character Device (যেমন keyboard) |

---

### পরের ৯টা Character - Permissions:

এই ৯টা character তিনটা গ্রুপে ভাগ করা থাকে, প্রতিটা গ্রুপে 3টা করে character:

```
rwx  r-x  r--
 ↑    ↑    ↑
Owner Group Others
```

প্রতিটা গ্রুপে তিনটা permission থাকে:

| Symbol | Permission | মানে |
|---|---|---|
| `r` | **Read** | দেখতে পারবে / পড়তে পারবে |
| `w` | **Write** | পরিবর্তন করতে পারবে |
| `x` | **Execute** | চালাতে পারবে (script/program) |
| `-` | **No Permission** | এই permission নেই |

---

### সম্পূর্ণ উদাহরণ বিশ্লেষণ:

```
-rwxr-xr--
```

| অংশ | মানে |
|---|---|
| `-` | এটা একটা regular file |
| `rwx` | Owner পারবে: read ✅, write ✅, execute ✅ |
| `r-x` | Group পারবে: read ✅, write ❌, execute ✅ |
| `r--` | Others পারবে: read ✅, write ❌, execute ❌ |

---

## Numeric (Octal) Mode - সংখ্যায় Permission

Linux-এ permission কে সংখ্যা দিয়েও বোঝানো যায়। এটাকে বলে **Octal Mode**।

প্রতিটা permission-এর একটা মান আছে:

| Permission | মান |
|---|---|
| `r` (Read) | **4** |
| `w` (Write) | **2** |
| `x` (Execute) | **1** |
| `-` (None) | **0** |

### কিভাবে সংখ্যা বের করবেন?

প্রতিটা গ্রুপের (Owner, Group, Others) তিনটা permission-এর মান যোগ করুন।

### উদাহরণ:

```
rwx  =  4+2+1  =  7
r-x  =  4+0+1  =  5
r--  =  4+0+0  =  4
```

তাহলে `rwxr-xr--` = **754**

---

### Common Numeric Permissions চার্ট:

| Numeric | Symbolic | মানে |
|---|---|---|
| `777` | `rwxrwxrwx` | সবাই সব করতে পারবে (বিপজ্জনক!) |
| `755` | `rwxr-xr-x` | Owner সব পারবে, বাকিরা শুধু read+execute |
| `644` | `rw-r--r--` | Owner read+write, বাকিরা শুধু read |
| `600` | `rw-------` | শুধু Owner read+write করতে পারবে |
| `700` | `rwx------` | শুধু Owner সব করতে পারবে |
| `400` | `r--------` | শুধু Owner read করতে পারবে (read-only) |

---

## Directory-এর Permission মানে আলাদা!

File এবং Directory-তে `rwx`-এর মানে কিন্তু একটু আলাদা:

| Permission | File-এ মানে | Directory-তে মানে |
|---|---|---|
| `r` | File পড়তে পারবে | Directory-র ভেতরের list দেখতে পারবে (`ls`) |
| `w` | File edit করতে পারবে | ভেতরে file তৈরি/মুছতে পারবে |
| `x` | File চালাতে পারবে | Directory-তে **ঢুকতে পারবে** (`cd`) |

> **গুরুত্বপূর্ণ:** Directory-তে `x` না থাকলে `cd` দিয়ে ঢোকাই যাবে না!

---

## হাতে-কলমে দেখি

### সব file-এর permission দেখুন:
```bash
ls -l
```
**Output:**
```
-rw-r--r-- 1 rahim rahim  512 Mar 4 09:00 notes.txt
-rwxr-xr-x 1 rahim rahim 1024 Mar 4 09:05 deploy.sh
drwxr-xr-x 2 rahim rahim 4096 Mar 4 09:10 configs
```

### একটা নির্দিষ্ট file-এর permission দেখুন:
```bash
ls -l deploy.sh
```
**Output:**
```
-rwxr-xr-x 1 rahim rahim 1024 Mar 4 09:05 deploy.sh
```

### Directory-সহ সব কিছু বিস্তারিত দেখুন:
```bash
ls -la
```
> `-a` flag দিলে hidden files (.bashrc, .profile) গুলোও দেখা যায়।

**Output:**
```
drwxr-xr-x 5 rahim rahim 4096 Mar 4 09:00 .
drwxr-xr-x 3 root  root  4096 Mar 3 08:00 ..
-rw-r--r-- 1 rahim rahim  220 Mar 3 08:00 .bash_logout
-rw-r--r-- 1 rahim rahim  512 Mar 4 09:00 notes.txt
```

### `stat` দিয়ে বিস্তারিত permission দেখুন:
```bash
stat notes.txt
```
**Output:**
```
  File: notes.txt
  Size: 512       Blocks: 8     IO Block: 4096  regular file
Device: 801h/2049d  Inode: 131073  Links: 1
Access: (0644/-rw-r--r--)  Uid: ( 1000/  rahim)   Gid: ( 1000/  rahim)
```
> এখানে `(0644/-rw-r--r--)` - numeric এবং symbolic দুটোই একসাথে দেখাচ্ছে!

---

## `ls -l` Output-এর বাকি অংশ বোঝা

```
-rwxr-xr-x  1  rahim  developers  1024  Mar 4 09:05  deploy.sh
     ↑       ↑    ↑        ↑        ↑        ↑            ↑
Permission  Link Owner   Group     Size     Date        Filename
Count
```

| অংশ | মানে |
|---|---|
| `-rwxr-xr-x` | Permission string |
| `1` | Hard link count |
| `rahim` | File-এর Owner (মালিক) |
| `developers` | File-এর Group |
| `1024` | File-এর size (bytes-এ) |
| `Mar 4 09:05` | শেষ কখন modify হয়েছে |
| `deploy.sh` | File-এর নাম |

---

## DevOps-এ কেন এটা এত গুরুত্বপূর্ণ?

DevOps জীবনে আপনি প্রতিদিন permission নিয়ে কাজ করবেন:

- **`deploy.sh`** script-কে executable করতে হবে → `chmod +x deploy.sh`
- **SSH private key**-কে secure রাখতে হবে → `chmod 600 ~/.ssh/id_rsa`
- **Web server** শুধু নির্দিষ্ট folder পড়তে পারবে → `chmod 755 /var/www/html`
- **Config file**-এ password আছে → অন্য কেউ পড়তে পারবে না → `chmod 600 config.env`
- **Log files** যেন accidentally মুছে না যায় → permission restrict করো

---

## 📝 Quick Summary

- প্রতিটা file/directory-তে তিন ধরনের user থাকে: **Owner, Group, Others**
- তিন ধরনের permission: **r (4), w (2), x (1)**
- Symbolic mode: `rwxr-xr--` human-friendly
- Numeric (Octal) mode: `754` সংখ্যায়
- Directory-তে `x` মানে **ঢোকার permission,** file-এ `x` মানে **চালানোর permission**
- `ls -l` দিয়ে permission দেখা যায়, `stat` দিয়ে বিস্তারিত দেখা যায়
- SSH key-এর জন্য `600`, script-এর জন্য `755`, web files-এর জন্য `644` এগুলো common DevOps pattern

---

## 🏋️ Practice Tasks

এগুলো নিজে করে দেখুন:

**Task 1:**
```bash
# একটা নতুন file তৈরি করুন এবং তার permission দেখুন
touch myfile.txt
ls -l myfile.txt
stat myfile.txt
# Permission কত? Numeric mode-এ কত হবে সেটা বের করুন
```

**Task 2:**
```bash
# একটা directory তৈরি করুন এবং তার permission দেখুন
mkdir mydir
ls -ld mydir
# Directory-র permission কি file-এর মতো নাকি আলাদা?
```

**Task 3:**
```bash
# আপনার home directory-র সব file দেখুন (hidden সহ)
ls -la ~
# কোন file-এর permission কত? .ssh folder থাকলে তার permission কত?
```

## ⏭️ What's Next?

**Chapter 2 - Lesson 2: Changing Permissions (`chmod`, `chown`, `chgrp`)**

পরের lesson-এ আমরা শিখবো permission কিভাবে **পরিবর্তন করতে হয়।** `chmod` দিয়ে permission বাড়ানো/কমানো, `chown` দিয়ে file-এর মালিক পরিবর্তন করা ইত্যাদি সব হাতে-কলমে শিখবো! 🚀
