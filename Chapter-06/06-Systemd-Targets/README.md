# Chapter 6 - Lesson 6: Systemd Targets

**Chapter 6 | Lesson 6 of 8**

## 🎯 এই Lesson-এ আমরা যা শিখব:

- Systemd Target কী?
- পুরনো Runlevel system কী ছিল?
- Runlevel vs Target comparison
- গুরুত্বপূর্ণ Targets এবং তাদের কাজ
- Target পরিবর্তন করার commands
- Default Target set করা
- DevOps-এ কখন এটা কাজে লাগে

## Runlevel কী?

Linux-এর পুরনো দিনে (SysV init system) **Runlevel** ছিল। Runlevel মানে হলো System এখন কোন mode-এ চলবে?

### Runlevel Table (পুরনো SysV):

| Runlevel | মানে |
|----------|------|
| 0 | Halt - System বন্ধ করো |
| 1 | Single-user mode - শুধু root, কোনো network নেই |
| 2 | Multi-user, network ছাড়া |
| 3 | Multi-user, network সহ (CLI only) |
| 4 | Unused / Custom |
| 5 | Multi-user, network + GUI (Graphical) |
| 6 | Reboot |

> Runlevel হলো একটা building-এর floor selector অর্থাৎ কোন floor-এ যাবে, সেটাই Runlevel। Systemd-তে এই floor selector-এর নাম হয়েছে Target।

## Systemd Target কী?

Systemd আসার পর Runlevel-এর জায়গা নিয়েছে Target। Target হলো এক ধরনের goal বা milestone - system কতটুকু boot করবে, কোন services চলবে, সেটা define করে।

Target file-গুলো শেষ হয় `.target` দিয়ে।

### Runlevel → Target Mapping:

| পুরনো Runlevel | নতুন Systemd Target | কাজ |
|----------------|---------------------|-----|
| 0 | `poweroff.target` | System বন্ধ |
| 1 | `rescue.target` | Single-user / rescue mode |
| 2, 3, 4 | `multi-user.target` | CLI, multi-user, network সহ |
| 5 | `graphical.target` | GUI + multi-user |
| 6 | `reboot.target` | System restart |
| - | `emergency.target` | Minimum environment, root only |
| - | `default.target` | Default target (symlink) |

## গুরুত্বপূর্ণ Targets বিস্তারিত

### 1️) `poweroff.target`
System সম্পূর্ণ বন্ধ করে দেয়। সব service stop হয়, power cut হয়।

### 2️) `rescue.target`
- শুধু root user login করতে পারে
- Network নেই, graphical নেই
- System repair বা troubleshoot করার জন্য
- পুরনো Runlevel 1 এর সমতুল্য

### 3️) `emergency.target`
- `rescue.target` এর চেয়েও বেশি minimal
- শুধু root filesystem mount হয়, অন্য কিছু না
- System একদম ভেঙে পড়লে এটা ব্যবহার হয়

### 4️) `multi-user.target` (DevOps-এর জন্য সবচেয়ে গুরুত্বপূর্ণ)
- Multi-user - অনেক user একসাথে login করতে পারে
- Network চালু থাকে
- CLI only - কোনো GUI নেই
- Server-এ এটাই সাধারণত default থাকে
- পুরনো Runlevel 3 এর সমতুল্য

### 5️) `graphical.target`
- `multi-user.target` এর সব সুবিধা + GUI (Desktop)
- Desktop Linux-এ এটা default থাকে
- Server-এ সাধারণত এটা লাগে না
- পুরনো Runlevel 5 এর সমতুল্য

### 6️) `reboot.target`
System restart করে।

### 7️) `default.target`
- এটা আসলে একটা symbolic link (symlink)
- সাধারণত `graphical.target` বা `multi-user.target`-এ point করে
- System boot হওয়ার সময় এই target দেখে boot হয়

## Commands - Target নিয়ে কাজ করা

### 1. বর্তমান Target দেখুন

```bash
systemctl get-default
```

**Output (server-এ):**
```
multi-user.target
```

**Output (desktop-এ):**
```
graphical.target
```

> এই command বলে দেয় system boot হলে কোন target-এ যাবে।

### 2. সব available Targets দেখুন

```bash
systemctl list-units --type=target
```

**Output:**
```
UNIT                   LOAD   ACTIVE SUB    DESCRIPTION
basic.target           loaded active active Basic System
cryptsetup.target      loaded active active Local Encrypted Volumes
getty.target           loaded active active Login Prompts
graphical.target       loaded active active Graphical Interface
local-fs.target        loaded active active Local File Systems
multi-user.target      loaded active active Multi-User System
network.target         loaded active active Network
network-online.target  loaded active active Network is Online
...
```

### 3. সব Targets দেখুন (inactive সহ)

```bash
systemctl list-units --type=target --all
```

### 4. Default Target পরিবর্তন করুন

#### CLI server mode সেট করুন (GUI বন্ধ):
```bash
sudo systemctl set-default multi-user.target
```

**Output:**
```
Removed /etc/systemd/system/default.target.
Created symlink /etc/systemd/system/default.target → /lib/systemd/system/multi-user.target.
```

> এখন থেকে system boot হলে **CLI mode**-এ যাবে।

#### GUI mode সেট করুন:
```bash
sudo systemctl set-default graphical.target
```

> এখন থেকে system boot হলে **Desktop/GUI**-তে যাবে।


### 5. এখনই (তাৎক্ষণিক) Target পরিবর্তন করো

`set-default` শুধু পরের boot এ কাজ করে। এখনই পরিবর্তন করতে `isolate` ব্যবহার করুন:

```bash
sudo systemctl isolate multi-user.target
```

> এই command এখনই system-কে CLI mode-এ নিয়ে যাবে। যদি GUI চালু থাকে, তাহলে সেটা বন্ধ হয়ে যাবে!

```bash
sudo systemctl isolate graphical.target
```

> এই command এখনই GUI চালু করবে।

⚠️ **সতর্কতা:** `isolate` সাথে সাথে কাজ করে। তাই সাবধানে ব্যবহার করুন। unsaved কাজ হারাতে পারেন।


### 6. Rescue Mode-এ যান (Troubleshooting)

```bash
sudo systemctl isolate rescue.target
```

**কখন ব্যবহার করবেন:**
- কোনো service system boot হতে দিচ্ছে না
- Password reset করতে হবে
- File system check করতে হবে

### 7. Target-এর details দেখুন

```bash
systemctl cat multi-user.target
```

**Output:**
```ini
# /lib/systemd/system/multi-user.target
[Unit]
Description=Multi-User System
Documentation=man:systemd.special(7)
Requires=basic.target
Conflicts=rescue.service rescue.target
After=basic.target rescue.service rescue.target
AllowIsolate=yes
```

> এখানে দেখুন `multi-user.target` চালু হতে হলে আগে `basic.target` চালু হতে হবে।

### 8. Target-এর dependency tree দেখুন

```bash
systemctl list-dependencies multi-user.target
```

**Output (সংক্ষিপ্ত):**
```
multi-user.target
● ├─atd.service
● ├─cron.service
● ├─dbus.service
● ├─getty.target
● │ └─getty@tty1.service
● ├─network.target
● ├─remote-fs.target
● └─basic.target
     ├─...
```

> এই tree দেখায় `multi-user.target` পৌঁছাতে হলে কোন কোন service/target আগে চালু হতে হবে।

## Target-এর Boot Hierarchy

System boot হওয়ার সময় Targets একটার পর একটা পূরণ হয়:

```
sysinit.target          ← Kernel চালু, /proc, /sys mount
      ↓
basic.target            ← Basic hardware, udev, timers
      ↓
network.target          ← Network interface চালু
      ↓
multi-user.target       ← সব system services চালু (CLI ready)
      ↓
graphical.target        ← Display manager চালু (GUI ready)
```

> এটা যেন একটা রকেট লঞ্চারের মতো checklist। প্রতিটা step পার হলে তবেই পরেরটায় যাওয়া যায়।


## default.target আসলে কী?

```bash
ls -la /etc/systemd/system/default.target
```

**Output:**
```
lrwxrwxrwx 1 root root 40 Jan 10 09:23 /etc/systemd/system/default.target -> /lib/systemd/system/graphical.target
```

> `default.target` আসলে একটা symlink এটা point করছে `graphical.target`-এ। `set-default` command এই symlink-কেই পরিবর্তন করে ফেলে।

## DevOps-এ Real-World Use Cases

### Use Case 1: Server-এ GUI বন্ধ রাখা
Production server-এ GUI লাগে না কারন resource waste করে:
```bash
sudo systemctl set-default multi-user.target
```

### Use Case 2: Boot হচ্ছে না - Rescue Mode
```bash
# Boot time-এ GRUB menu থেকে kernel parameter-এ যোগ করুন:
systemd.unit=rescue.target
# অথবা চালু থাকলে:
sudo systemctl isolate rescue.target
```

### Use Case 3: Script-এ current target check করা
```bash
#!/bin/bash
current=$(systemctl get-default)
if [ "$current" == "graphical.target" ]; then
    echo "Warning: Server is in GUI mode - unnecessary resource usage!"
fi
```

### Use Case 4: Service কোন Target-এ চলে জানা
```bash
systemctl show nginx.service | grep WantedBy
# Output: WantedBy=multi-user.target
```

> এর মানে nginx `multi-user.target`-এ enable হলে চলবে।

## Quick Reference Table

| Command | কাজ |
|---------|-----|
| `systemctl get-default` | বর্তমান default target দেখা যায় |
| `systemctl set-default multi-user.target` | Default target পরিবর্তন করে |
| `systemctl isolate rescue.target` | এখনই rescue mode-এ যাওয়া যায় |
| `systemctl list-units --type=target` | সব active target দেখা যায় |
| `systemctl list-dependencies graphical.target` | Target dependency দেখা যায় |
| `systemctl cat multi-user.target` | Target file দেখা যায় |

## 📝 Quick Summary

- Target হলো Systemd-এর Runlevel, system কোন mode-এ থাকবে তা নির্ধারণ করে
- `multi-user.target` = Server mode (CLI + Network) - DevOps-এর default
- `graphical.target` = Desktop mode (GUI + multi-user)
- `rescue.target` = Emergency repair mode
- `default.target` = একটা symlink যা বলে boot-এ কোন target ব্যবহার করতে হবে
- `set-default` পরের boot-এ কাজ করে, `isolate` এখনই কাজ করে


## 🏋️ Practice Tasks

**Task 1:** 

আপনার system-এর বর্তমান default target কী তা দেখুন এবং সব active target list করুন।

**Task 2:** 

`multi-user.target`-এর dependency tree দেখুন। কতগুলো service/target তার উপর নির্ভর করে তা গুণে দেখুন।

**Task 3:**

`systemctl cat graphical.target` রান করে দেখুন, এটা `multi-user.target`-এর সাথে কীভাবে connected।

---

## ⏭️ What's Next?

**Chapter 6 - Lesson 7: Service Dependencies & Ordering**

`After=`, `Requires=`, `Wants=`, `Before=` এই directives দিয়ে কীভাবে services-এর মধ্যে সম্পর্ক এবং চালু হওয়ার order নির্ধারণ করা হয়, সেটা শিখবো। DevOps-এ custom service লেখার সময় এটা অনেক কাজে লাগে! *Happy Learning* 🚀

<table width="100%">
  <tr>
    <td align="left">
      <a href="../05-Systemd-Timers">← Systemd Timers</a>
    </td>
    <td align="right">
      <a href="../07-Service-Dependencies-And-Ordering">Service Dependencies &amp; Ordering →</a>
    </td>
  </tr>
</table>