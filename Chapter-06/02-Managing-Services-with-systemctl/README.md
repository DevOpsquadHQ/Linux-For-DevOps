# Chapter 6 - Lesson 2: Managing Services with systemctl

**Chapter 6 | Lesson 2 of 8**


## 🎯 এই Lesson-এ আমরা যা শিখবো

আগের Lesson-এ আমরা জেনেছি Systemd কী এবং Unit, Target কীভাবে কাজ করে। এখন আমরা শিখবো কীভাবে Services manage করতে হয় যেমন start, stop, restart, enable, disable ইত্যাদি এই সবকিছু `systemctl` দিয়ে।

DevOps কাজে প্রতিদিন এই commands ব্যবহার করতে হয়। Nginx চলছে কিনা দেখা, SSH restart করা, কোনো service enable করা ইত্যাদি সব `systemctl` দিয়েই হয়।


মনে করেন আপনার বাসায় একটা **Generator** আছে।

| Generator এর কাজ | systemctl এর কাজ |
|---|---|
| Generator চালু করা | `systemctl start` |
| Generator বন্ধ করা | `systemctl stop` |
| Generator রিস্টার্ট করা | `systemctl restart` |
| লোডশেডিং হলে auto-on করা | `systemctl enable` |
| Auto-on বন্ধ করা | `systemctl disable` |
| Generator এর status চেক | `systemctl status` |


## systemctl - The Main Command

`systemctl` হলো Systemd-এর remote control। এটা দিয়ে আপনি যেকোনো service/unit control করতে পারবেন।

**Basic Syntax:**
```bash
systemctl [command] [unit-name]
```

### 1. Service Start করা

```bash
systemctl start nginx
```

**কী করে:** Nginx web server এখনই চালু করে। কিন্তু মনে রাখবেন এটা শুধু এই মুহূর্তের জন্য। Server reboot হলে আবার বন্ধ হয়ে যাবে।

**Example:**
```bash
sudo systemctl start nginx
```
কোনো output আসবে না, মানে সফল হয়েছে

> **DevOps Tip:** Linux-এ "no news is good news" - কোনো error না আসলে বুঝবেন command কাজ করেছে।


### 2. Service Stop করা

```bash
systemctl stop nginx
```

**কী করে:** Service টা এখনই বন্ধ করে দেয়। Running processes gracefully terminate হয়।

```bash
sudo systemctl stop nginx
```

### 3. Service Restart করা

```bash
systemctl restart nginx
```

**কী করে:** Service বন্ধ করে, তারপর আবার চালু করে। Config file change করার পর এটা দরকার হয়।

```bash
sudo systemctl restart nginx
```

> ⚠️ **সাবধান:** `restart` করলে চলমান connections briefly cut হয়। Production-এ সাবধানে ব্যবহার করুন।


### 4. Service Reload করা

```bash
systemctl reload nginx
```

**কী করে:** Service বন্ধ না করে শুধু configuration file আবার পড়ে। Nginx, Apache এই ধরনের services-এ কাজ করে।

| Command | Service বন্ধ হয়? | Config Reload হয়? |
|---|---|---|
| `restart` | ✅ হ্যাঁ | ✅ হ্যাঁ |
| `reload` | ❌ না | ✅ হ্যাঁ |

> **DevOps Tip:** Production-এ সবসময় `reload` prefer করতে হবে যাতে users disconnect না হয়।


### 5. reload-or-restart

```bash
systemctl reload-or-restart nginx
```

**কী করে:** আগে `reload` চেষ্টা করে। যদি service `reload` support না করে, তাহলে `restart` করে। এটা সবচেয়ে safe option।


### 6. Service Status দেখা

```bash
systemctl status nginx
```

**কী করে:** Service এর সম্পূর্ণ অবস্থা দেখায়। Service চলছে কিনা, PID কত, শেষ কয়েকটা log line ইত্যাদি।

**Example Output:**
```
● nginx.service - A high performance web server
     Loaded: loaded (/lib/systemd/system/nginx.service; enabled; vendor preset: enabled)
     Active: active (running) since Sun 2026-03-15 10:00:00 UTC; 5min ago
       Docs: man:nginx(8)
    Process: 1234 ExecStartPre=/usr/sbin/nginx -t (code=exited, status=0/SUCCESS)
   Main PID: 1235 (nginx)
      Tasks: 2 (limit: 4915)
     Memory: 5.4M
     CGroup: /system.slice/nginx.service
             ├─1235 nginx: master process /usr/sbin/nginx
             └─1236 nginx: worker process

Mar 15 10:00:00 server systemd[1]: Starting A high performance web server...
Mar 15 10:00:00 server systemd[1]: Started A high performance web server.
```

**Output বুঝবো কীভাবে:**

| Field | মানে |
|---|---|
| `Loaded` | Unit file কোথায় আছে এবং boot-এ enable আছে কিনা |
| `Active: active (running)` | Service চলছে |
| `Active: inactive (dead)` | Service বন্ধ |
| `Active: failed` | Service crash করেছে |
| `Main PID` | Service এর Process ID |
| `Memory` | কতটুকু RAM ব্যবহার করছে |
| নিচের log lines | সাম্প্রতিক ঘটনা |

### 7. Service Enable করা (Boot-এ auto-start)

```bash
systemctl enable nginx
```

**কী করে:** Server reboot হলে automatically Nginx চালু হবে। এটা `/etc/systemd/system/` ফোল্ডারে একটা symbolic link তৈরি করে।

```bash
sudo systemctl enable nginx
# Output:
# Created symlink /etc/systemd/system/multi-user.target.wants/nginx.service
#   → /lib/systemd/system/nginx.service
```

> ⚠️ **মনে রাখুন:** `enable` মানে এখনই চালু হবে না, শুধু পরের reboot থেকে auto-start হবে।


### 8. Enable + Start একসাথে

```bash
systemctl enable --now nginx
```

**কী করে:** একই সাথে এখনই চালু করে এবং `boot-এ auto-start` set করে। DevOps-এ এটা সবচেয়ে বেশি ব্যবহার হয়।

```bash
sudo systemctl enable --now nginx
```


### 9. Service Disable করা

```bash
systemctl disable nginx
```

**কী করে:** Boot-এ auto-start বন্ধ করে। কিন্তু যদি এখন চলতে থাকে, তাহলে চলতেই থাকবে। 

```bash
sudo systemctl disable nginx
```

**Output:**
Removed /etc/systemd/system/multi-user.target.wants/nginx.service


### 10. Disable + Stop একসাথে

```bash
systemctl disable --now nginx
```

**কী করে:** এখনই বন্ধ করে + boot auto-start ও বন্ধ করে।

### 11. Service Mask করা (পুরোপুরি lock করা)

```bash
systemctl mask nginx
```

**কী করে:** Service কে এমনভাবে block করে যে কেউ manually-ও start করতে পারবে না। `/dev/null` এ link করে দেয়।

```bash
sudo systemctl mask nginx
```
**Output:** Created symlink ... → /dev/null

```bash
sudo systemctl start nginx
```
**Output:** Failed to start nginx.service: Unit is masked.

**Unmask করতে:**
```bash
sudo systemctl unmask nginx
```

> **Use Case:** এমন কোনো service যেটা security risk, সেটাকে mask করে রাখুন।


### 12. সব Services-এর List দেখা

শুধু active/running services দেখুন

```bash
systemctl list-units --type=service
```

সব services (inactive সহ) দেখুন
```bash
systemctl list-units --type=service --all
```

Boot-এ enabled services দেখুন
```bash
systemctl list-unit-files --type=service
```

**Example Output (list-unit-files):**
```
UNIT FILE                    STATE
nginx.service                enabled
ssh.service                  enabled
bluetooth.service            disabled
cups.service                 disabled
```

| State | মানে |
|---|---|
| `enabled` | Boot-এ auto-start হবে |
| `disabled` | Boot-এ start হবে না |
| `masked` | সম্পূর্ণ blocked |
| `static` | অন্য service এর dependency হিসেবে চলে |

### 13. Daemon Reload

```bash
systemctl daemon-reload
```

**কী করে:** আপনি যদি কোনো `.service` file তৈরি বা edit করেন, systemd কে সেটা জানাতে হবে। `daemon-reload` দিলে systemd সব unit files আবার পড়ে/রিড করে।

```bash
sudo systemctl daemon-reload
```

> **Rule:** যখনই কোনো `.service` file edit করবেন, edit করার পরেই `daemon-reload`, তারপর `restart`।


### সব Commands এক জায়গায়


```bash
# চালু/বন্ধ/রিস্টার্ট
sudo systemctl start nginx
sudo systemctl stop nginx
sudo systemctl restart nginx
sudo systemctl reload nginx
sudo systemctl reload-or-restart nginx

# Status দেখা
systemctl status nginx
systemctl is-active nginx      # শুধু "active" বা "inactive"
systemctl is-enabled nginx     # শুধু "enabled" বা "disabled"
systemctl is-failed nginx      # crash হয়েছে কিনা

# Boot management
sudo systemctl enable nginx
sudo systemctl disable nginx
sudo systemctl enable --now nginx
sudo systemctl disable --now nginx

# Lock করা
sudo systemctl mask nginx
sudo systemctl unmask nginx

# List দেখা
systemctl list-units --type=service
systemctl list-unit-files --type=service

# Config reload
sudo systemctl daemon-reload
```

## Real DevOps Scenario

**Scenario:** আপনি একটা Linux server-এ Nginx install করলেন। এখন কী করবেন?

```bash
# Step 1: Install করা
sudo apt install nginx

# Step 2: এখনই চালু করুন এবং boot-এ enable করুন
sudo systemctl enable --now nginx

# Step 3: Status check করুন
systemctl status nginx

# Step 4: Config change করলে reload করুন (restart না)
sudo systemctl reload nginx

# Step 5: কোনো issue হলে is-failed দিয়ে চেক করুন
systemctl is-failed nginx
```


## 📝 Quick Summary

- `start` → এখনই চালু করো
- `stop` → এখনই বন্ধ করো
- `restart` → বন্ধ করে আবার চালু করো (brief downtime)
- `reload` → বন্ধ না করে config reload করো (no downtime)
- `status` → service এর সম্পূর্ণ অবস্থা দেখো
- `enable` → boot-এ auto-start set করো
- `disable` → boot auto-start বন্ধ করো
- `mask` → সম্পূর্ণ block করো
- `daemon-reload` → unit file change করলে systemd কে জানাও
- `--now` flag → enable/disable এর সাথে ব্যবহার করলে একসাথে start/stop হয়


## 🏋️ Practice Tasks

এই tasks গুলো নিজে নিজে try করুন:

**Task 1:**
```bash
# SSH service এর status চেক করুন
systemctl status ssh

# এটা enabled আছে কিনা দেখুন
systemctl is-enabled ssh
```

**Task 2:**
```bash
# সব enabled services এর list দেখুন
systemctl list-unit-files --type=service | grep enabled
# কতটা enabled service আছে count করুন
systemctl list-unit-files --type=service | grep enabled | wc -l
```

**Task 3:**
```bash
# cron service কে restart করুন, তারপর status চেক করুন
sudo systemctl restart cron
systemctl status cron
```


## ⏭️ What's Next

**Chapter 6 - Lesson 3: Checking Service Status & Logs**
`systemctl status` এর বাইরে `journalctl` দিয়ে কীভাবে service logs দেখতে হয়, filter করতে হয়, এবং কোনো service কেন crash করলো সেটা diagnose করতে হয় এগুলো সব শিখবো। এটা DevOps troubleshooting-এর একটা core skill! *Happy Learning* 🚀

<table width="100%">
  <tr>
    <td align="left">
      <a href="../01-Service-Management-And-Systemd">← Service Management &amp; Systemd</a>
    </td>
    <td align="right">
      <a href="../03-Checking-Service-Status-And-Logs">Checking Service Status &amp; Logs →</a>
    </td>
  </tr>
</table>