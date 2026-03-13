# Chapter 2 - Lesson 9: Switching Users (su, sudo su, runuser)

**Chapter 2 | Lesson 9 of 10**


## 🎯 এই Lesson-এ কী শিখবো?

- `su` দিয়ে অন্য user-এ switch করা
- `sudo su` দিয়ে root হওয়া
- `runuser` কী এবং কখন ব্যবহার হয়
- এগুলোর মধ্যে পার্থক্য কী
- DevOps-এ কখন কোনটা ব্যবহার করবো


### আগে একটু Context বুঝি

ধরুন আপনি একটা Office Building-এ আছেন। আপনার নিজের একটা Desk আছে (আপনার user account)।
এখন আপনাকে অন্য কারো Desk-এ বসে কাজ করতে হবে, সেটার জন্য আপনাকে **ঐ ব্যক্তির পরিচয় নিতে হবে।**

Linux-এ ঠিক এই কাজটাই করে **`su`** (Switch User)।


## 1. `su` - Switch User

### সহজ ভাষায়:
`su` মানে **"Substitute User"** বা **Switch User।**
এটা দিয়ে আপনি **current terminal-এ অন্য user হিসেবে** কাজ করতে পারবেন।

### Syntax:
```bash
su [options] [username]
```


### Example 1 - অন্য user-এ switch করুন:
```bash
su munir
```
```
Password:        ← munir-এর password চাইবে
```
Password দেওয়ার পর আপনি **munir** হিসেবে কাজ করতে পারবেন।


### Example 2 - root user-এ switch করুন:
```bash
su
# অথবা
su root
```
```
Password:        ← root-এর password চাইবে
```
> ⚠️ অনেক modern distro-তে (Ubuntu) root-এর password by default disabled থাকে।
> সেক্ষেত্রে `sudo su` ব্যবহার করতে হয়।


### Example 3 - Login shell সহ switch করা (`-` বা `-l` flag):

```bash
su - munir
# অথবা
su -l munir
```

#### `-` (dash) দিলে কী হয়?

| Without `-` | With `-` |
|---|---|
| শুধু user change হয় | পুরো environment change হয় |
| আগের user-এর PATH, HOME থাকে | নতুন user-এর HOME, PATH, shell সব load হয় |
| "আংশিক" পরিচয় বদল | "পুরোপুরি" ওই user হয়ে যাও |

**Real-world analogy:**
- `su munir` = জামা বদলালেন, কিন্তু পুরনো জুতা পরেই আছেন
- `su - munir` = পুরো outfit বদলে গেলে, মাথা থেকে পা পর্যন্ত নতুন পরিচয়

```bash
su - munir
```
```
Password:
[munir@server ~]$    ← munir-এর home directory-তে চলে গেছি
```


### Example 4 - নির্দিষ্ট একটা command রান করেন অন্য user হিসেবে (`-c` flag):
```bash
su -c "whoami" munir
```
```
Password:
munir
```
> Munir-এর password দিলে শুধু `whoami` command টা munir হিসেবে চলবে।
> তারপর আবার আপনি আপনার নিজের user-এ ফিরে আসবেন।


### Current user কে? - `whoami` ও `id`
```bash
whoami
```
```
munir
```
```bash
id
```
```
uid=1001(munir) gid=1001(munir) groups=1001(munir)
```


### Switch করার পর বের হওয়ার উপায়:
```bash
exit
# অথবা
logout
# অথবা Ctrl + D
```


## 2. `sudo su` - Root হওয়ার সবচেয়ে সহজ উপায়

### সহজ ভাষায়:
`sudo su` মানে - **"আমার নিজের password দিয়ে root হয়ে যান।"**

Ubuntu/Debian-এ root-এর password সরাসরি দেওয়া হয় না। তাই `su root` কাজ করে না।
কিন্তু যদি আপনি **sudoers** list-এ থাকেন, তাহলে `sudo su` দিয়ে root হতে পারবেন।

```bash
sudo su
```
```
[sudo] password for youruser:   ← আপনার নিজের password
root@server:/home/youruser#     ← এখন আপনি root!
```


### `sudo su` vs `sudo su -` - পার্থক্য:

```bash
sudo su
```
> Root হবে, কিন্তু **আপনার current directory ও environment** কিছুটা থেকে যাবে।

```bash
sudo su -
```
> Root হবে এবং **root-এর পুরো environment** load হবে।
> Root-এর home directory `/root`-এ নিয়ে যাবে।

```bash
sudo su -
```
```
root@server:~#
pwd
/root
```


### `su` vs `sudo su` পার্থক্য একটা Table-এ:

| বিষয় | `su` | `sudo su` |
|---|---|---|
| কার password লাগে? | **target user-এর** password | **তোমার নিজের** password |
| কখন ব্যবহার হয়? | root password জানলে | sudoers-এ থাকলে |
| Ubuntu-তে কাজ করে? | root password disabled বলে ❌ | হ্যাঁ |
| Security | কম safe | বেশি safe (audit trail থাকে) |


## 3. `runuser` - Service/Script-এর জন্য User Switch

### সহজ ভাষায়:
`runuser` হলো `su`-এর মতোই কিন্তু এটা **শুধু root** ব্যবহার করতে পারে।
Root থেকে অন্য কোনো user হিসেবে command চালাতে এটা ব্যবহৃত হয়।

> **মূল পার্থক্য:** `runuser` কোনো PAM authentication করে না (password লাগে না, কারণ আপনি আগেই root)

### Syntax:
```bash
runuser -l username -c "command"
```

### Example:
```bash
# প্রথমে root হও
sudo su -

# তারপর munir হিসেবে একটা script চালাও
runuser -l munir -c "bash /home/munir/backup.sh"
```


### DevOps-এ `runuser` কোথায় ব্যবহার হয়?

```bash
# Systemd service file-এ
[Service]
User=www-data
ExecStart=/usr/bin/python3 /app/server.py

# অথবা manually
runuser -l www-data -c "/opt/app/start.sh"
```

> যখন আপনি চান একটা script **নির্দিষ্ট service user** হিসেবে চলুক (যেমন `nginx`, `postgres`, `jenkins`), তখন `runuser` ব্যবহার হয়।


## 4. তিনটার মধ্যে পার্থক্য - Side by Side

| বিষয় | `su` | `sudo su` | `runuser` |
|---|---|---|---|
| **কে ব্যবহার করে?** | যেকেউ | sudoers user | শুধু root |
| **Password লাগে?** | Target user-এর | নিজের | ❌ লাগে না |
| **PAM auth করে?** | হ্যাঁ | হ্যাঁ | ❌ না |
| **DevOps use case** | User switch | Root access নেওয়া | Service script চালানো |
| **Safer কোনটা?** | মাঝারি | বেশি safe | Root context-এ safe |


## 5. Real-World DevOps Scenarios

### Scenario 1 - Production server-এ কাজ করতে root হওয়া:
```bash
sudo su -
# এখন সব root-level কাজ করো
apt update
systemctl restart nginx
exit   # কাজ শেষ হলে বের হয়ে আসো - সবসময়!
```

### Scenario 2 - App user হিসেবে দেখুন কোনো permission issue আছে কিনা:
```bash
su - appuser
ls -la /var/app/
cat /var/app/config.yml
exit
```

### Scenario 3 - Deployment script রান করুন নির্দিষ্ট user হিসেবে:
```bash
sudo su - deployuser -c "cd /opt/app && git pull && ./deploy.sh"
```

### Scenario 4 - Postgres DB-তে কাজ করা (postgres user হিসেবে):
```bash
sudo su - postgres
psql
\l          # সব database দেখাবে
\q
exit
```


## 6. Security Best Practices

 কাজ শেষ হলে সাথে সাথে exit করুন। Production-এ সরাসরি root login avoid করুন sudo su ব্যবহার করুন। এতে audit log থাকে runuser ব্যবহার করুন service script-এর জন্য।

- ❌ root হয়ে সারাক্ষণ কাজ করবেন না
- ❌ root password share করবেন না


**Audit log দেখতে চান?** `sudo` ব্যবহারের সব record থাকে:

```bash
sudo cat /var/log/auth.log | grep sudo
```


## 📝 Quick Summary

- **`su username`** → অন্য user হও (ওই user-এর password লাগবে)
- **`su -`** → Root হও পুরো environment সহ (root password লাগবে)
- **`su - username`** → ওই user-এর পুরো environment সহ switch করো
- **`sudo su`** → নিজের password দিয়ে root হও (Ubuntu-তে standard পদ্ধতি)
- **`sudo su -`** → Root-এর পুরো environment সহ root হও
- **`runuser -l user -c "cmd"`** → Root থেকে অন্য user হিসেবে command রান করো
- কাজ শেষে সবসময় **`exit`** করো


## 🏋️ Practice Tasks

**Task 1:**
```bash
# একটা নতুন user তৈরি করুন
sudo useradd -m testuser
sudo passwd testuser

# এখন su দিয়ে testuser হন
su - testuser

# আপনি কে সেটা verify করুন
whoami
pwd

# বের হয়ে আসুন
exit
```

**Task 2:**
```bash
# sudo su দিয়ে root হন
sudo su -

# Root-এর home directory দেখুন
pwd       # /root হওয়া উচিত

# কিছু root-only কাজ করুন
whoami
id

# বের হয়ে আসুন
exit
```

**Task 3:**
```bash
# su -c দিয়ে testuser হিসেবে শুধু একটা command রান করুন
su -c "id && pwd" testuser

# এরপর দেখুন আপনি আবার নিজের user-এ আছেন কিনা
whoami
```

---

## ⏭️ What's Next?

**Chapter 2 - Lesson 10:**
`/etc/passwd`, `/etc/shadow`, `/etc/group` - Understanding These Files। Linux কীভাবে user ও password তথ্য সংরক্ষণ করে, এই তিনটা গুরুত্বপূর্ণ file-এর structure ও প্রতিটা field-এর মানে বিস্তারিত জানবো। DevOps-এ এই file গুলো বোঝা অনেক জরুরি! *Happy Learning* 🚀
