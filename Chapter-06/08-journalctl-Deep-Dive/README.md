# Chapter 6 - Lesson 8: `journalctl` Deep Dive

**Chapter 6 | Lesson 8 of 8** | 🎉 Chapter 6 Complete!


## 🎯 এই Lesson-এ আমরা যা শিখবোঃ

- `journalctl` কী এবং কেন এটা এত powerful
- Logs filter করার সব উপায়
- Logs export করা (JSON, text)
- Persistent logs configure করা
- Real DevOps scenarios-তে কীভাবে ব্যবহার করবো


## `journalctl` কী?

মনে করেন আপনার system-এ সব কিছুর একটা diary আছে। প্রতিটা service কী করলো, কখন error হলো, কোন user কী করলো - সব কিছু লেখা থাকে। এই diary-র নাম হলো systemd journal, আর `journalctl` হলো সেই diary পড়ার tool।

```
systemd (PID 1)
    └── journald (daemon)
            └── /run/log/journal/  ← temporary (RAM)
            └── /var/log/journal/  ← persistent (disk)
```

> `journalctl` হলো একটা super-powered **search engine** আপনার system-এর সব log-এর জন্য।

## Basic Usage

### 1. সব logs দেখুন

```bash
journalctl
```

```
-- Logs begin at Mon 2024-01-15 10:00:00 UTC, end at Tue 2024-01-16 09:30:00 UTC. --
Jan 15 10:00:01 myserver systemd[1]: Started Session 1 of user root.
Jan 15 10:00:05 myserver sshd[1234]: Accepted publickey for root
Jan 15 10:05:12 myserver nginx[5678]: 2024/01/15 10:05:12 [error] 5678#0: ...
```

> এটা হাজার হাজার line দেখাবে। তাই আমরা filter ব্যবহার করবো।


### 2. সবচেয়ে নতুন logs আগে দেখুন

```bash
journalctl -r
```

`-r` মানে **reverse** - নতুনটা উপরে।

### 3. শুধু শেষের কিছু lines দেখুন

```bash
journalctl -n 20
```

```
# শেষের 20টা log entry দেখাবে
```

`-n` মানে **number of lines**। Default হলো 10।


### 4. Real-time logs follow করুন (সবচেয়ে বেশি ব্যবহৃত!)

```bash
journalctl -f
```

```
-- Logs begin at Mon 2024-01-15 10:00:00 UTC --
Jan 16 09:30:01 myserver sshd[9999]: Accepted password for deploy
Jan 16 09:30:05 myserver nginx[5678]: ... ← নতুন log আসতে থাকে
```

> এটা `tail -f` এর মতো কিন্তু অনেক বেশি powerful। DevOps-এ real-time debugging-এর সময় এটা সবচেয়ে বেশি দরকার।


## Filter করার উপায়

### 5. নির্দিষ্ট Service-এর logs দেখা

```bash
journalctl -u nginx
journalctl -u ssh
journalctl -u docker
```

**Syntax breakdown:**
```
journalctl -u <service_name>
           │
           └── -u = unit (systemd unit/service)
```

**Example output:**
```bash
journalctl -u nginx
```
```
-- Logs begin at Mon 2024-01-15 10:00:00 UTC --
Jan 15 10:00:10 myserver nginx[5678]: nginx: the configuration file /etc/nginx/nginx.conf syntax is ok
Jan 15 10:00:10 myserver nginx[5678]: nginx: configuration file /etc/nginx/nginx.conf test is successful
Jan 15 10:00:11 myserver systemd[1]: Started A high performance web server.
```


### 6. Multiple Services একসাথে দেখা

```bash
journalctl -u nginx -u mysql
```

দুটো service-এর logs একসাথে দেখাবে।


### 7. Real-time + নির্দিষ্ট Service

```bash
journalctl -u nginx -f
```

> এটা DevOps-এ সবচেয়ে বেশি ব্যবহার হয়! কোনো service deploy করার পরে তার real-time log দেখার প্রয়োজন পড়ে।


## Time-based Filtering

### 8. নির্দিষ্ট সময়ের logs দেখা

```bash
# আজকের logs
journalctl --since today

# গতকালের logs
journalctl --since yesterday

# নির্দিষ্ট সময় থেকে
journalctl --since "2024-01-15 10:00:00"

# সময়ের range
journalctl --since "2024-01-15 10:00:00" --until "2024-01-15 11:00:00"

# শেষ 1 ঘন্টার logs
journalctl --since "1 hour ago"

# শেষ 30 মিনিটের logs
journalctl --since "30 minutes ago"
```

**Real DevOps Example:**
```bash
# Production-এ রাত 2টায় কী হয়েছিল?
journalctl --since "2024-01-15 02:00:00" --until "2024-01-15 02:30:00"
```


## Priority/Level-based Filtering

Log-এর বিভিন্ন level আছে, যেমন সংবাদপত্রে breaking news আলাদা করা থাকে:

| Priority | Number | মানে |
|----------|--------|------|
| `emerg` | 0 | System crash হচ্ছে |
| `alert` | 1 | তাৎক্ষণিক action দরকার |
| `crit` | 2 | Critical condition |
| `err` | 3 | Error হয়েছে |
| `warning` | 4 | সতর্কতা |
| `notice` | 5 | সাধারণ কিন্তু গুরুত্বপূর্ণ |
| `info` | 6 | তথ্যমূলক |
| `debug` | 7 | Debug করার জন্য |

### 9. শুধু Error logs দেখা

```bash
# শুধু errors এবং তার চেয়ে বেশি critical
journalctl -p err

# শুধু warnings এবং উপরে
journalctl -p warning

# Number দিয়েও হয়
journalctl -p 3
```

**Example:**
```bash
journalctl -p err --since today
```
```
Jan 16 08:15:23 myserver mysql[1234]: [ERROR] Can't connect to local MySQL server
Jan 16 09:00:01 myserver nginx[5678]: open() "/var/www/html/index.html" failed (2: No such file or directory)
```

> **DevOps Tip:** Incident-এর সময় `journalctl -p err --since "1 hour ago"` দিয়ে শুধু errors দেখুন। হাজার হাজার info log-এর মধ্যে খুঁজতে হবে না।


### 10. Service + Error filter একসাথে

```bash
journalctl -u nginx -p err --since today
```

এটা দেখাবে - আজকের nginx-এর শুধু error logs।

## User ও PID-based Filtering

### 11. নির্দিষ্ট User-এর logs

```bash
# UID দিয়ে
journalctl _UID=1000

# Current user-এর logs
journalctl _UID=$(id -u)
```

### 12. নির্দিষ্ট PID-এর logs

```bash
journalctl _PID=1234
```

### 13. নির্দিষ্ট Program-এর logs

```bash
journalctl _COMM=nginx
journalctl _COMM=sshd
```


## Log Export করা

### 14. Plain Text-এ Export

```bash
journalctl -u nginx --since today > /tmp/nginx-logs.txt
```

### 15. JSON Format-এ Export (সবচেয়ে powerful!)

```bash
journalctl -u nginx -o json
```

```json
{"__REALTIME_TIMESTAMP":"1705395611000000","_HOSTNAME":"myserver","SYSLOG_IDENTIFIER":"nginx","MESSAGE":"server started","_PID":"5678"}
```

**Pretty JSON:**
```bash
journalctl -u nginx -o json-pretty
```
```json
{
    "__REALTIME_TIMESTAMP" : "1705395611000000",
    "_HOSTNAME" : "myserver",
    "SYSLOG_IDENTIFIER" : "nginx",
    "MESSAGE" : "server started",
    "_PID" : "5678"
}
```

> JSON format-এ export করলে পরে Python/bash script দিয়ে process করা যায়, অথবা **ELK Stack** (Elasticsearch, Logstash, Kibana)-এ পাঠানো যায়।

**Available Output Formats:**
```bash
journalctl -o short          # Default
journalctl -o short-precise  # Microsecond precision
journalctl -o verbose        # সব metadata সহ
journalctl -o json           # JSON format
journalctl -o json-pretty    # Pretty JSON
journalctl -o cat            # শুধু message, কোনো metadata নেই
```

## Persistent Logs Configure করা

### সমস্যাটা কী?

By default, অনেক system-এ journal logs RAM-এ (`/run/log/journal/`) রাখা হয়। System restart হলে সব logs মুছে যায়!

```bash
# Check করুন logs কোথায় আছে
ls /run/log/journal/    # temporary (RAM)
ls /var/log/journal/    # persistent (disk) - হয়তো নেই
```

### Persistent Logs চালু করা

**Step 1:** Directory তৈরি করুন
```bash
sudo mkdir -p /var/log/journal
```

**Step 2:** journald config file edit করুন
```bash
sudo nano /etc/systemd/journald.conf
```

**journald.conf-এ এই line খুঁজুন ও change করুন:**
```ini
[Journal]
Storage=persistent    # auto থেকে persistent করুন
```

**Storage options:**
| Option | মানে |
|--------|------|
| `auto` | `/var/log/journal/` থাকলে সেখানে, না হলে RAM-এ |
| `persistent` | সবসময় disk-এ |
| `volatile` | সবসময় RAM-এ |
| `none` | কোনো log রাখবে না |

**Step 3:** journald restart করুন
```bash
sudo systemctl restart systemd-journald
```

**Step 4:** Verify করুন
```bash
ls /var/log/journal/
```

## Disk Usage দেখা

```bash
# Journal কতটা disk নিচ্ছে?
journalctl --disk-usage
```

```
Archived and active journals take up 1.5G in the file system.
```

## Old Logs মুছে ফেলুন (Disk Management)

```bash
# 500MB-এর বেশি হলে বাকিটা মুছে ফেলা
sudo journalctl --vacuum-size=500M

# 7 দিনের পুরনো logs মুছে ফেলা
sudo journalctl --vacuum-time=7d

# শুধু 2টা archive file রাখা
sudo journalctl --vacuum-files=2
```

> DevOps-এ disk space কমে গেলে এই command কাজে আসে।


## journald.conf - Important Settings

```bash
sudo nano /etc/systemd/journald.conf
```

```ini
[Journal]
Storage=persistent          # persistent logs
Compress=yes                # logs compress করো
SystemMaxUse=1G             # সর্বোচ্চ 1GB disk ব্যবহার করবে
SystemKeepFree=500M         # disk-এ minimum 500MB free রাখবে
MaxRetentionSec=1month      # সর্বোচ্চ 1 মাসের logs রাখবে
MaxFileSec=1week            # প্রতি সপ্তাহে নতুন file
RateLimitIntervalSec=30s    # 30 সেকেন্ডে
RateLimitBurst=10000        # সর্বোচ্চ 10000 messages
```


## Journal-এ Grep করা

```bash
# নির্দিষ্ট keyword খুঁজুন
journalctl | grep "error"
journalctl -u nginx | grep "404"

# Case-insensitive
journalctl | grep -i "failed"
```


## Real DevOps Scenarios

### Scenario 1: Production Server-এ হঠাৎ Crash

```bash
# Step 1: শেষ কখন boot হয়েছে দেখুন
journalctl --list-boots
```
```
-2 abc123 Mon 2024-01-15 09:00:00 UTC-Mon 2024-01-15 23:00:00 UTC
-1 def456 Tue 2024-01-16 08:00:00 UTC-Tue 2024-01-16 09:00:00 UTC
 0 ghi789 Wed 2024-01-17 10:00:00 UTC-still running
```

```bash
# Step 2: Crash-এর আগের boot-এর logs দেখুন
journalctl -b -1          # আগের boot
journalctl -b -2          # তার আগের boot

# Step 3: সেই সময়ের errors দেখুন
journalctl -b -1 -p err
```

### Scenario 2: নতুন Service Deploy করা হয়েছে, কাজ করছে না

```bash
# Real-time logs দেখতে থাকুন
journalctl -u myapp -f

# অন্য terminal থেকে service start করুন
sudo systemctl start myapp

# Error দেখতে চাইলে
journalctl -u myapp -p err --since "5 minutes ago"
```

### Scenario 3: কেউ Unauthorized SSH Login করেছে কিনা চেক করুন

```bash
journalctl -u ssh --since today | grep "Failed password"
journalctl -u ssh --since today | grep "Invalid user"
```

### Scenario 4: Database Connection Issue Debug করুন

```bash
# MySQL/PostgreSQL logs দেখুন
journalctl -u mysql --since "1 hour ago" -p err

# Application + DB logs একসাথে
journalctl -u myapp -u mysql --since "30 minutes ago"
```

## সব Important Options - Quick Reference

```bash
# Basic
journalctl                    # সব logs
journalctl -r                 # Newest first
journalctl -n 50              # শেষের 50 lines
journalctl -f                 # Follow (real-time)

# Filter by unit
journalctl -u nginx           # nginx service
journalctl -u nginx -f        # nginx real-time

# Filter by time
journalctl --since today
journalctl --since yesterday
journalctl --since "1 hour ago"
journalctl --since "2024-01-15 10:00" --until "2024-01-15 11:00"

# Filter by priority
journalctl -p err             # errors
journalctl -p warning         # warnings
journalctl -p 0..3            # emerg থেকে err পর্যন্ত

# Boot-related
journalctl -b                 # Current boot
journalctl -b -1              # Previous boot
journalctl --list-boots       # সব boots-এর list

# Output format
journalctl -o json            # JSON
journalctl -o json-pretty     # Pretty JSON
journalctl -o cat             # শুধু message

# Disk management
journalctl --disk-usage
journalctl --vacuum-size=500M
journalctl --vacuum-time=7d
```

## 📝 Quick Summary

- `journalctl` হলো systemd-এর centralized log viewer
- `-f` দিয়ে real-time logs দেখা যায়
- `-u` দিয়ে নির্দিষ্ট service filter করা যায়
- `--since` / `--until` দিয়ে time range filter করা যায়
- `-p err` দিয়ে শুধু error logs দেখা যায়
- `-b -1` দিয়ে previous boot এর logs দেখা যায়
- Logs persistent করতে `/var/log/journal/` directory তৈরি করতে হয়
- `-o json` দিয়ে JSON format-এ export করা যায়
- `--vacuum-size` দিয়ে disk cleanup করা যায়


## 🏋️ Practice Tasks

**Task 1:**
SSH service-এর শেষ 30 মিনিটের logs দেখুন এবং শুধু error-level logs filter করুন।

**Task 2:**
`journalctl --list-boots` run করুন এবং current boot (`-b 0`) এর logs-এ শুধু errors দেখুন।

**Task 3:**
`/etc/systemd/journald.conf` file-এ `Storage=persistent` set করুন, `/var/log/journal/` directory তৈরি করুন এবং `journald` restart করার পরে `journalctl --disk-usage` দিয়ে verify করুন।


## অভিনন্দন আপনাকে! আপনি Chapter 6 সম্পূর্ণ করেছেন!

আপনি **Systemd & Service Management** chapter সফলভাবে শেষ করেছেন! এখন আপনি জানেন:

| ✅ | কী শিখলাম |
|----|-----------|
| ✅ | Systemd কী এবং কীভাবে কাজ করে |
| ✅ | Service start/stop/enable/disable |
| ✅ | Service status এবং logs দেখা |
| ✅ | Custom service unit file লেখা |
| ✅ | Systemd timers ব্যবহার করা |
| ✅ | Systemd targets বোঝা |
| ✅ | Service dependencies manage করা |
| ✅ | journalctl দিয়ে logs দেখা |

---

## ⏭️ What's Next

### Chapter 7 - Lesson 1: Disk Basics

> Block devices কী, `/dev/sda` মানে কী, partition কী, এবং MBR vs GPT - storage-এর একদম ভিত্তি থেকে শুরু করবো!

 *Happy Learning* 🚀

<table width="100%">
  <tr>
    <td align="left">
      <a href="../07-Service-Dependencies-And-Ordering">← Service Dependencies & Ordering</a>
    </td>
    <td align="right">
      <a href="../09-Assessment">Chapter 06 - Assessment →</a>
    </td>
  </tr>
</table>