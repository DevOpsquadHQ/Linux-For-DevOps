# Chapter 5 - Lesson 11: Real-World DevOps Scripts

**Chapter 5 | Lesson 11 of 11**

আজকের lesson টা Chapter 5 এর grand finale! এতদিন আমরা যা শিখেছি যেমন variables, loops, functions, arrays, text processing, error handling, cron সব কিছু আজকে একসাথে use করবো। Real DevOps engineer রা যে ধরনের scripts লেখে, সেগুলো আজকে আমরা build করবো।

আজকে তিনটা **production-grade script** তৈরি করবো:

1. **Log Rotation Script** - পুরনো logs automatically clean করবে
2. **Health Check Script** - server এর health monitor করবে
3. **Auto-Backup Script** - important files automatically backup করবে

## Script 1: Log Rotation Script

### এটা কী এবং কেন দরকার?

Real life এ application গুলো প্রতিদিন হাজার হাজার line log লেখে। যদি কেউ এগুলো পরিষ্কার না করে, তাহলে:
- Disk ভরে যাবে
- Server crash করবে
- নতুন logs লেখা বন্ধ হয়ে যাবে

একটা **log rotation script** automatically:
- পুরনো logs কে archive করে (compress করে রাখে)
- নির্দিষ্ট দিনের বেশি পুরনো logs delete করে
- একটা report তৈরি করে কী কী করা হলো

### Script টা লেখার আগে Structure Plan

```
/var/log/myapp/          ← এখানে application logs থাকে
    app.log              ← current log file
    app.log.2024-01-01.gz ← rotated & compressed old log
    app.log.2024-01-02.gz
```

### Script: `log_rotate.sh`

```bash
#!/bin/bash
# ============================================================
# Script Name : log_rotate.sh
# Description : Rotate, compress, and clean old log files
# Author      : DevOps Engineer
# ============================================================

# ---------- Error Handling ----------
set -euo pipefail

# ---------- Configuration ----------
LOG_DIR="/var/log/myapp"          # যে directory তে logs আছে
LOG_FILE="$LOG_DIR/app.log"       # main log file
MAX_AGE_DAYS=7                    # কত দিনের বেশি পুরনো logs delete হবে
MAX_SIZE_MB=50                    # log file কত MB হলে rotate হবে
ARCHIVE_DIR="$LOG_DIR/archive"    # compressed logs কোথায় রাখবো
REPORT_FILE="/var/log/rotation_report.log"  # rotation report

# ---------- Colors for output ----------
GREEN='\033[0;32m'
YELLOW='\033[1;33m'
RED='\033[0;31m'
NC='\033[0m' # No Color

# ---------- Logging Function ----------
log_message() {
    local level="$1"
    local message="$2"
    local timestamp
    timestamp=$(date '+%Y-%m-%d %H:%M:%S')
    echo "[$timestamp] [$level] $message" | tee -a "$REPORT_FILE"
}

# ---------- Check if running as root ----------
check_root() {
    if [[ $EUID -ne 0 ]]; then
        echo -e "${RED}Error: This script must be run as root!${NC}"
        exit 1
    fi
}

# ---------- Setup directories ----------
setup_dirs() {
    if [[ ! -d "$LOG_DIR" ]]; then
        mkdir -p "$LOG_DIR"
        log_message "INFO" "Created log directory: $LOG_DIR"
    fi

    if [[ ! -d "$ARCHIVE_DIR" ]]; then
        mkdir -p "$ARCHIVE_DIR"
        log_message "INFO" "Created archive directory: $ARCHIVE_DIR"
    fi
}

# ---------- Get file size in MB ----------
get_file_size_mb() {
    local file="$1"
    # du -m দিয়ে MB তে size বের করি
    du -m "$file" 2>/dev/null | awk '{print $1}'
}

# ---------- Rotate the log file ----------
rotate_log() {
    if [[ ! -f "$LOG_FILE" ]]; then
        log_message "WARNING" "Log file not found: $LOG_FILE - skipping rotation"
        return
    fi

    local file_size
    file_size=$(get_file_size_mb "$LOG_FILE")

    log_message "INFO" "Current log size: ${file_size}MB (limit: ${MAX_SIZE_MB}MB)"

    if (( file_size >= MAX_SIZE_MB )); then
        # নতুন archive filename তৈরি করো date দিয়ে
        local date_stamp
        date_stamp=$(date '+%Y-%m-%d_%H-%M-%S')
        local archive_name="$ARCHIVE_DIR/app.log.$date_stamp.gz"

        # Log file কে compress করে archive এ পাঠাও
        gzip -c "$LOG_FILE" > "$archive_name"

        # Original log file টা empty করো (truncate) - delete না!
        # কারণ application হয়তো এখনো এই file এ লিখছে
        truncate -s 0 "$LOG_FILE"

        log_message "INFO" "✅ Rotated: $LOG_FILE → $archive_name"
        echo -e "${GREEN}Log rotated successfully!${NC}"
    else
        log_message "INFO" "Log size is within limit. No rotation needed."
        echo -e "${YELLOW}No rotation needed (${file_size}MB < ${MAX_SIZE_MB}MB)${NC}"
    fi
}

# ---------- Delete old archives ----------
cleanup_old_archives() {
    log_message "INFO" "Cleaning archives older than $MAX_AGE_DAYS days..."

    # find দিয়ে পুরনো files খুঁজে delete করো
    local deleted_count=0
    while IFS= read -r old_file; do
        rm -f "$old_file"
        log_message "INFO" "🗑️  Deleted old archive: $old_file"
        (( deleted_count++ ))
    done < <(find "$ARCHIVE_DIR" -name "*.gz" -mtime +"$MAX_AGE_DAYS" 2>/dev/null)

    if (( deleted_count == 0 )); then
        log_message "INFO" "No old archives to delete."
    else
        log_message "INFO" "Total deleted: $deleted_count archive(s)"
    fi
}

# ---------- Show disk usage summary ----------
show_summary() {
    echo ""
    echo "=========================================="
    echo "       📊 LOG ROTATION SUMMARY"
    echo "=========================================="
    echo "Log Directory  : $LOG_DIR"
    echo "Archive Dir    : $ARCHIVE_DIR"
    echo "Archives kept  : $(find "$ARCHIVE_DIR" -name "*.gz" 2>/dev/null | wc -l) file(s)"

    local archive_size
    archive_size=$(du -sh "$ARCHIVE_DIR" 2>/dev/null | awk '{print $1}')
    echo "Archive Size   : $archive_size"
    echo "Report saved   : $REPORT_FILE"
    echo "=========================================="
}

# ---------- Main Function ----------
main() {
    echo -e "${GREEN}🔄 Starting Log Rotation Script...${NC}"
    log_message "INFO" "=== Log Rotation Started ==="

    check_root
    setup_dirs
    rotate_log
    cleanup_old_archives
    show_summary

    log_message "INFO" "=== Log Rotation Completed ==="
    echo -e "${GREEN}✅ Log rotation complete!${NC}"
}

# Script execute করো
main
```

### Script এর Key Concepts ব্যাখ্যা

| Part | কী করছে |
|------|---------|
| `set -euo pipefail` | Error হলে script বন্ধ হবে |
| `truncate -s 0` | File delete না করে empty করে - application crash এড়ায় |
| `gzip -c` | `-c` মানে stdout এ output দাও - original file রেখে দাও |
| `find -mtime +7` | ৭ দিনের বেশি পুরনো files খোঁজো |
| `tee -a` | Screen এ দেখাও এবং file এও লেখো |

## Script 2: Server Health Check Script

### এটা কী এবং কেন দরকার?

একজন DevOps engineer এর প্রতিদিন চেক করতে হয়:
- CPU কতটা busy?
- Memory কতটা ভরা?
- Disk কতটা ভরা?
- Important services চলছে কিনা?
- Server কতক্ষণ ধরে চালু আছে?

এই script automatically এগুলো চেক করবে এবং problem থাকলে **alert** দেবে।

### Script: `health_check.sh`

```bash
#!/bin/bash
# ============================================================
# Script Name : health_check.sh
# Description : Monitor server health - CPU, Memory, Disk, Services
# ============================================================

set -uo pipefail  # note: -e নেই কারণ কিছু checks fail করতে পারে স্বাভাবিকভাবে

# ---------- Configuration ----------
CPU_THRESHOLD=80        # CPU % এর বেশি হলে alert
MEMORY_THRESHOLD=85     # Memory % এর বেশি হলে alert
DISK_THRESHOLD=90       # Disk % এর বেশি হলে alert

# কোন services চেক করবো
SERVICES=("nginx" "ssh" "cron")

# Report কোথায় save হবে
REPORT_DIR="/var/log/health_reports"
REPORT_FILE="$REPORT_DIR/health_$(date '+%Y-%m-%d_%H-%M').txt"

# ---------- Colors ----------
RED='\033[0;31m'
GREEN='\033[0;32m'
YELLOW='\033[1;33m'
BLUE='\033[0;34m'
NC='\033[0m'

# ---------- Alert tracking ----------
ALERT_COUNT=0
ALERTS=()

# ---------- Helper: Print section header ----------
print_header() {
    echo ""
    echo -e "${BLUE}══════════════════════════════════════${NC}"
    echo -e "${BLUE}  $1${NC}"
    echo -e "${BLUE}══════════════════════════════════════${NC}"
}

# ---------- Helper: Add alert ----------
add_alert() {
    local message="$1"
    ALERTS+=("⚠️  $message")
    (( ALERT_COUNT++ ))
}

# ---------- Check 1: CPU Usage ----------
check_cpu() {
    print_header "🖥️  CPU CHECK"

    # top দিয়ে CPU idle % বের করো, তারপর 100 থেকে বিয়োগ করো
    local cpu_idle
    cpu_idle=$(top -bn1 | grep "Cpu(s)" | awk '{print $8}' | tr -d '%')

    # যদি awk দিয়ে না পাই, vmstat try করি
    if [[ -z "$cpu_idle" ]]; then
        cpu_idle=$(vmstat 1 2 | tail -1 | awk '{print $15}')
    fi

    local cpu_usage
    cpu_usage=$(echo "100 - $cpu_idle" | bc 2>/dev/null || echo "0")
    cpu_usage=${cpu_usage%.*}  # decimal part বাদ দাও

    echo "Current CPU Usage: ${cpu_usage}%"
    echo "Threshold        : ${CPU_THRESHOLD}%"

    if (( cpu_usage >= CPU_THRESHOLD )); then
        echo -e "${RED}❌ ALERT: CPU usage is HIGH (${cpu_usage}%)${NC}"
        add_alert "CPU usage is ${cpu_usage}% (threshold: ${CPU_THRESHOLD}%)"
    else
        echo -e "${GREEN}✅ CPU is OK (${cpu_usage}%)${NC}"
    fi

    # Top 3 CPU-consuming processes দেখাও
    echo ""
    echo "Top 3 CPU-consuming processes:"
    ps aux --sort=-%cpu 2>/dev/null | awk 'NR==2,NR==4 {printf "  %-20s CPU: %s%%\n", $11, $3}'
}

# ---------- Check 2: Memory Usage ----------
check_memory() {
    print_header "MEMORY CHECK"

    # free command দিয়ে memory info বের করো
    local mem_total mem_used mem_percent
    mem_total=$(free -m | awk '/^Mem:/ {print $2}')
    mem_used=$(free -m | awk '/^Mem:/ {print $3}')
    mem_percent=$(echo "scale=1; $mem_used * 100 / $mem_total" | bc)
    mem_percent_int=${mem_percent%.*}

    echo "Total Memory  : ${mem_total} MB"
    echo "Used Memory   : ${mem_used} MB"
    echo "Usage         : ${mem_percent}%"
    echo "Threshold     : ${MEMORY_THRESHOLD}%"

    if (( mem_percent_int >= MEMORY_THRESHOLD )); then
        echo -e "${RED}❌ ALERT: Memory usage is HIGH (${mem_percent}%)${NC}"
        add_alert "Memory usage is ${mem_percent}% (threshold: ${MEMORY_THRESHOLD}%)"
    else
        echo -e "${GREEN}✅ Memory is OK (${mem_percent}%)${NC}"
    fi

    # Swap info
    local swap_total swap_used
    swap_total=$(free -m | awk '/^Swap:/ {print $2}')
    swap_used=$(free -m | awk '/^Swap:/ {print $3}')
    echo ""
    echo "Swap: ${swap_used}MB used of ${swap_total}MB"
}

# ---------- Check 3: Disk Usage ----------
check_disk() {
    print_header "💾 DISK CHECK"

    # প্রতিটা mounted filesystem চেক করো
    echo "Filesystem Usage:"
    echo "--------------------------------------------"

    while IFS= read -r line; do
        # df output parse করো
        local usage_percent filesystem mountpoint
        usage_percent=$(echo "$line" | awk '{print $5}' | tr -d '%')
        filesystem=$(echo "$line" | awk '{print $1}')
        mountpoint=$(echo "$line" | awk '{print $6}')

        printf "  %-20s %s%% used at %s\n" "$filesystem" "$usage_percent" "$mountpoint"

        if (( usage_percent >= DISK_THRESHOLD )); then
            echo -e "  ${RED}❌ ALERT: Disk ${mountpoint} is ${usage_percent}% full!${NC}"
            add_alert "Disk $mountpoint is ${usage_percent}% full (threshold: ${DISK_THRESHOLD}%)"
        fi
    done < <(df -h | grep -v "^Filesystem" | grep -v "tmpfs" | grep -v "udev")
}

# ---------- Check 4: Service Status ----------
check_services() {
    print_header "⚙️  SERVICE CHECK"

    for service in "${SERVICES[@]}"; do
        # systemctl দিয়ে service status চেক করো
        if systemctl is-active --quiet "$service" 2>/dev/null; then
            echo -e "  ${GREEN}✅ $service is RUNNING${NC}"
        else
            echo -e "  ${RED}❌ $service is NOT RUNNING${NC}"
            add_alert "Service '$service' is not running!"
        fi
    done
}

# ---------- Check 5: System Uptime ----------
check_uptime() {
    print_header "UPTIME & LOAD"

    echo "Server Uptime : $(uptime -p)"
    echo "Current Time  : $(date)"
    echo ""

    # Load average বের করো
    local load_1 load_5 load_15
    read -r load_1 load_5 load_15 _ < /proc/loadavg

    echo "Load Average:"
    echo "  Last 1 min  : $load_1"
    echo "  Last 5 min  : $load_5"
    echo "  Last 15 min : $load_15"

    # CPU count বের করো
    local cpu_count
    cpu_count=$(nproc)
    echo "  CPU Cores   : $cpu_count"

    # যদি load average CPU count এর বেশি হয় → warning
    local load_int=${load_1%.*}
    if (( load_int > cpu_count )); then
        echo -e "${YELLOW}⚠️  Load average is higher than CPU count!${NC}"
        add_alert "High load average: $load_1 (CPU cores: $cpu_count)"
    fi
}

# ---------- Final Alert Summary ----------
show_alert_summary() {
    print_header "📋 HEALTH CHECK SUMMARY"

    echo "Check Time: $(date)"
    echo "Hostname  : $(hostname)"
    echo ""

    if (( ALERT_COUNT == 0 )); then
        echo -e "${GREEN} All checks PASSED! Server is healthy.${NC}"
    else
        echo -e "${RED} $ALERT_COUNT ALERT(S) FOUND:${NC}"
        echo ""
        for alert in "${ALERTS[@]}"; do
            echo -e "${RED}  $alert${NC}"
        done
    fi
    echo ""
}

# ---------- Save report ----------
save_report() {
    mkdir -p "$REPORT_DIR"
    # সব output কে report file এ save করো
    {
        echo "Health Check Report - $(date)"
        echo "Hostname: $(hostname)"
        echo "====================================="
        echo "Alerts Found: $ALERT_COUNT"
        for alert in "${ALERTS[@]}"; do
            echo "$alert"
        done
    } > "$REPORT_FILE"

    echo "📄 Report saved: $REPORT_FILE"
}

# ---------- Main ----------
main() {
    echo -e "${GREEN} Starting Server Health Check...${NC}"
    echo "Time: $(date)"

    check_cpu
    check_memory
    check_disk
    check_services
    check_uptime
    show_alert_summary
    save_report
}

main
```

### Script এর Key Concepts ব্যাখ্যা

| Technique | কোথায় ব্যবহার হয়েছে |
|-----------|---------------------|
| Array তে alerts জমা করা | `ALERTS+=("message")` |
| bc দিয়ে math | Memory percentage calculate |
| `while IFS= read -r` | df output line by line পড়া |
| `systemctl is-active --quiet` | Service চেক করা silently |
| `/proc/loadavg` | সরাসরি kernel থেকে load data |


## Script 3: Auto-Backup Script

### এটা কী এবং কেন দরকার?

Production server এ সবচেয়ে গুরুত্বপূর্ণ কাজ হলো **backup**। এই script:
- Important directories automatically backup করবে
- Backup কে compress করবে
- পুরনো backups delete করবে
- একটা log রাখবে সব কিছুর

### Script: `auto_backup.sh`

```bash
#!/bin/bash
# ============================================================
# Script Name : auto_backup.sh
# Description : Automated backup with compression & retention
# ============================================================

set -euo pipefail

# ---------- Configuration ----------
# কোন directories backup হবে
BACKUP_SOURCES=(
    "/etc"
    "/home"
    "/var/www"
)

# Backup কোথায় রাখবো
BACKUP_DEST="/backup"

# কত দিনের backup রাখবো
RETENTION_DAYS=7

# Backup filename এ date ব্যবহার করবো
DATE_STAMP=$(date '+%Y-%m-%d_%H-%M-%S')
HOSTNAME=$(hostname)
BACKUP_NAME="${HOSTNAME}_backup_${DATE_STAMP}.tar.gz"
BACKUP_PATH="$BACKUP_DEST/$BACKUP_NAME"

# Log file
LOG_FILE="/var/log/auto_backup.log"

# ---------- Colors ----------
GREEN='\033[0;32m'
RED='\033[0;31m'
YELLOW='\033[1;33m'
NC='\033[0m'

# ---------- Logging ----------
log() {
    local level="$1"
    local msg="$2"
    echo "[$(date '+%Y-%m-%d %H:%M:%S')] [$level] $msg" | tee -a "$LOG_FILE"
}

# ---------- Check disk space before backup ----------
check_disk_space() {
    log "INFO" "Checking available disk space..."

    local available_kb
    available_kb=$(df -k "$BACKUP_DEST" 2>/dev/null | awk 'NR==2 {print $4}' || echo "0")

    local available_mb=$(( available_kb / 1024 ))
    log "INFO" "Available space in $BACKUP_DEST: ${available_mb}MB"

    # কমপক্ষে 500MB দরকার
    if (( available_mb < 500 )); then
        log "ERROR" "Not enough disk space! Only ${available_mb}MB available (need 500MB)"
        exit 1
    fi
}

# ---------- Verify backup sources exist ----------
verify_sources() {
    log "INFO" "Verifying backup sources..."
    local valid_sources=()

    for source in "${BACKUP_SOURCES[@]}"; do
        if [[ -d "$source" ]]; then
            log "INFO" "  ✅ Found: $source"
            valid_sources+=("$source")
        else
            log "WARNING" "  ⚠️  Not found (skipping): $source"
        fi
    done

    if (( ${#valid_sources[@]} == 0 )); then
        log "ERROR" "No valid backup sources found! Exiting."
        exit 1
    fi

    # Valid sources দিয়ে array update করো
    BACKUP_SOURCES=("${valid_sources[@]}")
}

# ---------- Create the backup ----------
create_backup() {
    log "INFO" "Starting backup creation..."
    log "INFO" "Backup file: $BACKUP_PATH"

    # Backup destination তৈরি করো
    mkdir -p "$BACKUP_DEST"

    # tar দিয়ে compress করে backup নাও
    # --ignore-failed-read: কিছু file পড়া না গেলেও চালিয়ে যাও
    if tar -czf "$BACKUP_PATH" \
        --ignore-failed-read \
        "${BACKUP_SOURCES[@]}" 2>/dev/null; then

        local backup_size
        backup_size=$(du -sh "$BACKUP_PATH" | awk '{print $1}')
        log "INFO" "✅ Backup created successfully!"
        log "INFO" "   File: $BACKUP_PATH"
        log "INFO" "   Size: $backup_size"
    else
        log "ERROR" "❌ Backup creation failed!"
        exit 1
    fi
}

# ---------- Verify backup integrity ----------
verify_backup() {
    log "INFO" "Verifying backup integrity..."

    # tar -tzf দিয়ে backup test করো - extract না করেই
    if tar -tzf "$BACKUP_PATH" > /dev/null 2>&1; then
        local file_count
        file_count=$(tar -tzf "$BACKUP_PATH" 2>/dev/null | wc -l)
        log "INFO" "✅ Backup verified! Contains $file_count files/directories."
    else
        log "ERROR" "❌ Backup verification FAILED! File may be corrupted."
        exit 1
    fi
}

# ---------- Create checksum ----------
create_checksum() {
    log "INFO" "Creating checksum for verification..."
    local checksum_file="${BACKUP_PATH}.sha256"

    sha256sum "$BACKUP_PATH" > "$checksum_file"
    log "INFO" "✅ Checksum saved: $checksum_file"
}

# ---------- Remove old backups ----------
cleanup_old_backups() {
    log "INFO" "Removing backups older than $RETENTION_DAYS days..."

    local removed=0
    while IFS= read -r old_backup; do
        rm -f "$old_backup"
        rm -f "${old_backup}.sha256"  # checksum file ও delete করো
        log "INFO" " Removed: $old_backup"
        (( removed++ ))
    done < <(find "$BACKUP_DEST" -name "*.tar.gz" -mtime +"$RETENTION_DAYS" 2>/dev/null)

    if (( removed == 0 )); then
        log "INFO" "No old backups to remove."
    else
        log "INFO" "Removed $removed old backup(s)."
    fi
}

# ---------- Show backup summary ----------
show_summary() {
    echo ""
    echo "============================================"
    echo "        BACKUP SUMMARY"
    echo "============================================"
    echo "  Backup File : $BACKUP_NAME"
    echo "  Destination : $BACKUP_DEST"
    echo "  Backup Date : $(date)"
    echo ""
    echo "  All backups in $BACKUP_DEST:"

    # সব existing backups list করো
    find "$BACKUP_DEST" -name "*.tar.gz" -printf "    %f (%s bytes)\n" 2>/dev/null | sort

    echo ""
    echo "  Total backup storage:"
    du -sh "$BACKUP_DEST" 2>/dev/null | awk '{print "    " $1}'
    echo "============================================"
}

# ---------- Send notification (optional) ----------
send_notification() {
    # এটা optional - email বা slack notification পাঠাতে পারো
    # Example: mail -s "Backup Complete" admin@example.com <<< "Backup done: $BACKUP_NAME"
    log "INFO" "Notification: Backup completed successfully - $BACKUP_NAME"
}

# ---------- Trap for cleanup on error ----------
cleanup_on_error() {
    log "ERROR" "Script failed! Cleaning up incomplete backup..."
    rm -f "$BACKUP_PATH"
    exit 1
}

trap cleanup_on_error ERR

# ---------- Main ----------
main() {
    log "INFO" "======================================="
    log "INFO" "   AUTO-BACKUP SCRIPT STARTED"
    log "INFO" "   Hostname: $HOSTNAME"
    log "INFO" "======================================="

    check_disk_space
    verify_sources
    create_backup
    verify_backup
    create_checksum
    cleanup_old_backups
    show_summary
    send_notification

    log "INFO" "======================================="
    log "INFO" "   BACKUP COMPLETED SUCCESSFULLY"
    log "INFO" "======================================="

    echo -e "${GREEN}✅ Backup completed! See log: $LOG_FILE${NC}"
}

main
```

### Script এর Key Concepts ব্যাখ্যা

| Technique | কী করছে |
|-----------|---------|
| `tar -czf` | Compress করে archive তৈরি |
| `tar -tzf` | Extract না করেই integrity check |
| `sha256sum` | Backup এর checksum তৈরি - tamper detection |
| `trap cleanup_on_error ERR` | Error হলে incomplete backup delete করো |
| `find -printf "%f"` | Filename এবং size formatted output |


## Cron দিয়ে Scripts Schedule করা

এখন এই তিনটা script automatically চালানোর জন্য cron setup করবো:

```bash
# crontab edit করুন
sudo crontab -e
```

```cron
# ============================================
# DevOps Automation Cron Jobs
# ============================================

# Log Rotation - প্রতিদিন রাত 2টায়
0 2 * * * /opt/scripts/log_rotate.sh >> /var/log/cron_log_rotate.log 2>&1

# Health Check - প্রতি 15 মিনিটে
*/15 * * * * /opt/scripts/health_check.sh >> /var/log/cron_health.log 2>&1

# Auto Backup - প্রতিদিন রাত 3টায়
0 3 * * * /opt/scripts/auto_backup.sh >> /var/log/cron_backup.log 2>&1

# Weekly full report - প্রতি রবিবার সকাল 9টায়
0 9 * * 0 /opt/scripts/health_check.sh >> /var/log/weekly_report.log 2>&1
```


## Scripts কে Production-Ready করার Checklist

Real DevOps engineer রা script deploy করার আগে এই checklist follow করে:

```bash
# ✅ 1. Script executable করো
chmod +x /opt/scripts/health_check.sh

# ✅ 2. Syntax check করো
bash -n health_check.sh

# ✅ 3. Dry run করো
bash -x health_check.sh 2>&1 | head -50

# ✅ 4. ShellCheck দিয়ে lint করো (best practice checker)
shellcheck health_check.sh

# ✅ 5. Log file permissions ঠিক আছে কিনা দেখো
ls -la /var/log/*.log

# ✅ 6. Cron job সঠিকভাবে registered হয়েছে কিনা দেখো
crontab -l
```

## 📝 Quick Summary

- **Log Rotation Script** - disk ভরে যাওয়া থেকে server কে বাঁচায়; logs compress করে archive করে এবং পুরনো গুলো delete করে
- **Health Check Script** - CPU, Memory, Disk, Services monitor করে; threshold exceed হলে alert দেয়
- **Auto-Backup Script** - important directories tar.gz করে backup নেয়, integrity verify করে, checksum তৈরি করে, পুরনো backups cleanup করে
- সব scripts এ **error handling** (`set -euo pipefail`, `trap`), **logging**, **color-coded output** আছে
- **Cron** দিয়ে সব scripts automatically schedule করা হয়েছে

## 🏋️ Practice Tasks

**Task 1:** `health_check.sh` নামিয়ে নাও এবং `SERVICES` array তে তোমার system এ থাকা একটা service (যেমন `ssh`) যোগ করে run করো।

**Task 2:** `auto_backup.sh` এ `BACKUP_SOURCES` array modify করো - শুধু `/etc` backup নাও এবং script টা manually run করে দেখো backup file তৈরি হয় কিনা।

**Task 3:** তিনটা script কে `/opt/scripts/` এ রাখো, সবগুলোকে executable করো, এবং crontab এ add করো। তারপর `crontab -l` দিয়ে verify করো।

## Chapter 5 Complete!

তুমি Chapter 5: Shell Scripting & Automation সম্পূর্ণ শেষ করেছেন! 🏆

এই chapter এ আমরা যা শিখেছি:
- Shell scripting basics থেকে শুরু করে
- Variables, conditions, loops, functions
- Arrays ও string manipulation
- grep, awk, sed দিয়ে text processing
- Error handling ও debugging
- Cron দিয়ে scheduling
- Real-world production scripts

---

## ⏭️ What's Next

### **Chapter 5 - Assessment 5

<table width="100%">
  <tr>
    <td align="left">
      <a href="../10-Scheduling-Scripts">← Scheduling Scripts</a>
    </td>
    <td align="right">
      <a href="../12-Assessment">Chapter 5 - Assessment →</a>
    </td>
  </tr>
</table>