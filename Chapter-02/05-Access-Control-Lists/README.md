# Chapter 2 - Lesson 5: Access Control Lists (ACLs)

**Chapter 2 | Lesson 5 of 10**


## 🎯 এই Lesson-এ আমরা কী শিখবো?

- ACL কী এবং কেন দরকার?
- Traditional Permission-এর সীমাবদ্ধতা
- `getfacl` দিয়ে ACL দেখা
- `setfacl` দিয়ে ACL সেট করা
- Default ACL কী?
- Real-world DevOps use case


## ACL কী? - আগে সমস্যাটা বোঝা

আগের lessons-এ আমরা শিখেছি যে Linux-এ তিন ধরনের permission থাকে:

```
Owner (user) → Group → Others
```

কিন্তু এখানে একটা বড় সমস্যা আছে।

### Real-World সমস্যা:

ধরুন আপনার একটা file আছে: `project.txt`

| কে | Permission |
|---|---|
| `munir` (owner) | read, write |
| `devteam` (group) | read only |
| Others | no access |

এখন **boss** বলল: *"শুধু `arefin` কেও read permission দাও কিন্তু `arefin` `devteam` group-এ নেই!"*

Traditional permission দিয়ে এটা **সম্ভব না!** কারণ:
- `arefin`-কে owner বানাতে পারবে না
- `arefin`-কে group-এ add করলে সব group member-দের permission পাবে
- Others-এ দিলে সবাই পাবে

**এই সমস্যার সমাধান হলো ACL!**


## ACL কী?

**ACL = Access Control List**

ACL হলো একটা **বিশেষ permission system** যেটা দিয়ে আপনি **specific user বা specific group**-কে আলাদাভাবে permission দিতে পারবেন owner, group বা others-এর বাইরেও।

### Real-World Analogy:

একটা apartment building-এর কথা ভাবুন:
- **Traditional Permission** = শুধু building owner, building staff, বা general public এই তিন category
- **ACL** = আপনি চাইলে শুধু আপনার বন্ধু `muhib`-কে আলাদাভাবে একটা spare key দিতে পারবেন, সে না owner না staff

ACL হলো সেই **spare key system!**


## ACL Support আছে কিনা চেক করুন

ACL ব্যবহার করতে হলে file system-কে ACL support করতে হবে। Modern Linux-এ `ext4`, `xfs` সব major file system-এ ACL by default enabled থাকে।

চেক করতে:

```bash
tune2fs -l /dev/sda1 | grep "Default mount options"
```

অথবা `lsblk` দিয়ে আমরা দেখতে পারি আমাদের `partition` এর নাম কি?

যদি output-এ `acl` দেখেন, তাহলে enabled আছে।

অথবা সহজভাবে:

```bash
mount | grep "acl"
```


## ACL Package Install করুন

```bash
# Ubuntu/Debian
sudo apt install acl -y

# RHEL/CentOS/Fedora
sudo yum install acl -y
```


## Command 1: `getfacl` - ACL দেখা

`getfacl` মানে **"get file ACL"** এটা দিয়ে কোনো file বা directory-র ACL entries দেখা যায়।

### Syntax:
```bash
getfacl [filename]
```

### Example:

আগে একটা test file বানান:

```bash
mkdir ~/acl-practice
cd ~/acl-practice
touch project.txt
```

এখন `getfacl` দিয়ে দেখুন:

```bash
getfacl project.txt
```

### Expected Output:
```
# file: project.txt
# owner: munir
# group: munir
user::rw-
group::rw-
other::r--
```

### Output-এর মানে কী?

| Line | মানে |
|---|---|
| `# file: project.txt` | File-এর নাম |
| `# owner: munir` | File-এর owner |
| `# group: munir` | File-এর group |
| `user::rw-` | Owner-এর permission (read+write) |
| `group::rw-` | Group-এর permission |
| `other::r--` | Others-এর permission |

এখন পর্যন্ত এটা normal `ls -l`-এর মতোই। ACL add করলে পার্থক্য দেখবো।


## Command 2: `setfacl` - ACL সেট করা

`setfacl` মানে **"set file ACL"** এটা দিয়ে specific user বা group-কে আলাদা permission দেওয়া যায়।

### Syntax:
```bash
setfacl [options] [rule] [filename]
```

### গুরুত্বপূর্ণ Options:

| Option | মানে |
|---|---|
| `-m` | Modify (add বা update করো) |
| `-x` | Remove specific ACL entry |
| `-b` | সব ACL entries remove করো |
| `-R` | Recursive (directory-র ভেতরে সবকিছুতে apply হবে) |
| `-d` | Default ACL সেট করো |


### Example 1: Specific User-কে Permission দেওয়া

মনে করেন `arefin`-কে `project.txt`-এ **read permission** দিতে চান:

```bash
setfacl -m u:arefin:r project.txt
```

**Breakdown:**
- `-m` → modify করো
- `u:arefin:r` → **u** = user, **arefin** = username, **r** = read permission
- `project.txt` → কোন file-এ

এখন `getfacl` দিয়ে দেখুন:

```bash
getfacl project.txt
```

### Expected Output:
```
# file: project.txt
# owner: munir
# group: munir
user::rw-
user:arefin:r--          ← নতুন entry! শুধু arefin-এর জন্য
group::rw-
mask::rw-
other::r--
```

দেখুন `user:arefin:r--` নামে নতুন একটা line এসেছে! এটাই ACL!

এখন `ls -l` দিয়ে দেখুন:

```bash
ls -l project.txt
```

```
-rw-rw-r--+ 1 munir munir 0 Jun 10 10:00 project.txt
```

Permission-এর শেষে **`+`** চিহ্ন দেখেছেন? এই `+` মানে এই file-এ **ACL active আছে!**


### Example 2: Specific User-কে Read+Write Permission দিন

```bash
setfacl -m u:arefin:rw project.txt
```

```bash
getfacl project.txt
```

```
user:arefin:rw-          ← এখন arefin read+write পাবে
```

---

### Example 3: Specific Group-কে Permission দিন

ধরুন `contractors` নামের একটা group-কে **read-only** access দিতে চান:

```bash
setfacl -m g:contractors:r project.txt
```

**Breakdown:**
- `g:contractors:r` → **g** = group, **contractors** = group name, **r** = read

```bash
getfacl project.txt
```

```
user::rw-
user:arefin:rw-
group::rw-
group:contractors:r--     ← contractors group-এর আলাদা entry
mask::rw-
other::r--
```


### Example 4: ACL Entry Remove করা

`arefin`-এর ACL entry সরিয়ে দিতে:

```bash
setfacl -x u:arefin project.txt
```

```bash
getfacl project.txt
```

```
user::rw-
group::rw-
mask::rw-
other::r--
```

`arefin`-এর entry চলে গেছে!


### Example 5: সব ACL Remove করা

```bash
setfacl -b project.txt
```

এটা সব ACL entries মুছে ফেলবে এবং file আবার normal permission-এ ফিরে যাবে।


## mask কী? - গুরুত্বপূর্ণ Concept!

`getfacl` output-এ আপনি একটা `mask` line দেখেছিলেন। এটা কী?

```
mask::rw-
```

**mask** হলো একটা **maximum permission ceiling**। ACL দিয়ে কাউকে যতই permission দেই না কেন, mask-এর বাইরে যেতে পারবে না।

### Analogy:

ধরুন আপনার building-এ একটা rule আছে: *"কোনো visitor রাত ১০টার পরে থাকতে পারবে না।"*

আপনি চাইলেও আপনার বন্ধুকে রাত ১২টা পর্যন্ত রাখতে পারবে না, এটাই **mask!**

### Example:

```bash
setfacl -m u:arefin:rwx project.txt   # arefin-কে full permission দিলাম
setfacl -m mask::r-- project.txt   # কিন্তু mask শুধু read allow করছে
```

```bash
getfacl project.txt
```

```
user::rw-
user:arefin:rwx            #effective:r--   ← arefin rwx পেয়েছে কিন্তু effective শুধু r--
group::rw-              #effective:r--
mask::r--
other::r--
```

`#effective:r--` মানে **বাস্তবে** arefin শুধু read পাচ্ছে, যদিও তাকে rwx দেওয়া হয়েছিল।


## Default ACL - Directory-র জন্য বিশেষ Feature

Default ACL দিয়ে আপনি বলতে পারেন: *"এই directory-র ভেতরে যত নতুন file/folder তৈরি হবে, সবাই automatically এই permission পাবে।"*

### Analogy:

আপনি একটা office বানালেন এবং rule দিলেন: *"এই office-এ যত নতুন employee আসবে, সবাই by default `arefin`-এর সাথে document share করবে।"*

### Example:

```bash
mkdir ~/acl-practice/shared-dir
```

`shared-dir`-এ Default ACL সেট করুন:

```bash
setfacl -d -m u:arefin:rw ~/acl-practice/shared-dir
```

**Breakdown:**
- `-d` → default ACL সেট করো
- `-m u:arefin:rw` → arefin-কে read+write

এখন এই directory-র ভেতরে নতুন file বানান:

```bash
touch ~/acl-practice/shared-dir/newfile.txt
```

```bash
getfacl ~/acl-practice/shared-dir/newfile.txt
```

```
# file: shared-dir/newfile.txt
# owner: munir
# group: munir
user::rw-
user:arefin:rw-         ← automatically inherited হয়েছে!
group::r--
mask::rw-
other::r--
```

`newfile.txt` automatically `arefin`-এর জন্য ACL পেয়ে গেছে! আপনি manually কিছু করেননি।


## Recursive ACL - Directory-র সব Content-এ Apply করা

পুরো directory এবং তার ভেতরের সব file/folder-এ একসাথে ACL দিতে:

```bash
setfacl -R -m u:arefin:rx ~/acl-practice/shared-dir
```

**Breakdown:**
- `-R` → Recursive, মানে ভেতরের সব কিছুতে apply হবে


## Real-World DevOps Use Cases

| Scenario | Solution |
|---|---|
| Specific developer-কে log file পড়তে দাও কিন্তু সব user-কে না | `setfacl -m u:dev1:r /var/log/app.log` |
| CI/CD pipeline user-কে deploy directory-তে write access | `setfacl -m u:jenkins:rwx /var/www/html` |
| Contractor team-কে project folder read করতে দাও | `setfacl -R -m g:contractors:rx /opt/project` |
| নতুন file automatically shared হোক | Default ACL on shared directory |
| Audit করো কার কী access আছে | `getfacl -R /opt/project` |


## Quick Reference Table

```bash
# User-কে permission দেওয়া
setfacl -m u:username:rwx file

# Group-কে permission দেওয়া
setfacl -m g:groupname:rx file

# User-এর ACL রিমুভ করা
setfacl -x u:username file

# সব ACL রিমুভ করা
setfacl -b file

# ACL দেখুন
getfacl file

# Recursive ACL
setfacl -R -m u:username:rx directory/

# Default ACL (নতুন file-এর জন্য)
setfacl -d -m u:username:rw directory/
```


## 📝 Quick Summary

- ✅ **ACL** = traditional owner/group/others permission-এর বাইরে specific user বা group-কে আলাদা permission দেওয়ার system
- ✅ `getfacl` দিয়ে ACL দেখা যায়
- ✅ `setfacl -m` দিয়ে ACL add/modify করা যায়
- ✅ `setfacl -x` দিয়ে specific entry remove করা যায়
- ✅ `setfacl -b` দিয়ে সব ACL remove করা যায়
- ✅ **mask** হলো maximum permission ceiling
- ✅ **Default ACL** দিয়ে নতুন files automatically permission inherit করে
- ✅ `-R` দিয়ে পুরো directory-তে recursive ACL apply হয়
- ✅ ACL active থাকলে `ls -l`-এ `+` চিহ্ন দেখায়

---

## 🏋️ Practice Tasks

এগুলো নিজে নিজে করে দেখুন!

**Task 1:**
```
1. ~/acl-lab নামে একটা directory বানান
2. ভেতরে test.txt file তৈরি করুন
3. আপনার system-এ একটা নতুন user তৈরি করুন: sudo useradd testuser
4. testuser-কে test.txt-এ শুধু read permission দিন ACL দিয়ে
5. getfacl দিয়ে verify করুন
```

**Task 2:**
```
1. ~/shared নামে একটা directory বানান
2. Default ACL দিয়ে testuser-কে read+write permission দিন
3. ভেতরে দুটো নতুন file তৈরি করুন
4. getfacl দিয়ে verify করুন যে নতুন files automatically ACL পেয়েছে কিনা
```

**Task 3:**
```
1. আগের সব ACL remove করুন (setfacl -b)
2. getfacl দিয়ে confirm করুন ACL চলে গেছে
3. ls -l দিয়ে দেখুন + চিহ্ন আছে কিনা
```

---

## ⏭️ What's Next?

**Chapter 2 - Lesson 6: User Management**
> `useradd`, `usermod`, `userdel`, `passwd` Linux-এ user কীভাবে তৈরি, modify এবং delete করা যায়, সেটা শিখবো। DevOps-এ server access manage করার জন্য এটা অত্যন্ত গুরুত্বপূর্ণ!


আমরা এখন অনেকদুর এগিয়ে এসেছি। ACL একটু tricky মনে হলেও practice করলে সহজ হয়ে যাবে। যেকোনো জায়গায় আটকে গেলে জিজ্ঞেস করতে ভুলবেন না! *Happy Learning* 😊
