# Chapter 1 - Lesson 9: Input/Output Redirection & Pipes

**Chapter 1 | Lesson 9 of 10**

## 🎯 এই Lesson-এ কী শিখবো?

- Standard Input, Output, Error কী?
- Redirection operators: `>`, `>>`, `<`, `2>`, `&>`
- Pipes `|` কীভাবে কাজ করে
- `tee` command
- Real-world DevOps use cases

## "Data Flow" কী?

Linux-এ যখন আপনি কোনো command রান করেন, তখন তিনটা **channel** বা **stream** থাকে:

```
┌─────────────────────────────────────────────────────┐
│                                                     │
│   Keyboard ──► [ STDIN  - 0 ] ──►                   │
│                                  PROGRAM            │
│               [ STDOUT - 1 ] ──► Terminal (Normal)  │
│               [ STDERR - 2 ] ──► Terminal (Error)   │
│                                                     │
└─────────────────────────────────────────────────────┘
```

| Stream | নাম | Number | Default কোথায় যায়? |
|--------|-----|--------|---------------------|
| STDIN  | Standard Input  | 0 | Keyboard থেকে আসে |
| STDOUT | Standard Output | 1 | Terminal-এ দেখায় |
| STDERR | Standard Error  | 2 | Terminal-এ দেখায় (লাল রঙে) |

### Real-World Analogy:
> একটা রেস্টুরেন্ট কল্পনা করুন।
> - **STDIN** = Customer এর order (input)
> - **STDOUT** = খাবার যেটা টেবিলে আসে (normal output)
> - **STDERR** = Waiter যখন বলে "এই item নেই!" (error message)
>
> Redirection মানে হলো এই flow কে অন্যদিকে পাঠানো। যেমন খাবার টেবিলে না দিয়ে, একটা box-এ ভরে রাখা!


## STDOUT Redirection `>`

### কী করে?
Command-এর normal output কে **terminal-এ না দেখিয়ে**, একটা **file-এ লিখে দেয়**।
> ⚠️ সতর্কতা: `>` ইউজ করলে file-এ আগে থেকে কনটেন্ট থাকলে **সব মুছে দিয়ে** নতুন করে লেখে!

### Syntax:
```bash
command > filename
```

### Example:
```bash
echo "Hello, DevOps!" > output.txt
```
```bash
cat output.txt
```
```
Hello, DevOps!
```

আরেকটা example:
```bash
ls -l /etc > etc_list.txt
cat etc_list.txt
```
```
total 1048
-rw-r--r-- 1 root root    3028 Mar  1 10:00 adduser.conf
-rw-r--r-- 1 root root     411 Mar  1 10:00 apt
...
```
> Terminal-এ কিছু দেখাবে না, সব `etc_list.txt`-এ চলে গেছে!

### DevOps Use Case:
```bash
# Script-এর output log file-এ save করা
./deploy.sh > deploy_log.txt
```


## STDOUT Append `>>`

### কী করে?
File-এর **শেষে যোগ করে** বা append করে যে কারণে আগের content মুছে যায় না।

### Syntax:
```bash
command >> filename
```

### Example:
```bash
echo "Line 1" > myfile.txt
echo "Line 2" >> myfile.txt
echo "Line 3" >> myfile.txt
cat myfile.txt
```
```
Line 1
Line 2
Line 3
```

### `>` vs `>>` পার্থক্য:

| Operator | আগের content | কী করে |
|----------|-------------|--------|
| `>`  | মুছে দেয় ❌ | নতুন করে লেখে |
| `>>` | রেখে দেয় ✅ | শেষে যোগ করে |

### DevOps Use Case:
```bash
# প্রতিদিন log append করা
echo "$(date): Server is UP" >> server_status.log
```

## STDIN Redirection `<`

### কী করে?
Keyboard থেকে input নেওয়ার বদলে **file থেকে input নেয়**।

### Syntax:
```bash
command < filename
```

### Example:
```bash
# একটা file বানাই
cat > names.txt
Alice
Bob
Charlie
(বের হয়ে আসার জন্য Ctrl+D চাপুন)
```

```bash
# এখন file থেকে input দিন
sort < names.txt
```
```
Alice
Bob
Charlie
```
> `sort` command keyboard এর বদলে `names.txt` থেকে পড়লো!


## STDERR Redirection `2>`

### কী করে?
Error message গুলো terminal-এ না দেখিয়ে **file-এ পাঠায়**।

### Syntax:
```bash
command 2> error_file
```

### Example:
```bash
ls /nonexistent_folder 2> error.txt
cat error.txt
```
```
ls: cannot access '/nonexistent_folder': No such file or directory
```
> Terminal-এ কোনো error দেখাবে না, কারণ error message টি `error.txt` file-এ গেছে।


## STDOUT + STDERR একসাথে Redirect `&>`

### কী করে?
Normal output এবং error দুটোই **একই file-এ** পাঠায়।

### Syntax:
```bash
command &> all_output.txt
# অথবা পুরনো style:
command > all_output.txt 2>&1
```

### Example:
```bash
ls /etc /nonexistent &> combined.txt
cat combined.txt
```
```
ls: cannot access '/nonexistent': No such file or directory
/etc:
adduser.conf
apt
...
```

### DevOps Use Case:
```bash
# Cron job এর সব output capture করা
./backup.sh &> /var/log/backup.log
```

## Output ফেলে দেওয়া `/dev/null`

`/dev/null` হলো Linux-এর **"black hole"** এখানে যা পাঠাবেন, সব হারিয়ে যাবে!

```bash
# Error দেখতে না চাইলে
ls /nonexistent 2> /dev/null

# সব output দেখতে চাই না
./noisy_script.sh &> /dev/null
```

### Analogy:
> `/dev/null` হলো একটা আবর্জনার ঝুড়ি, যেটা নিজেই automatically খালি হয়ে যায় সবসময়!

## Pipe character `|`

এটাই Linux-এর সবচেয়ে **শক্তিশালী** feature!

### কী করে?
একটা command-এর **output** কে সরাসরি আরেকটা command-এর **input** হিসেবে দেয়।

```
Command1 ──► OUTPUT ──► | ──► INPUT ──► Command2
```

### Syntax:
```bash
command1 | command2
```

### Real-World Analogy:
> Factory assembly line ভাবুন
> - Machine 1 কাঁচামাল কাটে → output দেয়
> - Pipe সেই output কে Machine 2-তে পাঠায়
> - Machine 2 সেটা পালিশ করে → output দেয়
> - এভাবে chain করা যায়!

### Pipe Examples - এগুলো মুখস্থ করে ফেলতে হবে!

**Example 1: Long output filter করা**
```bash
ls -l /etc | less
```
> `/etc`-এর list টা `less`-এ দেখাবে, scroll করা যাবে।

**Example 2: নির্দিষ্ট কিছু খোঁজা**
```bash
ls /etc | grep "conf"
```
```
adduser.conf
ca-certificates.conf
debconf.conf
gai.conf
...
```
> "conf" আছে এমন সব file দেখাবে।

**Example 3: কতগুলো file আছে গণনা করা**
```bash
ls /etc | wc -l
```
```
167
```
> `/etc`-এ মোট ১৬৭টা item আছে।

**Example 4: Running process খোঁজা**
```bash
ps aux | grep "nginx"
```
```
root      1234  0.0  0.1  nginx: master process
www-data  1235  0.0  0.1  nginx: worker process
```
> সব process-এর মধ্যে `nginx` সংক্রান্ত filter গুলো হলো।

**Example 5: Multiple Pipes chain করা**
```bash
cat /etc/passwd | cut -d: -f1 | sort | head -5
```
```
_apt
backup
bin
daemon
games
```
> ধাপে ধাপে বোঝার চেস্টা করি:
> 1. `cat /etc/passwd` → পুরো file পড়লো
> 2. `cut -d: -f1` → প্রতি line থেকে username বের করলো
> 3. `sort` → alphabetically সাজালো
> 4. `head -5` → প্রথম ৫টা দেখালো


## `tee` Command

### কী করে?
Output একসাথে **terminal-এ দেখায়** এবং **file-এও লেখে**।

```
Command ──► OUTPUT ──► tee ──► Terminal (দেখা যায়)
                          └──► File (save হয়)
```

### Syntax:
```bash
command | tee filename
```

### Example:
```bash
ls /etc | tee etc_output.txt
```
```
adduser.conf
apt
bash.bashrc
...
(সব terminal-এ দেখাবে AND etc_output.txt-এও save হবে)
```

```bash
# Append করতে চাইলে -a flag
echo "New entry" | tee -a logfile.txt
```

### DevOps Use Case:
```bash
# Deploy script চালান এবং একই সাথে terminal-এ দেখা এবং log-ও রাখা
./deploy.sh | tee -a /var/log/deploy.log
```

### Analogy:
> `tee` হলো T-আকৃতির পানির pipe, পানি একদিক থেকে এসে **দুই দিকে** ভাগ হয়ে যায়!

## Here Document `<<`

একটু advanced কিন্তু অনেক কাজের।

### কী করে?
Multi-line input সরাসরি command-এ দেওয়া যায়।

```bash
cat << EOF
Hello World
This is a here document
Line 3
EOF
```
```
Hello World
This is a here document
Line 3
```

### DevOps Use Case:
```bash
# Script থেকে file তৈরি করা
cat << EOF > /etc/myapp/config.conf
host=localhost
port=8080
debug=false
EOF
```

## সব Operators একসাথে

| Operator | কী করে | Example |
|----------|--------|---------|
| `>` | STDOUT → file (overwrite) | `ls > out.txt` |
| `>>` | STDOUT → file (append) | `echo hi >> log.txt` |
| `<` | file → STDIN | `sort < names.txt` |
| `2>` | STDERR → file | `cmd 2> err.txt` |
| `2>>` | STDERR → file (append) | `cmd 2>> err.txt` |
| `&>` | STDOUT+STDERR → file | `cmd &> all.txt` |
| `\|` | cmd1 output → cmd2 input | `ls \| grep txt` |
| `tee` | output → terminal + file | `ls \| tee out.txt` |
| `/dev/null` | output ফেলে দেওয়া | `cmd 2> /dev/null` |


## Real-World DevOps Scenarios

### Scenario 1: Server Health Check Log
```bash
echo "=== $(date) ===" >> /var/log/health.log
df -h >> /var/log/health.log
free -m >> /var/log/health.log
echo "==================" >> /var/log/health.log
```

### Scenario 2: Error-only Log রাখা
```bash
./application_start.sh > /dev/null 2>> /var/log/app_errors.log
```
> Normal output দেখাবে না, শুধু error গুলো log-এ যাবে।

### Scenario 3: Top 5 Memory-consuming Process
```bash
ps aux --sort=-%mem | head -6
```

### Scenario 4: Unique IP থেকে কতবার Access হয়েছে
```bash
cat /var/log/nginx/access.log | awk '{print $1}' | sort | uniq -c | sort -rn | head -10
```

## 📝 Quick Summary

- ✅ **STDIN (0)** = Input source (default: keyboard)
- ✅ **STDOUT (1)** = Normal output (default: terminal)
- ✅ **STDERR (2)** = Error output (default: terminal)
- ✅ **`>`** = Output কে file-এ লেখে (overwrite করে)
- ✅ **`>>`** = Output কে file-এ যোগ করে (append)
- ✅ **`<`** = File থেকে input নেয়
- ✅ **`2>`** = Error কে file-এ পাঠায়
- ✅ **`&>`** = সব output (normal + error) file-এ পাঠায়
- ✅ **`|`** (pipe) = একটা command-এর output আরেকটা command-এর input হয়
- ✅ **`tee`** = Terminal-এ দেখায় + file-এ save করে
- ✅ **`/dev/null`** = Output ফেলে দেয় (black hole)


## 🏋️ Practice Tasks

এই তিনটা কাজ নিজে করুন:

**Task 1:**
```
/etc/hosts ফাইলের content terminal-এ দেখুন এবং
একই সাথে ~/hosts_backup.txt-এ save করুন।
(Hint: tee ব্যবহার করুন)
```

**Task 2:**
```
ps aux command রান করুন, তার output থেকে শুধু
"root" user এর processes filter করুন এবং
~/root_processes.txt ফাইলে save করুন।
(Hint: pipe + grep + redirection)
```

**Task 3:**
```
একটা file বানান যেখানে তিনটা line আছে:
"Start: $(date)"
df -h এর output
"End: $(date)"
সবকিছু ~/system_report.txt এ থাকবে।
(Hint: echo + >> এবং df -h >>)
```

## ⏭️ What's Next?

**Chapter 1 - Lesson 10: Archiving & Compression**
> `tar`, `gzip`, `gunzip`, `zip`, `unzip` ফাইল pack করা, compress করা, এবং extract করা। DevOps-এ backup নেওয়া এবং file transfer-এর জন্য অপরিহার্য!


আমরা এখন Linux-এর সবচেয়ে important concept গুলোর একটা শিখলাম! 🎉
Pipe এবং Redirection দিয়ে একজন DevOps Engineer অনেক জটিল কাজ মাত্র এক লাইনে করে ফেলতে পারে। Practice করতে থাকুন, না হলে শুধু পড়ে গেলে হবে না। প্র্যাকটিস না করলে কিছুই মনে থাকবে না!
