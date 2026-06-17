# 🐍 Python Cryptography Cheat Sheet for CTF

## Encoding & Decoding

| **Operation** | **Python Code** |
|---------------|-----------------|
| **Base64 Encode** | `import base64`<br>`base64.b64encode(b"flag").decode()` |
| **Base64 Decode** | `import base64`<br>`base64.b64decode("ZmxhZw==").decode()` |
| **Base32 Encode** | `import base64`<br>`base64.b32encode(b"flag").decode()` |
| **Base32 Decode** | `import base64`<br>`base64.b32decode("MZXW6===").decode()` |
| **Base85 Encode** | `import base64`<br>`base64.b85encode(b"flag").decode()` |
| **Base85 Decode** | `import base64`<br>`base64.b85decode("@WH8W").decode()` |
| **Hex Encode** | `"flag".encode().hex()`<br>`# หรือ`<br>`import binascii`<br>`binascii.hexlify(b"flag").decode()` |
| **Hex Decode** | `bytes.fromhex("666c6167").decode()`<br>`# หรือ`<br>`import binascii`<br>`binascii.unhexlify("666c6167").decode()` |
| **Binary to Text** | `binary = "01100110"`<br>`int(binary, 2).to_bytes((len(binary) + 7) // 8, 'big').decode()` |
| **Text to Binary** | `''.join(format(ord(c), '08b') for c in "flag")` |
| **ROT13** | `import codecs`<br>`codecs.encode("flag", "rot_13")` |
| **URL Encode** | `from urllib.parse import quote`<br>`quote("flag{test}")` |
| **URL Decode** | `from urllib.parse import unquote`<br>`unquote("flag%7Btest%7D")` |

## Hash Functions

| **Operation** | **Python Code** |
|---------------|-----------------|
| **MD5** | `import hashlib`<br>`hashlib.md5(b"flag").hexdigest()` |
| **SHA1** | `import hashlib`<br>`hashlib.sha1(b"flag").hexdigest()` |
| **SHA256** | `import hashlib`<br>`hashlib.sha256(b"flag").hexdigest()` |
| **SHA512** | `import hashlib`<br>`hashlib.sha512(b"flag").hexdigest()` |
| **HMAC-SHA256** | `import hmac, hashlib`<br>`hmac.new(b"key", b"message", hashlib.sha256).hexdigest()` |
| **Hash File** | `import hashlib`<br>`with open("file.txt", "rb") as f:`<br>`    hashlib.sha256(f.read()).hexdigest()` |

## RSA Operations

| **Operation** | **Python Code** |
|---------------|-----------------|
| **Basic RSA Decrypt** | `from Crypto.Util.number import long_to_bytes`<br>`from sympy import factorint`<br>`n = 12345`<br>`e = 65537`<br>`c = 6789`<br>`factors = factorint(n)`<br>`p, q = list(factors.keys())`<br>`phi = (p-1) * (q-1)`<br>`d = pow(e, -1, phi)  # Python 3.8+`<br>`m = pow(c, d, n)`<br>`flag = long_to_bytes(m)` |
| **RSA with PyCryptodome** | `from Crypto.PublicKey import RSA`<br>`from Crypto.Cipher import PKCS1_OAEP`<br>`key = RSA.import_key(open("private.pem").read())`<br>`cipher = PKCS1_OAEP.new(key)`<br>`plaintext = cipher.decrypt(ciphertext)` |
| **Small e Attack (e=3)** | `import gmpy2`<br>`from Crypto.Util.number import long_to_bytes`<br>`c = 12345`<br>`m = gmpy2.iroot(c, 3)[0]`<br>`long_to_bytes(int(m))` |
| **Common Modulus Attack** | `from sympy import gcdex`<br>`# มี n เดียวกัน แต่ e1, e2, c1, c2 ต่างกัน`<br>`s, t, _ = gcdex(e1, e2)`<br>`m = (pow(c1, s, n) * pow(c2, t, n)) % n`<br>`long_to_bytes(m)` |
| **Wiener's Attack** | `import owiener`<br>`# ใช้เมื่อ d เล็กมาก`<br>`d = owiener.attack(e, n)`<br>`if d:`<br>`    m = pow(c, d, n)`<br>`    long_to_bytes(m)` |
| **Factor with factordb** | `from factordb.factordb import FactorDB`<br>`f = FactorDB(n)`<br>`f.connect()`<br>`factors = f.get_factor_list()` |

## AES Operations

| **Operation** | **Python Code** |
|---------------|-----------------|
| **AES-ECB Encrypt** | `from Crypto.Cipher import AES`<br>`from Crypto.Util.Padding import pad`<br>`key = b"sixteen byte key"`<br>`cipher = AES.new(key, AES.MODE_ECB)`<br>`ct = cipher.encrypt(pad(b"plaintext", 16))` |
| **AES-ECB Decrypt** | `from Crypto.Cipher import AES`<br>`from Crypto.Util.Padding import unpad`<br>`cipher = AES.new(key, AES.MODE_ECB)`<br>`pt = unpad(cipher.decrypt(ct), 16)` |
| **AES-CBC Encrypt** | `from Crypto.Cipher import AES`<br>`from Crypto.Util.Padding import pad`<br>`iv = b"16 byte iv here!"`<br>`cipher = AES.new(key, AES.MODE_CBC, iv)`<br>`ct = cipher.encrypt(pad(plaintext, 16))` |
| **AES-CBC Decrypt** | `cipher = AES.new(key, AES.MODE_CBC, iv)`<br>`pt = unpad(cipher.decrypt(ct), 16)` |
| **AES-CTR** | `from Crypto.Cipher import AES`<br>`from Crypto.Util import Counter`<br>`ctr = Counter.new(128)`<br>`cipher = AES.new(key, AES.MODE_CTR, counter=ctr)`<br>`ct = cipher.encrypt(plaintext)` |
| **AES-GCM Encrypt** | `from Crypto.Cipher import AES`<br>`cipher = AES.new(key, AES.MODE_GCM)`<br>`ct, tag = cipher.encrypt_and_digest(plaintext)`<br>`nonce = cipher.nonce` |
| **AES-GCM Decrypt** | `cipher = AES.new(key, AES.MODE_GCM, nonce=nonce)`<br>`pt = cipher.decrypt_and_verify(ct, tag)` |

## XOR Operations

| **Operation** | **Python Code** |
|---------------|-----------------|
| **XOR String with Key** | `def xor(data, key):`<br>`    return bytes([b ^ key[i % len(key)] for i, b in enumerate(data)])`<br>`result = xor(b"flag", b"key")` |
| **XOR with Single Byte** | `data = b"encrypted"`<br>`key = 0x42`<br>`decrypted = bytes([b ^ key for b in data])` |
| **Find XOR Key** | `ct = b"ciphertext"`<br>`pt = b"plaintext_"`<br>`key = bytes([c ^ p for c, p in zip(ct, pt)])` |
| **Brute Force Single-Byte XOR** | `for key in range(256):`<br>`    result = bytes([b ^ key for b in data])`<br>`    if b"flag" in result:`<br>`        print(f"Key: {key}, Result: {result}")` |
| **Repeating-Key XOR** | `from itertools import cycle`<br>`key = b"KEY"`<br>`ct = bytes([p ^ k for p, k in zip(plaintext, cycle(key))])` |

## Classical Ciphers

| **Operation** | **Python Code** |
|---------------|-----------------|
| **Caesar Cipher Decrypt** | `def caesar(text, shift):`<br>`    result = ""`<br>`    for char in text:`<br>`        if char.isalpha():`<br>`            base = ord('A') if char.isupper() else ord('a')`<br>`            result += chr((ord(char) - base - shift) % 26 + base)`<br>`        else:`<br>`            result += char`<br>`    return result` |
| **Vigenere Decrypt** | `def vigenere_decrypt(ct, key):`<br>`    key = (key * (len(ct) // len(key) + 1))[:len(ct)]`<br>`    return ''.join(chr((ord(c) - ord(k)) % 26 + ord('A')) for c, k in zip(ct, key))` |
| **Atbash Cipher** | `import string`<br>`atbash = str.maketrans(`<br>`    string.ascii_lowercase + string.ascii_uppercase,`<br>`    string.ascii_lowercase[::-1] + string.ascii_uppercase[::-1])`<br>`text.translate(atbash)` |

## Utilities & Conversions

| **Operation** | **Python Code** |
|---------------|-----------------|
| **Bytes to Long** | `from Crypto.Util.number import bytes_to_long`<br>`num = bytes_to_long(b"flag")` |
| **Long to Bytes** | `from Crypto.Util.number import long_to_bytes`<br>`text = long_to_bytes(12345)` |
| **GCD (หารร่วมมาก)** | `import math`<br>`gcd = math.gcd(a, b)` |
| **Extended GCD** | `from sympy import gcdex`<br>`s, t, gcd = gcdex(a, b)` |
| **Modular Inverse** | `pow(a, -1, m)  # Python 3.8+`<br>`# หรือ`<br>`from sympy import mod_inverse`<br>`mod_inverse(a, m)` |
| **Chinese Remainder Theorem** | `from sympy.ntheory.modular import crt`<br>`result = crt([m1, m2], [a1, a2])[0]` |
| **Is Prime** | `from sympy import isprime`<br>`isprime(n)` |
| **Factor Integer** | `from sympy import factorint`<br>`factorint(n)` |
| **Random Bytes** | `import os`<br>`os.urandom(16)` |
| **Padding (PKCS7)** | `from Crypto.Util.Padding import pad, unpad`<br>`padded = pad(data, 16)`<br>`unpadded = unpad(padded, 16)` |

## Frequency Analysis

| **Operation** | **Python Code** |
|---------------|-----------------|
| **Character Frequency** | `from collections import Counter`<br>`freq = Counter(ciphertext)`<br>`freq.most_common(10)` |
| **English Score** | `def english_score(text):`<br>`    freq = {'E': 12.70, 'T': 9.06, 'A': 8.17, 'O': 7.51,`<br>`            'I': 6.97, 'N': 6.75, 'S': 6.33, 'H': 6.09}`<br>`    score = 0`<br>`    for char in text.upper():`<br>`        if char in freq:`<br>`            score += freq[char]`<br>`    return score` |
| **Index of Coincidence** | `def ioc(text):`<br>`    N = len(text)`<br>`    freq = Counter(text)`<br>`    return sum(f * (f-1) for f in freq.values()) / (N * (N-1))` |

## Network & Protocol

| **Operation** | **Python Code** |
|---------------|-----------------|
| **Connect to Server** | `import socket`<br>`s = socket.socket(socket.AF_INET, socket.SOCK_STREAM)`<br>`s.connect(("example.com", 1234))`<br>`s.send(b"data")`<br>`response = s.recv(1024)` |
| **pwntools (แนะนำ)** | `from pwn import *`<br>`r = remote("example.com", 1234)`<br>`r.sendline(b"data")`<br>`response = r.recvline()` |
| **TLS/SSL Connection** | `import ssl, socket`<br>`context = ssl.create_default_context()`<br>`with socket.create_connection(("example.com", 443)) as sock:`<br>`    with context.wrap_socket(sock, server_hostname="example.com") as ssock:`<br>`        ssock.send(b"data")` |

## Padding Oracle Attack Template

```python
def oracle(ct):
    # ส่ง ciphertext ไปที่ server
    # return True ถ้า padding ถูกต้อง, False ถ้าไม่
    pass

def padding_oracle_attack(ct, iv, block_size=16):
    plaintext = b""
    for block in range(len(ct) // block_size):
        decrypted_block = b""
        for byte_pos in range(block_size-1, -1, -1):
            padding_value = block_size - byte_pos
            for guess in range(256):
                fake_iv = bytearray(block_size)
                for i in range(byte_pos+1, block_size):
                    fake_iv[i] = decrypted_block[i-byte_pos-1] ^ padding_value
                fake_iv[byte_pos] = guess
                if oracle(bytes(fake_iv) + ct[block*block_size:(block+1)*block_size]):
                    decrypted_byte = guess ^ padding_value
                    decrypted_block = bytes([decrypted_byte]) + decrypted_block
                    break
        plaintext += decrypted_block
    return plaintext
```

## Useful Libraries to Install

```bash
# ติดตั้ง libraries ที่ใช้บ่อยใน CTF
pip install pycryptodome   # AES, RSA, และอื่นๆ
pip install gmpy2          # Fast math operations
pip install sympy          # Symbolic mathematics
pip install pwntools       # CTF framework
pip install owiener        # Wiener's attack
pip install factordb-pycli # FactorDB API
pip install z3-solver      # SMT solver
pip install requests       # HTTP requests
pip install pycrypto       # Legacy crypto library
```

## Quick CTF Workflow

```python
# 1. อ่านไฟล์
with open("encrypted.txt", "rb") as f:
    data = f.read()

# 2. ลอง decode แบบต่างๆ
import base64
try:
    decoded = base64.b64decode(data)
    print(f"Base64: {decoded}")
except:
    pass

# 3. ลอง XOR single byte
for key in range(256):
    result = bytes([b ^ key for b in data])
    if b"flag" in result.lower() or b"ctf" in result.lower():
        print(f"XOR key {key}: {result}")

# 4. ถ้าเป็น RSA ให้ลอง factor n
from factordb.factordb import FactorDB
from Crypto.Util.number import long_to_bytes

# n, e, c = ...
f = FactorDB(n)
f.connect()
factors = f.get_factor_list()
if len(factors) == 2:
    p, q = factors
    phi = (p-1) * (q-1)
    d = pow(e, -1, phi)
    m = pow(c, d, n)
    print(long_to_bytes(m))
```

## Common CTF Patterns

| **Pattern** | **Solution** |
|-------------|--------------|
| `n, e, c` มีให้ | Factor `n` ด้วย factordb หรือ sympy → คำนวณ `d` → decrypt |
| `e = 3` และ `c` เล็ก | Small e attack: `m = iroot(c, e)` |
| Hex string ยาวๆ | `bytes.fromhex(hex_string)` แล้วลอง decode ต่อ |
| Base64 ซ้อนหลายชั้น | Decode วนไปเรื่อยๆ จนได้ plaintext |
| XOR repeated key | Guess key length → frequency analysis → brute force |
| Padding oracle | ใช้ template ข้างบน + ส่ง request ไป server |
| ECB mode (มี pattern ซ้ำ) | Block-by-block analysis หรือ cut-and-paste attack |

## Advanced RSA Attacks

```python
# Hastad's Broadcast Attack (e=3, ส่ง message เดียวกันหลายครั้ง)
from sympy.ntheory.modular import crt
import gmpy2
from Crypto.Util.number import long_to_bytes

# มี c1, c2, c3 และ n1, n2, n3 (e=3)
m_cubed = crt([n1, n2, n3], [c1, c2, c3])[0]
m = gmpy2.iroot(m_cubed, 3)[0]
print(long_to_bytes(int(m)))

# Fermat's Factorization (p และ q ใกล้กัน)
def fermat_factor(n):
    a = gmpy2.isqrt(n) + 1
    b2 = a*a - n
    while not gmpy2.is_square(b2):
        a += 1
        b2 = a*a - n
    b = gmpy2.isqrt(b2)
    return int(a - b), int(a + b)

# RSA with Chinese Remainder Theorem
def rsa_crt_decrypt(c, p, q, e):
    dp = pow(e, -1, p-1)
    dq = pow(e, -1, q-1)
    qinv = pow(q, -1, p)
    m1 = pow(c, dp, p)
    m2 = pow(c, dq, q)
    h = (qinv * (m1 - m2)) % p
    m = m2 + h * q
    return m
```

## Elliptic Curve Cryptography (ECC)

```python
from Crypto.PublicKey import ECC
from Crypto.Hash import SHA256

# Generate key pair
key = ECC.generate(curve='P-256')
private_key = key.d
public_key = key.pointQ

# Sign message
from Crypto.Signature import DSS
signer = DSS.new(key, 'fips-186-3')
hash_obj = SHA256.new(b"message")
signature = signer.sign(hash_obj)

# Verify signature
verifier = DSS.new(key.public_key(), 'fips-186-3')
try:
    verifier.verify(hash_obj, signature)
    print("Valid signature")
except ValueError:
    print("Invalid signature")
```

## Common Mistakes to Exploit

```python
# 1. No padding (textbook RSA)
# → Small e attack, chosen plaintext attack

# 2. Same modulus, different exponents
# → Common modulus attack

# 3. d is small (d < n^0.25)
# → Wiener's attack

# 4. p, q are close
# → Fermat's factorization

# 5. ECB mode
# → Block rearrangement, pattern analysis

# 6. Weak random (predictable seed)
import random
random.seed(known_seed)
predicted = random.randint(0, 100)

# 7. Reused nonce in AES-CTR or stream cipher
# XOR ciphertexts to cancel the keystream
ct1_xor_ct2 = bytes([a ^ b for a, b in zip(ct1, ct2)])
# นี่คือ pt1 XOR pt2
```

## Bit Flipping Attack (CBC/CTR)

```python
# CBC Bit Flipping
def flip_bit(ct, iv, position, original_byte, target_byte):
    """Flip bit in CBC ciphertext to change plaintext"""
    iv = bytearray(iv)
    iv[position] ^= original_byte ^ target_byte
    return bytes(iv)

# CTR Bit Flipping
def flip_ctr(ct, position, original_byte, target_byte):
    """Flip bit in CTR ciphertext"""
    ct = bytearray(ct)
    ct[position] ^= original_byte ^ target_byte
    return bytes(ct)
```

## Length Extension Attack (Hash)

```python
import hashpumpy

# สำหรับ SHA-1, SHA-256 ที่ไม่ใช้ HMAC
original_data = b"original"
original_hash = "abc123..."
append_data = b"&admin=true"
key_length = 16  # ต้องเดา

new_hash, new_message = hashpumpy.hashpump(
    original_hash, 
    original_data, 
    append_data, 
    key_length
)
```

## Tips & Tricks

```python
# 1. อ่านไฟล์ hex dump
with open("file.hex", "r") as f:
    hex_data = f.read().replace(" ", "").replace("\n", "")
    data = bytes.fromhex(hex_data)

# 2. แปลง PEM to numbers
from Crypto.PublicKey import RSA
key = RSA.import_key(open("public.pem").read())
n = key.n
e = key.e

# 3. ดู hex dump
import binascii
print(binascii.hexlify(data))

# 4. ลอง decode ทุกอย่าง
import codecs
encodings = ['utf-8', 'latin-1', 'ascii', 'utf-16']
for enc in encodings:
    try:
        print(f"{enc}: {data.decode(enc)}")
    except:
        pass

# 5. Find patterns in hex
from collections import Counter
blocks = [data[i:i+16] for i in range(0, len(data), 16)]
counter = Counter(blocks)
if counter.most_common(1)[0][1] > 1:
    print("ECB detected! Repeated blocks found")
```
