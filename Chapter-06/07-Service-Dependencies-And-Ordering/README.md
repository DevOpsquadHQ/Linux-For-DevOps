# Chapter 6 - Lesson 7: Service Dependencies & Ordering

**Chapter 6 | Lesson 7 of 8**

## 🎯 এই Lesson-এ আমরা যা শিখবো

Systemd-তে services গুলো কখন start হবে, কোনটার পরে কোনটা চলবে এই সব নিয়ন্ত্রণ করা হয় **Dependencies & Ordering** দিয়ে। এটা DevOps-এ অনেক গুরুত্বপূর্ণ কারণ real-world-এ একটা service অন্য একটার উপর নির্ভর করে।


মনে করেন আপনি একটা **Restaurant** চালান:
- আগে **Gas supply** চালু করতে হবে
- তারপর **Kitchen** চালু হবে
- তারপর **Waiter service** দেয়া শুরু করবে

যদি Gas supply না থাকে, তাহলে Kitchen চালু করা বোকামি। এই ধারণাটাই Systemd Dependencies।

## Dependency Keywords-এর বিস্তারিত

### 1. `After=` - শুধু Ordering নিয়ন্ত্রণ করে

```ini
After=network.target
```

> আমি `network.target`-এর পরে start হবো।
>
> কিন্তু এটা শুধু sequence বলে, dependency না। মানে network না উঠলেও এই service চেষ্টা করবে।


### 2. `Before=` - উল্টো দিক থেকে Ordering

```ini
Before=nginx.service
```

আমি `nginx.service`-এর আগে start হবো।

### 3. `Requires=` - Hard Dependency

```ini
Requires=postgresql.service
```

আমার জন্য `postgresql.service` অবশ্যই চালু থাকতে হবে। না থাকলে আমিও বন্ধ হয়ে যাবো।

এটা কঠোর (strict), dependency fail করলে এই service-ও fail করবে।


### 4. `Wants=` - Soft Dependency

```ini
Wants=redis.service
```

আমি চাই `redis.service` চালু থাকুক কিন্তু না থাকলেও আমি চলতে পারবো।

এটা নরম (mild), dependency fail করলেও এই service চলতে থাকবে।

### 5. `BindsTo=` - Requires-এর চেয়েও কঠোর

```ini
BindsTo=docker.service
```

`docker.service` বন্ধ হলে আমিও সাথে সাথে বন্ধ হয়ে যাবো।

`Requires=` শুধু start-time চেক করে। কিন্তু `BindsTo=` runtime-এও চেক রাখে।


### 6. `PartOf=` - গ্রুপের অংশ

```ini
PartOf=app.target
```

আমি `app.target`-এর একটা অংশ। `app.target` বন্ধ বা restart হলে আমিও হবো।


### 7. `Conflicts=` - একসাথে চলতে পারবে না

```ini
Conflicts=apache2.service
```

`apache2.service` রান থাকলে আমি রান থাকবো না। একে অপরের সাথে conflict করে।

উদাহরণ: nginx আর apache2 একই port-এ থাকলে conflict।


## Keywords একনজরে - তুলনামূলক টেবিল

| Keyword | কাজ | Failure হলে কী হয়? |
|---|---|---|
| `After=` | পরে start হবে | কিছু হয় না |
| `Before=` | আগে start হবে | কিছু হয় না |
| `Requires=` | Hard dependency | এই service-ও fail করবে |
| `Wants=` | Soft dependency | চলতে থাকবে |
| `BindsTo=` | Runtime hard link | সাথে সাথে বন্ধ হবে |
| `PartOf=` | গ্রুপের অংশ | গ্রুপ বন্ধ হলে বন্ধ |
| `Conflicts=` | একসাথে চলবে না | অন্যটা বন্ধ হবে |


## Real-World Example - একটি Web App Service

মনে করেন আপনার একটা web application আছে যেটা:
- **Database** (PostgreSQL) ছাড়া চলে না
- **Network** ছাড়া চলে না
- **Redis** থাকলে ভালো, না থাকলেও চলে
- **Apache** এর সাথে conflict করে

```ini
# /etc/systemd/system/myapp.service

[Unit]
Description=My Web Application
After=network.target postgresql.service
Requires=postgresql.service
Wants=redis.service
Conflicts=apache2.service

[Service]
Type=simple
User=appuser
WorkingDirectory=/opt/myapp
ExecStart=/opt/myapp/start.sh
Restart=on-failure

[Install]
WantedBy=multi-user.target
```

**এই file-এর মানে:**
- `After=network.target postgresql.service` → Network আর PostgreSQL start হওয়ার পরে চালু হবে
- `Requires=postgresql.service` → PostgreSQL না থাকলে এই app-ও চলবে না
- `Wants=redis.service` → Redis চেষ্টা করবে, না পেলেও চলবে
- `Conflicts=apache2.service` → Apache চললে এটা চলবে না


## Dependency দেখার Commands

### একটা service-এর সব dependency দেখুন:

```bash
systemctl list-dependencies nginx.service
```

**Output:**
```
nginx.service
● ├─system.slice
● ├─network.target
● └─sysinit.target
    ├─dev-hugepages.mount
    └─...
```

### Reverse dependency - কোন services এটার উপর নির্ভর করে:

```bash
systemctl list-dependencies --reverse nginx.service
```

### কোনো service-এর পুরো unit file দেখুন:

```bash
systemctl cat nginx.service
```

**Output:**
```ini
# /lib/systemd/system/nginx.service
[Unit]
Description=A high performance web server
After=network.target
...
```

### Service-এর সব properties দেখুন:

```bash
systemctl show nginx.service
```

শুধু নির্দিষ্ট property দেখতে:

```bash
systemctl show nginx.service --property=After
systemctl show nginx.service --property=Requires
systemctl show nginx.service --property=Wants
```

**Output:**
```
After=sysinit.target system.slice network.target
Requires=system.slice
Wants=
```

## Hands-On - নিজে একটা Dependency Chain তৈরি করুন

ধরেন আপনি একটা **database-dependent app** তৈরি করবেন। প্রথমে একটা fake database service বানান:

### Step 1 - Fake Database Service

```bash
sudo nano /etc/systemd/system/fakedb.service
```

```ini
[Unit]
Description=Fake Database Service

[Service]
Type=oneshot
RemainAfterExit=yes
ExecStart=/bin/echo "FakeDB started"
ExecStop=/bin/echo "FakeDB stopped"

[Install]
WantedBy=multi-user.target
```

### Step 2 - Dependent App Service

```bash
sudo nano /etc/systemd/system/fakeapp.service
```

```ini
[Unit]
Description=Fake App (depends on FakeDB)
After=fakedb.service
Requires=fakedb.service

[Service]
Type=oneshot
RemainAfterExit=yes
ExecStart=/bin/echo "FakeApp started successfully"

[Install]
WantedBy=multi-user.target
```

### Step 3 - Reload ও Test করুন

```bash
# Daemon reload
sudo systemctl daemon-reload

# শুধু fakeapp চালু করুন, fakedb আপনা-আপনি চালু হবে
sudo systemctl start fakeapp.service

# Status চেক করুন
systemctl status fakeapp.service
systemctl status fakedb.service
```

**Output দেখবে:**
```
● fakeapp.service - Fake App (depends on FakeDB)
     Loaded: loaded (/etc/systemd/system/fakeapp.service)
     Active: active (exited)
```

এখন fakedb বন্ধ করে দেখুন fakeapp কী করে:

```bash
sudo systemctl stop fakedb.service
systemctl status fakeapp.service
```

`Requires=` থাকার কারণে fakeapp-ও বন্ধ হয়ে যাবে, এটাই Hard Dependency!


## Wants= vs Requires= - Live পার্থক্য

### Requires= দিয়ে test:
```ini
Requires=fakedb.service   # fakedb বন্ধ হলে fakeapp-ও বন্ধ
```

```bash
sudo systemctl stop fakedb.service
systemctl is-active fakeapp.service
# Output: inactive ← fakeapp-ও বন্ধ হয়ে গেছে!
```

### Wants= দিয়ে test করলে:
```ini
Wants=fakedb.service   # fakedb বন্ধ হলেও fakeapp চলবে
```

```bash
sudo systemctl stop fakedb.service
systemctl is-active fakeapp.service
# Output: active ← fakeapp চলছে!
```

## `.target` ফাইল দিয়ে Service Group তৈরি

DevOps-এ প্রায়ই একসাথে অনেক service চালু করতে হয়। Target দিয়ে সেটা সহজ হয়।

```bash
sudo nano /etc/systemd/system/mystack.target
```

```ini
[Unit]
Description=My Full Application Stack
Requires=fakedb.service fakeapp.service
After=fakedb.service fakeapp.service
```

```bash
sudo systemctl daemon-reload
sudo systemctl start mystack.target
# এখন fakedb আর fakeapp দুটোই একসাথে চালু হবে!
```

## Override করার সঠিক পদ্ধতি - `drop-in` files

কোনো existing service-এ dependency যোগ করতে চাইলে সরাসরি unit file edit না করে **drop-in** ব্যবহার করুন:

```bash
sudo systemctl edit nginx.service
```

এটা একটা editor খুলবে। আপনি লিখবেন:

```ini
[Unit]
After=postgresql.service
Wants=redis.service
```

Save করলে এটা তৈরি হবে:
```
/etc/systemd/system/nginx.service.d/override.conf
```

এতে original file নিরাপদ থাকে, আর আপনার changes আলাদা file-এ থাকে।


## 📋 Quick Summary

- `After=` / `Before=` → শুধু order নিয়ন্ত্রণ করে, dependency না
- `Requires=` → Hard dependency, fail করলে এটাও fail
- `Wants=` → Soft dependency, fail করলেও চলে
- `BindsTo=` → Runtime-এও চেক রাখে, সাথে সাথে বন্ধ হয়
- `Conflicts=` → দুটো service একসাথে চলতে পারবে না
- `systemctl list-dependencies` → dependency tree দেখায়
- `systemctl edit` → drop-in file দিয়ে safely override করা যায়
- `.target` → একাধিক service-এর group তৈরি করা যায়


## 🏋️ Practice Tasks

1. **fakedb.service** এবং **fakeapp.service** তৈরি করুন। `Requires=` দিয়ে test করুন, fakedb বন্ধ করলে fakeapp-ও বন্ধ হয় কিনা দেখুন।

2. এখন `Requires=` বদলে `Wants=` দিন। আবার test করুন, এবার fakeapp চালু থাকে কিনা দেখুন।

3. `systemctl list-dependencies nginx.service` (বা যেকোনো installed service) run করুন এবং output-টা দেখুন।

---

## ⏭️ What's Next

**Chapter 6 - Lesson 8: journalctl Deep Dive**
journalctl দিয়ে logs filter করা, export করা, persistent logs configure করা এবং real DevOps debugging scenarios! *Happy Learning* 🚀

<table width="100%">
  <tr>
    <td align="left">
      <a href="../06-Systemd-Targets">← Systemd Targets</a>
    </td>
    <td align="right">
      <a href="../08-journalctl-Deep-Dive">journalctl Deep Dive →</a>
    </td>
  </tr>
</table>