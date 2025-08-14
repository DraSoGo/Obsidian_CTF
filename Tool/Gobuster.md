#web
# What
- หา path website ด้วยการ Bruteforce
# Command
dir : หา path ตามหลังปกติ
```bash
gobuster dir --help gobuster dir -u "http://www.example.thm/" -w /usr/share/wordlists/dirbuster/directory-list-2.3-small.txt
```
dns : หา subdomain
```bash
gobuster dns --help gobuster dns -d http://www.example.thm/ -w /usr/share/wordlists/dirbuster/directory-list-2.3-small.txt
```
vhost : หาเว็บไซต์ที่แตกต่างกันบนเครื่องเดียวกัน
```bash
gobuster vhost -u "http://10.10.117.32" --domain offensivetools.thm -w /usr/share/wordlists/dirbuster/directory-list-2.3-small.txt --append-domain --exclude-length 200-350
```