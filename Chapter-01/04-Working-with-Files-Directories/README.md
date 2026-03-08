# Chapter 1 - Lesson 4: Working with Files & Directories

**Chapter 1 | Lesson 4 of 10**


## 🎯 আজকের Lesson-এ কী শিখবো?

আজকে আমরা শিখবো Linux-এ files ও directories নিয়ে কাজ করার সবচেয়ে গুরুত্বপূর্ণ commands গুলো। এগুলো হলো DevOps Engineer-দের **daily tools**। প্রতিদিন DevOps-এর কাজে এগুলো ব্যবহার হয়।

আজকের commands:
| Command | কাজ কী করে |
|---------|------------|
| `touch` | নতুন file তৈরি করে |
| `mkdir` | নতুন directory তৈরি করে |
| `cp` | file/directory copy করে |
| `mv` | file/directory move বা rename করে |
| `rm` | file/directory delete করে |
| `rmdir` | খালি (empty) directory delete করে |


## শুরু করার আগে

Linux-এর file system কে একটা **অফিসের filing cabinet** মনে করো।

- **Directory** = Cabinet-এর drawer বা folder
- **File** = সেই folder-এর ভেতরের কাগজ
- আজকে আমরা শিখবো কীভাবে নতুন ফাইল তৈরি করতে হয়, copy করতে হয়, সরাতে (move) হয়, এবং ফেলে (delete/remove) দিতে হয়।

## 1️⃣ `touch` - নতুন File তৈরি করে

### এটা কী করে?
`touch` command দিয়ে একটা **নতুন, খালি file** তৈরি করা যায়। এছাড়া যদি file আগে থেকেই থাকে, তাহলে সেটার **last modified time** আপডেট করে।

> অফিসে একটা নতুন খালি কাগজ বের করে টেবিলে রাখলে কিছু লেখা নেই, কিন্তু কাগজটা তৈরি।

### Syntax:
```bash
touch filename
```

### উদাহরণ:
```bash
touch hello.txt
```
```bash
# Output: (কোনো output দেখাবে না, কিন্তু file তৈরি হয়ে যাবে)
ls -l hello.txt
```
```
-rw-r--r-- 1 user user 0 Mar 4 10:00 hello.txt
```

### একসাথে অনেক file তৈরি:
```bash
touch file1.txt file2.txt file3.txt
```

### DevOps-এ কখন ব্যবহার করবে:
- Log file placeholder তৈরি করতে
- Script test করার জন্য dummy file তৈরি করতে
- Configuration file এর জায়গা আগে থেকে তৈরি রাখতে

## 2️⃣ `mkdir` নতুন Directory তৈরি করো

### এটা কী করে?
**mkdir = Make Directory।** নতুন folder/directory তৈরি করে।

> অফিসের cabinet-এ একটা নতুন drawer বা folder খোলা।

### Syntax:
```bash
mkdir directory_name
```

### উদাহরণ:
```bash
mkdir myproject
ls
```
```
myproject/
```

### Nested directories একসাথে তৈরি (`-p` flag):
```bash
mkdir -p /home/user/projects/devops/scripts
```
> `-p` মানে **parent** মাঝের সব directories যদি না থাকে, সেগুলোও তৈরি করে দেবে। এটা ছাড়া error দেবে।

```bash
# -p ছাড়া চেষ্টা করলে:
mkdir /home/user/projects/devops/scripts
# Error: cannot create directory: No such file or directory

# -p দিলে:
mkdir -p /home/user/projects/devops/scripts
# সব কিছু একসাথে তৈরি হয়ে যাবে
```

### একসাথে অনেক directory:
```bash
mkdir frontend backend database
```

### DevOps-এ কখন ব্যবহার করবে:
- Project structure তৈরি করতে
- Deployment folder তৈরি করতে
- Log directory, config directory তৈরি করতে

## 3️⃣ `cp` File বা Directory Copy করো

### এটা কী করে?
**cp = Copy**। একটা file বা directory-র **duplicate** তৈরি করে নতুন জায়গায়।

> একটা কাগজ photocopy করে অন্য drawer-এ রাখা। Original জায়গায় থাকে, নতুন জায়গায়ও যায়।

### Syntax:
```bash
cp source destination
```

### উদাহরণ ১ - File copy:
```bash
cp hello.txt /tmp/hello_backup.txt
```
```
# hello.txt এখনো আছে, /tmp/hello_backup.txt নামে একটা copy তৈরি হয়েছে
```

### উদাহরণ ২ - Same directory-তে rename করে copy:
```bash
cp hello.txt hello_copy.txt
```

### উদাহরণ ৩ - Directory copy (`-r` flag):
```bash
cp -r myproject/ /tmp/myproject_backup/
```
> `-r` মানে **recursive**। directory-র ভেতরের সব files ও sub-folders সহ copy করে।
> Directory copy করতে হলে `-r` **অবশ্যই** লাগবে।

### Common Flags:
| Flag | মানে | কখন ব্যবহার করবে |
|------|------|-----------------|
| `-r` | Recursive | Directory copy করতে |
| `-v` | Verbose | কী copy হচ্ছে দেখাবে |
| `-i` | Interactive | Overwrite করার আগে জিজ্ঞেস করবে |
| `-p` | Preserve | Permissions ও timestamps রক্ষা করে |

```bash
cp -rv myproject/ /tmp/myproject_backup/
```
```
'myproject/app.py' -> '/tmp/myproject_backup/app.py'
'myproject/config.yml' -> '/tmp/myproject_backup/config.yml'
```

### DevOps-এ কখন ব্যবহার করবে:
- Config file backup নেওয়ার আগে edit করতে
- Deployment-এর আগে current version backup রাখতে
- Template থেকে নতুন config তৈরি করতে


## 4️⃣ `mv` - Move বা Rename করো

### এটা কী করে?
**mv = Move**। দুটো কাজ করে:
1. File/directory **অন্য জায়গায় সরায়**
2. File/directory **rename করে**

> একটা কাগজ এক drawer থেকে তুলে অন্য drawer-এ রাখা (move), অথবা কাগজের উপরের নাম কেটে নতুন নাম লেখা (rename)।

### Syntax:
```bash
mv source destination
```

### উদাহরণ ১ - Rename করা:
```bash
mv hello.txt goodbye.txt
ls
```
```
goodbye.txt
# hello.txt আর নেই, goodbye.txt হয়ে গেছে
```

### উদাহরণ ২ - অন্য directory-তে move করা:
```bash
mv goodbye.txt /tmp/
ls /tmp/goodbye.txt
```
```
/tmp/goodbye.txt
# file এখন /tmp/ তে চলে গেছে
```

### উদাহরণ ৩ - Move করার সময় rename:
```bash
mv /tmp/goodbye.txt /home/user/final.txt
# /tmp/ থেকে তুলে /home/user/ তে নিয়ে গেছে এবং নামও বদলেছে
```

### উদাহরণ ৪ - Directory rename:
```bash
mv myproject myapp
# myproject folder-এর নাম myapp হয়ে গেছে
```

### Common Flags:
| Flag | মানে |
|------|------|
| `-i` | (interactive) Overwrite করার আগে জিজ্ঞেস করবে |
| `-v` | (verbose) কী হচ্ছে দেখাবে |
| `-n` | (No clobber) Already existing file overwrite করবে না |
| `-f` | (Force) জিজ্ঞেস না করে জোর করে overwrite করবে |

```
# দুটো file তৈরি করুন
echo "I am new" > new.txt
echo "I am old" > old.txt

# -n ছাড়া move — old.txt overwrite হয়ে যাবে
mv new.txt old.txt
cat old.txt

I am new   # পুরনো content চলে গেছে!
```
```
# আবার তৈরি করুন
echo "I am new" > new.txt
echo "I am old" > old.txt

# -n দিয়ে move — old.txt safe থাকবে
mv -n new.txt old.txt
cat old.txt

I am old   # পুরনো content অক্ষত আছে
```

### DevOps-এ কখন ব্যবহার করবে:
- Config file rename করতে (যেমন `.bak` extension দিতে)
- Files organized করতে সঠিক folder-এ সরাতে
- Old deployment folder rename করতে

> DevOps Tip: Production-এ important config files move করার সময় `-n` ব্যবহার করা ভালো অভ্যাস। ভুলে কোনো critical file overwrite হওয়ার ভয় থাকে না।

## 5️⃣ `rm` File বা Directory Delete

### এটা কী করে?
**rm = Remove**। File বা directory **permanently delete** করে।

> ⚠️ **সাবধান:** Linux-এ **Recycle Bin নেই**। `rm` করলে file চলে যাবে, ফেরত আনা **অনেক কঠিন বা অসম্ভব।**

> কাগজ ছিঁড়ে ফেলে দেওয়া। Recycle Bin-এ না, সরাসরি আবর্জনায়।

### Syntax:
```bash
rm filename
```

### উদাহরণ ১ - Single file delete:
```bash
rm hello.txt
```
```
# কোনো output নেই, file চলে গেছে
```

### উদাহরণ ২ - Confirm করে delete (`-i` flag):
```bash
rm -i important.txt
```
```
rm: remove regular file 'important.txt'? y
# y চাপলে delete হবে, n চাপলে হবে না
```

### উদাহরণ ৩ - Directory ও সব contents delete (`-r` flag):
```bash
rm -r myproject/
# myproject folder এবং তার ভেতরের সব কিছু চলে যাবে
```

### উদাহরণ ৪ - Force delete (confirmation ছাড়া):
```bash
rm -rf oldfolder/
```
> `-r` = recursive (directory সহ), `-f` = force (কোনো প্রশ্ন না করে)

### ⚠️ WARNING - এই Command কখনো ভুলেও run করবেন না:
```bash
rm -rf /          # পুরো system মুছে যাবে!
rm -rf /*         # সব files মুছে যাবে!
rm -rf ~/         # আপনার পুরো home directory মুছে যাবে!
```
> Production server-এ `rm -rf` চালানোর আগে **দুইবার ভাবা উচিৎ**। অনেক বড় বড় company এই ভুলে সব data হারিয়েছে।

### Common Flags:
| Flag | মানে |
|------|------|
| `-r` | Recursive - directory delete করতে |
| `-f` | Force - কোনো confirmation ছাড়া |
| `-i` | Interactive - প্রতিটার জন্য জিজ্ঞেস করবে |
| `-v` | Verbose - কী delete হচ্ছে দেখাবে |

### DevOps-এ কখন ব্যবহার করবেন:
- Old log files clean up করতে
- Temp files delete করতে
- Failed deployment folder সরাতে


## 6️⃣ `rmdir` - খালি/empty Directory Delete করে

### এটা কী করে?
**rmdir = Remove Directory**। শুধুমাত্র **সম্পূর্ণ খালি** বা empty directory delete করে। ভেতরে কিছু থাকলে error দেবে।

> একটা সম্পূর্ণ খালি drawer সরিয়ে ফেলা। ভেতরে কিছু থাকলে সরানো যাবে না।

### Syntax:
```bash
rmdir directory_name
```

### উদাহরণ:
```bash
mkdir emptydir
rmdir emptydir
# সফলভাবে delete হবে
```

```bash
mkdir notempty
touch notempty/file.txt
rmdir notempty
```
```
rmdir: failed to remove 'notempty': Directory not empty
# Error! কারণ ভেতরে file আছে
```

> **Practical Tip:** বাস্তবে `rmdir` কম ব্যবহার হয়। বেশিরভাগ সময় `rm -r` ব্যবহার করা হয়। তবে `rmdir` safe, ভুলে ভেতরের কিছু delete হওয়ার ভয় নেই।


## 🔄 সব Commands একসাথে - একটা Real DevOps Scenario

ধরুন আপনি একটা web server-এ নতুন deployment করছেন:

```bash
# Step 1: Project folder তৈরি করুন
mkdir -p /var/www/myapp/config
mkdir -p /var/www/myapp/logs

# Step 2: Config template copy করুন
cp /etc/nginx/nginx.conf /var/www/myapp/config/nginx.conf

# Step 3: Log file তৈরি করুন
touch /var/www/myapp/logs/access.log
touch /var/www/myapp/logs/error.log

# Step 4: Old backup rename করুন
mv /var/www/myapp_old /var/www/myapp_backup_2024

# Step 5: কাজ শেষে temp files মুছে ফেলুন
rm -rf /var/www/myapp/tmp/
```

## 📋 Quick Summary

- ✅ **`touch`** - নতুন খালি file তৈরি করে
- ✅ **`mkdir`** - নতুন directory তৈরি করে; `-p` দিয়ে nested directories
- ✅ **`cp`** - file/directory copy করে; directory-র জন্য `-r` লাগে
- ✅ **`mv`** - file move বা rename করে
- ✅ **`rm`** - file/directory delete করে। directory-র জন্য `-r`; সাবধানে ব্যবহার করবেন।
- ✅ **`rmdir`** - শুধু খালি directory delete করে
- ⚠️ **`rm -rf`** - শক্তিশালী কিন্তু বিপজ্জনক। সবসময় double-check করতে হবে।


## 🏋️ Practice Tasks

এই তিনটা task নিজে নিজে করার চেস্টা করুন:

**Task 1:**
```
১. আপনার home directory-তে "devops_practice" নামে একটা folder তৈরি করুন
২. তার ভেতরে "scripts" ও "configs" নামে আরো দুটো sub-folder তৈরি করুন
৩. scripts folder-এ "deploy.sh" ও "backup.sh" নামে দুটো file তৈরি করুন
```

**Task 2:**
```
১. deploy.sh ফাইলটা configs folder-এ copy করুন
২. তারপর সেই copy-র নাম বদলে "deploy_backup.sh" করুন
```

**Task 3:**
```
১. একটা "temp" নামে folder তৈরি করুন
২. তার ভেতরে কিছু dummy files তৈরি করুন
৩. পুরো temp folder টা delete করুন
৪. ls দিয়ে confirm করুন যে ডিলিট হয়েছে কিনা
```

---

## ⏭️ What's Next?

**Lesson 5: Viewing File Contents**
`cat`, `less`, `more`, `head`, `tail`, `tac` এই commands দিয়ে আপনি যেকোনো file-এর ভেতরে কী আছে দেখতে পারবেন। Log files দেখা, config check করা, সব কিছুতে এগুলো লাগে।
