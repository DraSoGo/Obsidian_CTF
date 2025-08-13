#web
- หา path website ด้วยการ Bruteforce
- dir : หา path ตามหลังปกติ gobuster dir --help gobuster dir -u "`http://www.example.thm/`" -w `/usr/share/wordlists/dirbuster/directory-list-2.3-small.txt`
- dns : หา subdomain gobuster dns --help gobuster dns -d `http://www.example.thm/` -w `/usr/share/wordlists/dirbuster/directory-list-2.3-small.txt`
- vhost : หาเว็บไซต์ที่แตกต่างกันบนเครื่องเดียวกัน gobuster vhost --help gobuster vhost -u "`http://10.10.117.32`" --domain `offensivetools.thm` -w `/usr/share/wordlists/dirbuster/directory-list-2.3-small.txt` --append-domain --exclude-length 200-350