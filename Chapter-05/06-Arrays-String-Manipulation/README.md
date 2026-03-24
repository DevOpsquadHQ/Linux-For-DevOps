# Chapter 5 - Lesson 6: Arrays & String Manipulation

**Chapter 5 | Lesson 6 of 11**


## 🎯 এই Lesson-এ আমরা যা শিখবো

- Bash-এ Array কী এবং কীভাবে কাজ করে
- Indexed Array এবং Associative Array
- Array-এর উপর বিভিন্ন Operations
- String Manipulation - Bash-এ strings নিয়ে কাজ করা
- Real-world DevOps use cases


## PART 1: Arrays in Bash

### Array কী?

সহজ ভাষায় বলতে গেলে Array হলো একটা বাক্স যেখানে অনেকগুলো জিনিস একসাথে রাখা যায়।

> ধরুন আপনার কাছে একটা ডিমের ট্রে আছে। প্রতিটা ঘরে একটা করে ডিম রাখা যায়। ঘরগুলোর নম্বর হলো 0, 1, 2, 3... এটাই Array!

Normal variable-এ একটাই value রাখা যায়:

```bash
name="Munir"
```

কিন্তু Array-তে অনেকগুলো যেমনঃ
```bash
names=("Munir" "Karim" "Jamal" "Salim")
```

## PART 2: Indexed Array (সংখ্যা দিয়ে index)

### Array তৈরি করা

```bash
# Method 1: একসাথে সব value দিয়ে
fruits=("apple" "banana" "mango" "orange")

# Method 2: একটা একটা করে assign করে
fruits[0]="apple"
fruits[1]="banana"
fruits[2]="mango"

# Method 3: declare দিয়ে
declare -a servers=("web01" "web02" "db01" "db02")
```

### Array-এর Values Access করা

```bash
fruits=("apple" "banana" "mango" "orange")

# একটা নির্দিষ্ট element access করা
echo ${fruits[0]}        # apple
echo ${fruits[2]}        # mango

# সব element একসাথে দেখা
echo ${fruits[@]}        # apple banana mango orange
echo ${fruits[*]}        # apple banana mango orange

# Array-এর মোট কতটা element আছে
echo ${#fruits[@]}       # 4

# একটা নির্দিষ্ট element-এর length
echo ${#fruits[1]}       # 6 (banana = 6 letters)

# সব index দেখা
echo ${!fruits[@]}       # 0 1 2 3
```

**Output:**
```
apple
mango
apple banana mango orange
apple banana mango orange
4
6
0 1 2 3
```

### Array Loop করা

```bash
#!/bin/bash
servers=("web01" "web02" "db01" "cache01")

echo "=== All Servers ==="
for server in "${servers[@]}"; do
    echo "Server: $server"
done
```

**Output:**
```
=== All Servers ===
Server: web01
Server: web02
Server: db01
Server: cache01
```

**Index সহ loop:**
```bash
#!/bin/bash
servers=("web01" "web02" "db01" "cache01")

for i in "${!servers[@]}"; do
    echo "Index $i → ${servers[$i]}"
done
```

**Output:**
```
Index 0 → web01
Index 1 → web02
Index 2 → db01
Index 3 → cache01
```

### Array Modify করা

```bash
#!/bin/bash
fruits=("apple" "banana" "mango")

# নতুন element যোগ করুন (append)
fruits+=("orange")
fruits+=("grape" "kiwi")   # একসাথে একাধিকও যোগ করা যায়

echo ${fruits[@]}
# apple banana mango orange grape kiwi

# একটা element পরিবর্তন করুন
fruits[1]="strawberry"
echo ${fruits[@]}
# apple strawberry mango orange grape kiwi

# একটা element delete করুন
unset fruits[2]
echo ${fruits[@]}
# apple strawberry orange grape kiwi

# পুরো array delete করুন
unset fruits
```

### Array Slicing (একটা অংশ নেওয়া)

```bash
#!/bin/bash
numbers=(10 20 30 40 50 60 70 80)

# Syntax: ${array[@]:start:length}
echo ${numbers[@]:2:3}    # index 2 থেকে শুরু, 3টা element নিন
# Output: 30 40 50

echo ${numbers[@]:0:4}    # শুরু থেকে 4টা
# Output: 10 20 30 40

echo ${numbers[@]:5}      # index 5 থেকে শেষ পর্যন্ত
# Output: 60 70 80
```

## PART 3: Associative Array (Key-Value pair)

এটা Python-এর dictionary-র মতো। নম্বরের বদলে নামের index ব্যবহার করা যায়।

> Phone book-এর মতো, নাম দিয়ে নম্বর খোজা!

```bash
#!/bin/bash

# Associative array তৈরি করতে declare -A লাগবেই
declare -A server_ips

server_ips["web01"]="192.168.1.10"
server_ips["web02"]="192.168.1.11"
server_ips["db01"]="192.168.1.20"
server_ips["cache01"]="192.168.1.30"

# একটা value access করো
echo ${server_ips["web01"]}    # 192.168.1.10
echo ${server_ips["db01"]}     # 192.168.1.20

# সব keys দেখাও
echo ${!server_ips[@]}
# web01 web02 db01 cache01

# সব values দেখাও
echo ${server_ips[@]}
# 192.168.1.10 192.168.1.11 192.168.1.20 192.168.1.30

# Loop করো
for key in "${!server_ips[@]}"; do
    echo "$key → ${server_ips[$key]}"
done
```

**Output:**
```
192.168.1.10
192.168.1.20
web01 → 192.168.1.10
web02 → 192.168.1.11
db01 → 192.168.1.20
cache01 → 192.168.1.30
```

### DevOps Real Example: Server Health Check with Array

```bash
#!/bin/bash
declare -A server_status

servers=("web01" "web02" "db01")

for server in "${servers[@]}"; do
    # ping করুন, যদি সাড়া দেয় তাহলে UP নইলে DOWN
    if ping -c 1 -W 1 "$server" &>/dev/null; then
        server_status[$server]="✅ UP"
    else
        server_status[$server]="❌ DOWN"
    fi
done

echo "=== Server Health Report ==="
for server in "${!server_status[@]}"; do
    echo "$server : ${server_status[$server]}"
done
```

## PART 4: String Manipulation

String Manipulation মানে হলো text/string নিয়ে বিভিন্ন কাজ করা যেমন লেন্থ বের করা, কাটা, বদলানো, খোঁজা ইত্যাদি।

### 4.1 String Length বের করা

```bash
name="DevOps Engineer"
echo ${#name}        # 15

city="Dhaka"
echo ${#city}        # 5
```

### 4.2 Substring Extraction (একটা অংশ বের করা)

```bash
# Syntax: ${string:start:length}
text="Hello DevOps World"

echo ${text:6:6}      # DevOps  (index 6 থেকে শুরু, 6 character)
echo ${text:0:5}      # Hello   (শুরু থেকে 5 character)
echo ${text:13}       # World   (index 13 থেকে শেষ পর্যন্ত)
echo ${text: -5}      # World   (শেষ থেকে 5 character)
```

### 4.3 String Replace করা

```bash
# Syntax: ${string/old/new}  → প্রথমটা replace করে
# Syntax: ${string//old/new} → সবগুলো replace করে

sentence="I love cats and cats love me"

echo ${sentence/cats/dogs}     # I love dogs and cats love me
echo ${sentence//cats/dogs}    # I love dogs and dogs love me

# শুধু শুরুতে replace করো
path="/home/user/file.txt"
echo ${path/#\/home/\/root}    # /root/user/file.txt

# শুধু শেষে replace করো
filename="report.txt"
echo ${filename/%txt/pdf}      # report.pdf
```

### 4.4 String Delete করা

```bash
# Syntax: ${string#pattern}   → শুরু থেকে সবচেয়ে ছোট match delete করে
# Syntax: ${string##pattern}  → শুরু থেকে সবচেয়ে বড় match delete করে
# Syntax: ${string%pattern}   → শেষ থেকে সবচেয়ে ছোট match delete করে
# Syntax: ${string%%pattern}  → শেষ থেকে সবচেয়ে বড় match delete করে

filepath="/var/log/nginx/access.log"

# Path থেকে শুধু filename বের করো (শুরু থেকে সব / সহ delete করো)
echo ${filepath##*/}       # access.log

# File থেকে extension বাদ দাও
filename="backup_2024.tar.gz"
echo ${filename%.*}        # backup_2024.tar   (শেষ . এর পর বাদ)
echo ${filename%%.*}       # backup_2024        (প্রথম . এর পর সব বাদ)

# Path থেকে filename বাদ দিয়ে directory রাখো
echo ${filepath%/*}        # /var/log/nginx
```

**মনে রাখার trick:**

> - `#` এবং `##` → **শুরু** থেকে কাটে (# keyboard-এ left দিকে)
> - `%` এবং `%%` → **শেষ** থেকে কাটে (% keyboard-এ right দিকে)
> - একটা `#` বা `%` → **ছোট** match
> - দুটো `##` বা `%%` → **বড়** match (greedy)

### 4.5 Upper/Lower Case Conversion

```bash
name="hello devops world"

# Uppercase করো
echo ${name^^}          # HELLO DEVOPS WORLD

# শুধু প্রথম letter uppercase
echo ${name^}           # Hello devops world

# Lowercase করো (যদি uppercase থাকে)
NAME="HELLO DEVOPS"
echo ${NAME,,}          # hello devops

# শুধু প্রথম letter lowercase
echo ${NAME,}           # hELLO DEVOPS
```

### 4.6 Default Value / Variable Substitution

```bash
# যদি variable empty থাকে তাহলে default value ব্যবহার করো
# Syntax: ${variable:-default}

username=""
echo ${username:-"guest"}     # guest (কারণ username empty)

username="admin"
echo ${username:-"guest"}     # admin (কারণ username set আছে)

# যদি variable না থাকে তাহলে set করে দাও
# Syntax: ${variable:=default}
echo ${city:="Dhaka"}         # Dhaka (city set হয়ে গেলো)
echo $city                    # Dhaka

# Error message দেখাও যদি variable না থাকে
# Syntax: ${variable:?error message}
echo ${required_var:?"This variable is required!"}
# Output: bash: required_var: This variable is required!
```

### 4.7 String Contains Check

```bash
#!/bin/bash
main_string="Hello DevOps World"
search="DevOps"

if [[ "$main_string" == *"$search"* ]]; then
    echo "✅ '$search' found in the string!"
else
    echo "❌ '$search' not found!"
fi
```

**Output:**
```
✅ 'DevOps' found in the string!
```

### 4.8 String Split করা (IFS ব্যবহার করে)

```bash
#!/bin/bash
# IFS = Internal Field Separator

csv_data="web01,web02,db01,cache01"

# comma দিয়ে split করো
IFS=',' read -ra servers <<< "$csv_data"

for server in "${servers[@]}"; do
    echo "Server: $server"
done
```

**Output:**
```
Server: web01
Server: web02
Server: db01
Server: cache01
```

## PART 5: Real-World DevOps Script

এখন সব কিছু মিলিয়ে একটা real script লিখি:

```bash
#!/bin/bash
# ============================================
# DevOps Log Analyzer with Arrays & Strings
# ============================================

LOG_DIR="/var/log"
declare -A log_sizes
declare -a large_logs

echo "========================================="
echo "           Log File Analyzer             "
echo "========================================="

# সব .log file খুঁজে বের করো
while IFS= read -r -d '' logfile; do
    filename="${logfile##*/}"           # শুধু filename বের করো
    size=$(du -sh "$logfile" 2>/dev/null | cut -f1)   # size বের করো
    log_sizes["$filename"]="$size"     # Associative array-তে store করো
done < <(find "$LOG_DIR" -maxdepth 1 -name "*.log" -print0 2>/dev/null)

# Display করো
echo ""
echo "📁 Log Files Found:"
echo "-----------------------------------------"
for log in "${!log_sizes[@]}"; do
    echo "  ${log,,} → Size: ${log_sizes[$log]}"
done

echo ""
echo "✅ Total log types found: ${#log_sizes[@]}"
echo "========================================="
```


## Quick Reference Table

| Operation | Syntax | Example |
|-----------|--------|---------|
| Array তৈরি | `arr=(a b c)` | `fruits=(apple mango)` |
| Element access | `${arr[n]}` | `${fruits[0]}` |
| সব element | `${arr[@]}` | `${fruits[@]}` |
| Array length | `${#arr[@]}` | `${#fruits[@]}` |
| Append | `arr+=(x)` | `fruits+=(orange)` |
| Delete element | `unset arr[n]` | `unset fruits[1]` |
| String length | `${#str}` | `${#name}` |
| Substring | `${str:start:len}` | `${text:0:5}` |
| Replace first | `${str/old/new}` | `${s/cat/dog}` |
| Replace all | `${str//old/new}` | `${s//cat/dog}` |
| Delete prefix | `${str#pattern}` | `${path##*/}` |
| Delete suffix | `${str%pattern}` | `${file%.*}` |
| Uppercase | `${str^^}` | `${name^^}` |
| Lowercase | `${str,,}` | `${NAME,,}` |
| Default value | `${var:-default}` | `${user:-guest}` |


## 📝 Lesson Summary

- **Indexed Array** - সংখ্যা দিয়ে index হয়, `arr=(a b c)` দিয়ে তৈরি করো
- **Associative Array** - key-value pair, `declare -A` দিয়ে তৈরি করতে হয়
- **`${arr[@]}`** - সব element, **`${#arr[@]}`** - মোট count
- **`${!arr[@]}`** - সব index/key দেখায়
- **String length** → `${#str}`, **Substring** → `${str:start:len}`
- **Replace** → `/` একটা, `//` সবগুলো
- **Delete prefix** → `#/##`, **Delete suffix** → `%/%%`
- **Case change** → `^^` uppercase, `,,` lowercase
- **Default value** → `${var:-default}` খুব useful scripting-এ


## 🏋️ Practice Tasks

**Task 1:**
একটা array তৈরি করুন যেখানে আপনার পছন্দের ৫টা programming language থাকবে। তারপর loop করে প্রতিটার নাম uppercase-এ print করুন।

**Task 2:**
একটা string নেন: `"server_backup_2024_01_15.tar.gz"` - এখন:
- শুধু extension বাদ দিয়ে দেখান
- শুধু filename থেকে date অংশটা (`2024_01_15`) বের করুন

**Task 3:**
একটা Associative Array তৈরি করুন যেখানে ৩টা service-এর নাম key হিসেবে এবং তাদের port number value হিসেবে থাকবে (nginx=80, ssh=22, mysql=3306)। Loop করে সব দেখান।

---

## ⏭️ What's Next?

**Chapter 5 - Lesson 7: Text Processing**
`grep`, `awk`, `sed`, `cut`, `sort`, `uniq`, `tr` এগুলো হলো Linux-এর সবচেয়ে শক্তিশালী text processing tools। DevOps কাজে log analysis, config parsing, data transformation সবখানেই এগুলো লাগে। পরের lesson-এ এগুলোর উপরে mastery করবো! *Happy Learning* 🚀


<table width="100%">
  <tr>
    <td align="left">
      <a href="../05-Functions-in-Bash">← Functions in Bash</a>
    </td>
    <td align="right">
      <a href="../07-Text-Processing">Text Processing →</a>
    </td>
  </tr>
</table>