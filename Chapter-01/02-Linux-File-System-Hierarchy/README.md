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
