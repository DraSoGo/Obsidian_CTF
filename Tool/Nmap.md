#network #web 
# What
Nmap (Network Mapper) เป็น network scanning tool สำหรับค้นหา hosts, ports, services, และช่องโหว่ในระบบ [[Network]] ใช้ในการ [[Web Reconnaissance]] และวางแผนการโจมตี

# Key Features
- Port scanning: ตรวจสอบ open/closed/filtered ports
- Service detection: ระบุ service version ที่รันอยู่
- OS detection: คาดเดา operating system
- Script scanning (NSE): ตรวจหาช่องโหว่และเก็บข้อมูลเพิ่มเติม
- Network mapping: วาด network topology

# CTF Use Cases

## Scenario 1: Initial Reconnaissance
เริ่มต้นการทำ CTF ต้องสแกนหา open ports และ services ที่รันอยู่เพื่อหาจุดเข้าโจมตี
```bash
# Quick scan - Top 100 ports เพื่อดูภาพรวมเร็วๆ
nmap -F -T4 10.10.10.50

# Full scan - All 65535 ports เพื่อหาทุก services
nmap -p- -T4 10.10.10.50 -v

# Service detection - หา version เพื่อค้นหา exploits
nmap -sV -sC -p 22,80,443,3306 10.10.10.50 -oN scan.txt

# Aggressive scan - All techniques รวมกัน
nmap -A -T4 10.10.10.50 -oA full_scan
```

## Scenario 2: Web Application Discovery
เจอ web server บน port 80/443 ต้องการสแกนหา hidden services, subdirectories, และช่องโหว่
```bash
# HTTP enumeration ด้วย NSE scripts
nmap -p 80,443,8080,8443 --script http-enum,http-headers,http-methods 10.10.10.50

# ตรวจหา common web vulnerabilities
nmap -p 80 --script http-sql-injection,http-stored-xss 10.10.10.50

# ตรวจสอบ SSL/TLS configuration
nmap -p 443 --script ssl-cert,ssl-enum-ciphers 10.10.10.50

# หา HTTP title และ server information
nmap -p 80,8080 --script http-title,http-server-header 10.10.10.50 -v
```

## Scenario 3: Vulnerability Scanning & Exploit Planning
หลังจากเจอ services แล้ว ต้องสแกนหาช่องโหว่เพื่อหา exploit ที่เหมาะสม
```bash
# Vulnerability scan - ตรวจหาช่องโหว่ที่รู้จัก
nmap -sV --script vuln -v 10.10.10.50

# SMB enumeration - หา SMB shares และ vulnerabilities
nmap -p 445 --script smb-enum-shares,smb-vuln-ms17-010 10.10.10.50

# SSH enumeration - หา SSH algorithms และ version
nmap -p 22 --script ssh-auth-methods,ssh2-enum-algos 10.10.10.50

# MySQL enumeration - หา MySQL users และ databases
nmap -p 3306 --script mysql-enum,mysql-info 10.10.10.50
```

# Command Examples

## Basic Scanning

ดูว่าจะสแกนอะไรบ้าง / subnet - ipa - ipb (list scan)
```bash
nmap -sL 192.168.0.1/24
```

สแกนด้วย ping (host discovery)
```bash
nmap -sn 192.168.0.1/24
```

สแกนด้วย TCP Connect (ไม่ต้องการ root)
```bash
nmap -sT 192.168.0.1
```

สแกนด้วย TCP SYN (เร็วกว่าและแอบมากกว่า - ต้องการ root)
```bash
sudo nmap -sS 192.168.0.1
```

สแกนด้วย UDP (สำหรับ DNS, SNMP, DHCP)
```bash
sudo nmap -sU 192.168.0.1
```

สแกนเฉพาะ 100 port ยอดนิยม (เร็ว)
```bash
nmap -F 192.168.0.1
```

สแกน port range 10-101
```bash
nmap -p10-101 192.168.0.1
```

สแกน host ที่ไม่ตอบ ping (bypass firewall)
```bash
nmap -Pn 192.168.0.1
```

สแกนทุก 65535 ports (ช้าแต่ครบถ้วน)
```bash
nmap -p- 192.168.0.1 -T4 -v
```

สแกน specific ports (web services)
```bash
nmap -p 80,443,8080,8443 192.168.0.1
```

## Service Detection

detect OS (ต้องการ root privileges)
```bash
sudo nmap -O 192.168.0.1
```

detect service version
```bash
nmap -sV 192.168.0.1
```

detect ทุกอย่าง (OS + Version + Scripts + Traceroute)
```bash
nmap -A 192.168.0.1
```

detect version แบบ aggressive (ช้าแต่แม่นยำ)
```bash
nmap -sV --version-intensity 9 192.168.0.1
```

detect พร้อม default NSE scripts
```bash
nmap -sC -sV 192.168.0.1
```

## NSE Script Scanning

ตรวจหาช่องโหว่ทั่วไป (vulnerability scan)
```bash
nmap -sV --script vuln -v 10.10.136.62
```

HTTP enumeration scripts
```bash
nmap -p 80 --script http-enum,http-headers,http-methods 192.168.0.1
```

SMB enumeration และ vulnerability check
```bash
nmap -p 445 --script smb-os-discovery,smb-vuln-* 192.168.0.1
```

SSH brute force (ระวัง rate limiting)
```bash
nmap -p 22 --script ssh-brute --script-args userdb=users.txt,passdb=pass.txt 192.168.0.1
```

MySQL enumeration
```bash
nmap -p 3306 --script mysql-enum,mysql-databases,mysql-users 192.168.0.1
```

DNS enumeration
```bash
nmap -p 53 --script dns-brute,dns-zone-transfer target.com
```

## CTF-Specific Strategies

Quick initial scan (2-3 minutes)
```bash
# สแกนเร็วเพื่อดูภาพรวม
nmap -sV -sC -T4 -F 10.10.10.50 -v -oN quick_scan.txt
```

Full comprehensive scan (10-20 minutes)
```bash
# สแกนครบทุก ports และ services
sudo nmap -sS -sV -sC -p- -T4 -A 10.10.10.50 -v -oA full_scan
```

Stealth scan (หลบ IDS/IPS)
```bash
# ใช้ SYN scan + random data length + decoy IPs
sudo nmap -sS -T2 -f -D RND:10 --data-length 32 10.10.10.50
```

UDP services scan (สำหรับ DNS, SNMP, TFTP)
```bash
# UDP ช้ามาก ต้องใช้เวลานาน
sudo nmap -sU -sV --top-ports 100 10.10.10.50 -v
```

Web-focused scan
```bash
# เน้นที่ web ports และ HTTP scripts
nmap -p 80,443,8000,8080,8443 --script http-* 10.10.10.50 -oN web_scan.txt
```

## Performance Tuning

### Timing Templates
```bash
-T0  # Paranoid - ช้ามาก, หลบ IDS
-T1  # Sneaky - ช้า, หลบ IDS
-T2  # Polite - ช้า, ไม่กิน bandwidth
-T3  # Normal (default)
-T4  # Aggressive - เร็ว, แนะนำสำหรับ CTF
-T5  # Insane - เร็วมาก, อาจพลาด ports
```

### Parallelism & Rate Control
```bash
# เพิ่มจำนวน parallel probes
--min-parallelism 100 --max-parallelism 300

# กำหนดอัตราการส่ง packets ต่อวินาที
--min-rate 500 --max-rate 1000

# กำหนด timeout สูงสุดต่อ host
--host-timeout 5m
```

### CTF Optimization
```bash
# เร็วสุด - เสี่ยงพลาด ports
nmap -T5 --min-rate 1000 -p- 10.10.10.50

# เร็วและแม่นยำ - แนะนำสำหรับ CTF
nmap -T4 --min-rate 500 -p- 10.10.10.50 -v
```

## Output & Reporting

Normal output (อ่านง่าย)
```bash
nmap -oN scan_output.txt 192.168.0.1
```

XML output (สำหรับ import ใน tools อื่น)
```bash
nmap -oX scan_output.xml 192.168.0.1
```

Grepable output (สำหรับ grep/parse)
```bash
nmap -oG scan_output.gnmap 192.168.0.1
```

All formats at once
```bash
nmap -oA scan_output 192.168.0.1
# สร้างไฟล์: scan_output.nmap, scan_output.xml, scan_output.gnmap
```

Verbose output (แสดง progress realtime)
```bash
nmap -v -p- 192.168.0.1
nmap -vv -p- 192.168.0.1  # Very verbose
```

Debug output (troubleshooting)
```bash
nmap -d 192.168.0.1   # Debug level 1
nmap -d9 192.168.0.1  # Debug level 9 (most verbose)
```

## Advanced Techniques

### Firewall/IDS Evasion
```bash
# Fragment packets เพื่อหลบ firewall
nmap -f 192.168.0.1

# ใช้ decoy IPs (ปลอม source IP)
nmap -D RND:10 192.168.0.1
nmap -D decoy1,decoy2,ME,decoy3 192.168.0.1

# เปลี่ยน source port
nmap --source-port 53 192.168.0.1

# สุ่มลำดับ host scanning
nmap --randomize-hosts 192.168.0.0/24

# เพิ่ม random data เพื่อหลบ signature detection
nmap --data-length 32 192.168.0.1
```

### Custom Script Execution
```bash
# รัน specific NSE script
nmap --script http-title 192.168.0.1

# รัน multiple scripts
nmap --script "http-*,ssh-*" 192.168.0.1

# รัน script categories
nmap --script vuln,exploit 192.168.0.1

# ส่ง arguments ไปยัง scripts
nmap --script http-brute --script-args http-brute.path=/admin 192.168.0.1
```

### Network Mapping
```bash
# Traceroute + OS detection
nmap -A --traceroute 192.168.0.1

# ค้นหา gateway และ routes
nmap -sn --traceroute 192.168.0.0/24
```

# Quick Reference - Common Flags

## Scan Types
- `-sT` : TCP Connect scan
- `-sS` : TCP SYN scan (stealth, requires root)
- `-sU` : UDP scan
- `-sn` : Ping scan (no port scan)
- `-sL` : List scan
- `-Pn` : Skip host discovery (treat all as online)

## Port Specification
- `-p-` : All 65535 ports
- `-p 22,80,443` : Specific ports
- `-p 1-1000` : Port range
- `-F` : Fast scan (top 100 ports)
- `--top-ports 1000` : Top N most common ports

## Service/OS Detection
- `-sV` : Version detection
- `-O` : OS detection
- `-A` : Aggressive (OS + Version + Scripts + Traceroute)
- `-sC` : Default NSE scripts
- `--version-intensity <0-9>` : Detection intensity

## Timing & Performance
- `-T<0-5>` : Timing template (0=paranoid, 5=insane)
- `--min-rate <n>` : Minimum packets per second
- `--max-rate <n>` : Maximum packets per second
- `--host-timeout <time>` : Maximum time per host

## Output
- `-v` : Verbose
- `-oN <file>` : Normal output
- `-oX <file>` : XML output
- `-oG <file>` : Grepable output
- `-oA <basename>` : All formats

## NSE Scripts
- `--script <script>` : Run specific script
- `--script vuln` : Run vulnerability scripts
- `--script-args <args>` : Script arguments

# NSE Script Categories

## Discovery
- `broadcast` : Network discovery via broadcast
- `discovery` : Host and service discovery

## Vulnerability Detection
- `vuln` : ตรวจหาช่องโหว่ที่รู้จัก
- `exploit` : พยายาม exploit ช่องโหว่

## Brute Force
- `brute` : Password brute forcing
- `auth` : Authentication testing

## Intrusive
- `intrusive` : Scripts ที่อาจทำให้ระบบผิดปกติ
- `dos` : Denial of Service testing

## Common NSE Scripts for CTF
```bash
# Web enumeration
http-enum, http-headers, http-methods, http-robots.txt, http-title

# Vulnerability scanning
vuln, vulscan, vulners

# SMB enumeration
smb-os-discovery, smb-enum-shares, smb-enum-users, smb-vuln-*

# SSH enumeration
ssh-auth-methods, ssh-brute, ssh-hostkey, ssh2-enum-algos

# Database enumeration
mysql-enum, mysql-databases, mysql-users, mysql-vuln-*
```

# NSE Script Documentation
https://nmap.org/nsedoc/categories/default.html

# Troubleshooting

## Problem: Nmap ช้าเกินไป
**Solutions:**
1. เพิ่ม `-T4` หรือ `-T5` สำหรับ faster scanning
2. ใช้ `-F` สแกนเฉพาะ top 100 ports ก่อน
3. เพิ่ม `--min-rate 500` เพื่อ maintain scan speed
4. สแกนเฉพาะ ports ที่สนใจแทน `-p-`

## Problem: Missing ports / inaccurate results
**Solutions:**
1. ใช้ `-Pn` ถ้า host ไม่ตอบ ping
2. ลด timing `-T2` หรือ `-T3` เพื่อความแม่นยำ
3. เพิ่ม `--max-retries 3` เพื่อ retry packets
4. ใช้ TCP SYN `-sS` แทน TCP Connect `-sT`

## Problem: Firewall blocking scan
**Solutions:**
1. ใช้ `-f` เพื่อ fragment packets
2. ใช้ `-D RND:10` เพื่อใช้ decoy IPs
3. เปลี่ยน `--source-port 53` (DNS port)
4. ลด timing `-T2` เพื่อหลบ IDS/IPS

## Problem: Permission denied (requires root)
**Solutions:**
1. ใช้ `sudo` สำหรับ SYN scan `-sS`, OS detection `-O`
2. หรือใช้ TCP Connect `-sT` ซึ่งไม่ต้องการ root
3. ติดตั้ง nmap capabilities: `sudo setcap cap_net_raw,cap_net_admin,cap_net_bind_service+eip /usr/bin/nmap`

## Problem: UDP scan ช้ามาก
**Solutions:**
1. UDP เป็น connectionless protocol ทำให้ช้าเสมอ
2. สแกนเฉพาะ top ports: `--top-ports 100`
3. เพิ่ม rate: `--min-rate 500`
4. ใช้ `-T4` หรือ `-T5`

# Related Topics
- [[Network]] - การทำความเข้าใจ network fundamentals
- [[Web Reconnaissance]] - การรวบรวมข้อมูล web applications
- [[Port Scanning]] - เทคนิคการสแกน ports
- [[Service Enumeration]] - การ enumerate services เพิ่มเติม
- [[🌐 Web application]] - การทดสอบ web services ที่เจอ
- [[Vulnerability Scanning]] - การตรวจหาช่องโหว่
- [[Network Mapping]] - การวาด network topology