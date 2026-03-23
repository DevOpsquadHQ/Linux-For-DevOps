# Chapter 5 - Lesson 3: Conditionals (if, elif, else, test, [ ], [[ ]])

**Chapter 5 | Lesson 3 of 11**


## 🎯 এই Lesson-এ কী শিখবো?

- `if`, `elif`, `else` দিয়ে decision making
- `test` command কী এবং কীভাবে কাজ করে
- `[ ]` vs `[[ ]]` - পার্থক্য ও সঠিক ব্যবহার
- Numeric comparison, String comparison, File conditions
- Real-world DevOps examples


## Conditional কী?

ধরুন আপনি একজন Security Guard। যদি visitor-এর ID card থাকে তাহলে ভেতরে ঢুকতে দেন আর  যদি না থাকে তাহলে আর ভিতরে ঢুকতে দেন না বা বাইরে রাখেন

এটাই হলো conditional logic, একটা condition check করো, তারপর সিদ্ধান্ত নাও।

Bash scripting-এ condition-ও ঠিক এভাবেই কাজ করে।


## Basic `if` Syntax

```bash
if [ condition ]; then
    # condition true হলে এটা run হবে
fi
```

**মনে রাখুন:**
> - `if` এর পরে `then` লাগবেই
> - শেষে `fi` দিয়ে block বন্ধ করতে হবে (`if` উল্টো করলে `fi`)
> - `[ condition ]` - এর ভেতরে **দুই পাশে space** দিতেই হবে


```bash
age=60

if [ age ]; then
    echo $age
fi
```

## if / elif / else Structure

```bash
if [ condition1 ]; then
    # condition1 true হলে
elif [ condition2 ]; then
    # condition2 true হলে
else
    # কোনোটাই true না হলে
fi
```

### Simple Example:

```bash
#!/bin/bash

age=20

if [ $age -ge 18 ]; then
    echo "You are an adult"
else
    echo "You are a underage"
fi
```

**Output:**
```
You are an adult
```


## 🔢 Numeric Comparison Operators

| Operator | মানে | Example |
|----------|------|---------|
| `-eq` | equal (==) | `[ $a -eq $b ]` |
| `-ne` | not equal (!=) | `[ $a -ne $b ]` |
| `-gt` | greater than (>) | `[ $a -gt $b ]` |
| `-lt` | less than (<) | `[ $a -lt $b ]` |
| `-ge` | greater than or equal (>=) | `[ $a -ge $b ]` |
| `-le` | less than or equal (<=) | `[ $a -le $b ]` |

### Example - Disk Usage Alert:

```bash
#!/bin/bash

disk_usage=85

if [ $disk_usage -ge 90 ]; then
    echo "🔴 CRITICAL: The disk is almost full."
elif [ $disk_usage -ge 75 ]; then
    echo "🟡 WARNING: Disk usage is high"
else
    echo "🟢 OK: Normal disk usage"
fi
```

**Output:**
```
🟡 WARNING: Disk usage is high
```

> **DevOps Use Case:** Production server-এ disk 90% ভরে গেলে auto-alert পাঠাতে হবে।


## String Comparison Operators

| Operator | মানে | Example |
|----------|------|---------|
| `=` | equal | `[ "$a" = "$b" ]` |
| `!=` | not equal | `[ "$a" != "$b" ]` |
| `-z` | string empty (zero length) | `[ -z "$a" ]` |
| `-n` | string not empty | `[ -n "$a" ]` |

> String comparison-এ variable সবসময় **"double quotes"** এ রাখুন, অন্যথায় error আসতে পারে।

### Example - Environment Check:

```bash
#!/bin/bash

environment="production"

if [ "$environment" = "production" ]; then
    echo "Production environment! Be careful with your work।"
elif [ "$environment" = "staging" ]; then
    echo "Staging environment - test properly to be sure"
else
    echo "Development environment"
fi
```

**Output:**
```
Production environment! Be careful with your work।
```


## File Test Operators - DevOps-এ সবচেয়ে বেশি ব্যবহৃত!

| Operator | মানে |
|----------|------|
| `-f` | file exist করে এবং regular file |
| `-d` | directory exist করে |
| `-e` | file বা directory exist করে (যেকোনো) |
| `-r` | file readable |
| `-w` | file writable |
| `-x` | file executable |
| `-s` | file exist করে এবং empty না |

### Example - Backup Script:

```bash
#!/bin/bash

backup_dir="/var/backups/myapp"

if [ -d "$backup_dir" ]; then
    echo "Backup directory exists, backup starting..."
else
    echo "Directory does not exist, creating..."
    mkdir -p "$backup_dir"
    echo "Directory created!"
fi
```

### Example - Config File Check:

```bash
#!/bin/bash

config="/etc/myapp/config.yml"

if [ ! -f "$config" ]; then
    echo "Config file not found: $config"
    echo "Script closing."
    exit 1
fi

echo "Config file found, starting..."
```

> `!` মানে **NOT** - condition উল্টে দেয়


## `test` Command - `[ ]` এর আসল রূপ

`[ condition ]` আসলে `test` command-এর একটা shorthand।

এই দুটো **একই কাজ করে:**

```bash
test -f /etc/passwd
[ -f /etc/passwd ]
```

আপনি চাইলে directly `test` command-ও ব্যবহার করতে পারেন:

```bash
#!/bin/bash

if test -f "/etc/passwd"; then
    echo "File exists"
fi
```

কিন্তু `[ ]` syntax বেশি পরিচিত ও বহুল ব্যবহৃত।


## `[ ]` vs `[[ ]]` - পার্থক্য কী?

এটা একটু confusing, কিন্তু খুব important!

| বিষয় | `[ ]` (POSIX) | `[[ ]]` (Bash Extended) |
|-------|--------------|------------------------|
| Shell | সব shell-এ চলে | শুধু Bash/Zsh-এ চলে |
| Word splitting | হয় (তাই quotes লাগে) | হয় না |
| `&&` / `\|\|` inside | কাজ করে না | কাজ করে |
| Regex match | নেই | `=~` দিয়ে আছে |
| String compare `<` `>` | `\<` `\>` দিতে হয় | সরাসরি `<` `>` |

### `[[ ]]` এর বিশেষ সুবিধা:

**১. `&&` এবং `||` সরাসরি ব্যবহার:**
```bash
#!/bin/bash

cpu=85
memory=90

# [[ ]] এর ভেতরে && ব্যবহার
if [[ $cpu -gt 80 && $memory -gt 80 ]]; then
    echo "🔴 Both CPU and Memory are critical!"
fi
```

**২. Regex Matching (`=~`):**
```bash
#!/bin/bash

filename="backup_2024_01_15.tar.gz"

if [[ "$filename" =~ ^backup_[0-9]{4} ]]; then
    echo "Valid backup file name"
else
    echo "Invalid file name"
fi
```

**Output:**
```
Valid backup file name
```

**কোনটা ব্যবহার করবো?**

> Bash script লিখলে `[[ ]]` ব্যবহার করুন, এটা বেশি powerful ও safe।
> Portability দরকার হলে (সব shell-এ চালাতে হবে) `[ ]` ব্যবহার করুন।


## Multiple Conditions - AND / OR

### `[ ]` এ লেখার নিয়ম:

```bash
# AND
if [ condition1 ] && [ condition2 ]; then

# OR  
if [ condition1 ] || [ condition2 ]; then
```

### `[[ ]]` এ লেখার নিয়ম:

```bash
# AND
if [[ condition1 && condition2 ]]; then

# OR
if [[ condition1 || condition2 ]]; then
```

### Real Example - Service Health Check:

```bash
#!/bin/bash

service_name="nginx"
port=80

# Service running কিনা check (exit code 0 মানে running)
systemctl is-active --quiet "$service_name"
service_running=$?

# Port open কিনা check
ss -tlnp | grep -q ":$port "
port_open=$?

if [[ $service_running -eq 0 && $port_open -eq 0 ]]; then
    echo "$service_name is running and port $port is open"
elif [[ $service_running -ne 0 ]]; then
    echo "$service_name is not running."
else
    echo "$service_name is running but port $port is closed."
fi
```


## Real-World DevOps Script - User Input Validation

```bash
#!/bin/bash

echo "Give a name to the environment (dev/staging/prod):"
read env

if [[ -z "$env" ]]; then
    echo "Nothing provided! Script closing."
    exit 1
elif [[ "$env" == "prod" || "$env" == "production" ]]; then
    echo "Deploying to production..."
    echo "Sure? (yes/no):"
    read confirm
    if [[ "$confirm" == "yes" ]]; then
        echo "Production deployment started..."
    else
        echo "Deploy cancelled।"
        exit 0
    fi
elif [[ "$env" == "staging" ]]; then
    echo "Starting staging deployment..."
elif [[ "$env" == "dev" ]]; then
    echo "Starting dev deployment..."
else
    echo "Unknown environment: $env"
    exit 1
fi
```


## 📝 Quick Summary

- `if / elif / else / fi` দিয়ে decision tree বানানো যায়
- Numeric comparison: `-eq`, `-ne`, `-gt`, `-lt`, `-ge`, `-le`
- String comparison: `=`, `!=`, `-z` (empty), `-n` (not empty)
- File checks: `-f` (file), `-d` (dir), `-e` (exist), `-x` (executable)
- `[ ]` = POSIX standard, সব shell-এ চলে
- `[[ ]]` = Bash extended, বেশি powerful - `&&`, `||`, `=~` support করে
- `!` দিয়ে condition উল্টানো যায়
- String variable সবসময় `"$variable"` quote করুন


## 🏋️ Practice Tasks

**Task 1:**
একটা script লিখুন যেটা একটা number নিবে এবং বলবে সেটা positive, negative নাকি zero।

**Task 2:**
একটা script লিখুন যেটা check করবে `/tmp/test.txt` file আছে কিনা। যদি থাকে তাহলে content দেখাবে, না থাকলে file তৈরি করে "Hello DevOps" লিখবে।

**Task 3:**
একটা script লিখুন যেটা user-এর কাছ থেকে username নিবে এবং check করবে সেই user system-এ আছে কিনা।
*(Hint: `id username` command সফল হলে exit code 0 দেয়)*

---

## ⏭️ What's Next?

**Chapter 5 - Lesson 4: Loops (for, while, until, break, continue)**

> একই কাজ বারবার না করে loop দিয়ে automate করা এবং 100টা server-এ একসাথে কাজ করানোর কাজে খুব ব্যবহার হয়! *Happy Learning* 🚀


<table width="100%">
  <tr>
    <td align="left">
      <a href="../02-Variables-Data-Types-N-User-Input">← Variables, Data Types & User Input</a>
    </td>
    <td align="right">
      <a href="../04-Loops">Loops →</a>
    </td>
  </tr>
</table>