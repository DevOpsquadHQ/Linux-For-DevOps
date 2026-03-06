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
