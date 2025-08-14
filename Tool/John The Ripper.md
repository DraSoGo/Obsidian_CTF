#cryptography 
# What
- ไว้แก้ hash โหดหว่า hashcat
# Command
## Normal
```bash
john --format=raw-md5 --wordlist=/usr/share/wordlists/rockyou.txt hash_to_crack.txt
```
## Shadow
หารหัสจาก shadow และ passwd บน Unix และ Linux
```bash
unshadow local_passwd local_shadow > hash.txt john --wordlist=/usr/share/wordlists/rockyou.txt --format=sha512crypt hash.txt
```
## Single Crack Mode
เดารหัสเอาเองแตต่ต้องใส่ข้อความเริ่มต้นให้
1efee03cdcb96d90ad48ccc7b8666033 → mike:1efee03cdcb96d90ad48ccc7b8666033
```bash
john --single --format=raw-sha256 hashes.txt
```
## Zip
หารหัสไฟล์ zip
```bash
zip2john secure.zip > hash.txt
john --wordlist=/usr/share/wordlists/rockyou.txt hash.txt
```
## Rar
หารหัสไฟล์ rar
```bash
rar2john secure.rar > hash.txt
john --wordlist=/usr/share/wordlists/rockyou.txt hash.txt
```
## SSH
หารหัส ssh ผ่านไฟล์ rsa
```bash
ssh2john id_rsa > hash.txt
john --wordlist=/usr/share/wordlists/rockyou.txt hash.txt
```
# Rule

| **Command**                                                           | **Mean**                                                                                                  |
| --------------------------------------------------------------------- | --------------------------------------------------------------------------------------------------------- |
| sudo mousepad /etc/john/john.conf                                     | config rule  <br>ตัวอย่าง  <br>`[List.Rules:PoloPassword]`  <br>`cAz"[0-9] [!£$%@]"`  <br>hello → Hello0! |
| john --wordlist=[path to wordlist] --rule=PoloPassword [path to file] | ใช้ rule                                                                                                  |
| Az                                                                    | เติมท้ายคำ                                                                                                |
| A0                                                                    | เติมหน้าคำ                                                                                                |
| c                                                                     | ตัวแรกเป็นตัวพิมพ์ใหญ่                                                                                    |
| [1]                                                                   | เติม 1                                                                                                    |
| [0-9]                                                                 | เติม 0-9                                                                                                  |
| [A-Z]                                                                 | เติม A-Z                                                                                                  |
| [a-z]                                                                 | เติม a-z                                                                                                  |
| [!£$%@]                                                               | เติมอักขระพิเศษ                                                                                           |
