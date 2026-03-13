# Chapter 2 - Lesson 8: `sudo` & the Sudoers File

**Chapter 2 | Lesson 8 of 10**


## 🎯 এই Lesson-এ কী শিখবো?

- `sudo` কী এবং কেন দরকার
- `sudo` vs `su` - পার্থক্য কী?
- `/etc/sudoers` file কী?
- `visudo` দিয়ে sudoers edit করা
- Sudo rules লেখার syntax
- Real-world DevOps-এ sudo কীভাবে ব্যবহার হয়


## Analogy দিয়ে বুঝি আগে

কল্পনা করুন আপনি একটা বড় office-এ কাজ করেন।

- **Regular employee (normal user)** → শুধু নিজের desk-এর কাজ করতে পারে
- **Manager (root user)** → পুরো office-এর সব কিছু access করতে পারে
- **`sudo`** → Boss আপনাকে বললো, *"এই কাজটা আপনি আমার permission-এ করতে পারবেন"* - মানে temporarily manager-এর power পেলেন, কিন্তু আপনি manager হয়ে গেলে না!

এটাই হলো `sudo` - **Superuser DO**।


## `sudo` কী?

`sudo` মানে হলো **"Superuser Do"**।

একটা normal user যখন কোনো command-এর আগে `sudo` লেখে, তখন সেই command টা **root (admin) permission** দিয়ে run হয়।

```bash
sudo apt update
```

এই command টা normally root ছাড়া রান হয় না কিন্তু `sudo` দিলে রান হয়।


## `sudo` vs `su` - পার্থক্য

| বিষয় | `sudo` | `su` |
|---|---|---|
| মানে | Superuser Do | Switch User |
| কী করে | একটা command root হিসেবে run করে | পুরোপুরি অন্য user-এ switch করে |
| Password চায় | **নিজের** password | **Target user-এর** password (root-এর) |
| নিরাপদ? | ✅ বেশি নিরাপদ | ⚠️ কম নিরাপদ |
| Log রাখে? | ✅ হ্যাঁ, সব log হয় | ❌ না |
| DevOps-এ use | বেশি preferred | কম preferred |


## Basic `sudo` Commands

### 1. একটা command root হিসেবে run করুন

```bash
sudo command_name
```

**Example:**
```bash
sudo apt install nginx
```

```
[sudo] password for devuser:
Reading package lists... Done
...
nginx installed successfully
```

> আপনার নিজের password দিলেই হবে, root-এর password লাগবে না।


### 2. Root shell-এ যাও (সাময়িকভাবে)

```bash
sudo -i
```

```
root@server:~#
```

> এখন আপনি পুরোপুরি root হিসেবে কাজ করছেন। কাজ শেষে `exit` দিয়ে বের হয়ে যান।


### 3. অন্য user হিসেবে command run করা

```bash
sudo -u username command
```

**Example:**
```bash
sudo -u deploy ls /home/deploy/
```

> `-u` মানে **"এই user হিসেবে run করো"**


### 4. Sudo permission আছে কিনা দেখুন

```bash
sudo -l
```

```
Matching Defaults entries for devuser on server:
    env_reset, mail_badpass

User devuser may run the following commands on server:
    (ALL : ALL) ALL
```

> এই output মানে `devuser` সব কিছু করতে পারে। এটা দেখে বুঝবেন আপনার কতটুকু permission আছে।


### 5. কতক্ষণ password না চেয়ে sudo চলবে?

Default-এ একবার password দিলে **15 মিনিট** পর্যন্ত আর চাইবে না।

আগেই timeout করতে চাইলে:

```bash
sudo -k
```

> এটা sudo session বাতিল/ক্যান্সেল করে দেয়, পরের বার আবার password চাইবে।


## `/etc/sudoers` File কী?

`/etc/sudoers` হলো Linux-এর **"permission registry"** - এখানে লেখা থাকে কোন user কোন command sudo দিয়ে চালাতে পারবে।

```bash
cat /etc/sudoers
```

> ⚠️ এই file সরাসরি `nano` বা `vim` দিয়ে **কখনো edit করা যাবে না!** ভুল হলে পুরো sudo system নষ্ট হয়ে যেতে পারে!


## `visudo` - সঠিক উপায়ে Sudoers Edit করা

```bash
sudo visudo
```

`visudo` হলো একটা special editor যেটা:
- sudoers file edit করতে দেয়
- Save করার আগে **syntax check** করে
- ভুল থাকলে **warn করে** এবং নষ্ট file save হতে দেয় না

```
# visudo খুললে এরকম দেখাবে:
# This file MUST be edited with the 'visudo' command as root.
#
# See the man page for details on how to write a sudoers file.
#
Defaults        env_reset
Defaults        mail_badpass
Defaults        secure_path="/usr/local/sbin:..."

# User privilege specification
root    ALL=(ALL:ALL) ALL

# Members of the admin group may gain root privileges
%admin ALL=(ALL) ALL

# Allow members of group sudo to execute any command
%sudo   ALL=(ALL:ALL) ALL
```

## Sudo Rule-এর Syntax বোঝা

```
WHO  WHERE=(AS_WHOM)  WHAT
```

| Part | মানে | Example |
|---|---|---|
| `WHO` | কোন user বা group | `devuser` বা `%developers` |
| `WHERE` | কোন machine-এ | `ALL` মানে সব জায়গায় |
| `AS_WHOM` | কোন user হিসেবে run হবে | `(ALL)` মানে যেকোনো user হিসেবে |
| `WHAT` | কোন command চালাতে পারবে | `ALL` বা specific path |


### Example 1: সব কিছুর permission দেওয়া

```bash
devuser ALL=(ALL:ALL) ALL
```

> `devuser` যেকোনো machine-এ, যেকোনো user হিসেবে, যেকোনো command চালাতে পারবে।


### Example 2: Password ছাড়াই sudo রান করা

```bash
devuser ALL=(ALL) NOPASSWD: ALL
```

> DevOps automation-এ এটা অনেক কাজে লাগে, script-এ password type করা possible না, তাই `NOPASSWD` দেওয়া হয়।


### Example 3: শুধু নির্দিষ্ট command-এর permission দিয়ে কাজ করা

```bash
devuser ALL=(ALL) NOPASSWD: /usr/bin/systemctl restart nginx
```

> এখন `devuser` শুধু nginx restart করতে পারবে - অন্য কিছু না।


### Example 4: একটা group-কে sudo দিয়ে কাজ করা

```bash
%developers ALL=(ALL) ALL
```

> `%` মানে এটা group। `developers` group-এর সবাই sudo পাবে।


## `/etc/sudoers.d/` - আলাদা File-এ Rule রাখা

Best practice হলো sudoers file সরাসরি edit না করে `/etc/sudoers.d/` folder-এ আলাদা file রাখা।

```bash
sudo visudo -f /etc/sudoers.d/devuser
```

এই file-এ লিখুন:

```
devuser ALL=(ALL) NOPASSWD: /usr/bin/systemctl
```

সেভ করুন। এখন এই rule automatically active হয়ে যাবে।

> এতে মূল sudoers file নিরাপদ থাকে এবং manage করা সহজ হয়।


## Sudo Log দেখা - কে কী করেছে?

`sudo` প্রতিটা command log করে রাখে। এটাই sudo-কে নিরাপদ করে।

```bash
sudo cat /var/log/auth.log | grep sudo
```

```
Mar 5 10:32:14 server sudo: devuser : TTY=pts/0 ;
PWD=/home/devuser ; USER=root ; COMMAND=/usr/bin/apt update

Mar 5 10:35:22 server sudo: devuser : TTY=pts/0 ;
PWD=/home/devuser ; USER=root ; COMMAND=/usr/bin/systemctl restart nginx
```

> DevOps-এ এটা দিয়ে **audit** করা যায় - কে কখন কী command চালিয়েছে।


## Sudo দিয়ে কিছু করতে না দিতে চাইলে?

```bash
devuser ALL=(ALL) ALL, !/usr/bin/passwd root
```

> `!` মানে **"এই command বাদে সব"**। এখানে devuser root-এর password change করতে পারবে না।


## Real-World DevOps-এ Sudo কীভাবে ব্যবহার হয়?

| Scenario | Sudo Rule |
|---|---|
| Deploy user শুধু nginx restart করতে পারবে | `deploy ALL=(ALL) NOPASSWD: /usr/bin/systemctl restart nginx` |
| CI/CD pipeline-এ password ছাড়া sudo | `jenkins ALL=(ALL) NOPASSWD: ALL` |
| Developer শুধু log দেখতে পারবে | `dev ALL=(ALL) NOPASSWD: /usr/bin/journalctl` |
| Monitoring script root ছাড়া চলবে | `monitor ALL=(ALL) NOPASSWD: /usr/bin/ss, /usr/bin/netstat` |


## ⚠️ Common Mistakes & সাবধানতা

| ভুল | সমস্যা | সমাধান |
|---|---|---|
| `vim /etc/sudoers` দিয়ে edit করা | Syntax ভুল হলে sudo কাজ করবে না | সবসময় `visudo` ব্যবহার করা উচিৎ|
| সবাইকে `NOPASSWD: ALL` দেওয়া | Security risk | শুধু প্রয়োজনীয় command-এ দিন |
| sudoers নষ্ট হয়ে গেলে | sudo কাজ করবে না | Recovery mode-এ boot করে ঠিক করতে হবে |


## 📝 Quick Summary

- `sudo` = একটা command temporarily root হিসেবে run করা
- `sudo -l` = আমার sudo permission কী কী আছে দেখা
- `sudo -i` = root shell-এ যাওয়া
- `/etc/sudoers` = sudo-র permission registry
- `visudo` = sudoers file safely edit করার tool
- `NOPASSWD` = password ছাড়া sudo (automation-এ দরকার)
- `/etc/sudoers.d/` = আলাদা file-এ rule রাখার best practice
- Sudo সব কিছু log করে - audit করা যায়


## 🏋️ Practice Tasks

**Task 1:**
নিজের user-এর sudo permission দেখুন:
```bash
sudo -l
```
Output-এ কী দেখাচ্ছে সেটা বোঝার চেষ্টা করুন।

**Task 2:**
একটা নতুন user `deploy` তৈরি করুন এবং তাকে শুধু `systemctl` command-এ NOPASSWD sudo দিন:

```bash
sudo useradd deploy
sudo visudo -f /etc/sudoers.d/deploy

# এটা লিখে দিন: deploy ALL=(ALL) NOPASSWD: /usr/bin/systemctl
```

**Task 3:**
Sudo log দেখুন এবং আপনি আজকে কোন কোন sudo command চালিয়েছেন সেটা খুজুন:
```bash
sudo grep sudo /var/log/auth.log | tail -20
```

---

## ⏭️ What's Next?

**Lesson 9: Switching Users - `su`, `sudo su`, `runuser`**
পরের lesson-এ শিখবো কীভাবে এক user থেকে আরেক user-এ switch করা যায়, `su` আর `sudo su`-র পার্থক্য কী, এবং `runuser` কীভাবে কাজ করে। *Happy Learning* 🚀
