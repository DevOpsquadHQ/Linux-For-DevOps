# Chapter 2 - Lesson 10: User Auth Files (/etc/passwd, /etc/shadow, /etc/group)

**Chapter 2 | Lesson 10 of 10**


## 🎯 এই Lesson-এ কী শিখবো?

Linux-এ তিনটি অত্যন্ত গুরুত্বপূর্ণ ফাইল আছে যেগুলোতে সব user ও group-এর তথ্য সংরক্ষিত থাকে:

| ফাইল | কী রাখে |
|---|---|
| `/etc/passwd` | সব user-এর basic তথ্য |
| `/etc/shadow` | user-দের encrypted password |
| `/etc/group` | সব group-এর তথ্য |

এই তিনটি ফাইল একসাথে কাজ করে Linux-এর পুরো **authentication system** রান করে।


## /etc/passwd - User Database

### এটা কী?

এটা Linux-এর **user phonebook** - প্রতিটি user সম্পর্কে basic তথ্য এখানে থাকে। এটা সবাই পড়তে পারে (world-readable), কারণ এতে password নেই।

### ফাইলটি দেখার command:

```bash
cat /etc/passwd
```

### Example Output:

```
root:x:0:0:root:/root:/bin/bash
daemon:x:1:1:daemon:/usr/sbin:/usr/sbin/nologin
www-data:x:33:33:www-data:/var/www:/usr/sbin/nologin
devops:x:1001:1001:DevOps Engineer:/home/devops:/bin/bash
```


### প্রতিটি line-এর structure (৭টি field, `:` দিয়ে আলাদা):

```
username : password : UID : GID : GECOS : home_dir : shell
```

| Field | নাম | মানে কী |
|---|---|---|
| 1 | **username** | Login name (যেমন: `devops`) |
| 2 | **password** | `x` মানে password `/etc/shadow`-এ আছে |
| 3 | **UID** | User ID (root = 0, normal user = 1000+) |
| 4 | **GID** | Primary Group ID |
| 5 | **GECOS** | Full name বা description (optional) |
| 6 | **home_dir** | User-এর home directory |
| 7 | **shell** | Login shell (যেমন `/bin/bash`) |


### Important UID ranges:

| UID Range | কারা |
|---|---|
| `0` | root (superuser) |
| `1 - 999` | System/service users (daemon, www-data, etc.) |
| `1000+` | Normal human users |

### Real Example বিশ্লেষণ:

```
devops:x:1001:1001:DevOps Engineer:/home/devops:/bin/bash
```

- **devops** → username
- **x** → password shadow-এ আছে
- **1001** → UID
- **1001** → GID
- **DevOps Engineer** → full name/description
- **/home/devops** → home directory
- **/bin/bash** → login করলে bash shell পাবেন


### nologin shell মানে কী?

```
www-data:x:33:33:www-data:/var/www:/usr/sbin/nologin
```

`/usr/sbin/nologin` মানে এই user দিয়ে **login করা যাবে না।** এটা service account, শুধু web server চালানোর জন্য।


### Useful Commands for /etc/passwd:

```bash
# একটি নির্দিষ্ট user দেখুন
grep "devops" /etc/passwd

# শুধু username গুলো দেখুন
cut -d: -f1 /etc/passwd

# UID দিয়ে user খোঁজো
awk -F: '$3 == 1001' /etc/passwd

# কতজন user আছে count করো
wc -l /etc/passwd

# সব normal user দেখুন (UID >= 1000)
awk -F: '$3 >= 1000 && $3 != 65534' /etc/passwd
```


## /etc/shadow - Password Database

### এটা কী?

এটা **"top secret vault"** - শুধু root পড়তে পারে। প্রতিটি user-এর **encrypted (hashed) password** এখানে থাকে। এছাড়া password expiry সংক্রান্ত সব তথ্যও এখানে থাকে।

### দেখার command:

```bash
sudo cat /etc/shadow
```

### Example Output:

```
root:$6$xyz123$AbCdEfGhIjKlMnOpQrStUvWxYz...:19500:0:99999:7:::
devops:$6$abc456$XyZaBcDeFgHiJkLmNoPqRsTuV...:19800:0:99999:7:::
locked_user:!:19700:0:99999:7:::
no_passwd_user::19600:0:99999:7:::
```

### প্রতিটি line-এর structure (৯টি field):

```
username : hashed_password : last_changed : min : max : warn : inactive : expire : reserved
```

| Field | নাম | মানে কী |
|---|---|---|
| 1 | **username** | User-এর নাম |
| 2 | **hashed_password** | Encrypted password |
| 3 | **last_changed** | শেষ কবে password বদলানো হয়েছে (epoch day) |
| 4 | **min_days** | কত দিনের আগে আবার বদলানো যাবে না |
| 5 | **max_days** | কত দিন পর বাধ্যতামূলক বদলাতে হবে |
| 6 | **warn_days** | expire হওয়ার কতদিন আগে warning দেবে |
| 7 | **inactive_days** | expire-এর পর কতদিন account active থাকবে |
| 8 | **expire_date** | Account কবে expire হবে (epoch day) |
| 9 | **reserved** | ভবিষ্যতের জন্য reserved |


### Password Hash বোঝা:

```
$6$xyz123$AbCdEfGhIjKlMnOpQrStUvWxYz...
```

| Part | মানে |
|---|---|
| `$6$` | SHA-512 hashing algorithm ব্যবহার হয়েছে |
| `$xyz123$` | Salt (random value - security বাড়ায়) |
| বাকিটা | আসল hashed password |

**Algorithm codes:**

| Code | Algorithm |
|---|---|
| `$1$` | MD5 (পুরনো, দুর্বল) |
| `$5$` | SHA-256 |
| `$6$` | SHA-512 (আধুনিক, শক্তিশালী) |
| `$y$` | yescrypt (সবচেয়ে নতুন) |


### ⚠️ Special password field values:

| Value | মানে |
|---|---|
| `$6$...` | Normal hashed password |
| `!` বা `!!` | Account **locked** - login করা যাবে না |
| `` (empty) | Password নেই - যে কেউ login করতে পারবে (বিপজ্জনক!) |
| `*` | System account - password login disabled |


### Shadow সংক্রান্ত Commands:

```bash
# একটি user-এর password info দেখুন
sudo chage -l devops
```

**Output:**
```
Last password change                    : Jan 15, 2025
Password expires                        : never
Password inactive                       : never
Account expires                         : never
Minimum number of days between change   : 0
Maximum number of days between change   : 99999
Number of days of warning before expiry : 7
```

```bash
# Password expiry সেট করুন (90 দিনে expire)
sudo chage -M 90 devops

# Account lock করুন
sudo passwd -l devops

# Account unlock করুন
sudo passwd -u devops

# Account expire date সেট করুন
sudo chage -E 2025-12-31 devops
```


## /etc/group - Group Database

### এটা কী?

এটা Linux-এর **"group directory"** - সব group-এর তথ্য এখানে থাকে। কোন group-এ কোন কোন user আছে সেটাও এখানে দেখা যায়।

### দেখার command:

```bash
cat /etc/group
```

### Example Output:

```
root:x:0:
sudo:x:27:devops,munir
docker:x:999:devops,alice,bob
developers:x:1002:devops,munir,alice
devops:x:1001:
```


### প্রতিটি line-এর structure (৪টি field):

```
group_name : password : GID : members
```

| Field | নাম | মানে কী |
|---|---|---|
| 1 | **group_name** | Group-এর নাম |
| 2 | **password** | Group password (প্রায়ই `x` বা empty) |
| 3 | **GID** | Group ID |
| 4 | **members** | Group-এর সদস্যরা (comma দিয়ে আলাদা) |


### Real Example বিশ্লেষণ:

```
docker:x:999:devops,alice,bob
```

- **docker** → group নাম
- **x** → group password (ব্যবহার হয় না)
- **999** → GID
- **devops,alice,bob** → এই তিনজন docker group-এর member


### Group সংক্রান্ত Useful Commands:

```bash
# একটি user কোন কোন group-এ আছে
groups devops
# অথবা
id devops

# একটি নির্দিষ্ট group-এর তথ্য দেখুন
grep "docker" /etc/group

# সব group-এর নাম দেখুন
cut -d: -f1 /etc/group

# Current user-এর group দেখুন
groups
```


## তিনটি ফাইল কীভাবে একসাথে কাজ করে?

এটা একটা **তিন-তলা বিল্ডিং** এর মতো:

```
আপনি login করলে → Linux প্রথমে /etc/passwd চেক করে
                    ↓
         Username আছে কিনা দেখে, UID/GID নেয়
                    ↓
         /etc/shadow-এ গিয়ে password verify করে
                    ↓
         /etc/group-এ গিয়ে আপনার group membership দেখে
                    ↓
         সব ঠিক থাকলে → Login সফল! 🎉
```


## Security দৃষ্টিকোণ থেকে এই ফাইলগুলো:

```bash
# ফাইলের permissions দেখো
ls -l /etc/passwd /etc/shadow /etc/group
```

**Output:**
```
-rw-r--r-- 1 root root   2847 Jan 15 10:00 /etc/passwd
-rw-r----- 1 root shadow 1543 Jan 15 10:00 /etc/shadow
-rw-r--r-- 1 root root    987 Jan 15 10:00 /etc/group
```

| ফাইল | Permissions | কারণ |
|---|---|---|
| `/etc/passwd` | `644` (সবাই পড়তে পারে) | Password নেই, তাই safe |
| `/etc/shadow` | `640` (শুধু root ও shadow group) | Hashed password আছে |
| `/etc/group` | `644` (সবাই পড়তে পারে) | Group info public থাকা দরকার |


## DevOps-এ Real-World ব্যবহার:

### Scenario 1: নতুন server-এ user কনফিগার হয়েছে কিনা দেখা

```bash
grep "deploy" /etc/passwd && echo "User exists" || echo "User not found"
```

### Scenario 2: কোন user-দের sudo access আছে দেখা

```bash
grep "sudo" /etc/group
```

### Scenario 3: সব locked account দেখা

```bash
sudo grep "^[^:]*:!" /etc/shadow | cut -d: -f1
```

### Scenario 4: Service account গুলো দেখা (login নেই)

```bash
grep "nologin\|false" /etc/passwd | cut -d: -f1
```

### Scenario 5: Ansible বা script দিয়ে user তৈরির পর verify করা

```bash
id newuser && grep newuser /etc/passwd && grep newuser /etc/group
```


## /etc/gshadow - Bonus ফাইল

একটা ৪র্থ ফাইলও আছে যেটা অনেকে জানে না:

```bash
sudo cat /etc/gshadow
```

এটা `/etc/shadow`-এর মতোই, কিন্তু **group-এর জন্য।** Group password ও admin তথ্য এখানে থাকে। DevOps-এ এটা কম ব্যবহার হয়।


## 📝 Quick Summary

- `/etc/passwd` - সব user-এর basic info (7 fields), সবাই পড়তে পারে
- `/etc/shadow` - Encrypted passwords ও expiry info (9 fields), শুধু root পড়তে পারে
- `/etc/group` - Group তথ্য ও membership (4 fields), সবাই পড়তে পারে
- Password field-এ `!` মানে locked, `*` মানে system account
- UID 0 = root, 1-999 = system, 1000+ = normal user
- `$6$` মানে SHA-512 hash ব্যবহার হয়েছে
- এই তিনটি ফাইল একসাথে Linux authentication চালায়
- `chage` দিয়ে password policy, `passwd -l` দিয়ে lock করা যায়


## 🏋️ Practice Tasks

**Task 1:** নিজের system-এ `/etc/passwd` দেখুন এবং কতজন user `nologin` shell ব্যবহার করছে সেটা count করুন।
```bash
grep -c "nologin" /etc/passwd
```

**Task 2:** একটি নতুন user তৈরি করুন, তারপর `/etc/passwd`, `/etc/shadow`, `/etc/group` এ সে যোগ হয়েছে কিনা confirm করুন।
```bash
sudo useradd testuser123
grep testuser123 /etc/passwd /etc/shadow /etc/group
```

**Task 3:** `chage -l` দিয়ে আপনার নিজের user-এর password policy দেখুন এবং maximum password age 90 দিনে সেট করুন।
```bash
chage -l $(whoami)
sudo chage -M 90 $(whoami)
```


## Chapter 2 সম্পন্ন!

অভিনন্দন! 🏆 আপনি **Chapter 2: File Permissions & User Management** সম্পূর্ণ করেছেন!

এই Chapter-এ আমরা যা শিখেছি:
- rwx permissions, chmod, chown
- Special permissions (SUID, SGID, Sticky Bit)
- umask, ACLs
- User ও Group management
- sudo ও sudoers
- এবং এখন Linux-এর core authentication files
