# Chapter 3 - Lesson 1: Process & Performance Management

**Chapter 3 | Lesson 1 of 10**


## Process কী? - সহজ ভাষায়

কল্পনা করুন আপনি একটা **রেস্টুরেন্ট** চালাচ্ছেন।

- **রেসিপি** = Program (disk-এ থাকা file)
- **রান্না করার কাজ** = Process (রেসিপি execute হচ্ছে)
- **রান্নাঘর** = RAM/CPU (যেখানে কাজ হচ্ছে)

অর্থাৎ **যখন কোনো program চালু হয়, সেটাই তখন Process হয়ে যায়।**

একই program থেকে **একাধিক process** তৈরি হতে পারে। যেমন আপনই একই সাথে দুটো terminal খুললে, দুটো আলাদা process চলবে।


## PID - Process ID

Linux-এ প্রতিটা process-কে একটা **unique number** দেওয়া হয়, এটাই **PID (Process ID)**। 

মনে করুন প্রতিটা process-এর একটা জাতীয় পরিচয়পত্র আছে। সেই ID নম্বরই হলো PID।


| PID | Process |
|-----|---------|
| 1 | systemd (সবার বাবা - প্রথম process) |
| 2 | kthreadd (kernel thread) |
| 1234 | আপনার bash session |
| 5678 | আপনার চালু করা nginx |

> **PID 1 সবসময় `systemd`** এটাই Linux boot হওয়ার পর প্রথম চালু হয়, এবং বাকি সব process এর থেকেই জন্ম নেয়।


## PPID - Parent Process ID

Linux-এ প্রতিটা process কোনো না কোনো **parent process থেকে জন্ম নেয়**।

```
systemd (PID 1)
    └── bash (PID 500)
            └── ls (PID 501)
```

- **PPID** = Parent Process-এর PID
- `ls` command রান করলে, bash তার **child process** তৈরি করে
- child মরে গেলে parent-কে জানায় তারপর parent সেটা clean করে

> এটা একটা **family tree** এর মতো। সব process-এর একটা বাবা আছে।


## Foreground vs Background Process

### Foreground Process

Foreground Process terminal আটকে রাখে, যার কারনে আমরা অন্য কোন কাজ করতে পারি না।

**উদাহরণ:**
```bash
sleep 100
```
এটা রান করলে terminal আটকে যাবে 100 সেকেন্ড পর্যন্ত।


### Background Process

Background Process terminal আটকায় না যার কারনে আমরা অন্য কাজ করতে পারি।

**উদাহরণ:**
```bash
sleep 100 &
```

শেষে `&` দিলে process **background-এ** চলে যায়।

Output-এ এমন কিছু আসবে:
```
[1] 4523
```
- `[1]` = job number
- `4523` = PID


## Process-এর Lifecycle (জীবনচক্র)

```
Created (তৈরি)
    ↓
Running (চলছে) ←→ Waiting (কিছুর জন্য অপেক্ষা)
    ↓
Stopped (থামানো হয়েছে)
    ↓
Zombie (কাজ শেষ কিন্তু parent এখনো clean করেনি)
    ↓
Terminated (সম্পূর্ণ শেষ)
```

### Process States সহজ ভাষায়:

| State | Code | মানে |
|-------|------|------|
| Running | R | এখন CPU-তে কাজ করছে |
| Sleeping | S | কিছুর জন্য অপেক্ষা করছে (I/O, event) |
| Stopped | T | কেউ থামিয়ে দিয়েছে (Ctrl+Z) |
| Zombie | Z | কাজ শেষ কিন্তু parent এখনো তুলে নেয়নি |
| Idle | I | Kernel thread, কিছু করছে না |


## Commands


### 1️⃣ `echo $$` - নিজের PID দেখা

**কী করে:** বর্তমান shell-এর PID দেখায়

```bash
echo $$
```

**Output:**
```
1842
```
> তোমার bash-এর PID হলো 1842

---

### 2️⃣ `echo $PPID` - Parent PID দেখা

**কী করে:** বর্তমান shell-এর parent-এর PID দেখায়

```bash
echo $PPID
```

**Output:**
```
1801
```
