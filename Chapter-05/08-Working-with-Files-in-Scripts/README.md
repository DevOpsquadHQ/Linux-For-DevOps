# Chapter 5 - Lesson 8: Working with Files in Scripts

**Chapter 5 | Lesson 8 of 11**


## 🎯 এই Lesson-এ আমরা যা শিখব

Shell script-এর ভেতর থেকে file নিয়ে কাজ করা যেমন read করা, write করা, append করা, এবং file exist করে কিনা check করা। Real DevOps কাজে এটা প্রতিদিনের কাজ।


## কেন এটা জানা দরকার?

DevOps Engineer হিসেবে তোমাকে প্রতিদিন:
- Config file read করতে হবে
- Log file-এ data লিখতে হবে
- Backup script-এ file check করতে হবে
- Report generate করতে হবে

এই সব কাজ script দিয়ে automate করতে হলে file handling জানাটা must।

## এই Lesson-এর Structure

```
1. File exist করে কিনা check করা
2. File থেকে data read করা
3. File-এ data write করা
4. File-এ data append করা
5. Line-by-line read করা
6. File-এর properties check করা
7. Real DevOps example script
```

## File Exist করে কিনা Check করা

Script চালানোর আগে file আছে কিনা দেখা, না হলে error হবে। এটা একটা **defensive programming** technique।

### File Test Operators (Flags)

| Flag | মানে কী |
|------|---------|
| `-e` | File exist করে? (যেকোনো ধরনের) |
| `-f` | Regular file exist করে? |
| `-d` | Directory exist করে? |
| `-r` | File readable? |
| `-w` | File writable? |
| `-x` | File executable? |
| `-s` | File-এর size > 0? (empty না?) |
| `-L` | Symbolic link? |

### Syntax

```bash
if [ -f "/path/to/file" ]; then
    echo "File exists"
else
    echo "File does not exist"
fi
```

### উদাহরণ ১ - Basic file check

```bash
#!/bin/bash

FILE="/etc/passwd"

if [ -f "$FILE" ]; then
    echo "✅ File exists: $FILE"
else
    echo "❌ File not found: $FILE"
fi
```

**Output:**
```
✅ File exists: /etc/passwd
```

### উদাহরণ ২ - Multiple checks একসাথে

```bash
#!/bin/bash

FILE="data.txt"

if [ ! -e "$FILE" ]; then
    echo "❌ File does not exist at all"
elif [ ! -f "$FILE" ]; then
    echo "⚠️  Path exists but it's not a regular file"
elif [ ! -r "$FILE" ]; then
    echo "🔒 File exists but is not readable"
else
    echo "✅ File exists and is readable"
fi
```

> **`!` মানে NOT** অর্থাৎ condition উল্টে দেয়।

### উদাহরণ ৩ - Directory check

```bash
#!/bin/bash

DIR="/var/log"

if [ -d "$DIR" ]; then
    echo "Directory exists: $DIR"
else
    echo "Creating directory: $DIR"
    mkdir -p "$DIR"
fi
```

## File থেকে Data Read করা

### Method 1 - `cat` দিয়ে পুরো file read

```bash
#!/bin/bash

FILE="config.txt"

if [ -f "$FILE" ]; then
    content=$(cat "$FILE")
    echo "File content:"
    echo "$content"
fi
```

> **`$(...)`** মানে command-এর output একটা variable-এ রাখো। এটাকে **Command Substitution** বলে।

### Method 2 - `while read` দিয়ে Line-by-Line Read (সবচেয়ে গুরুত্বপূর্ণ)

এটা DevOps-এ সবচেয়ে বেশি ব্যবহার হয়।

```bash
#!/bin/bash

FILE="servers.txt"

while IFS= read -r line; do
    echo "Processing: $line"
done < "$FILE"
```

**প্রতিটা অংশ বুঝি:**

| অংশ | মানে |
|-----|------|
| `while` | Loop চালাতে থাকো |
| `IFS=` | Internal Field Separator খালি রাখো (যাতে leading/trailing space preserve হয়) |
| `read -r line` | এক লাইন পড়ো `line` variable-এ। `-r` মানে backslash `\` কে literal treat করো |
| `done < "$FILE"` | File-কে input হিসেবে দাও |


### Real Example - servers.txt থেকে server list read করে ping

**servers.txt:**
```
192.168.1.1
192.168.1.2
google.com
```

**Script:**
```bash
#!/bin/bash

FILE="servers.txt"

if [ ! -f "$FILE" ]; then
    echo "❌ servers.txt not found!"
    exit 1
fi

while IFS= read -r server; do
    if ping -c 1 -W 1 "$server" &>/dev/null; then
        echo "✅ $server is UP"
    else
        echo "❌ $server is DOWN"
    fi
done < "$FILE"
```

**Output:**
```
✅ 192.168.1.1 is UP
❌ 192.168.1.2 is DOWN
✅ google.com is UP
```

### Method 3 - Specific Line Read করা

```bash
#!/bin/bash

# File-এর ৩য় line পড়া
line=$(sed -n '3p' file.txt)
echo "Third line: $line"
```

```bash
# প্রথম line পড়া
first_line=$(head -n 1 file.txt)

# শেষ line পড়া
last_line=$(tail -n 1 file.txt)
```

## File-এ Data Write করা (Overwrite)

`>` operator দিয়ে file-এ লেখা হয়। পুরনো content মুছে যায়।

```bash
#!/bin/bash

OUTPUT_FILE="report.txt"

echo "=== System Report ===" > "$OUTPUT_FILE"
echo "Date: $(date)" >> "$OUTPUT_FILE"
echo "Hostname: $(hostname)" >> "$OUTPUT_FILE"

echo "✅ Report saved to $OUTPUT_FILE"
```

### `echo` vs `printf` - কোনটা ভালো?

```bash
# echo - simple, newline auto add করে
echo "Hello World" > file.txt

# printf - বেশি control, format string support করে
printf "Name: %s\nAge: %d\n" "Alice" 30 > file.txt
```

**printf output:**
```
Name: Alice
Age: 30
```

## File-এ Data Append করা

`>>` operator দিয়ে পুরনো content রেখে নতুন data যোগ করা হয়।

```bash
#!/bin/bash

LOG_FILE="app.log"

log_message() {
    local level="$1"
    local message="$2"
    local timestamp=$(date '+%Y-%m-%d %H:%M:%S')
    echo "[$timestamp] [$level] $message" >> "$LOG_FILE"
}

log_message "INFO"  "Application started"
log_message "WARN"  "High memory usage detected"
log_message "ERROR" "Database connection failed"

echo "✅ Logs written to $LOG_FILE"
```

**app.log এর content:**
```
[2025-03-15 10:30:00] [INFO]  Application started
[2025-03-15 10:30:01] [WARN]  High memory usage detected
[2025-03-15 10:30:02] [ERROR] Database connection failed
```

> এই pattern টা DevOps-এ *logging function* হিসেবে সব script-এ ব্যবহার হয়। এটা খেয়াল রাখা খুব জরুরি!

## `tee` দিয়ে Screen এবং File দুটোতেই লেখা

```bash
#!/bin/bash

echo "Deployment started at $(date)" | tee deploy.log
echo "Building Docker image..." | tee -a deploy.log
echo "Pushing to registry..." | tee -a deploy.log
```

| অংশ | মানে |
|-----|------|
| `tee` | Screen-এ দেখায় এবং file-এ লেখে (overwrite) |
| `tee -a` | Screen-এ দেখায় এবং file-এ append করে |

**Output (screen-এও দেখবেন, deploy.log-এও লেখা হবে):**

```
Deployment started at Sat Mar 15 10:30:00 UTC 2025
Building Docker image...
Pushing to registry...
```

## File-এর Properties Check করা

```bash
#!/bin/bash

FILE="data.csv"

# File size check করা
if [ -s "$FILE" ]; then
    echo "✅ File has content"
    
    # File size bytes-এ
    size=$(wc -c < "$FILE")
    echo  Size: $size bytes"
    
    # কত line আছে
    lines=$(wc -l < "$FILE")
    echo "📄 Lines: $lines"
    
    # কত word আছে
    words=$(wc -w < "$FILE")
    echo "🔤 Words: $words"
else
    echo "⚠️  File is empty!"
fi
```

### `stat` দিয়ে বিস্তারিত file info

```bash
stat myfile.txt
```

**Output:**
```
  File: myfile.txt
  Size: 1024        Blocks: 8     IO Block: 4096  regular file
Device: fd00h/64768d    Inode: 123456   Links: 1
Access: (0644/-rw-r--r--)  Uid: ( 1000/   user)   Gid: ( 1000/   user)
Access: 2025-03-15 10:00:00.000
Modify: 2025-03-15 09:55:00.000
Change: 2025-03-15 09:55:00.000
```

## File Read করে Process করা - CSV Example

DevOps-এ config বা inventory CSV file থেকে data নেওয়া খুব common।

**inventory.csv:**
```
server1,192.168.1.10,web
server2,192.168.1.11,db
server3,192.168.1.12,cache
```

**Script:**
```bash
#!/bin/bash

FILE="inventory.csv"

echo "=== Server Inventory ==="
echo ""

while IFS=',' read -r name ip role; do
    echo "🖥️  Server : $name"
    echo "   IP     : $ip"
    echo "   Role   : $role"
    echo "-------------------"
done < "$FILE"
```

> **`IFS=','`** মানে comma (`,`) দিয়ে প্রতিটা field আলাদা করুন।

**Output:**
```
=== Server Inventory ===

  Server : server1
   IP     : 192.168.1.10
   Role   : web
-------------------
  Server : server2
   IP     : 192.168.1.11
   Role   : db
-------------------
  Server : server3
   IP     : 192.168.1.12
   Role   : cache
-------------------
```

## Temp File ব্যবহার করা

Script-এর মধ্যে temporary file দরকার হলে `mktemp` ব্যবহার করুন। Script শেষে এটা delete হয়ে যায়।

```bash
#!/bin/bash

# Secure temp file তৈরি করো
TMPFILE=$(mktemp /tmp/myapp.XXXXXX)

echo "Temp file created: $TMPFILE"

# Temp file-এ কিছু লেখো
echo "temporary data" > "$TMPFILE"

# কাজ করো...
cat "$TMPFILE"

# শেষে delete করো (trap দিয়ে এমনকি error হলেও delete হবে)
trap "rm -f $TMPFILE" EXIT

echo "Done. Temp file will be cleaned up."
```

> **`trap "command" EXIT`** মানে - script যেভাবেই শেষ হোক (normal বা error), `command` execute হবে। এটা cleanup-এর জন্য best practice।

## Real-World DevOps Script - Config File Reader & Validator

এই script টা একটা config file পড়বে, validate করবে, এবং report তৈরি করবে।

**app.conf:**
```
DB_HOST=localhost
DB_PORT=5432
DB_NAME=myapp
APP_PORT=8080
DEBUG=false
```

**Script: validate_config.sh**
```bash
#!/bin/bash

CONFIG_FILE="app.conf"
REPORT_FILE="config_report.txt"
REQUIRED_KEYS=("DB_HOST" "DB_PORT" "DB_NAME" "APP_PORT")

# ---- Functions ----

log() {
    echo "[$(date '+%H:%M:%S')] $1" | tee -a "$REPORT_FILE"
}

check_config() {
    local key="$1"
    local value

    value=$(grep "^${key}=" "$CONFIG_FILE" | cut -d'=' -f2)

    if [ -z "$value" ]; then
        log "❌ MISSING: $key"
        return 1
    else
        log "✅ FOUND:   $key = $value"
        return 0
    fi
}

# ---- Main ----

# পুরনো report মুছে নতুন করে শুরু করুন
> "$REPORT_FILE"

log "=== Config Validation Report ==="
log "Config file: $CONFIG_FILE"
log ""

# Config file exist করে কিনা
if [ ! -f "$CONFIG_FILE" ]; then
    log "❌ Config file not found: $CONFIG_FILE"
    exit 1
fi

# Config file empty কিনা
if [ ! -s "$CONFIG_FILE" ]; then
    log "⚠️  Config file is empty!"
    exit 1
fi

# প্রতিটা required key check করো
errors=0
for key in "${REQUIRED_KEYS[@]}"; do
    if ! check_config "$key"; then
        ((errors++))
    fi
done

log ""
if [ "$errors" -eq 0 ]; then
    log "🎉 All required keys found. Config is VALID."
else
    log "💥 $errors key(s) missing. Config is INVALID."
fi

log ""
log "Report saved to: $REPORT_FILE"
```

**Output:**
```
[10:30:00] === Config Validation Report ===
[10:30:00] Config file: app.conf
[10:30:00]
[10:30:00] FOUND:   DB_HOST = localhost
[10:30:00] FOUND:   DB_PORT = 5432
[10:30:00] FOUND:   DB_NAME = myapp
[10:30:00] FOUND:   APP_PORT = 8080
[10:30:00]
[10:30:00] All required keys found. Config is VALID.
[10:30:00]
[10:30:00] Report saved to: config_report.txt
```

## Quick Reference - File Handling Cheatsheet

| কাজ | Command/Method |
|-----|----------------|
| File exist check | `[ -f "$file" ]` |
| Directory exist check | `[ -d "$dir" ]` |
| File empty কিনা | `[ -s "$file" ]` (empty হলে false) |
| পুরো file read | `content=$(cat file)` |
| Line-by-line read | `while IFS= read -r line; do ... done < file` |
| File-এ write (overwrite) | `echo "text" > file` |
| File-এ append | `echo "text" >> file` |
| Screen + file দুটোতে | `echo "text" \| tee -a file` |
| Line count | `wc -l < file` |
| File size | `wc -c < file` |
| Temp file | `mktemp /tmp/app.XXXXXX` |
| Cleanup on exit | `trap "rm -f $tmp" EXIT` |


## 📝 Lesson Summary

- **`[ -f ]`, `[ -d ]`, `[ -s ]`** - file এর existence এবং properties check করে
- **`while IFS= read -r line`** - file line-by-line read করার best practice
- **`>`** - overwrite করে, **`>>`** - append করে
- **`tee -a`** - screen এবং file দুটোতেই output পাঠায়
- **`wc -l`, `wc -c`** - line ও size count করে
- **`mktemp` + `trap`** - safe temporary file handling
- **`IFS=','`** - CSV file parse করতে ব্যবহার হয়
- **`$(cat file)` বা `$(command)`** - command substitution দিয়ে output variable-এ রাখে


## 🏋️ Practice Tasks

**Task 1:**
একটা file `fruits.txt` তৈরি করুন যেখানে প্রতিটা line-এ একটা fruit-এর নাম আছে (কমপক্ষে ৫টা)। একটা script লিখুন যেটা file পড়বে এবং প্রতিটা line-এর সামনে একটা নম্বর যোগ করে print করবে (1. Apple, 2. Banana...)।

**Task 2:**
একটা script লিখুন `disk_check.sh` যেটা:
- `df -h` এর output একটা file `disk_report.txt`-এ save করবে
- সেই file কত line আছে তা বলবে
- File-এর size bytes-এ বলবে

**Task 3:**
একটা CSV file `users.csv` তৈরি করুন এই format-এ:
```
alice,admin,active
bob,developer,inactive
carol,devops,active
```

একটা script লিখুন যেটা শুধু `active` user-দের নাম এবং role print করবে।

---

## ⏭️ What's Next

**Chapter 5 - Lesson 9: Error Handling & Debugging**
`set -euo pipefail`, `bash -x`, এবং `trap` দিয়ে script-কে robust এবং debug-friendly করা, real production script-এর জন্য must-know! *Happy Learning* 🚀


<table width="100%">
  <tr>
    <td align="left">
      <a href="../07-Text-Processing">← Text Processing</a>
    </td>
    <td align="right">
      <a href="../09-Error-Handling-N-Debugging">Error Handling &amp; Debugging →</a>
    </td>
  </tr>
</table>