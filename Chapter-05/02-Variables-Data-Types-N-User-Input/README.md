# Chapter 5 - Lesson 2: Variables, Data Types & User Input

**Chapter 5 | Lesson 2 of 11**

## 🎯 এই Lesson-এ আমরা যা শিখব

- Variable কী এবং কীভাবে কাজ করে
- Variable declare ও use করার নিয়ম
- Data types in Bash
- Environment Variables vs Local Variables
- Special Variables (`$0`, `$1`, `$#`, `$@`, `$$`, `$?`)
- User থেকে Input নেওয়া (`read` command)
- Real-world examples


## Variable কী? - সহজ ভাষায়

মনে করুন আপনি একটা বাক্সে কিছু রাখলেন এবং সেই বাক্সের একটা নাম দিলেন। পরে যখনই সেই জিনিসটা দরকার, আপনি বাক্সের নাম ধরে ডাকলেই হবে।

*Variable = নাম দেওয়া বাক্স, যেখানে data রাখা হয়।*

```
name = "Munir"   →  "Munir" নামের বাক্সে রাখা আছে
```


## Variable Declare করার নিয়ম

```bash
variable_name=value
```

### ⚠️ গুরুত্বপূর্ণ Rules:

| Rule | সঠিক ✅ | ভুল ❌ |
|------|---------|--------|
| `=` এর দুই পাশে **কোনো space নেই** | `name="Munir"` | `name = "Munir"` |
| নাম lowercase রাখা best practice | `server_name` | `Server Name` |
| নাম letter বা `_` দিয়ে দিয়ে শুরু হবে | `_count=5` | `1count=5` |
| কোনো special character নেই নামে | `my_var` | `my-var` |


## Variable Use করা - `$` দিয়ে

Variable-এর মান বের করতে `$` ব্যবহার করতে হবে:

```bash
vim var.sh

#!/bin/bash

name="Munir"
echo "My name is: $name"
echo "My name is: ${name}"   # এটাও valid - curly brace দিয়ে
```

**Output:**
```
My name is: Munir
My name is: Munir
```

> `${name}` curly brace কেন? যখন variable-এর পরেই আরো text থাকে, তখন curly brace দিয়ে clearly বোঝানো যায় variable কোথায় শেষ।

```bash
file="backup"
echo "${file}_2026.tar.gz"   # Output: backup_2026.tar.gz
echo "$file_2026.tar.gz"     # Output: .tar.gz  ❌ (ভুল - $file_2026 বলে কিছু নেই)
```

## Bash-এ Data Types

Bash-এ formally data type নেই, কিন্তু practically আমরা এভাবে ভাবি:

| Type | Example | ব্যাখ্যা |
|------|---------|---------|
| String | `name="Munir"` | Text data |
| Integer | `age=25` | সংখ্যা |
| Array | `fruits=("Mango" "Jackfruit")` | একাধিক value (পরের lesson-এ বিস্তারিত) |
| Boolean | `true` / `false` | Bash-এ explicitly নেই, 0/1 দিয়ে কাজ হয় |

### Integer নিয়ে কাজ:

```bash
vim number.sh

#!/bin/bash

num1=10
num2=5

# Arithmetic করতে $(( )) ব্যবহার করতে হয়

sum=$((num1 + num2))
diff=$((num1 - num2))
product=$((num1 * num2))
quotient=$((num1 / num2))
remainder=$((num1 % num2))

echo "Addition: $sum"
echo "Subtraction: $diff"
echo "Multiplication: $product"
echo "Division: $quotient"
echo "Remainder: $remainder"
```

**Output:**
```
Addition: 15
Subtraction: 5
Multiplication: 50
Division: 2
Remainder: 0
```

> `$(( ))` = Arithmetic Expansion। এটা ছাড়া Bash সংখ্যাকেও string হিসেবে দেখে।


## Local Variable vs Environment Variable

### Local Variable:
শুধু **current script বা session**-এর মধ্যে কাজ করে।

```bash
city="Dhaka"
echo $city    # কাজ করবে এই script-এ
```

### Environment Variable:
**System-wide** - সব process ও child process দেখতে পারে। `export` দিয়ে তৈরি করতে হয়।

```bash
export APP_ENV="production"
echo $APP_ENV
```

### পার্থক্য দেখুন:

```bash
#!/bin/bash

local_var="I am local"
export env_var="I am environment"

bash -c 'echo "Local: $local_var"'    # Output: Local:  (খালি - child দেখে না)
bash -c 'echo "Env: $env_var"'        # Output: Env: আমি environment
```

### কিছু Built-in Environment Variables:

```bash
echo $HOME       # /home/munir - তোমার home directory
echo $USER       # Munir - current username
echo $PATH       # যেসব directory-তে commands খোঁজা হয়
echo $PWD        # current directory
echo $SHELL      # /bin/bash - কোন shell চলছে
echo $HOSTNAME   # machine-এর নাম
```


## Special Variables - খুবই গুরুত্বপূর্ণ!

এগুলো Bash automatically তৈরি করে, আপনার তৈরি করার দরকার নাই:

| Variable | অর্থ | Example |
|----------|------|---------|
| `$0` | Script-এর নিজের নাম | `./myscript.sh` |
| `$1`, `$2`... | Script-এ দেওয়া Arguments (1st, 2nd...) | `./script.sh arg1 arg2` |
| `$#` | মোট কতটা argument দেওয়া হয়েছে | `3` |
| `$@` | সব arguments (আলাদা আলাদা) | `"arg1" "arg2" "arg3"` |
| `$*` | সব arguments (একসাথে একটা string) | `"arg1 arg2 arg3"` |
| `$?` | সর্বশেষ command-এর exit status (0=সফল, 1+=ব্যর্থ) | `0` |
| `$$` | Current script-এর PID | `12345` |
| `$!` | সর্বশেষ background process-এর PID | `12346` |

### উদাহরণ script:

```bash
#!/bin/bash
# script name: greet.sh

echo "Script name: $0"
echo "First argument: $1"
echo "Second argument: $2"
echo "Total arguments: $#"
echo "All arguments: $@"
echo "Script PID: $$"
```

**এভাবে রান করুন:**
```bash
bash greet.sh Munir DevOps
```

**Output:**
```
Script name: greet.sh
First argument: Munir
Second argument: DevOps
Total arguments: 2
All arguments: Munir DevOps
Script PID: 1902
```


### `$?` - Exit Status (DevOps-এ অনেক কাজের!)

```bash
ls /home          # সফল command
echo $?           # Output: 0  (সফল)

ls /nonexistent   # ব্যর্থ command
echo $?           # Output: 2  (ব্যর্থ)
```

> DevOps-এ এটা দিয়ে check করা হয় - "আগের command সফল হয়েছে কিনা।" পরের lesson-এ (conditionals) এটা অনেক ব্যবহার হবে।


## User থেকে Input নেওয়া - `read` Command

### Basic Syntax:

```bash
read variable_name
```

### উদাহরণ:

```bash
vim read_var.sh

#!/bin/bash

echo "What is your name?"
read username
echo "Welcome, $username!"
```

**Output:**
```
What is your name?
Munir Mahmud        ← user এটা type করে
Welcome, Munir Mahmud!
```

### `read` এর Options:

| Option | কাজ | Example |
|--------|-----|---------|
| `-p` | Prompt একই line-এ দেখায় | `read -p "নাম: " name` |
| `-s` | Input hide করে (password-এর জন্য) | `read -s -p "Password: " pass` |
| `-n` | নির্দিষ্ট কয়টা character নেবে | `read -n 1 choice` |
| `-t` | কত seconds অপেক্ষা করবে | `read -t 10 answer` |
| `-r` | Backslash-কে literal character হিসেবে নেয় (best practice) | `read -r filename` |

### Practical উদাহরণ:

```bash
#!/bin/bash

read -p "Your Name is: " name
read -p "Your age is: " age
read -s -p "Your password: " pass
echo ""    # password-এর পর নতুন line

echo "Name: $name"
echo "Age: $age"
echo "Password is taken (will not be shown)"
```

**Output:**
```
Your Name is: Munir
Your age is: 25
Your password: 
Name: Munir
Age: 25
Password is taken (will not be shown)
```


### একসাথে একাধিক Variable-এ Input:

```bash
#!/bin/bash

read -p "Write city and country (Keep space): " city country
echo "City: $city"
echo "Country: $country"
```

**Output:**
```
Write city and country (Keep space): Dhaka Bangladesh
City: Dhaka
Country: Bangladesh
```


## Real-World DevOps Script - সব একসাথে

```bash
#!/bin/bash
# deploy.sh - A simple deployment script

echo "=============================="
echo "   Server Deployment Script   "
echo "=============================="

read -p "Environment (dev/staging/prod): " environment
read -p "Application Name: " app_name
read -p "Version number: " version
read -s -p "Deploy key: " deploy_key
echo ""

echo ""
echo "Deploy is starting..."
echo "Environment: $environment"
echo "App: $app_name"
echo "Version: $version"
echo "Script is ran by: $USER"
echo "Script Name: $0"
echo "Deploy is being running to machine: $HOSTNAME"

# Simulate deployment
sleep 2
echo "$app_name v$version successfully deployed to $environment!"
```

**Output:**
```
==============================
   Server Deployment Script   
==============================
Environment (dev/staging/prod): prod
Application Name: myapp
Version number: 2.1.0
Deploy key: 
Deploy is being started...
Environment: prod
App: myapp
Version: 2.1.0
Script is ran by: Munir
Script Name: ./deploy.sh
Deploy is being running to machine: devserver-01
myapp v2.1.0 successfully deployed to prod!
```


### Variable-এ Default Value দেওয়া:

এটা একটা advanced কিন্তু অনেক কাজের trick:

```bash
#!/bin/bash

# যদি user কিছু না দেয়, default value ব্যবহার হবে
read -p "Environment (default: dev): " env
environment=${env:-dev}    # যদি $env খালি থাকে, "dev" ব্যবহার করো

echo "Environment: $environment"
```

**Output (user কিছু না দিলে):**
```
Environment (default: dev): 
Environment: dev
```

> `${variable:-default}` = "variable যদি খালি থাকে, তাহলে default ব্যবহার করো" - DevOps script-এ এটা অনেক দেখবেন।


## 📝 Quick Summary

- Variable declare করতে `name=value` (কোনো space নেই `=` এর পাশে)
- Variable use করতে `$name` বা `${name}`
- Arithmetic এর জন্য `$(( ))` ব্যবহার করুন
- `export` দিয়ে Environment Variable তৈরি হয়
- Special variables: `$0`, `$1`, `$#`, `$@`, `$?`, `$$`
- `$?` দিয়ে command সফল হয়েছে কিনা check করা যায়
- `read` দিয়ে user থেকে input নেওয়া যায়
- `read -p` (prompt), `read -s` (secret), `read -t` (timeout)
- `${var:-default}` দিয়ে default value set করা যায়


## 🏋️ Practice Tasks

**Task 1:** একটা script লিখুন যেটা user-এর নাম ও বয়স নেবে এবং বলবে "Your name X, You are Y years old।"

**Task 2:** একটা script লিখুন যেটা দুটো number argument হিসেবে নেবে (`$1`, `$2`) এবং তাদের যোগ, বিয়োগ, গুণ, ভাগ দেখাবে।

**Task 3:** একটা script লিখুন যেটা username ও password নেবে (`read -s` দিয়ে password hide করুন), তারপর print করবে "User: X logged in successfully।"

---

## ⏭️ What's Next?

**Chapter 5 - Lesson 3: Conditionals**
`if`, `elif`, `else`, `test`, `[ ]`, `[[ ]]` script-এ decision making শেখা। যেমন: "যদি environment prod হয়, তাহলে এটা করো, নাহলে ওটা করো।" *Happy Learning* 🚀


<table width="100%">
  <tr>
    <td align="left">
      <a href="../01-Shell-Scripting-Fundamentals">← Shell Scripting Fundamentals</a>
    </td>
    <td align="right">
      <a href="../03-Conditionals">Conditionals →</a>
    </td>
  </tr>
</table>