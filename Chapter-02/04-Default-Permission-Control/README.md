# Chapter 2 - Lesson 4: Default Permission Control (umask)

**Chapter 2 | Lesson 4 of 10**


## 🎯 আজকে কী শিখবো?

আজকে আমরা শিখবো **umask**, এটা হলো Linux-এর একটা "default permission filter" যেটা automatically decide করে যে নতুন file বা directory তৈরি হলে সেটার permission কেমন হবে।


## Real-World Analogy

কল্পনা করুন আপনি একটা **stamp factory**-তে কাজ করেন।

- সব নতুন document তৈরি হওয়ার আগে একটা **default template** আছে
- সেই template-এ কিছু অংশ **automatically blocked/hidden** থাকে
- যেটুকু block করা হয়, সেটুকু permission **দেওয়া হয় না**

**umask হলো সেই "blocking template"**, এটা বলে দেয় কোন permission গুলো **default-এ বন্ধ রাখতে হবে।**


## Permission Basics - Quick Recap

| Permission | Numeric Value |
|-----------|--------------|
| read (r)  | 4 |
| write (w) | 2 |
| execute (x) | 1 |
| rwx (full) | 7 |


## Linux-এর Default Maximum Permission

Linux-এ নতুন কিছু তৈরি করলে সেটার **maximum possible permission** হলো:

| Type | Maximum Permission | Numeric |
|------|-------------------|---------|
| **File** | `rw-rw-rw-` | **666** |
| **Directory** | `rwxrwxrwx` | **777** |

> File-এ default-এ **execute (x) নেই** কারণ সব file execute করার দরকার নেই, এটা security-র জন্য।


## umask কীভাবে কাজ করে?

**Formula:**

```
Final Permission = Maximum Permission - umask
```

উদাহরণ: যদি umask = **022** হয়

```
File:      666 - 022 = 644  →  rw-r--r--
Directory: 777 - 022 = 755  →  rwxr-xr--
```

মানে হলো:
- **Owner** → read + write (file), read + write + execute (dir)
- **Group** → শুধু read
- **Others** → শুধু read


## umask Command

### ১. বর্তমান umask দেখুন

```bash
umask
```

**Output:**
```
0022
```

> প্রথম `0` টা special permission-এর জন্য (এখন ignore করুন), বাকি `022` হলো actual umask।


### ২. Symbolic format-এ দেখুন

```bash
umask -S
```

**Output:**
```
u=rwx,g=rx,o=rx
```

এটা বলছে:
- **u (user/owner)** → rwx পাবে
- **g (group)** → rx পাবে (write নেই)
- **o (others)** → rx পাবে (write নেই)


## Practical Example - নিজে দেখুন

### Step 1: বর্তমান umask check করুন
```bash
umask
```
```
0022
```

### Step 2: একটা file তৈরি করুন
```bash
touch testfile.txt
ls -l testfile.txt
```
**Output:**
```
-rw-r--r-- 1 user user 0 Mar 4 10:00 testfile.txt
```
→ Permission: **644** (666 - 022 = 644)

### Step 3: একটা directory তৈরি করুন
```bash
mkdir testdir
ls -ld testdir
```
**Output:**
```
drwxr-xr-x 2 user user 4096 Mar 4 10:00 testdir
```
→ Permission: **755** (777 - 022 = 755)


## umask পরিবর্তন করা

### Syntax:
```bash
umask [value]
```


### Example 1: umask 000 - সব permission দিন

```bash
umask 000
touch openfile.txt
ls -l openfile.txt
```
**Output:**
```
-rw-rw-rw- 1 user user 0 Mar 4 10:00 openfile.txt
```
→ Permission: **666** (666 - 000 = 666) সবাই read+write করতে পারবে

> এটা **insecure**, production-এ কখনো করবেন না!


### Example 2: umask 077 শুধু owner-এর জন্য

```bash
umask 077
touch privatefile.txt
ls -l privatefile.txt
```
**Output:**
```
-rw------- 1 user user 0 Mar 4 10:00 privatefile.txt
```
→ Permission: **600** (666 - 077 = 600) শুধু owner পড়তে ও লিখতে পারবে

> এটা **highly secure** private key files-এর জন্য perfect!


### Example 3: umask 002 - Group-friendly

```bash
umask 002
touch groupfile.txt
ls -l groupfile.txt
```
**Output:**
```
-rw-rw-r-- 1 user user 0 Mar 4 10:00 groupfile.txt
```
→ Permission: **664** (666 - 002 = 664) owner ও group দুজনেই write করতে পারবে

> Team collaboration project-এ এটা useful


## Common umask Values - Quick Reference Table

| umask | File Permission | Directory Permission | কোথায় ব্যবহার হয় |
|-------|----------------|---------------------|-----------------|
| **022** | 644 (rw-r--r--) | 755 (rwxr-xr-x) | Default (most systems) |
| **027** | 640 (rw-r-----) | 750 (rwxr-x---) | Secure servers |
| **077** | 600 (rw-------) | 700 (rwx------) | Highly sensitive data |
| **002** | 664 (rw-rw-r--) | 775 (rwxrwxr-x) | Team/shared projects |
| **000** | 666 (rw-rw-rw-) | 777 (rwxrwxrwx) | Never use! |


## umask Permanently Set করা

এখন যদি আপনি `umask 077` দেন এটা **শুধু current session**-এ কাজ করবে। Terminal বন্ধ করলে আবার আগের umask ফিরে আসবে।

**Permanently set করতে হলে:**

### For a specific user:
```bash
nano ~/.bashrc
```
সবার নিচে এই line যোগ করুন:
```bash
umask 027
```
তারপর:
```bash
source ~/.bashrc
```


### For all users (system-wide):
```bash
sudo nano /etc/profile
```
এই line যোগ করুন:
```bash
umask 022
```

- `/etc/profile` সব user-এর জন্য apply হয়
- `~/.bashrc` বা `~/.profile` শুধু সেই specific user-এর জন্য


## DevOps Real-World Use Cases

| Scenario | Recommended umask | কারণ |
|----------|------------------|------|
| **Web server files** | 022 | Public read দরকার |
| **SSH private keys** | 077 | শুধু owner দেখবে |
| **Shared dev environment** | 002 | Team-এর সবাই edit করতে পারবে |
| **Backup scripts** | 027 | Group read, others কিছুই না |
| **Kubernetes secrets** | 077 | Maximum security |


## Bonus: umask এর Bitwise Logic (Optional Deep Dive)

Technically, umask **subtraction** করে না, এটা **bitwise AND with complement** করে।

```
Final = Max_Permission AND (NOT umask)
```

কিন্তু বেশিরভাগ ক্ষেত্রে simple subtraction-ই কাজ করে। শুধু মনে রাখবেন:

- **666 - 077 = 600** → সঠিক
- কিন্তু **777 - 678 = 099** → এই ধরনের edge case-এ bitwise logic দরকার হয়, কিন্তু practically কেউ এমন umask দেয় না।


## 📝 Quick Summary

- **umask** = একটা filter যেটা নতুন file/directory-র default permission নির্ধারণ করে
- **Formula:** `Final = Max - umask` (File: 666, Dir: 777)
- Default umask বেশিরভাগ সিস্টেমে **022**
- **umask** দিয়ে current value দেখুন, **umask -S** দিয়ে symbolic দেখুন
- Permanently set করতে `~/.bashrc` বা `/etc/profile` edit করুন
- **077** = most secure, **002** = team-friendly, **022** = standard


## 🏋️ Practice Tasks

এগুলো নিজে নিজে try করুন:

1. **Task 1:** Terminal-এ `umask` রান করুন এবং current value দেখুন। তারপর `umask -S` রান করুন এবং দুটোর মধ্যে সম্পর্ক বোঝার চেস্টা করুন।

2. **Task 2:** umask **077** set করুন, একটা file এবং একটা directory তৈরি করুন। `ls -l` দিয়ে permission দেখুন। তারপর umask **002** set করুন এবং আবার তৈরি করুন, পার্থক্য লক্ষ্য করুন।

3. **Task 3:** আপনার `~/.bashrc` file-এ `umask 027` add করুন, `source ~/.bashrc` রান করুন, তারপর নতুন file তৈরি করে permission verify করুন।

---

## ⏭️ What's Next?

**Chapter 2 - Lesson 5: ACLs - Access Control Lists (getfacl, setfacl)**

> পরের lesson-এ আমরা শিখবো **ACL**, যেটা standard permission-এর বাইরে গিয়ে **specific user বা group**-কে আলাদাভাবে permission দেওয়ার সুযোগ দেয়। এটা enterprise environment-এ অনেক বেশি ব্যবহৃত হয়! 🚀
