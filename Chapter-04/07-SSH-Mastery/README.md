# Chapter 4 - Lesson 7: SSH Mastery

**Chapter 4 | Lesson 7 of 11**


আপনাকে স্বাগতম আজকের সবচেয়ে গুরুত্বপূর্ণ DevOps lesson-এ! 🎉

SSH ছাড়া DevOps কল্পনাই করা যায় না। আপনাকে প্রতিদিন এটা ব্যবহার করতে হবে - remote server manage করতে, CI/CD pipeline-এ, cloud infrastructure-এ। এটা শুধু একটা tool না, এটা আপনার daily weapon।


## SSH কী? (What is SSH?)

**SSH = Secure Shell**

সহজ ভাষায়: SSH হলো একটা **encrypted tunnel** যেটার মাধ্যমে আপনি একটা remote computer-এ বসে এমনভাবে কাজ করতে পারবেন যেন আপনি সেই machine-এর সামনেই বসে আছেন।


> কল্পনা করুন আপনি Dhaka থেকে London-এর একটা office-এ কাজ করতে চান। SSH হলো সেই **secure encrypted phone line** যেটা দিয়ে আপনি London-এর office-এ সরাসরি কথা বলতে পারেন। কেউ মাঝখানে আপনার কথা শুনতে পারবে না।

### SSH কেন দরকার?
| পুরনো পদ্ধতি (Telnet) | SSH |
|---|---|
| সব data plain text | সব data encrypted |
| Password চুরি হতে পারে | Password secure |
| Man-in-the-middle attack সহজ | Attack প্রায় অসম্ভব |
| DevOps-এ ব্যবহার নিষিদ্ধ | DevOps-এর standard |


## SSH-এর Parts (Components)

```
আপনার Machine (Client)  ←--- SSH Tunnel ---→  Remote Server (Server)
   ssh client                                    sshd (daemon)
```

- **ssh** → আপনার machine-এ থাকা client tool
- **sshd** → remote server-এ চলতে থাকা daemon (service)
- **Port 22** → SSH-এর default port
- **~/.ssh/** → সব SSH keys ও config এখানে থাকে


## Part 1: Password দিয়ে SSH Connect করা

### Basic Syntax:
```bash
ssh username@hostname_or_IP
```

### Example:
```bash
ssh devops@192.168.1.100
```

```
# Expected Output:
The authenticity of host '192.168.1.100 (192.168.1.100)' can't be established.
ED25519 key fingerprint is SHA256:xxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxx.
Are you sure you want to continue connecting (yes/no/[fingerprint])? yes

Warning: Permanently added '192.168.1.100' (ED25519) to the list of known hosts.
devops@192.168.1.100's password: ______

Welcome to Ubuntu 22.04 LTS
devops@remote-server:~$     ← এখন আপনি remote server-এ আছেন!
```

> ⚠️ প্রথমবার connect করলে "fingerprint" জিজ্ঞেস করবে - `yes` লিখলে এটা `~/.ssh/known_hosts` file-এ save হয়ে যায়।

### Different port-এ connect করতে:
```bash
ssh -p 2222 devops@192.168.1.100
```
*(যখন server-এর SSH default port 22 না হয়ে 2222-তে রান করে)*


## Part 2: SSH Key Generation (সবচেয়ে গুরুত্বপূর্ণ!)

Password দিয়ে login করা অনিরাপদ ও ঝামেলার। DevOps-এ সবসময় SSH Key-based authentication ব্যবহার করা হয়।


> Password হলো একটা secret word যেটা আপনি বলে দেন। Key-based auth হলো একটা unique lock-and-key system - আপনার কাছে private key (চাবি), server-এ public key (তালা) থাকে। চাবি ছাড়া তালা খোলা যাবে না।

### SSH Key Pair কীভাবে কাজ করে:

```
আপনার Machine                    Remote Server
┌─────────────────┐              ┌─────────────────┐
│  Private Key    │              │  Public Key     │
│  (id_ed25519)   │◄── Match ───►│ (authorized_keys│
│  SECRET! কাউকে │              │  এখানে থাকে)   │
│  দেখাবে না      │              │                 │
└─────────────────┘              └─────────────────┘
```

### Step 1: Key Generate করুন

```bash
ssh-keygen -t ed25519 -C "your_email@example.com"

#OR

ssh-keygen -t rsa -b 4096 -C "your_email@example.com"
```

**প্রতিটা অংশের মানে:**
- `ssh-keygen` → SSH key তৈরির tool
- `-t ed25519` → key-এর type (algorithm) - ed25519 সবচেয়ে modern ও secure
- `-C "email"` → একটা comment/label (optional, কিন্তু helpful)

```
# Output:
Generating public/private ed25519 key pair.
Enter file in which to save the key (/home/devops/.ssh/id_ed25519):  ← Enter চাপো
Enter passphrase (empty for no passphrase):  ← চাইলে extra password দাও
Enter same passphrase again:

Your identification has been saved in /home/devops/.ssh/id_ed25519
Your public key has been saved in /home/devops/.ssh/id_ed25519.pub
The key fingerprint is:
SHA256:AbCdEfGhIjKlMnOpQrStUvWxYz your_email@example.com

+--[ED25519 256]--+
|        .o+      |
|       . o.      |
|      . + .      |
|     . + =       |
|    . S * o      |
|     . B + .     |
|      = o .      |
|     . + .       |
+----[SHA256]-----+
```

> **Passphrase কী?** - এটা আপনার private key-এর উপর আরেকটা layer of protection। Production-এ দেওয়া ভালো।

### Step 2: তৈরি হওয়া files দেখুন

```bash
ls -la ~/.ssh/
```

```
-rw-------  1 devops devops  411 Mar 15 10:00 id_ed25519       ← Private Key (SECRET!)
-rw-r--r--  1 devops devops   98 Mar 15 10:00 id_ed25519.pub   ← Public Key (share করো)
```

### Public Key দেখুন:
```bash
cat ~/.ssh/id_ed25519.pub
```
```
ssh-ed25519 AAAAC3NzaC1lZDI1NTE5AAAAIBkjH8s... your_email@example.com
```


## Part 3: Public Key Server-এ Copy করা

### Method 1: `ssh-copy-id` (সবচেয়ে সহজ)

```bash
ssh-copy-id devops@192.168.1.100
```

```
# Output:
/usr/bin/ssh-copy-id: INFO: Source of key(s) to be installed: "/home/devops/.ssh/id_ed25519.pub"
Number of key(s) added: 1

Now try logging into the machine, with:   "ssh 'devops@192.168.1.100'"
```

এটা automatically আপনার public key → server-এর `~/.ssh/authorized_keys` file-এ add করে দেয়।

### Method 2: Manual Copy (যখন ssh-copy-id নেই)

```bash
cat ~/.ssh/id_ed25519.pub | ssh devops@192.168.1.100 "mkdir -p ~/.ssh && cat >> ~/.ssh/authorized_keys"
```

### Step 3: এখন password ছাড়াই login করুন!

```bash
ssh devops@192.168.1.100
```
```
Welcome to Ubuntu 22.04 LTS
devops@remote-server:~$    ← কোনো password ছাড়াই!
```


## Part 4: ssh-agent (Passphrase ঝামেলা দূর করুন)

যদি আপনি private key-এ passphrase দিয়ে থাকেন, তাহলে প্রতিবার SSH connect করতে passphrase দিতে হবে - এটা বিরক্তিকর।

**ssh-agent** হলো একটা background program যেটা আপনার key memory-তে ধরে রাখে।


> ssh-agent হলো আপনার key ring - একবার চাবি ঢুকিয়ে দিয়ে, সারাদিন পকেটে রাখেন। বারবার locker খুলতে হয় না।

```bash
# Agent start করো
eval "$(ssh-agent -s)"
```
```
Agent pid 12345
```

```bash
# Key add করো
ssh-add ~/.ssh/id_ed25519
```
```
Enter passphrase for /home/devops/.ssh/id_ed25519: ****
Identity added: /home/devops/.ssh/id_ed25519 (your_email@example.com)
```

```bash
# কোন keys loaded আছে দেখো
ssh-add -l
```
```
256 SHA256:AbCdEf... your_email@example.com (ED25519)
```

এখন সারাদিন আর passphrase দিতে হবে না!


## Part 5: SSH Config File (~/.ssh/config) - Game Changer!

এটা অনেকেই জানে না, কিন্তু এটা আপনার **productivity বহুগুণ বাড়িয়ে দিবে।**

### সমস্যা:
```bash
# এই long command মনে রাখা কঠিন:
ssh -p 2222 -i ~/.ssh/my_special_key devops@192.168.1.100
```

### সমাধান - SSH Config File:
```bash
nano ~/.ssh/config
```

```
# ~/.ssh/config

# Server 1 - Production
Host prod
    HostName 192.168.1.100
    User devops
    Port 2222
    IdentityFile ~/.ssh/id_ed25519

# Server 2 - Staging
Host staging
    HostName 192.168.1.200
    User ubuntu
    Port 22
    IdentityFile ~/.ssh/id_ed25519

# GitHub
Host github.com
    User git
    IdentityFile ~/.ssh/id_ed25519
    AddKeysToAgent yes
```

### এখন connect করুন এভাবে:
```bash
ssh prod        # এটাই যথেষ্ট! পুরো command লাগবে না
ssh staging
```

### Config file-এর permissions ঠিক রাখুন:
```bash
chmod 600 ~/.ssh/config
```


## Part 6: SSH Tunneling (Advanced কিন্তু জরুরি!)

SSH Tunneling মানে - SSH-এর ভেতর দিয়ে অন্য traffic পাঠানো। DevOps-এ এটা অনেক কাজে লাগে।

### Local Port Forwarding:

**Use case:** Remote server-এ একটা database আছে (port 5432), সরাসরি access নেই - SSH দিয়ে আপনার local machine-এ নিয়ে আসতে পারেন।

```bash
ssh -L 5432:localhost:5432 devops@192.168.1.100
```

**মানে:**
- `-L` → Local forwarding
- `5432` → আপনার local machine-এর port
- `localhost:5432` → remote server থেকে যে address-এ যাবে
- `devops@192.168.1.100` → SSH server

এখন আপনি `localhost:5432`-এ connect করলে, সেটা actually remote server-এর 5432-এ যাবে!

```bash
# এখন local machine থেকেই remote DB access করুন:
psql -h localhost -p 5432 -U postgres
```

### Remote Port Forwarding:

**Use case:** আপনার local machine-এ একটা app আছে (port 3000), remote server থেকে সেটা access করতে চান।

```bash
ssh -R 9090:localhost:3000 devops@192.168.1.100
```

### Dynamic Port Forwarding (SOCKS Proxy):

```bash
ssh -D 1080 devops@192.168.1.100
```

এটা একটা SOCKS proxy তৈরি করে - সব browser traffic SSH-এর ভেতর দিয়ে পাঠাতে পারে।


## Part 7: SSH Security Best Practices

### Server-side সেটিংস (`/etc/ssh/sshd_config`):

```bash
sudo nano /etc/ssh/sshd_config
```

```
# Root login বন্ধ করে দিন
PermitRootLogin no

# Password login বন্ধ করে দিন (key-only)
PasswordAuthentication no

# Default port পরিবর্তন করে দিন (optional)
Port 2222

# Idle connection timeout
ClientAliveInterval 300
ClientAliveCountMax 2

# শুধু নির্দিষ্ট user allow করে দিন
AllowUsers devops munir
```

সেটিং পরিবর্তনের পর:
```bash
sudo systemctl restart sshd
```


## Part 8: Useful SSH Commands Summary

| Command | কাজ |
|---|---|
| `ssh user@host` | Basic connect |
| `ssh -p 2222 user@host` | Custom port |
| `ssh -i ~/.ssh/mykey user@host` | Specific key দিয়ে connect |
| `ssh-keygen -t ed25519` | New key pair তৈরি |
| `ssh-copy-id user@host` | Public key server-এ copy |
| `ssh-add ~/.ssh/mykey` | Key agent-এ add |
| `ssh-add -l` | Loaded keys দেখা |
| `ssh -L local:remote:port user@host` | Local port forward |
| `ssh -v user@host` | Verbose/debug mode |
| `ssh user@host 'command'` | Remote command run করো |

### Remote Command Run করার Example:
```bash
# Remote server-এ disk usage দেখা - login না করেই!
ssh devops@192.168.1.100 'df -h'

# Remote server-এ একটা file তৈরি করা:
ssh devops@192.168.1.100 'touch /tmp/testfile'
```


## 📝 Quick Summary

- SSH হলো encrypted remote connection tool - DevOps-এর backbone
- `ssh-keygen` দিয়ে key pair তৈরি করা যায় (private + public)
- `ssh-copy-id` দিয়ে server-এ public key পাঠানো যায়
- `ssh-agent` দিয়ে passphrase ঝামেলা এড়ানোর জন্য ব্যাবহার করা হয়
- `~/.ssh/config` দিয়ে shortcuts বানান - সময় বাঁচান
- SSH Tunneling দিয়ে secure port forwarding করা যায়
- Production-এ সবসময় `PasswordAuthentication no` রাখুন


## 🏋️ Practice Tasks

**Task 1:**
```bash
# নতুন একটা ED25519 key pair generate করুন
# এবং দুটো file (private + public) locate করুন
ssh-keygen -t ed25519 -C "practice@devops.com"
ls -la ~/.ssh/
```

**Task 2:**
```bash
# ~/.ssh/config file তৈরি করুন
# একটা alias "myserver" নামে add করুন যেকোনো host-এর জন্য
nano ~/.ssh/config
# তারপর: ssh myserver দিয়ে connect করার চেষ্টা করুন
```

**Task 3:**
```bash
# ssh-agent চালু করুন এবং আপনার key add করুন
eval "$(ssh-agent -s)"
ssh-add ~/.ssh/id_ed25519
ssh-add -l   # confirm করুন key loaded হয়েছে কিনা
```

---

## ⏭️ What's Next?

**Chapter 4 - Lesson 8: Firewall Management**
> `iptables`, `ufw`, `firewalld` Linux-এ কীভাবে network traffic control করা হয়, কোন port খোলা বা বন্ধ করতে হয়, এবং DevOps environment-এ firewall rules কীভাবে manage করতে হয়।


আপনি আজকে SSH-এর সবচেয়ে গুরুত্বপূর্ণ topics cover করেছেন! 🎉 এই knowledge আপনার DevOps career-এ প্রতিদিন কাজে আসবে। Practice করুন এবং যেকোনো সমস্যা হলে জানাতে ভুলবেন না! *Happy Learning* 🚀


<table width="100%">
  <tr>
    <td align="left">
      <a href="../06-Open-Ports-Sockets">← Open Ports &amp; Sockets</a>
    </td>
    <td align="right">
      <a href="../08-Firewall-Management">Firewall Management →</a>
    </td>
  </tr>
</table>