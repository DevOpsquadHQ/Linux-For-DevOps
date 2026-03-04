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

