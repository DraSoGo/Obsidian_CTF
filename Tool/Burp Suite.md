#web 
# What
Burp Suite เป็น integrated platform สำหรับทดสอบความปลอดภัยของ [[🌐 Web application]] ใช้ในการดักจับและวิเคราะห์ HTTP/HTTPS traffic, ปรับแต่ง requests, ทดสอบช่องโหว่ [[SQL Injection]], [[XSS]], และ [[Authentication]] bypass

# Key Features
- **Proxy**: ดักจับและแก้ไข HTTP/HTTPS traffic แบบ real-time
- **Repeater**: ส่ง request ซ้ำและแก้ไขเพื่อทดสอบ
- **Intruder**: ยิง payloads แบบอัตโนมัติ (fuzzing, brute force)
- **Scanner**: สแกนหาช่องโหว่อัตโนมัติ (Pro version)
- **Decoder**: encode/decode ข้อมูลหลายรูปแบบ
- **Comparer**: เปรียบเทียบ responses

# CTF Use Cases

## Scenario 1: Authentication Bypass Testing
เจอหน้า login ที่ต้องการทดสอบ [[SQL Injection]] และ brute force credentials
```
1. เปิด Burp Proxy และตั้งค่า browser ให้ผ่าน 127.0.0.1:8080
2. ลองล็อกอินด้วย test credentials
3. ดัก POST request ใน Proxy → HTTP history
4. คลิกขวาที่ request → Send to Repeater (เพื่อทดสอบ SQLi)
5. คลิกขวาที่ request → Send to Intruder (เพื่อ brute force)
```

**Repeater Example - Testing SQL Injection:**
```http
POST /login HTTP/1.1
Host: target.com
Content-Type: application/x-www-form-urlencoded

username=admin' OR '1'='1&password=anything
```

**Intruder Example - Brute Force:**
```
1. Intruder → Positions → Clear $ → เลือก password value → Add $
2. Intruder → Payloads → Load wordlist (rockyou.txt)
3. Start attack → ดู response length/status code เพื่อหา valid password
```

## Scenario 2: API Parameter Fuzzing
เจอ API endpoint ที่ต้องการทดสอบ parameter injection และหา hidden parameters
```
1. ดัก GET/POST request ไปยัง API endpoint
2. Send to Intruder
3. เลือก Attack type: Cluster bomb (ทดสอบหลาย parameters พร้อมกัน)
4. เพิ่ม position markers ที่ parameters และ values
5. Load payload lists: common parameters, SQLi payloads, XSS payloads
6. วิเคราะห์ responses หา status code 200, error messages, หรือ unexpected behavior
```

**Example Request:**
```http
GET /api/user?id=§1§&role=§admin§&debug=§true§ HTTP/1.1
Host: api.target.com
```

## Scenario 3: Session Token Analysis & JWT Manipulation
วิเคราะห์ session management และแก้ไข JWT tokens เพื่อ privilege escalation
```
1. Login แล้วดัก request ใน Proxy
2. ส่ง request ที่มี JWT ไปยัง Decoder
3. Decode JWT (Base64) → เห็น payload: {"user":"normal","role":"user"}
4. แก้ไข role เป็น "admin" → Encode กลับ
5. ส่งไปยัง Repeater → เปลี่ยน token ใน Authorization header
6. ส่ง request → ถ้า server ไม่ verify signature = privilege escalation success
```

# Core Modules

## 1. Proxy
ดักจับและแก้ไข HTTP/HTTPS traffic แบบ real-time

**Setup:**
```
1. Proxy → Options → Proxy Listeners → 127.0.0.1:8080 (running)
2. Browser → Proxy Settings → Manual: 127.0.0.1:8080
3. Proxy → Intercept → Intercept is on
4. Browse website → requests จะถูกดักไว้
5. Forward (ส่งต่อ) / Drop (ยกเลิก) / Send to Repeater/Intruder
```

**ดัก HTTPS traffic:**
```
1. Browse to http://burp → Download CA Certificate
2. Install certificate ใน browser (Firefox: Settings → Certificates → Import)
3. Restart browser → HTTPS traffic จะถูกดักได้
```

**Useful Filters:**
```
- Filter by scope: เฉพาะ target domain
- Filter by file extension: ซ่อน .js, .css, .png
- Filter by status code: แสดงเฉพาะ 200, 302, 404, 500
```

## 2. Repeater
ส่ง request ซ้ำและแก้ไข parameters/headers เพื่อทดสอบแบบ manual

**Workflow:**
```
1. ดัก request จาก Proxy → Send to Repeater
2. แก้ไข request: parameters, headers, body
3. คลิก Send → ดู Response
4. ลองแก้ไข payload: SQLi, XSS, IDOR
5. เปรียบเทียบ responses (คลิกขวา → Request in browser)
```

**Example - Testing IDOR:**
```http
# Original request
GET /api/user/profile?id=123 HTTP/1.1

# Modified request - ทดสอบ IDOR
GET /api/user/profile?id=1 HTTP/1.1
GET /api/user/profile?id=999 HTTP/1.1
```

**Example - Testing SQL Injection:**
```http
# ทดสอบ payloads ต่างๆ
POST /search HTTP/1.1
Content-Type: application/x-www-form-urlencoded

query=test' OR '1'='1
query=test' UNION SELECT NULL--
query=test'; DROP TABLE users--
```

## 3. Intruder
ยิง payloads อัตโนมัติเพื่อ fuzzing และ brute force

**Attack Types:**
- **Sniper**: ทดสอบทีละ position (ช้าแต่แม่นยำ)
- **Battering ram**: ใช้ payload เดียวกันทุก positions
- **Pitchfork**: ใช้ payload ที่ match กันตาม index
- **Cluster bomb**: ทดสอบทุกคู่ combinations (เยอะที่สุด)

**Workflow - Brute Force Login:**
```
1. ดัก login POST request → Send to Intruder
2. Positions → Clear $ → เลือก username และ password → Add $
3. Payloads → Set 1 (username): Load list: usernames.txt
4. Payloads → Set 2 (password): Load list: passwords.txt
5. Options → Grep - Match → Add: "Invalid credentials", "Success"
6. Start attack → Sort by Length/Status → หา valid credentials
```

**Workflow - Parameter Fuzzing:**
```
1. ดัก API request → Send to Intruder
2. เลือก parameter value → Add $
3. Payloads → Load: SQLi payloads, XSS payloads, Path traversal
4. Options → Redirections: Always follow
5. Start attack → ดู responses หา errors, different lengths, 500 status
```

**Example - Finding Admin Panel:**
```http
GET /§admin§ HTTP/1.1
Host: target.com

# Payloads: admin, administrator, wp-admin, phpmyadmin, dashboard, panel, etc.
# ดู status code 200 หรือ 302 (redirect) แทน 404
```

**Rate Limiting:**
```
Options → Resource Pool → Maximum concurrent requests: 1
Options → Request Engine → Throttle: Add delay (ms)
```

## 4. Scanner (Pro Only)
สแกนหาช่องโหว่อัตโนมัติ

**Active Scan:**
```
1. Proxy → HTTP history → เลือก request → คลิกขวา → Do active scan
2. Scanner → Scan queue → ดู progress
3. Scanner → Results → ดู vulnerabilities ที่เจอ
```

**Passive Scan:**
```
- ทำงานอัตโนมัติเมื่อ traffic ผ่าน Proxy
- ไม่ส่ง additional requests
- ตรวจหา: missing headers, cookies without secure flag, information disclosure
```

## 5. Decoder
Encode/Decode ข้อมูลหลายรูปแบบ

**Supported Formats:**
```
- URL encoding
- HTML encoding
- Base64
- Hex
- ASCII hex
- Gzip
- JWT (ต้องใช้ extension: JWT Editor)
```

**Workflow - Decode JWT:**
```
1. Copy JWT token จาก request
2. Decoder → Paste token
3. Decode as → Base64 → เห็น header และ payload
4. แก้ไข payload → Encode กลับ → Copy ไปใช้ใน Repeater
```

**Workflow - URL Decode:**
```
1. Paste URL-encoded string: %27%20OR%20%271%27%3D%271
2. Decode as → URL → เห็น: ' OR '1'='1
```

## 6. Comparer
เปรียบเทียบ requests หรือ responses หาความแตกต่าง

**Workflow:**
```
1. Repeater → ส่ง request 2 ครั้งด้วย payloads ต่างกัน
2. คลิกขวาที่ response → Send to Comparer (response)
3. ทำซ้ำกับ response อื่น
4. Comparer → Words หรือ Bytes → Compare → ดู highlighted differences
```

**Use Case - Blind SQL Injection:**
```
- ส่ง request กับ payload: ' AND 1=1-- และ ' AND 1=2--
- Compare responses → ถ้า different = SQL injection exists
```

# Command Examples (Shortcuts)

## Proxy Shortcuts
```
Ctrl+R - Send to Repeater
Ctrl+I - Send to Intruder
Ctrl+Shift+B - Send to Scanner
Forward - ส่ง request ต่อ
Drop - ยกเลิก request
Intercept is on/off - เปิด/ปิดการดัก traffic
```

## Repeater Shortcuts
```
Ctrl+Space - Send request
Ctrl+R - Rename tab
Ctrl+Shift+R - Show response in browser
```

## Intruder Shortcuts
```
Ctrl+I - Insert position marker
Ctrl+Shift+P - Clear all position markers
```

## General Shortcuts
```
Ctrl+F - Find in current view
Ctrl+S - Save item
Ctrl+T - New tab
```

# Quick Reference - Common Workflows

## 1. Basic Proxy Setup
```
1. Proxy → Options → 127.0.0.1:8080 running ✓
2. Browser proxy: 127.0.0.1:8080
3. Install Burp CA cert สำหรับ HTTPS
4. Intercept is on
```

## 2. SQLi Testing Workflow
```
1. ดัก request → Send to Repeater
2. แก้ไข parameter: ' OR '1'='1
3. ส่ง request → ดู response
4. ถ้าเจอ → Send to SQLMap: Save request → sqlmap -r request.txt
```

## 3. Brute Force Workflow
```
1. ดัก login request → Send to Intruder
2. Add $ ที่ username และ password
3. Load wordlists
4. Start attack → Sort by Length
5. หา response ที่ different = valid credentials
```

## 4. JWT Token Manipulation
```
1. ดัก request ที่มี JWT → Copy token
2. Decoder → Decode Base64 → แก้ไข payload
3. Encode กลับ → Copy
4. Repeater → Paste token ใหม่ → Send
```

## 5. Parameter Fuzzing
```
1. ดัก API request → Send to Intruder
2. Add $ ที่ parameter value
3. Load payload list: SQLi, XSS, Path traversal
4. Start attack → หา 500 errors, different responses
```

# Related Topics
- [[SQL Injection]] - ทดสอบ SQL injection ด้วย Repeater และ Intruder
- [[XSS]] - ทดสอบ Cross-Site Scripting payloads
- [[Authentication]] - Bypass authentication และ brute force
- [[🌐 Web application]] - ทดสอบความปลอดภัย web apps
- [[Web Reconnaissance]] - รวบรวมข้อมูลจาก HTTP traffic
- [[API Security]] - ทดสอบ API endpoints
- [[Session Management]] - วิเคราะห์ cookies และ tokens