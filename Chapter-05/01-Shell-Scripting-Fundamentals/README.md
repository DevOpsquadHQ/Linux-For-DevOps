# Chapter 5 - Lesson 1: What is Shell Scripting?

**Chapter 5 | Lesson 1 of 11**


🎉 অভিনন্দন! আপনি Chapter 5-এ পৌঁছে গেছেন!

Chapter 1 থেকে 4 পর্যন্ত আপনি Linux-এর fundamentals, permissions, processes, এবং networking শিখেছেন। এখন সবচেয়ে exciting part "Shell Scripting & Automation" শুরু হচ্ছে!

এই chapter-টা আপনাকে একজন real DevOps Engineer বানাবে কারণ scripting ছাড়া DevOps অসম্পূর্ণ।


## Shell Scripting কী?

ধরুন আপনি প্রতিদিন সকালে একই রকম breakfast তৈরি করেন যেমন ডিম সিদ্ধ করা, চা বানানো, টোস্ট রেডি করা ইত্যাদি। প্রতিদিন একই কাজ বারবার করতে হয়। এখন যদি আপনি একটা recipe card লিখে রাখেন, এবং বলেন "এই card follow করো" - তাহলে যেকেউ (বা যেকোনো machine) সেই কাজগুলো automatically করতে পারবে।

**Shell Script = সেই recipe card।** আপনি Linux commands-গুলো একটা file-এ লিখে রাখেন, তারপর একবার run করলেই সব automatically হয়ে যায়।


## Shell কী?

Shell হলো আপনার এবং Linux kernel-এর মধ্যে মধ্যস্থতাকারী (interpreter)।


আপনি (human) → Shell → Linux Kernel → Hardware


আপনি যখন terminal-এ `ls` বা `cd` লিখেন, Shell সেটা বুঝে kernel-কে বলে কী করতে হবে।

### Shell-এর types:

| Shell | Full Name | বিশেষত্ব |
|-------|-----------|----------|
| `sh` | Bourne Shell | সবচেয়ে পুরনো, basic |
| `bash` | Bourne Again Shell | সবচেয়ে popular, feature-rich |
| `zsh` | Z Shell | macOS default, extra features |
| `fish` | Friendly Interactive Shell | beginner-friendly |
| `dash` | Debian Almquist Shell | fast, lightweight |

> **DevOps-এ সবচেয়ে বেশি ব্যবহার হয় `bash`।** আমরা এই পুরো chapter-এ bash নিয়ে কাজ করবো।


## Shell Script দেখতে কেমন?

একটা simple shell script এইরকম দেখতে:

```bash
#!/bin/bash
# This is my first script

echo "Hello, DevOps World!"
echo "Today's Date: $(date)"
echo "Who am I: $(whoami)"
```

**Output:**
```
Hello, DevOps World!
Today's Date: Sun Mar 15 10:30:00 UTC 2026
Who am I: munir
```

## Shebang (`#!`) - সবচেয়ে গুরুত্বপূর্ণ line

Script-এর একদম প্রথম line-এ থাকে:

```bash
#!/bin/bash
```

এটাকে বলে **Shebang** (বা Hashbang)।

### Shebang কেন দরকার?

Shebang দরকার কারণ আপনি যখন `./script.sh` দিয়ে একটা file run করেন, তখন Linux নিজে সেই file-টা পড়তে পারে না। তার একটা program দরকার যেটা দিয়ে সে file-টা execute করবে।

Shebang সেই program-এর **ঠিকানা** দিয়ে দেয়।

```bash
#!/bin/bash     → bash দিয়ে run করো
#!/bin/sh       → sh দিয়ে run করো
#!/usr/bin/python3  → Python দিয়ে run করো
```

`#!` মানে "এই interpreter দিয়ে এই script চালাও"। Shebang না থাকলে Linux confuse হয়ে যায় "কোন program দিয়ে চালাবো?"


## প্রথম Script লেখা - Step by Step

### Step 1: একটা file তৈরি করুন

```bash
vim hello.sh
```

> `.sh` extension দেওয়া **mandatory না**, কিন্তু এটা convention। দেখলেই যাতে বোঝা যায় এটা shell script।

### Step 2: Script লেখা

(hash) # দিয়ে শুরু হওয়া line = comment, execute হয় না

```bash
#!/bin/bash
# This is my first shell script

echo "=============================="
echo "  My first Shell Script"
echo "=============================="
echo ""
echo "Hostname: $(hostname)"
echo "Current User: $(whoami)"
echo "Current Date: $(date)"
echo "Current Directory: $(pwd)"
echo ""
echo "Script Done!"
```

`:wq` দিয়ে save করে বের হয়ে আসুন।

### Step 3: Script দেখা

```bash
cat hello.sh
```

**Output:**
```
#!/bin/bash
# This is my first shell script
...
```

## Script Execute করার Permission দেওয়া

Script লেখার পর সরাসরি run করলে error আসবে:

```bash
./hello.sh
```
```
bash: ./hello.sh: Permission denied
```

কারণ নতুন file-এ default-এ execute permission থাকে না।

### Permission দিন:

```bash
chmod +x hello.sh
```

এখন permission check করুন:
```bash
ls -l hello.sh
```
```
-rwxr-xr-x 1 munir munir 245 Mar 15 10:30 hello.sh
```

`x` দেখা যাচ্ছে, এখন execute করা যাবে।


## Script Execute করার ৩টি উপায়

### উপায় ১: `./script.sh` (সবচেয়ে common)

```bash
./hello.sh
```

> `./` মানে "current directory থেকে এই file রান করুন"

**Output:**
```
==============================
  My first Shell Script
==============================

Hostname: devops-server
Current User: munir
Current Date: Sun Mar 15 10:30:00 UTC 2026
Current Directory: /home/munir

Script Done!
```

### উপায় ২: `bash script.sh` (permission না থাকলেও চলে)

```bash
bash hello.sh
```

এই উপায়ে execute permission না থাকলেও script run হবে। কারণ আপনি explicitly `bash` দিয়ে রান করেছেন।

### উপায় ৩: `source script.sh` বা `. script.sh`

```bash
source hello.sh
# অথবা
. hello.sh
```

> এটা একটু আলাদা - এই উপায়ে script "current shell-এর মধ্যেই" run হয়। Variables এবং changes current session-এ থাকে।

### তিনটার পার্থক্য এক নজরে:

| উপায় | Execute Permission লাগে? | নতুন Sub-shell তৈরি হয়? |
|-------|--------------------------|--------------------------|
| `./hello.sh` | ✅ হ্যাঁ | ✅ হ্যাঁ |
| `bash hello.sh` | ❌ না | ✅ হ্যাঁ |
| `source hello.sh` | ❌ না | ❌ না (same shell) |


## Bash vs Sh - কোনটা ব্যবহার করবো?

```bash
#!/bin/bash   ← এটাই সবসময় ব্যবহার করুন
#!/bin/sh     ← এটা limited features দেয়
```

> DevOps কাজে সবসময় `#!/bin/bash` লিখুন।

আপনার system-এ bash কোথায় আছে দেখুন:

```bash
which bash
```
```
/bin/bash
```


## Comments - Script-এ Notes লেখা

```bash
#!/bin/bash

# এটা single-line comment
# এই line execute হবে না

echo "Hello"  # এটাও comment - line-এর শেষে

# Multi-line comment এভাবে লেখা হয়:
# প্রতিটা line-এ # দিতে হয়
# কোনো shortcut নেই bash-এ
```

> **ভালো DevOps engineer সবসময় comment লেখে** ৬ মাস পরে নিজেই script বুঝতে পারবে না comment ছাড়া!


## `echo` command - Output দেখানো

Shell scripting-এ সবচেয়ে বেশি ব্যবহৃত command হলো `echo`:

```bash
echo "Simple text"
echo "Today is: $(date)"            # Command substitution
echo "Home directory: $HOME"        # Variable
echo -n "No new line in this line"  # -n flag
echo -e "Line1\nLine2\nLine3"       # -e flag: escape characters বোঝে
```

**Output:**
```
Simple text
Today is: Sun Mar 15 10:30:00 UTC 2026
Home directory: /home/munir
No new line in this line
Line1
Line2
Line3
```


## একটা Real DevOps Script দেখি

ধরুন আপনাকে প্রতিদিন server-এর basic info check করতে হয়:

```bash
#!/bin/bash
# Script: system_info.sh
# Purpose: Server-এর basic health information দেখায়
# Author: DevOps Engineer
# Date: 2026-03-15

echo "======================================="
echo "       SYSTEM INFORMATION REPORT"
echo "======================================="
echo ""

echo "Hostname    : $(hostname)"
echo "Logged User : $(whoami)"
echo "Date & Time : $(date '+%Y-%m-%d %H:%M:%S')"
echo "Uptime      : $(uptime -p)"
echo ""

echo "--- CPU Info ---"
echo "CPU Model : $(lscpu | grep 'Model name' | cut -d: -f2 | xargs)"
echo ""

echo "--- Memory Usage ---"
free -h | grep Mem
echo ""

echo "--- Disk Usage ---"
df -h / | tail -1
echo ""

echo "--- Last 3 Logged In Users ---"
last -n 3
echo ""

echo "======================================="
echo "  Report Generated Successfully"
echo "======================================="
```

এই script একবার run করলেই আপনার server-এর সব important info এক জায়গায় দেখাবে - manually কোনো command চালাতে হবে না!

## Script কোথায় রাখবো?

| Location | কখন ব্যবহার করবো |
|----------|-----------------|
| `/home/username/scripts/` | Personal scripts |
| `/usr/local/bin/` | সব user-এর জন্য available করতে হলে |
| `/opt/scripts/` | Team বা application-specific scripts |
| `/etc/cron.d/` | Scheduled scripts |

> নিজের জন্য একটা `~/scripts/` folder তৈরি করুন এবং সব script সেখানে রাখুন।


## Quick Summary

- **Shell** হলো আপনার এবং Linux kernel-এর মধ্যে bridge
- **Bash** সবচেয়ে popular shell - DevOps-এ এটাই ব্যবহার হয়
- **Shebang (`#!/bin/bash`)** script-এর প্রথম line-এ থাকে যা কোন interpreter ব্যবহার করতে হবে তা বলে দেয়
- Script run করার আগে **`chmod +x`** দিয়ে পারমিশন চেঞ্জ করতে হয়
- Script execute করার ৩ উপায়: `./script.sh`, `bash script.sh`, `source script.sh`
- **`#`** দিয়ে comment লেখা হয় যা execute হয় না
- **`echo`** দিয়ে output দেখানো হয়
- **`$(command)`** দিয়ে command-এর output script-এর মধ্যে use করা যায়


## 🏋️ Practice Tasks

**Task 1:**
একটা `my_info.sh` script লিখুন যেটা run করলে দেখাবে:
- আপনার username
- আপনার home directory
- Current date ও time
- আপনি কোন shell ব্যবহার করছেন (`echo $SHELL`)

**Task 2:**
Script-টাতে `chmod +x` দিন এবং তিনটা উপায়ে (`./`, `bash`, `source`) run করুন - পার্থক্য observe করুন।

**Task 3:**
`/usr/local/bin/` directory-তে আপনার script copy করুন এবং যেকোনো জায়গা থেকে শুধু নাম দিয়ে run করার চেষ্টা করুন:
```bash
sudo cp my_info.sh /usr/local/bin/myinfo
sudo chmod +x /usr/local/bin/myinfo
myinfo   # যেকোনো জায়গা থেকে run করুন
```

---

## ⏭️ What's Next?

**Chapter 5 - Lesson 2: Variables, Data Types & User Input**

পরের lesson-এ শিখবো:
- Script-এ variables কীভাবে declare ও use করতে হয়
- User থেকে input নেওয়া যায় (`read` command)
- Special variables (`$0`, `$1`, `$#`, `$?`, `$$`)
- Environment variables vs Local variables

Shell scripting-এর সবচেয়ে interesting part শুরু হচ্ছে। variables ছাড়া কোনো script কাজে লাগে না! *Happy Learning* 🚀


<table width="100%">
  <tr>
    <td align="left">
      <a href="../../12-Assessment">← Chapter 4 - Assessment</a>
    </td>
    <td align="right">
      <a href="../02-Variables-Data-Types-N-User-Input">Variables, Data Types & User Input →</a>
    </td>
  </tr>
</table>