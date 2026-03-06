# Chapter 1 - Lesson 7: File Editors (nano, vim)

**Chapter 1 | Lesson 7 of 10**


## এই Lesson-এ কী শিখবো?

- Terminal-এ file edit করার দরকার কেন?
- **nano** - beginner-friendly editor
- **vim** - powerful editor যা প্রতিটা DevOps Engineer-কে জানতে হয়
- vim-এর modes, navigation, editing, save & quit
- Real-world DevOps/Sys admin-এ কোনটা কখন ব্যবহার হয়



## Terminal Editor কেন দরকার?

কল্পনা করুন আপনি একটা remote server-এ SSH করে আছেন। সেখানে কোনো GUI নেই, কোনো VS Code নেই। আপনাকে একটা config file edit করতে হবে।

এই মুহূর্তে **terminal-based editor** ছাড়া কোনো উপায় নেই।

DevOps-এ প্রতিদিন এরকম কাজ হয়:
- `/etc/nginx/nginx.conf` edit করা
- `/etc/ssh/sshd_config` hardening করা
- shell script লেখা
- crontab edit করা

## PART 1: nano - The Beginner-Friendly Editor

### nano কী?

**nano** হলো একটা simple text editor। এটা **What You See Is What You Get** মানে ওপেন করলেই লিখতে পারবে, কোনো mode নেই।

নিচে screen-এ সব shortcut দেখা যায়, তাই মনে রাখার ঝামেলা কম।


### nano দিয়ে file ওপেন করা

```bash
nano filename.txt
```

যদি file না থাকে, নতুন file তৈরি হবে।


### nano-র Screen Layout

```
  GNU nano 6.2         filename.txt

[এখানে আপনি লিখবেন]




^G Help    ^O Write Out   ^W Where Is   ^K Cut      ^T Execute
^X Exit    ^R Read File   ^\ Replace    ^U Paste    ^J Justify
```

নিচের `^` মানে **Ctrl** key।


### nano-র Important Shortcuts

| Shortcut | কাজ |
|----------|-----|
| `Ctrl + O` | File save করো (Write Out) |
| `Ctrl + X` | nano থেকে বের হও |
| `Ctrl + W` | কোনো word খোঁজো |
| `Ctrl + K` | পুরো একটা line cut করো |
| `Ctrl + U` | Cut করা line paste করো |
| `Ctrl + G` | Help দেখো |
| `Ctrl + /` | নির্দিষ্ট line number-এ যাও |


### nano Real Example

**Step 1:** একটা নতুন file ওপেন করুন:
```bash
nano myfirstfile.txt
```

**Step 2:** কিছু লিখুন:
```
Hello, I am Munir Mahmud. I am learning Linux for DevOps!
This is my first file in nano.
DevOps is awesome.
```

**Step 3:** Save করুন → `Ctrl + O` চাপুন → Enter চাপুন

**Step 4:** বের হওয়ার জন্য → `Ctrl + X` চাপুন

**Step 5:** দেখুন file তৈরি হয়েছে কিনা:
```bash
cat myfirstfile.txt
```

**Output:**
```
Hello, I am Munir Mahmud. I am learning Linux for DevOps!
This is my first file in nano.
DevOps is awesome.
```


### nano-তে Existing Config File Edit করা (DevOps Example)

```bash
sudo nano /etc/hosts
```

এখানে আপনি নতুন hostname entry add করতে পারবেন। এটা DevOps-এ খুবই common কাজ।


## PART 2: vim - The Powerful Editor

### vim কী?

**vim** মানে **Vi IMproved**। এটা `vi` editor-এর উন্নত version।

vim কে অনেকে ভয় পায় কারণ এটা একটু আলাদাভাবে কাজ করে। এর **modes** আছে। কিন্তু একবার শিখলে এটা অবিশ্বাস্য রকম powerful।

> **Real Talk:** Production server-এ অনেক সময় শুধু `vi` বা `vim` থাকে, `nano` থাকে না। তাই vim জানাটা **mandatory** একজন DevOps Engineer-এর জন্য।

### vim-এর Modes - সবচেয়ে গুরুত্বপূর্ণ Concept

vim-এ একটাই screen, কিন্তু **আলাদা আলাদা mode**-এ আলাদা কাজ হয়।

**Analogy:** মনে করেন আপনার একটা গাড়ি আছে।
- **Normal Mode** = গাড়ি neutral-এ আছে, আপনি দেখছেন কিন্তু চালাচ্ছেন না
- **Insert Mode** = গাড়ি চলছে, আপনি type করছেন
- **Command Mode** = আপনি dashboard-এ command দিচ্ছেন (save, quit, search)
- **Visual Mode** = আপনি text select করছেন

### vim-এর 4টি Main Mode

```
┌─────────────────────────────────────────────┐
│                                             │
│   NORMAL MODE  ──(i, a, o)──►  INSERT MODE  │
│       ▲                            │        │
│       └──────────── Esc ───────────┘        │
│       │                                     │
│      (:)                                    │
│       │                                     │
│   COMMAND MODE                              │
│  (:w, :q, :wq)                              │
│                                             │
│   NORMAL MODE ──(v)──► VISUAL MODE          │
└─────────────────────────────────────────────┘
```

| Mode | কীভাবে ঢুকবেন | কী করতে পারবেন |
|------|-------------|---------------|
| **Normal Mode** | Default / `Esc` চাপলে | Navigate, copy, paste, delete |
| **Insert Mode** | `i` চাপলে | Text type করা যায় |
| **Command Mode** | `:` চাপলে | Save, quit, search & replace |
| **Visual Mode** | `v` চাপলে | Text select করা যায় |

---

### vim দিয়ে File ওপেন করা

```bash
vim filename.txt
```

অথবা existing file:
```bash
vim /etc/hosts
```


### Step-by-Step: vim দিয়ে প্রথমবার কাজ করা

**Step 1: vim খুলুন**
```bash
vim practice.txt
```
আপনি এখন **Normal Mode**-এ আছেন। কিছু type করলে কাজ হবে না।

**Step 2: Insert Mode-এ যান**
```
i চাপুন
```
নিচে দেখবেন লেখা আছে: `-- INSERT --`
এখন আপনি type করতে পারবেন।

**Step 3: কিছু লিখুন**
```
My name is Munir Mahmud. I am a DevOps Engineer.
I am learning vim today.
vim is powerful!
```

**Step 4: Normal Mode-এ ফিরে যান**
```
Esc চাপুন
```
নিচের `-- INSERT --` চলে যাবে।

**Step 5: Save করুন**
```
:w লিখে তারপর Enter
```

**Step 6: vim বন্ধ করুন**
```
:q লিখে তারপর Enter
```

অথবা **একসাথে save করে বের হওয়া:**
```
:wq লিখে তারপর Enter
```

### vim-এর Normal Mode Navigation

Mouse ব্যবহার না করে keyboard দিয়ে navigate করা vim-এর superpower।

| Key | কাজ |
|-----|-----|
| `h` | বামে যাওয়া (←) |
| `l` | ডানে যাওয়া (→) |
| `j` | নিচে যাওয়া (↓) |
| `k` | উপরে যাওয়া (↑) |
| `gg` | File-এর একদম শুরুতে যাওয়া |
| `G` | File-এর একদম শেষে যাওয়া |
| `0` | Line-এর শুরুতে যাওয়া |
| `$` | Line-এর শেষে যাওয়া |
| `w` | পরের word-এ যাওয়া |
| `b` | আগের word-এ যাওয়া |
| `5G` | 5 নম্বর line-এ যাওয়া |

### vim-এ Insert Mode-এর বিভিন্ন উপায়

| Key | কাজ |
|-----|-----|
| `i` | Cursor-এর আগে insert শুরু করে |
| `a` | Cursor-এর পরে insert শুরু করে (append) |
| `o` | নিচে নতুন line তৈরি করে insert করে |
| `O` | উপরে নতুন line তৈরি করে insert করে |
| `I` | Line-এর একদম শুরুতে insert করে |
| `A` | Line-এর একদম শেষে insert করে |


### vim-এ Delete / Cut করা (Normal Mode-এ)

| Command | কাজ |
|---------|-----|
| `x` | একটা character delete করে |
| `dd` | পুরো line delete/cut করে |
| `d5d` বা `5dd` | 5টা line delete করে |
| `dw` | একটা word delete করে |
| `D` | Cursor থেকে line-এর শেষ পর্যন্ত delete |


### vim-এ Copy & Paste (Normal Mode-এ)

vim-এ copy কে বলে **yank**।

| Command | কাজ |
|---------|-----|
| `yy` | পুরো line copy (yank) করো |
| `y5y` বা `5yy` | 5টা line copy করো |
| `yw` | একটা word copy করো |
| `p` | Cursor-এর নিচে paste করো |
| `P` | Cursor-এর উপরে paste করো |


### vim-এ Undo & Redo

| Command | কাজ |
|---------|-----|
| `u` | Undo (Normal Mode-এ) |
| `Ctrl + r` | Redo |


### vim Command Mode (:) Save, Quit & Search

| Command | কাজ |
|---------|-----|
| `:w` | Save করো |
| `:q` | Quit করো (unsaved changes থাকলে হবে না) |
| `:wq` | Save করে quit করো |
| `:q!` | Save না করেই force quit করো |
| `:wq!` | Force save করে quit করো |
| `:set nu` | Line number দেখাও |
| `:set nonu` | Line number লুকাও |
| `:/word` | নিচের দিকে word খোঁজো |
| `:?word` | উপরের দিকে word খোঁজো |
| `n` | পরের search result-এ যাও |
| `:%s/old/new/g` | সব জায়গায় "old" কে "new" দিয়ে replace করো |


### vim Real DevOps Example: Search & Replace

মনে করেন আপনি একটা config file-এ সব জায়গায় `localhost` কে `192.168.1.10` দিয়ে replace করতে চান:

```bash
vim app.conf
```

Command mode-এ এই command দিন:
```
:%s/localhost/192.168.1.10/g
```

এক লাইনেই পুরো file-এর সব `localhost` replace হয়ে যাবে!

### vim-এ Visual Mode দিয়ে Text Select করা

1. Normal mode-এ `v` চাপুন
2. Arrow key বা `h/j/k/l` দিয়ে text select করুন
3. `y` চাপুন → copy হবে
4. `d` চাপুন → delete হবে
5. `p` চাপুন → paste হবে


### একটা সাধারণ vim Session দেখুন

```bash
$ vim server.conf

# vim ওপেন হলো, Normal Mode-এ আছি

i                    # Insert Mode-এ গেলাম
port=8080            # টাইপ করলাম
Esc                  # Normal Mode-এ ফিরলাম
:set nu              # Line number দেখালাম
/port                # "port" word খুঁজলাম
:%s/8080/9090/g      # 8080 কে 9090 দিয়ে replace করলাম
:wq                  # Save করে বের হয়ে গেলাম
```

## nano vs vim কোনটা কখন ব্যবহার করবেন?

| বিষয় | nano | vim |
|-------|------|-----|
| **Difficulty** | সহজ | কঠিন (শিখতে সময় লাগে) |
| **Speed** | ধীর | অনেক দ্রুত |
| **Server-এ পাওয়া যায়?** | সবসময় না | প্রায় সবসময় |
| **Large file edit** | কষ্টকর | সহজ |
| **Search & Replace** | সীমিত | অনেক powerful |
| **DevOps use** | Quick edits | Production level |
| **কখন ব্যবহার করবেন** | Quick config change | Complex editing, scripting |

> **Advice:** শুরুতে nano দিয়ে practice করতে পারেন। কিন্তু vim শেখা বন্ধ করা যাবে না, এটা career-এ অনেক কাজে আসবে।


## vim চিটশিট (Quick Reference)

```
vim এ ঢোকা:         vim filename
Insert Mode:         i (before), a (after), o (new line below)
Normal Mode:         Esc
Save:                :w
Quit:                :q
Save & Quit:         :wq
Force Quit:          :q!
Delete line:         dd
Copy line:           yy
Paste:               p
Undo:                u
Search:              /keyword
Replace all:         :%s/old/new/g
Go to line 10:       :10 or 10G
Line numbers:        :set nu
```

## 📋 Lesson Summary

- **nano** হলো simple, beginner-friendly editor, shortcut screen-এই দেখা যায়
- **vim** হলো powerful editor, modes আছে: Normal, Insert, Command, Visual
- vim-এ সবসময় **`Esc`** চাপলে Normal Mode-এ ফিরবে
- **`:wq`** = save করে বের হওয়া, **`:q!`** = save না করেই বের হওয়া
- **`dd`** = line delete, **`yy`** = line copy, **`p`** = paste
- **`:%s/old/new/g`** = পুরো file-এ search & replace
- Production server-এ vim জানা **অপরিহার্য**

## 🏋️ Practice Tasks

**Task 1 - nano practice:**
```bash
nano my_info.txt
```
নিচের তথ্য লিখে, save করে, বের হও:
```
Name: [your name]
Role: DevOps Engineer
Favorite OS: Linux
```

**Task 2 - vim basics:**
```bash
vim devops_notes.txt
```
- Insert mode-এ ঢুকে 5টা line লিখুন
- Esc চাপো
- `dd` দিয়ে একটা line delete করুন
- `yy` দিয়ে একটা line copy করুন
- `p` দিয়ে paste করুন
- `:wq` দিয়ে save করে বের হয়ে যান

**Task 3 - Search & Replace:**
```bash
vim replace_practice.txt
```
- "hello" word 3 বার লিখুন আলাদা line-এ
- `:wq` দিয়ে save করুন
- আবার vim দিয়ে খোলো
- `:%s/hello/world/g` command দিয়ে সব "hello" কে "world" দিয়ে রিপ্লেস করুন
- Save করে বের হন, `cat` দিয়ে verify করুন

---

## ⏭️ What's Next?

**Chapter 1 - Lesson 8: Links in Linux**
পরের lesson-এ শিখবো **Hard Links এবং Symbolic (Soft) Links** কী, কীভাবে তৈরি করতে হয়, এবং DevOps-এ এগুলো কোথায় কাজে লাগে। এটা একটা super interesting topic! 
