# Chapter 1 - Final Assessment 

**Total 20 Practice Tasks**


> ⚠️ **নিয়ম:** প্রতিটা task নিজে terminal-এ try করতে হবে। উত্তর মুখস্থ না করে বুঝে বুঝে করার চেস্টা করুন। এর পরে এই এ্যাসাইনমেন্টের শেষে উত্তর দেয়া আছে সেখান থেকে মিলিয়ে দেখতে পারেন।


## Section A - Easy (Task 1–7) | File System & Navigation

### Task 1

আপনার home directory-তে যান এবং currently আপনি কোথায় আছেন সেটা print করুন।
কোন ২টা command use করবেন?

---

### Task 2

`/etc` directory-তে যান এবং সেখানকার সব file ও folder print করুন **hidden files সহ**, এবং **human-readable size সহ** (long format-এ)।

---

### Task 3

আপনার home directory-তে নিচের structure তৈরি করুন:

```
devops_practice/
├── scripts/
├── logs/
└── configs/
```

শুধুমাত্র একটি command দিয়ে এটা করুন।

---

### Task 4

`devops_practice/logs/` folder-এর ভেতরে এই ৩টা empty file তৈরি করুন একসাথে:

```
app.log   error.log   access.log
```

---

### Task 5

`/etc/hostname` file-এর content print করুন। তারপর `/etc/os-release` file-এর **শুধু প্রথম 5 লাইন** print করুন।

---

### Task 6

`/var/log` directory-তে যান। সেখানে `.log` extension আছেনে এমন সব file **find** করুন।

---

### Task 7

`cat` command ব্যবহার করে `/etc/passwd` file print করুন, কিন্তু শুধু **শেষ 3 লাইন** print করুন। (hint: একটাই command)

---

## Section B - Medium (Task 8–14) | Files, Permissions & Search

### Task 8

`devops_practice/configs/` folder-এ `nginx.conf` নামে একটা file তৈরি করুন।
তারপর সেটাকে `devops_practice/scripts/` folder-এ **copy** করুন।
তারপর original file-টা **rename** করুন `nginx.conf.bak` নামে।

---

### Task 9
এই command টা run করুন এবং explain করুন প্রতিটা column মানে কী:

```bash
ls -la /etc/passwd
```

Expected output কিছুটা এরকম হবে:
```
-rw-r--r-- 1 root root 2847 Jan 10 10:00 /etc/passwd
```

প্রতিটা part-এর মানে লেখো।

---

### Task 10

`/proc` directory সম্পর্কে ৩টা জিনিস বলুন:
- এটা কী ধরনের filesystem?
- এখানে কি real data store হয়?
- DevOps-এর কাজে এটা কোথায় কাজে লাগে?

(এটা theory task - command নয়)

---

### Task 11

আপনার system-এ `bash` binary টা কোথায় আছে, সেটা ৩টা আলাদা command দিয়ে বের করুন।

---

### Task 12

`devops_practice/logs/app.log` file-এ নিচের text টা **append** করুন (overwrite করবেন না):

```
Server started at port 8080
Database connected successfully
Error: timeout on connection
Server started at port 8080
```

তারপর শুধু **"Error"** শব্দ আছে এমন line গুলো filter করে print করুন।

---

### Task 13

`devops_practice/` folder টাকে **compress** করুন `.tar.gz` format-এ। Output file-এর নাম হবে: `devops_backup.tar.gz` তারপর সেই compressed file-এর ভেতরের contents list করুন, extract না করেই।

---

### Task 14

`/tmp` directory-তে যান এবং `testfile` নামে যেকোনো file খুজুন সম্পূর্ণ filesystem জুড়ে। (find command use করুন, permission error গুলো suppress করুন)

---

## Section C - Challenging (Task 15–20) | Real DevOps Thinking

### Task 15

নিচের command-টা explain করুন, প্রতিটা `|` এর পরে কী হচ্ছে step by step:

```bash
cat /etc/passwd | grep "/bin/bash" | cut -d: -f1 | sort
```

এটা কী output দিবে এবং কেন?

---

### Task 16

`devops_practice/logs/` directory-তে একটা **symbolic link** তৈরি করুন যেটা `/var/log` কে point করবেন। Link-এর নাম হবে `system_logs`।
তারপর verify করুন যে link টা সঠিকভাবে কাজ করছে।

---

### Task 17

এই scenario টা solve করুন:

> আপনার একটা log file আছে যেটা **real-time update** হচ্ছে। আপনি সেই file monitor করতে চান, নতুন line আসলে সাথে সাথে দেখতেও চান। কোন command use করবেন এবং কীভাবে?

Bonus: শুধু "ERROR" বা "WARN" আছে এমন line গুলো real-time-এ filter করে দেখাতে চাইলে কিভাবে দেখাবেন?

---

### Task 18

`/etc` directory-এ এমন সব file খুজুন যেগুলো:
- **শুধু root** read করতে পারে (permission: 600 বা 640)
- **Size 1KB এর বেশি**

(একটাই find command-এ করুন)

---

### Task 19

নিচের ৩টা directory-র পার্থক্য explain করুন। প্রতিটাতে কী ধরনের জিনিস থাকে এবং DevOps-এ কোনটা কখন কাজে লাগে:

| Directory | কী আছেনে? | DevOps use case |
|-----------|---------|-----------------|
| `/etc` | ? | ? |
| `/var` | ? | ? |
| `/tmp` | ? | ? |

---

### Task 20 ⭐ (Boss Task) এটা একটা real DevOps scenario:

> আপনার server-এ disk space কমে যাচ্ছে। আপনাকে খুঁজে বের করতে হবে:
> 1. কোন directory সবচেয়ে বেশি space নিচ্ছে (`/var` এর ভেতরে)
> 2. ১০০MB এর বেশি সব file খুঁজে বের করুন (entire system-এ)
> 3. `/tmp` folder কতটুকু space নিচ্ছে

তিনটা আলাদা command লিখুন।

---

## এখন অ্যাসেসমেন্ট মিলিয়ে নেয়ার পালা!

এই ২০টা task নিজে নিজে চেষ্টা করুন। যখন শেষ হবে অথবা কোথাও আটকে যাবেন google এ search দেন/AI কে বলেন। যদিও নিচে সবগুলোর উত্তর দেয়া আছে।

> **মনে রাখবেন:** এই test-এ fail করার কিছু নেই। এটা আপনার শেখাকে **solid** করার জন্য। চেষ্টা করাটাই সবচেয়ে গুরুত্বপূর্ণ! ভুলে গেলে অথবা মনে না পড়লে আবার ঐ নির্দিষ্ট লেসনে যান, একটা রিভিশন দিয়ে আসুন। প্র্যাকটিস করতে হবে অনেক অনেক বেশি।


## Answers for Final Assessment 

**Easy Tasks (1–7)**

### Task 1: Home directory-তে যান এবং location print করুন

```bash
cd ~
pwd
```

**Explanation:**
- `cd ~` → আপনাকে সরাসরি home directory-তে নিয়ে যায় (যেমন `/home/yourname`)
- `pwd` → Print Working Directory, আপনি এখন কোথায় আছেন সেটা দেখায়

**Expected Output:**

```
/home/yourname
```

> `cd` এবং `cd $HOME` এই দুটোও same কাজ করে!

---

### Task 2: /etc directory-তে hidden files সহ সব print করুন

```bash
cd /etc
ls -lah
```

**Explanation:**
- `-l` → long format (permissions, owner, size সব দেখায়)
- `-a` → hidden files সহ (যেগুলো `.` দিয়ে শুরু)
- `-h` → human-readable size (1K, 2M, 3G এভাবে)

**Expected Output (কিছুটা এরকম):**
```
total 1.4M
drwxr-xr-x 102 root root  4.0K Jan 10 10:00 .
drwxr-xr-x  20 root root  4.0K Jan 10 09:00 ..
-rw-r--r--   1 root root  3.0K Jan 10 10:00 adduser.conf
-rw-r--r--   1 root root   411 Jan 10 10:00 bash.bashrc
```

---

### Task 3: একটি command দিয়ে পুরো folder structure তৈরি করুন

```bash
mkdir -p ~/devops_practice/{scripts,logs,configs}
```

**Explanation:**
- `-p` → parent directory না থাকলে তৈরি করে দেয়, already থাকলে error দেয় না
- `{scripts,logs,configs}` → **brace expansion** একসাথে ৩টা folder তৈরি করে

**Verify করুন:**
```bash
tree ~/devops_practice
```

**Expected Output:**
```
devops_practice/
├── configs/
├── logs/
└── scripts/
```

---

### Task 4: একসাথে ৩টা empty file তৈরি করুন

```bash
touch ~/devops_practice/logs/app.log ~/devops_practice/logs/error.log ~/devops_practice/logs/access.log
```

**অথবা আরো smart way:**
```bash
touch ~/devops_practice/logs/{app,error,access}.log
```

**Verify করুন:**
```bash
ls ~/devops_practice/logs/
```

**Expected Output:**
```
access.log  app.log  error.log
```

> `touch` মূলত file-এর timestamp update করে কিন্তু file না থাকলে নতুন empty file তৈরি করে!

---

### Task 5: hostname print করুন এবং os-release এর প্রথম 5 লাইন

```bash
cat /etc/hostname
```

```bash
head -5 /etc/os-release
```

**Expected Output:**
```
# /etc/hostname
ubuntu-server

# head -5 /etc/os-release
PRETTY_NAME="Ubuntu 22.04.3 LTS"
NAME="Ubuntu"
VERSION_ID="22.04"
VERSION="22.04.3 LTS (Jammy Jellyfish)"
ID=ubuntu
```

> `head -n 5` এবং `head -5` দুটোই same কাজ করে!

---

### Task 6: /var/log এ .log extension এর সব file খুজুন

```bash
find /var/log -name "*.log"
```

**Explanation:**
- `find` → file খোঁজার command
- `/var/log` → কোথায় খুঁজবে
- `-name "*.log"` → যেসব file এর নাম `.log` দিয়ে শেষ

**Expected Output (কিছুটা এরকম):**
```
/var/log/syslog
/var/log/auth.log
/var/log/kern.log
/var/log/dpkg.log
```

---

### Task 7: /etc/passwd এর শেষ 3 লাইন print করুন

```bash
tail -3 /etc/passwd
```

**Expected Output:**
```
messagebus:x:103:106::/nonexistent:/usr/sbin/nologin
sshd:x:104:65534::/run/sshd:/usr/sbin/nologin
yourname:x:1000:1000::/home/yourname:/bin/bash
```

> `-3` মানে শেষের ৩ লাইন। `tail -f` দিয়ে real-time monitor করা যায়, এটা DevOps-এ খুব বেশি use হয়!

---

## Section B - Medium Tasks (8–14)

### Task 8: File তৈরি, copy এবং rename

```bash
# Step 1: file তৈরি করুন
touch ~/devops_practice/configs/nginx.conf

# Step 2: scripts folder-এ copy করুন
cp ~/devops_practice/configs/nginx.conf ~/devops_practice/scripts/

# Step 3: original file rename করুন
mv ~/devops_practice/configs/nginx.conf ~/devops_practice/configs/nginx.conf.bak
```

**Verify করুন:**
```bash
ls ~/devops_practice/configs/
ls ~/devops_practice/scripts/
```

**Expected Output:**
```
# configs/
nginx.conf.bak

# scripts/
nginx.conf
```

> `mv` command শুধু move না, same directory-তে use করলে rename হিসেবে কাজ করে!

---

### Task 9: ls -la output-এর প্রতিটা column explain করুন

```bash
ls -la /etc/passwd
```

**Output:**
```
-rw-r--r-- 1 root root 2847 Jan 10 10:00 /etc/passwd
```

**প্রতিটা Part-এর মানে:**

| Part | মানে |
|------|------|
| `-` | File type `-` মানে regular file, `d` মানে directory, `l` মানে link |
| `rw-` | **Owner** এর permission read ✅ write ✅ execute ❌ |
| `r--` | **Group** এর permission read ✅ write ❌ execute ❌ |
| `r--` | **Others** এর permission read ✅ write ❌ execute ❌ |
| `1` | Hard link count |
| `root` | File এর **owner** |
| `root` | File এর **group** |
| `2847` | File size (bytes-এ) |
| `Jan 10 10:00` | Last modified time |
| `/etc/passwd` | File এর নাম/path |

---

### Task 10: /proc directory সম্পর্কে ৩টা জিনিস

**১. এটা কী ধরনের filesystem?**
> `/proc` হলো একটা **Virtual Filesystem** এটা disk-এ exist করে না। Linux kernel এটাকে **RAM-এ তৈরি করে** রাখে। প্রতিবার boot হলে নতুন করে তৈরি হয়।

**২. এখানে কি real data store হয়?**
> না! এখানে কোনো real data store হয় না। এটা আসলে **kernel-এর window**। kernel তার ভেতরের information গুলো এখানে file হিসেবে দেখায় যাতে আপনি পড়তে পারেন।

**৩. DevOps কাজে কোথায় লাগে?**
```bash
cat /proc/meminfo      # RAM কতটুকু available
cat /proc/cpuinfo      # CPU কত core, কী model
cat /proc/1/status     # PID 1 (systemd) এর status
cat /proc/uptime       # System কতক্ষণ ধরে চলছে
```

> Monitoring tools যেমন `htop`, `free`, `top` এরা সবাই `/proc` থেকেই data নেয়!

---

### Task 11: bash binary কোথায় থাকে ৩টা command দিয়ে বের করুন

```bash
# Command 1
which bash

# Command 2
whereis bash

# Command 3
find / -name "bash" -type f 2>/dev/null
```

**Expected Output:**
```
# which bash
/usr/bin/bash

# whereis bash
bash: /usr/bin/bash /usr/share/man/man1/bash.1.gz

# find output
/usr/bin/bash
```

**পার্থক্য:**
| Command | কী দেখায় |
|---------|----------|
| `which` | শুধু executable-এর path (PATH variable থেকে) |
| `whereis` | Binary + manual page + source সব |
| `find` | পুরো filesystem জুড়ে খোঁজে |

---

### Task 12: File-এ text append করুন এবং filter করুন

```bash
# Text append করুন (>> মানে append, > মানে overwrite)
cat >> ~/devops_practice/logs/app.log << 'EOF'
Server started at port 8080
Database connected successfully
Error: timeout on connection
Server started at port 8080
EOF
```

**অথবা একটা একটা করে:**
```bash
echo "Server started at port 8080" >> ~/devops_practice/logs/app.log
echo "Database connected successfully" >> ~/devops_practice/logs/app.log
echo "Error: timeout on connection" >> ~/devops_practice/logs/app.log
echo "Server started at port 8080" >> ~/devops_practice/logs/app.log
```

**এখন শুধু "Error" line filter করুন:**
```bash
grep "Error" ~/devops_practice/logs/app.log
```

**Expected Output:**
```
Error: timeout on connection
```

> সবসময় মনে রাখবেন `>>` (double) **append** আর `>` (single) হলো **overwrite**, ভুলে গেলে পুরো file মুছে যাবে! এটা production-এ ভুল করলে অনেক বড় ক্ষতি হতে পারেে।

---

### Task 13: Folder compress করুন এবং contents list করুন

```bash
# Compress করুন
tar -czvf devops_backup.tar.gz ~/devops_practice/

# Contents list করুন (extract না করে)
tar -tzvf devops_backup.tar.gz
```

**Flag গুলোর মানে:**
| Flag | মানে |
|------|------|
| `-c` | Create - নতুন archive তৈরি করুন |
| `-z` | gzip দিয়ে compress করুন |
| `-v` | Verbose - process print করুন |
| `-f` | File - archive file-এর নাম দেয় |
| `-t` | list - contents print করুন |

**Expected Output (list করলে):**
```
drwxr-xr-x user/user    0 2024-01-10 devops_practice/
drwxr-xr-x user/user    0 2024-01-10 devops_practice/logs/
-rw-r--r-- user/user  150 2024-01-10 devops_practice/logs/app.log
```

---

### Task 14: testfile নামের file খুজুন, permission error suppress করুন

```bash
find / -name "testfile" 2>/dev/null
```

**Explanation:**
- `find /` → root থেকে পুরো filesystem খুজুন
- `-name "testfile"` → এই নামের file খুজুন
- `2>/dev/null` → **stderr** (error messages) গুলো `/dev/null` এ পাঠান, মানে suppress করুন

> `2>/dev/null` এটা DevOps scripting-এ অনেক বেশি use হয়। `2>` মানে stderr redirect। `/dev/null` হলো Linux-এর **black hole**, এখানে যা পাঠাবেন সব হারিয়ে যাবে অর্থাৎ কোথাও কোন রেকর্ড থাকবে না!

---

## Section C - Challenging Tasks (15–20)

### Task 15: Command pipeline step by step explain করুন

```bash
cat /etc/passwd | grep "/bin/bash" | cut -d: -f1 | sort
```

**Step by step breakdown:**

```
Step 1: cat /etc/passwd
```
> পুরো `/etc/passwd` file-এর content দেখায়। প্রতি লাইনে একজন user-এর info থাকে।

```
Step 2: | grep "/bin/bash"
```
> আগের output থেকে শুধু সেই লাইন গুলো রাখুন যেখানে `/bin/bash` আছে, মানে যেসব user bash shell use করে।

```
Step 3: | cut -d: -f1
```
> প্রতি লাইনকে `:` দিয়ে split করুন এবং শুধু ১ম field নেন, মানে শুধু username।
> (`-d:` মানে delimiter হলো colon, `-f1` মানে field 1)

```
Step 4: | sort
```
> Username গুলো alphabetically sort করুন।

**Expected Output:**
```
root
yourname
```

> এটাই **Unix Philosophy** ছোট ছোট command একসাথে জুড়ে powerful কাজ করে!

---

### Task 16: Symbolic link তৈরি করুন এবং verify করুন

```bash
# Symbolic link তৈরি করুন
ln -s /var/log ~/devops_practice/logs/system_logs

# Verify করুন - link print করুন
ls -la ~/devops_practice/logs/

# Link কাজ করছে কিনা দেখো
ls ~/devops_practice/logs/system_logs/
```

**Expected Output:**
```
# ls -la দিলে
lrwxrwxrwx 1 user user 8 Jan 10 system_logs -> /var/log

# ls system_logs/ দিলে /var/log এর contents দেখাবে
auth.log  syslog  kern.log  dpkg.log ...
```

**`l` মানে link** `ls -la` output-এ প্রথম character `l` হলে বুঝবেন এটা symbolic link।

> Real DevOps use case: অনেক application fixed path-এ log লেখে। Symbolic link দিয়ে সেটাকে অন্য জায়গায় redirect করা যায়, actual folder না সরিয়েই!

---

### Task 17ঃ Real-time log monitoring

**Basic real-time monitoring:**
```bash
tail -f ~/devops_practice/logs/app.log
```

**শুধু ERROR বা WARN filter করে real-time print করুন:**
```bash
tail -f ~/devops_practice/logs/app.log | grep --line-buffered -E "ERROR|WARN"
```

**Explanation:**
| Part | মানে |
|------|------|
| `tail -f` | file-এর শেষে wait করে, নতুন line আসলে সাথে সাথে দেখায় |
| `--line-buffered` | প্রতি line আসার সাথে সাথে process করুন (pipe-এ দরকার) |
| `-E "ERROR\|WARN"` | Extended regex, ERROR অথবা WARN যেকোনোটা match করুন |

**Multiple files একসাথে monitor করতে:**
```bash
tail -f /var/log/syslog /var/log/auth.log
```

> এটা DevOps-এর সবচেয়ে বেশি use হওয়া command গুলোর একটা। Production-এ কোনো issue হলে প্রথমেই `tail -f` দিয়ে log দেখা হয়!

---

### Task 18: Permission ও size দিয়ে file খুজুন

```bash
find /etc -maxdepth 1 \( -perm 600 -o -perm 640 \) -size +1k 2>/dev/null
```

**Explanation:**
| Part | মানে |
|------|------|
| `find /etc` | `/etc` এ খুজুন |
| `-maxdepth 1` | শুধু এই directory-তে (sub-folder-এ না) |
| `\( ... \)` | Grouping - এই condition গুলো একসাথে |
| `-perm 600` | Exactly 600 permission |
| `-o` | OR operator |
| `-perm 640` | Exactly 640 permission |
| `-size +1k` | 1KB এর বেশি |
| `2>/dev/null` | Permission error suppress |

**Expected Output (কিছুটা এরকম):**
```
/etc/shadow
/etc/gshadow
/etc/ssl/private/ssl-cert-snakeoil.key
```

---

### Task 19: /etc, /var, /tmp এর পার্থক্য

| Directory | কী আছেনে? | DevOps Use Case |
|-----------|---------|-----------------|
| `/etc` | System-এর সব **configuration files** যেমন software settings, network config, user info | Nginx config, SSH config, DNS settings পরিবর্তন করতে |
| `/var` | **Variable data**: যেসব data সময়ের সাথে বদলায় যেমন logs, databases, mail queue, package cache | Log analysis, disk space monitoring, database files |
| `/tmp` | **Temporary files**: reboot হলে মুছে যায়, সব user write করতে পারে | Script-এ temporary file রাখা, build artifacts, short-lived data |

**সহজ analogy:**
- `/etc` = আপনার **settings/preferences**: stable, rarely changes
- `/var` = আপনার **diary**: প্রতিদিন নতুন কিছু লেখা হয়
- `/tmp` = **rough paper**: কাজ শেষে ফেলে দেওয়া হয়

---

### Task 20: ⭐ Boss Task: Disk Space Investigation

```bash
# 1. /var এর ভেতরে কোন directory সবচেয়ে বেশি space নিচ্ছে
du -sh /var/* 2>/dev/null | sort -rh | head -10
```

```bash
# 2. Entire system-এ 100MB এর বেশি সব file খুজুন
find / -type f -size +100M 2>/dev/null
```

```bash
# 3. /tmp folder কতটুকু space নিচ্ছে
du -sh /tmp
```

**Explanation:**

**Command 1:**
| Part | মানে |
|------|------|
| `du -sh` | **D**isk **U**sage - `-s` summary, `-h` human readable |
| `/var/*` | /var এর প্রতিটা item |
| `sort -rh` | Human-readable size অনুযায়ী reverse sort (বড় আগে) |
| `head -10` | শুধু top 10 print করুন |

**Expected Output:**
```
2.1G    /var/log
1.4G    /var/cache
850M    /var/lib
120M    /var/backups
```

**Command 2 Expected Output:**
```
/var/log/large_app.log
/home/user/ubuntu.iso
/opt/database/data.db
```

**Command 3 Expected Output:**
```
45M     /tmp
```

> এই তিনটা command মুখস্থ করে রাখুন, Production server-এ disk full হলে এগুলোই আপনাকে বাঁচাবে!

---

## 🏆 Chapter 1 - Complete! Congratulations

```
╔════════════════════════════════════════╗
║   Chapter 1: Linux Fundamentals        ║
║   Status: ✅ COMPLETE                  ║
║   Lessons: 10/10 Done                  ║
║   Assessment: 20/20 Tasks Covered      ║
╚════════════════════════════════════════╝
```

### 👉 [Chapter 2 - Lesson 1: Understanding File Permissions](https://github.com/munirmahmud/Linux-For-DevOps/tree/main/Chapter-02/01-Understanding-File-Permissions)

Linux-এ Security-র ভিত্তি হলো Permissions।
কোন user কোন file পড়তে, লিখতে বা run করতে পারবে, সেটা কীভাবে কাজ করে সেটা শিখবো।
