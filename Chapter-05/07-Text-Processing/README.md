# Chapter 5 - Lesson 7: Text Processing 

**Chapter 5 | Lesson 7 of 11**

আপনি এতদিনে Shell Scripting-এর অনেক কিছুই শিখে ফেলেছেন যেমন variables, loops, functions, arrays ইত্যাদি। আজকের lesson টা একটু বিশেষ কারণ এই tools গুলো ছাড়া কোনো real-world DevOps কাজই সম্পূর্ণ হয় না।

> কল্পনা করুন আপনার কাছে একটা বিশাল log file আছে, লাখো লাইন। আপনি কি সবগুলো লাইন এক এক করে পড়বেন? না! আপনি এই tools দিয়ে মুহূর্তের মধ্যে যা দরকার তা বের করে দিবে। এগুলো হলো আপনার **"টেক্সট-এর ছুরি-কাঁচি"**

## আজকের Topics

| Tool | কাজ |
|------|-----|
| `grep` | Pattern খোঁজা |
| `awk` | Column/Field processing |
| `sed` | Find & Replace / Edit |
| `cut` | নির্দিষ্ট column কাটা |
| `sort` | Sort করা |
| `uniq` | Duplicate সরানো |
| `tr` | Character replace/delete |


## Practice File তৈরি করুন

আজকের সব example-এ এই file টা ব্যবহার করবো। তাই আগে থেকেই এটা তৈরি করে রাখুন:

```bash
cat > employees.txt << 'EOF'
Alice   Developer   75000   Dhaka
Bob     DevOps      90000   Chittagong
Charlie Developer   80000   Dhaka
Diana   Manager     95000   Sylhet
Eve     DevOps      88000   Dhaka
Frank   Developer   72000   Rajshahi
Grace   Manager     100000  Dhaka
Henry   DevOps      85000   Chittagong
EOF
```

এবং একটা log file:

```bash
cat > server.log << 'EOF'
2024-01-15 10:23:01 ERROR Database connection failed
2024-01-15 10:23:45 INFO  Server started successfully
2024-01-15 10:25:12 ERROR Disk space critically low
2024-01-15 10:26:00 WARN  Memory usage at 85%
2024-01-15 10:27:33 INFO  Backup completed
2024-01-15 10:28:11 ERROR Cannot connect to Redis
2024-01-15 10:29:55 INFO  Health check passed
2024-01-15 10:30:22 WARN  CPU usage at 90%
EOF
```


## `grep` - Pattern খোঁজার Master Tool

`grep` মানে Globally search for a Regular Expression and Print।

সহজ কথায় কোনো একটা file-এ নির্দিষ্ট কোনো word বা pattern আছে কিনা খুঁজে বের করে।

### Basic Syntax:
```bash
grep [options] "pattern" filename
```

### Basic Examples:

```bash
# "DevOps" শব্দ খোঁজো employees.txt-এ
grep "DevOps" employees.txt
```
**Output:**
```
Bob     DevOps      90000   Chittagong
Eve     DevOps      88000   Dhaka
Henry   DevOps      85000   Chittagong
```

```bash
# Case-insensitive search (-i)
grep -i "devops" employees.txt
# "devops", "DevOps", "DEVOPS" - সব match করবে
```

```bash
# Line number সহ দেখাও (-n)
grep -n "ERROR" server.log
```
**Output:**
```
1:2024-01-15 10:23:01 ERROR Database connection failed
3:2024-01-15 10:25:12 ERROR Disk space critically low
6:2024-01-15 10:28:11 ERROR Cannot connect to Redis
```

```bash
# Pattern যেসব line-এ নেই সেগুলো দেখাও (-v = invert)
grep -v "ERROR" server.log
# ERROR বাদে বাকি সব line দেখাবে
```


```bash
# কতবার pattern পাওয়া গেছে শুধু সেই count দেখাও (-c)
grep -c "ERROR" server.log
```
**Output:**
```
3
```


```bash
# Recursive search - সব folder/file-এ খোঁজো (-r)
grep -r "DevOps" /home/user/
# পুরো directory তে খুঁজবে
```

```bash
# Multiple patterns একসাথে (-E বা egrep)
grep -E "ERROR|WARN" server.log
```
**Output:**
```
2024-01-15 10:23:01 ERROR Database connection failed
2024-01-15 10:25:12 ERROR Disk space critically low
2024-01-15 10:26:00 WARN  Memory usage at 85%
2024-01-15 10:28:11 ERROR Cannot connect to Redis
2024-01-15 10:30:22 WARN  CPU usage at 90%
```

```bash
# শুধু matching word টা দেখাও, পুরো line না (-o)
grep -o "ERROR\|WARN" server.log
```

```bash
# Match-এর আগের ও পরের lines দেখাও (context)
grep -A 2 "ERROR" server.log   # After: পরের 2 line
grep -B 1 "ERROR" server.log   # Before: আগের 1 line
grep -C 1 "ERROR" server.log   # Context: আগে-পরে 1 line
```

### DevOps Use Cases:
```bash
# Production log থেকে error বের করো
grep "ERROR" /var/log/app.log

# কোনো process চলছে কিনা দেখো
ps aux | grep nginx

# Config file-এ কোনো setting আছে কিনা দেখো
grep "PermitRootLogin" /etc/ssh/sshd_config

# যেসব IP address access করেছে
grep "Failed password" /var/log/auth.log
```

## `awk` - Powerful Column Processor

`awk` হলো একটি পুরো programming language! এটা মূলত **column/field-based processing** করে।

> কল্পনা করুন Excel spreadsheet। `awk` হলো সেই tool যা আপনাকে যেকোনো column নিয়ে কাজ করতে দেয়।

### Basic Concept:
`awk` প্রতিটা line কে **fields** এ ভাগ করে।
- `$1` = প্রথম column
- `$2` = দ্বিতীয় column
- `$NF` = শেষ column
- `$0` = পুরো line

Default delimiter হলো **space বা tab**।

### Basic Syntax:
```bash
awk 'pattern { action }' filename
```

### Basic Examples:

```bash
# শুধু প্রথম column (নাম) দেখাও
awk '{ print $1 }' employees.txt
```
**Output:**
```
Alice
Bob
Charlie
Diana
Eve
Frank
Grace
Henry
```

```bash
# নাম এবং salary দেখাও (column 1 এবং 3)
awk '{ print $1, $3 }' employees.txt
```
**Output:**
```
Alice 75000
Bob 90000
Charlie 80000
...
```

```bash
# Custom format দিয়ে print করো
awk '{ print "Name: " $1 "\tSalary: " $3 }' employees.txt
```
**Output:**
```
Name: Alice     Salary: 75000
Name: Bob       Salary: 90000
```

```bash
# Condition দিয়ে filter করো - DevOps দের দেখাও
awk '$2 == "DevOps" { print $1, $3 }' employees.txt
```
**Output:**
```
Bob 90000
Eve 88000
Henry 85000
```

```bash
# Salary 85000-এর বেশি যাদের
awk '$3 > 85000 { print $1, $2, $3 }' employees.txt
```
**Output:**
```
Bob DevOps 90000
Diana Manager 95000
Eve DevOps 88000
Grace Manager 100000
```

```bash
# Custom delimiter ব্যবহার করো (-F)
# /etc/passwd file comma-separated নয়, colon(:) separated
awk -F: '{ print $1, $3 }' /etc/passwd
# username এবং UID দেখাবে
```

```bash
# সব salary এর মোট যোগফল বের করো
awk '{ total += $3 } END { print "Total Salary:", total }' employees.txt
```
**Output:**
```
Total Salary: 685000
```


```bash
# Line count (NR = Number of Records)
awk 'END { print "Total employees:", NR }' employees.txt
```
**Output:**
```
Total employees: 8
```

```bash
# Log file থেকে ERROR গুলো আলাদা করো
awk '/ERROR/ { print NR": "$0 }' server.log
```
**Output:**
```
1: 2024-01-15 10:23:01 ERROR Database connection failed
3: 2024-01-15 10:25:12 ERROR Disk space critically low
6: 2024-01-15 10:28:11 ERROR Cannot connect to Redis
```

```bash
# BEGIN এবং END block
awk 'BEGIN { print "=== Employee Report ===" }
     { print $1, $2, $3 }
     END { print "=== End of Report ===" }' employees.txt
```
**Output:**
```
=== Employee Report ===
Alice Developer 75000
Bob DevOps 90000
...
=== End of Report ===
```

### DevOps Use Cases:
```bash
# Disk usage থেকে শুধু percentage বের করো
df -h | awk '{ print $5, $6 }'

# Process list থেকে memory usage দেখো
ps aux | awk '{ print $2, $4, $11 }'

# Apache log থেকে IP address গুলো বের করো
awk '{ print $1 }' /var/log/apache2/access.log
```

## `sed` - Stream Editor (Find & Replace Master)

`sed` মানে **S**tream **Ed**itor। এটা file-এর content modify করতে পারে - মূলত **find & replace** এর জন্য বিখ্যাত।

> Microsoft Word-এর "Find & Replace" (Ctrl+H) কিন্তু terminal-এ এবং অনেক বেশি powerful!

### Basic Syntax:
```bash
sed 'command' filename
sed 's/old/new/' filename    # s = substitute
```

### Basic Examples:

```bash
# "Developer" কে "Engineer" দিয়ে replace করো
sed 's/Developer/Engineer/' employees.txt
```
**Output:**
```
Alice   Engineer   75000   Dhaka
Bob     DevOps      90000   Chittagong
Charlie Engineer   80000   Dhaka
...
```

⚠️ **মনে রাখুন:** এটা শুধু প্রথম occurrence replace করে প্রতিটা line-এ।


```bash
# সব occurrence replace করো (g = global)
sed 's/Developer/Engineer/g' employees.txt
```


```bash
# Case-insensitive replace (I flag)
sed 's/developer/Engineer/gI' employees.txt
```


```bash
# File কে directly modify করো (-i flag)
sed -i 's/Developer/Engineer/g' employees.txt
# এটা original file বদলে দেবে!

# Safe way - backup রেখে modify করো
sed -i.bak 's/Developer/Engineer/g' employees.txt
# employees.txt.bak নামে backup তৈরি হবে
```


```bash
# নির্দিষ্ট line number-এ কাজ করো
sed '3s/Developer/Senior Developer/' employees.txt
# শুধু 3 নম্বর line-এ replace হবে
```


```bash
# নির্দিষ্ট line delete করো (d = delete)
sed '2d' employees.txt
# 2 নম্বর line delete হবে

# Pattern match করা line delete করো
sed '/ERROR/d' server.log
# ERROR আছে এমন সব line delete হবে
```


```bash
# নির্দিষ্ট line print করো (p = print, -n = suppress other lines)
sed -n '2,4p' employees.txt
# 2 থেকে 4 নম্বর line দেখাবে
```

```bash
# Line-এর শুরুতে কিছু add করো
sed 's/^/>> /' employees.txt
# প্রতিটা line-এর শুরুতে ">> " যোগ হবে
```

```bash
# Line-এর শেষে কিছু add করো
sed 's/$/ [BD]/' employees.txt
# প্রতিটা line-এর শেষে " [BD]" যোগ হবে
```

```bash
# Multiple commands একসাথে (-e)
sed -e 's/Developer/Engineer/g' -e 's/Manager/Lead/g' employees.txt
```

### DevOps Use Cases:
```bash
# Config file-এ port number change করো
sed -i 's/port=8080/port=9090/' /etc/app/config.conf

# Comment করা line uncomment করো
sed -i 's/^#PermitRootLogin/PermitRootLogin/' /etc/ssh/sshd_config

# Log file থেকে sensitive data মুছে দাও
sed 's/password=[^ ]*/password=***HIDDEN***/g' app.log
```

## 4️⃣ `cut` - Column কাটার Simple Tool

`cut` হলো `awk` এর ছোট ভাই শুধু নির্দিষ্ট column বা character বের করার জন্য।

### Basic Syntax:
```bash
cut [options] filename
```

| Option | মানে |
|--------|------|
| `-f` | Field number (column) |
| `-d` | Delimiter (default: tab) |
| `-c` | Character position |

### Examples:

```bash
# Tab-separated file থেকে 1ম এবং 3য় column বের করো
cut -f1,3 employees.txt
```
**Output:**
```
Alice   75000
Bob     90000
Charlie 80000
```

```bash
# Custom delimiter দিয়ে - /etc/passwd থেকে username বের করো
cut -d: -f1 /etc/passwd
```
**Output:**
```
root
daemon
bin
...
```

```bash
# /etc/passwd থেকে username এবং home directory
cut -d: -f1,6 /etc/passwd
```

```bash
# Character position দিয়ে কাটো
echo "Hello World" | cut -c1-5
```
**Output:**
```
Hello
```

```bash
# Log file থেকে date বের করো
cut -d' ' -f1 server.log
```
**Output:**
```
2026-03-24
2026-03-24
...
```

## `sort` - Sort করার Tool

### Basic Syntax:
```bash
sort [options] filename
```

### Examples:

```bash
# Alphabetically sort করো
sort employees.txt
```

```bash
# Reverse order-এ sort (-r)
sort -r employees.txt
```

```bash
# নির্দিষ্ট column দিয়ে sort করো (-k = key)
sort -k3 employees.txt
# 3য় column (salary) দিয়ে sort করবে
```

```bash
# Numeric sort (-n) - number হিসেবে sort
sort -k3 -n employees.txt
# Salary ascending order-এ
```

```bash
# Numeric + Reverse (highest salary first)
sort -k3 -nr employees.txt
```
**Output:**
```
Grace   Manager     100000  Dhaka
Diana   Manager     95000   Sylhet
Bob     DevOps      90000   Chittagong
...
```

```bash
# Unique lines শুধু রাখো (-u)
sort -u employees.txt
```

## `uniq` - Duplicate সরানোর Tool

⚠️ **Important:** `uniq` শুধু adjacent (পাশাপাশি) duplicate সরায়। তাই আগে `sort` করতে হয়!

### Examples:

```bash
# City গুলো বের করো এবং duplicate সরাও
awk '{ print $4 }' employees.txt | sort | uniq
```
**Output:**
```
Chittagong
Dhaka
Rajshahi
Sylhet
```

```bash
# প্রতিটা city কতবার আছে গণনা করো (-c)
awk '{ print $4 }' employees.txt | sort | uniq -c
```
**Output:**
```
      2 Chittagong
      4 Dhaka
      1 Rajshahi
      1 Sylhet
```

```bash
# শুধু duplicate lines দেখাও (-d)
awk '{ print $4 }' employees.txt | sort | uniq -d
```
**Output:**
```
Chittagong
Dhaka
```

```bash
# শুধু unique (একবারই আছে এমন) lines দেখাও (-u)
awk '{ print $4 }' employees.txt | sort | uniq -u
```
**Output:**
```
Rajshahi
Sylhet
```

### DevOps Use Case:
```bash
# কতগুলো unique IP আমার server-এ access করেছে
awk '{ print $1 }' /var/log/nginx/access.log | sort | uniq -c | sort -nr | head -10
```

## `tr` - Character Translate/Replace

`tr` মানে **tr**anslate। এটা character-level replacement করে।

### Basic Syntax:
```bash
echo "text" | tr 'old_chars' 'new_chars'
```

### Examples:

```bash
# Lowercase থেকে Uppercase
echo "hello world" | tr 'a-z' 'A-Z'
```
**Output:**
```
HELLO WORLD
```

```bash
# Uppercase থেকে Lowercase
echo "HELLO WORLD" | tr 'A-Z' 'a-z'
```
**Output:**
```
hello world
```

```bash
# Space কে underscore দিয়ে replace করো
echo "hello world foo" | tr ' ' '_'
```
**Output:**
```
hello_world_foo
```

```bash
# নির্দিষ্ট character delete করো (-d)
echo "Hello, World! 123" | tr -d '0-9'
```
**Output:**
```
Hello, World! 
```

```bash
# Duplicate characters squeeze করো (-s)
echo "heeello    woorld" | tr -s 'eo '
```
**Output:**
```
helo world
```

```bash
# Newline কে space দিয়ে replace করো
cat employees.txt | tr '\n' ' '
# সব line এক line-এ চলে আসবে
```

## Combining Tools - Real Power!

এই tools গুলোর আসল শক্তি হলো এগুলোকে **pipe (|)** দিয়ে একসাথে ব্যবহার করা।

### Scenario 1: Top 3 highest paid employees কারা?
```bash
sort -k3 -nr employees.txt | head -3
```
**Output:**
```
Grace   Manager     100000  Dhaka
Diana   Manager     95000   Sylhet
Bob     DevOps      90000   Chittagong
```

### Scenario 2: কতজন কোন role-এ আছে?
```bash
awk '{ print $2 }' employees.txt | sort | uniq -c | sort -nr
```
**Output:**
```
      3 Developer
      3 DevOps
      2 Manager
```

### Scenario 3: Log থেকে error summary তৈরি করো
```bash
grep "ERROR" server.log | awk '{ print $4, $5, $6, $7 }' | sort | uniq -c
```
**Output:**
```
      1 Cannot connect to Redis
      1 Database connection failed
      1 Disk space critically low
```

### Scenario 4: সব Dhaka-তে থাকা employees-দের salary-র গড়
```bash
awk '$4 == "Dhaka" { total += $3; count++ } END { print "Average:", total/count }' employees.txt
```
**Output:**
```
Average: 83250
```

### Scenario 5: /etc/passwd থেকে regular users (UID >= 1000) দের list
```bash
awk -F: '$3 >= 1000 { print $1, $3, $6 }' /etc/passwd
```

### Scenario 6: একটা config file থেকে comment ও blank line সরিয়ে দেখাও
```bash
grep -v '^#' /etc/ssh/sshd_config | grep -v '^$'
```

## Quick Reference Table

| Tool | Best For | Example |
|------|----------|---------|
| `grep` | Pattern/word খোঁজা | `grep "ERROR" app.log` |
| `awk` | Column processing, math | `awk '{print $1,$3}' file` |
| `sed` | Find & Replace, edit | `sed 's/old/new/g' file` |
| `cut` | Simple column extract | `cut -d: -f1 /etc/passwd` |
| `sort` | Sort lines | `sort -k3 -nr file` |
| `uniq` | Remove duplicates | `sort file \| uniq -c` |
| `tr` | Character replace | `echo "hi" \| tr 'a-z' 'A-Z'` |

## 📝 Quick Summary

- **`grep`** - file-এ pattern খোঁজে, `-i` case-insensitive, `-v` invert, `-r` recursive
- **`awk`** - column-based processing, condition দিয়ে filter, math করতে পারে
- **`sed`** - find & replace master, line delete/add, `-i` দিয়ে file directly edit
- **`cut`** - simple column extract, `-d` delimiter, `-f` field number
- **`sort`** - alphabetical/numeric sort, `-k` column, `-r` reverse, `-n` numeric
- **`uniq`** - duplicate সরায়, আগে `sort` করতে হয়, `-c` count দেখায়
- **`tr`** - character-level replace/delete, case conversion করে
- **Pipe (`|`)** - এই সব tools কে একসাথে জুড়ে দেওয়াই আসল power

## 🏋️ Practice Tasks

**Task 1:** `employees.txt` থেকে শুধু **DevOps** role-এর employees-দের নাম ও salary বের করুন, এবং salary অনুযায়ী descending order-এ sort করুন।

**Task 2:** `server.log` থেকে শুধু WARN এবং ERROR entries বের করুন, তারপর প্রতিটার timestamp (প্রথম দুটো column) এবং message দেখান।

**Task 3:** `/etc/passwd` file থেকে সব users-এর username ও home directory বের করুন (`cut` ব্যবহার করুন), তারপর alphabetically sort করুন এবং output-কে uppercase-এ convert করুন (`tr` ব্যবহার করুন)।

---

## ⏭️ What's Next

**Chapter 5 - Lesson 8: Working with Files in Scripts**
> কিভাবে script-এ files read করবে, write করবে, append করবে এবং file exist করে কিনা check করবে - সব দেখবো real-world examples সহ! *Happy Learning* 🚀


<table width="100%">
  <tr>
    <td align="left">
      <a href="../06-Arrays-String-Manipulation">← Arrays &amp; String Manipulation</a>
    </td>
    <td align="right">
      <a href="../08-Working-with-Files-in-Scripts">Working with Files in Scripts →</a>
    </td>
  </tr>
</table>