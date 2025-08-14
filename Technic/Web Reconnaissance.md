#web 
# Content Discover
## Dirb
### Command
```bash
dirb http://172.18.0.64/
```
## Buster
### Command
```bash
sudo gobuster dir -w /usr/share/wordlists/dirbuster/directory-list-2.3-medium.txt --url http://172.18.0.33/database/backup/secret/ -x json --random-agent --exclude-length 1591 -b 403
```
[[Gobuster]]
### Mean
- -x กำหนดชนิดไฟล์
- -b ข้าม 403/404
- --random-agent random user ทำให้จับไม่ได้ว่ามาจาก gobuster
- --exclude-length กำหนด length ให้ตรงกับที่ gobuster ต้องการ
## Backup
ลองเติม .bak ดู
# RECON & enumeration
## RECON & enumeration
### Command
ดูว่ามีพอร์ตใดเปิดอยู่
```bash
sudo nmap -sC 172.18.0.70
```
[[Nmap]]