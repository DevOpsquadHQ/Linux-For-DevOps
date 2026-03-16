# Chapter 2 - Final Assessment 

**Total 20 Practice Tasks**

এখন সময় নিজেকে যাচাই করার! আমি আমার মতে করে একটা structure তৈরি করেছি আপনারা আপনাদের মতো করে তৈরি করতে পারেন বা এই এ্যাসাইনমেন্টও ফলো করতে পারেন।

এখানে আমি ২৫টি টাস্ক তৈরি করেছি। প্রথমে, টাস্কগুলো উল্লেখ করেছি এবং এর পরে এর সমাধান উল্লেখ করেছি। টাস্কগুলো নিজে নিজে করার চেস্টা করবেন এর পরে সমাধান দেখে মিলিয়ে নিবেন। 

এখানে Tasks আছে **3 ধরনের**: 
- Conceptual (বোঝার)
- Command-based (করার)
- Scenario-based (real-world)


## SECTION A - Conceptual Questions (বোঝার প্রশ্ন)
> এগুলো short answer প্রশ্ন। নিজে নিজে চেস্টা করুন, পরে মিলিয়ে নিবেন।


**Q1.** একটা file-এর permission দেখালো: `-rwxr-x--x`

এই permission-এ:
- Owner কী কী করতে পারবে?
- Group কী কী করতে পারবে?
- Others কী করতে পারবে?
- Numeric (octal) value কত হবে এটার?

---

**Q2.** `chmod 644 file.txt` আর `chmod u=rw,go=r file.txt` - এই দুটো command কি same কাজ করে? কেন?

---

**Q3.** SUID, SGID, আর Sticky Bit - তিনটার পার্থক্য **এক লাইনে** বলুন এবং কোনটা কোথায় use হয় উদাহরণ দিন।

---

**Q4.** `umask 027` set থাকলে:
- নতুন file তৈরি হলে permission কত হবে?
- নতুন directory তৈরি হলে permission কত হবে?

---

**Q5.** `/etc/passwd` আর `/etc/shadow` file-এর মধ্যে পার্থক্য কী? Password কোনটায় থাকে এবং কেন?

---

**Q6.** Hard link আর Symbolic link-এর মধ্যে পার্থক্য কী? (এটা Chapter 1 থেকে - bonus!)

---

## ⌨️ SECTION B - Command-Based Tasks (হাতে-কলমে করার চেস্টা করুন)
> এগুলো আপনার terminal-এ actually run করে দেখুন।

---

**Task 7.** নিচের কাজগুলো **একটার পর একটা** করুন:
```
1. /tmp/devops_test/ নামে একটা directory তৈরি করুন
2. সেখানে script.sh, config.cfg, data.txt - তিনটা file তৈরি করুন
3. script.sh কে → Owner: rwx, Group: r-x, Others: --- করুন (numeric দিয়ে)
4. config.cfg কে → সবার জন্য read-only করুন (numeric দিয়ে)
5. data.txt কে → symbolic mode দিয়ে group-এর write permission রিমুভ করে দিন
```

---

**Task 8.** `devops` নামে একটা নতুন user তৈরি করো যেখানে:
- Home directory হবে `/home/devops`
- Default shell হবে `/bin/bash`
- Comment/description হবে "DevOps Engineer"
তারপর verify করো user টা তৈরি হয়েছে কিনা।

---

**Task 9.** `developers` নামে একটা group তৈরি করুন এবং `devops` user-কে সেই group-এ add করুন। Verify করুন।

---

**Task 10.** `/tmp/devops_test/config.cfg` file-এর:
- Owner পরিবর্তন করুন `devops` user-এ
- Group পরিবর্তন করুন `developers` group-এ
- একটাই command-এ দুটো কাজ করুন

---

**Task 11.** `/tmp/devops_test/` directory-তে **Sticky Bit** set করুন। তারপর `ls -ld` দিয়ে verify করুন।

---

**Task 12.** `devops` user-এর জন্য `/etc/sudoers`-এ এমন একটা rule add করুন যাতে সে **password ছাড়াই** `systemctl` command চালাতে পারে। (`visudo` use করুন)

---

**Task 13.** `script.sh` file-এ **SUID** bit set করুন। তারপর `ls -l` দিয়ে দেখুন - output-এ কী পরিবর্তন এলো?

---

**Task 14.** নিচের command run করুন এবং output explain করুন:
```bash
getfacl /tmp/devops_test/data.txt
```
তারপর `devops` user-কে ঐ file-এ **ACL দিয়ে** read+write permission দিন (যদিও সে owner না)।

---

**Task 15.** `devops` user-এর password set করুন `DevOps@2025` দিয়ে। তারপর account-টাকে **lock** করুন এবং verify করুন যে locked হয়েছে।

---

**Task 16.** বর্তমান system-এর `umask` value দেখুন। তারপর current session-এ `umask` change করুন `022` থেকে `077`-এ। একটা নতুন file তৈরি করে দেখুন permission কেমন আসে।

---

## 🔥 SECTION C - Scenario-Based Tasks (Real DevOps Problems)
> এগুলো real-world situations - চিন্তা করে solve করুন।

---

**Scenario 17. Production Server Problem:**

আপনার team-এর সবাই `/var/www/html/` directory-তে file তৈরি করে, কিন্তু সমস্যা হলো একজন অন্যজনের file delete করে ফেলছে।
- কোন special permission দিলে এই সমস্যা solve হবে?
- Exact command লিখুন।

---

**Scenario 18. Security Audit:**

আপনি একটা server audit করতে গিয়ে দেখলেন `/usr/bin/passwd` file-এ SUID bit আছে।
- এটা কি normal নাকি security risk?
- কেন এখানে SUID থাকা দরকার? Explain করুন।

---

**Scenario 19. Team Setup:**

আপনার company-তে নতুন একটা project শুরু হয়েছে। আপনাকে বলা হলো:
- `alice`, `bob`, `charlie` - তিনজন developer
- তাদের জন্য `webteam` group বানান
- `/var/project/` directory বানান যেখানে শুধু `webteam` group read+write+execute করতে পারবে
- অন্য কেউ কিছু করতে পারবে না
- যে file-ই তৈরি হোক, সেটা automatically `webteam` group-এর হবে

সব commands লিখুন step by step।

---

**Scenario 20. User Investigation:**

কেউ আপনাকে বললো `john` নামের একজন user system-এ আছে। আপনাকে বের করতে হবে:
- সে কোন groups-এ আছে?
- তার home directory কোথায়?
- তার account locked নাকি active?
- সে কতদিন ধরে password change করেনি?

শুধু commands লিখুন (actually john না থাকলেও commands জানলেই হবে)।

---

**Scenario 21. DevOps Automation:**

আপনি একটা script লিখতে চান যেটা:
- `deploy` নামে user আছে কিনা check করবে
- না থাকলে তৈরি করবে
- `deployers` group-এ add করবে
- sudo access দেবে (NOPASSWD)

শুধু ধাপগুলো এবং commands লিখুন।

---

**Scenario 22. File Permission Puzzle:**

একটা file-এর permission: `rwsr-xr-x` এবং owner `root`।
- এই file কে run করলে কার permission-এ চলবে?
- এটা কি safe? কখন এটা ব্যবহার হয়?

---

**Scenario 23. (BONUS 🌟)** `/etc/shadow` file-এর একটা line দেখুন:
```
devops:$6$xyz$hashedpassword:19500:0:90:7:30::
```
এই line থেকে বলুন:
- Password কি set আছে?
- Password কতদিন পর expire হবে?
- Expire হওয়ার কত দিন আগে warning আসবে?
- Account কতদিন পর disable হবে (expire-এর পর)?

---

**Scenario 24. (BONUS 🌟)** আপনি `su - devops` command দিলে কী হয়? শুধু `su devops` দিলে কী পার্থক্য হয়?

---

**Scenario 25. (BONUS 🌟)** নিচের দুটো command-এর মধ্যে পার্থক্য কী?
```bash
sudo su -
sudo -i
```

---

## ANSWER KEY
> ⚠️ আগে নিজে নিজে চেষ্টা করুন! তারপর এখানে দেখুন।

---

### SECTION A - Answers

**A1:**
```
-rwxr-x--x  breakdown:
- Owner (rwx)   → read, write, execute করতে পারবে
- Group (r-x)   → read আর execute পারবে, write পারবে না
- Others (--x)  → শুধু execute পারবে

Numeric value:
r=4, w=2, x=1
Owner:  4+2+1 = 7
Group:  4+0+1 = 5
Others: 0+0+1 = 1
→ 751
```

---

**A2:**
```
হ্যাঁ, দুটো same কাজ করে।
644 মানে:
- u = rw (4+2=6)
- g = r  (4=4)
- o = r  (4=4)

u=rw,go=r মানেও exactly এটাই।
শুধু notation আলাদা - একটা numeric, একটা symbolic।
```

---

**A3:**
```
SUID  → File run হলে owner-এর permission-এ চলে। 
        Example: /usr/bin/passwd (root-এর মতো চলে)

SGID  → File run হলে group-এর permission-এ চলে।
        Directory-তে দিলে নতুন file সেই group পায়।
        Example: /var/project/ (team directory)

Sticky Bit → Directory-তে নিজের file শুধু নিজে delete করতে পারবে।
             Example: /tmp (সবাই লিখতে পারে, কিন্তু অন্যেরটা মুছতে পারে না)
```

---

**A4:**
```
umask 027 মানে:
- File default max: 666
- Directory default max: 777

File:      666 - 027 = 640  → rw-r-----
Directory: 777 - 027 = 750  → rwxr-x---
```

---

**A5:**
```
/etc/passwd:
- সব user-এর basic info থাকে (username, UID, GID, home, shell)
- সবাই read করতে পারে
- Password field-এ শুধু 'x' লেখা থাকে

/etc/shadow:
- Actual hashed password থাকে
- Password aging info থাকে
- শুধু root read করতে পারে (security-র জন্য)
```

---

**A6 (Bonus):**
```
Hard Link:
- Same inode point করে
- Original file delete হলেও link কাজ করে
- Different filesystem-এ হয় না

Symbolic Link:
- আলাদা inode, original file-এর path point করে
- Original file delete হলে link broken হয়
- Different filesystem-এও হয়
```

---

### ⌨️ SECTION B - Answers

**Task 7:**
```bash
mkdir /tmp/devops_test
cd /tmp/devops_test
touch script.sh config.cfg data.txt

chmod 750 script.sh          # rwxr-x---
chmod 444 config.cfg         # r--r--r--
chmod g-w data.txt           # group থেকে write সরাও
```

---

**Task 8:**
```bash
useradd -m -d /home/devops -s /bin/bash -c "DevOps Engineer" devops

# Verify:
cat /etc/passwd | grep devops
id devops
```

---

**Task 9:**
```bash
groupadd developers
usermod -aG developers devops

# Verify:
groups devops
cat /etc/group | grep developers
```

---

**Task 10:**
```bash
chown devops:developers /tmp/devops_test/config.cfg

# Verify:
ls -l /tmp/devops_test/config.cfg
```

---

**Task 11:**
```bash
chmod +t /tmp/devops_test/
# অথবা
chmod 1755 /tmp/devops_test/

# Verify:
ls -ld /tmp/devops_test/
# Output-এ 't' দেখা যাবে: drwxr-xr-t
```

---

**Task 12:**
```bash
visudo
# নিচের line add করুন:
devops ALL=(ALL) NOPASSWD: /bin/systemctl
```

---

**Task 13:**
```bash
chmod u+s script.sh
# অথবা
chmod 4750 script.sh

# ls -l output:
-rwsr-x--- (s দেখা যাবে x-এর জায়গায়)
```

---

**Task 14:**
```bash
getfacl /tmp/devops_test/data.txt
# দেখাবে: owner, group, other permissions

# ACL দিয়ে devops user-কে permission দিন:
setfacl -m u:devops:rw /tmp/devops_test/data.txt

# Verify:
getfacl /tmp/devops_test/data.txt
```

---

**Task 15:**
```bash
passwd devops               # password set

passwd -l devops            # lock করুন

# Verify (locked হলে ! দেখাবে):
passwd -S devops
# অথবা
cat /etc/shadow | grep devops   # hash-এর আগে ! থাকবে
```

---

**Task 16:**
```bash
umask                        # বর্তমান value দেখুন (সাধারণত 022)
umask 077                    # change করুন

touch testfile_new.txt
ls -l testfile_new.txt
# Output: -rw------- (শুধু owner read+write)
```

---

### 🔥 SECTION C - Answers

**Scenario 17:**
```bash
# Sticky Bit দিতে হবে
chmod +t /var/www/html/
# এখন কেউ অন্যের file delete করতে পারবে না
```

---

**Scenario 18:**
```
এটা সম্পূর্ণ NORMAL এবং দরকারী।

কারণ: passwd command সব user ব্যবহার করে নিজের password change করতে।
কিন্তু password /etc/shadow-এ থাকে যেটা শুধু root লিখতে পারে।

SUID থাকার কারণে passwd চালালে সেটা root-এর permission-এ চলে,
তাই সাধারণ user-ও নিজের password change করতে পারে।

এটা intentional design - security risk না।
```

---

**Scenario 19:**
```bash
# Users তৈরি
useradd alice
useradd bob
useradd charlie

# Group তৈরি
groupadd webteam

# Users কে group-এ add করুন
usermod -aG webteam alice
usermod -aG webteam bob
usermod -aG webteam charlie

# Directory তৈরি
mkdir /var/project

# Ownership set করুন
chown root:webteam /var/project

# Permission: owner=rwx, group=rwx, others=---
chmod 770 /var/project

# SGID set করুন (নতুন file automatically webteam group পাবে)
chmod g+s /var/project

# Final verify:
ls -ld /var/project
# drwxrws--- (s দেখাবে group execute-এর জায়গায়)
```

---

**Scenario 20:**
```bash
# Groups দেখুন
groups john
id john

# Home directory দেখুন
grep john /etc/passwd

# Account locked কিনা দেখুন
passwd -S john

# Password aging info দেখুন
chage -l john
```

---

**Scenario 21:**
```bash
# User আছে কিনা check
id deploy 2>/dev/null || useradd -m -s /bin/bash deploy

# Group তৈরি ও add
groupadd deployers 2>/dev/null
usermod -aG deployers deploy

# Sudo access
echo "deploy ALL=(ALL) NOPASSWD: ALL" >> /etc/sudoers.d/deploy
```

---

**Scenario 22:**
```
rwsr-xr-x এবং owner root মানে SUID bit set।

এই file run করলে → ROOT-এর permission-এ চলবে
(যে-ই run করুক না কেন)

Safe কিনা: Depends করে।
- /usr/bin/passwd → safe (intentional)
- Random script-এ SUID → বিপজ্জনক! (privilege escalation attack হতে পারে)

Real DevOps-এ: SUID files নিয়মিত audit করা উচিত।
Command: find / -perm -4000 -type f 2>/dev/null
```

---

**Scenario 23 (BONUS):**
```
devops:$6$xyz$hashedpassword:19500:0:90:7:30::

Field breakdown:
- $6$ → SHA-512 hashing (password set আছে)
- 19500 → Last password change (epoch days থেকে date)
- 0 → Minimum days (0 মানে যেকোনো সময় change করা যাবে)
- 90 → Maximum days → 90 দিন পর password expire
- 7 → Warning → expire-এর 7 দিন আগে warning
- 30 → Inactivity → expire-এর 30 দিন পরেও না বদলালে account disable
```

---

**Scenario 24 (BONUS):**
```
su - devops:
- devops user-এ switch করে
- devops-এর environment load করে (home directory, PATH সব)
- Login shell শুরু হয়
- cd ~ করলে /home/devops যাবে

su devops:
- User switch করে কিন্তু
- আগের user-এর environment থাকে
- Current directory same থাকে
- PATH আগেরটাই থাকে

DevOps-এ সবসময় su - (dash সহ) use করা better practice।
```

---

**Scenario 25 (BONUS):**
```
sudo su -:
- প্রথমে current user-এর sudo দিয়ে su চালায়
- তারপর root-এর login shell শুরু হয়
- দুটো step

sudo -i:
- Directly root-এর login shell শুরু করে
- Cleaner এবং recommended way
- একটাই step

দুটোই root shell দেয়, কিন্তু sudo -i বেশি preferred
কারণ এটা আরো predictable environment দেয়।
```


## 🏆 নিজেকে Score দিন!

| Score Range | মানে |
|---|---|
| 23-25 | 🏆 Champion! Chapter 2 আপনার আয়ত্তে |
| 18-22 | 💪 খুব ভালো! একটু review করলেই perfect |
| 13-17 | 📖 ভালো, কিন্তু weak in this chapter তাহলে আবার প্রাকটিস করুন|
| 10-12 | 🔄 Chapter 2 আরেকবার revise করুন |

---

যেকোনো answer নিয়ে confused হলে/কোন প্রশ্ন থাকলে বলুন। আমি বিস্তারিত explain করবো! আমরা এখন তৃতীয় চ্যাপ্টার শুরু করবো।
