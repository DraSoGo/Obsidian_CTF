#web #recon #enumeration #osint

# Web Reconnaissance

> [!info] Overview
> Web Reconnaissance คือขั้นตอนแรกของการโจมตีเว็บแอปพลิเคชัน ใช้เพื่อรวบรวมข้อมูลเกี่ยวกับเป้าหมายมากที่สุดเท่าที่จะเป็นไปได้ ครอบคลุมตั้งแต่การค้นหา subdomain, directory enumeration, technology fingerprinting, ไปจนถึงการหา sensitive information

## Table of Contents
- [[#Port Scanning]]
- [[#Subdomain Enumeration]]
- [[#Directory and File Discovery]]
- [[#Technology Fingerprinting]]
- [[#DNS Enumeration]]
- [[#Information Gathering]]

---

## Port Scanning

### Nmap - Network Mapper
ใช้สำหรับ scan พอร์ตและบริการที่เปิดอยู่บนเครื่องเป้าหมาย

```bash
# Basic scan - ดูว่ามีพอร์ตใดเปิดอยู่
sudo nmap -sC 172.18.0.70

# Comprehensive scan - scan ทุกพอร์ตและแสดงเวอร์ชั่นของบริการ
sudo nmap -sV -p- 172.18.0.70

# Aggressive scan - รวม OS detection, version detection, script scanning
sudo nmap -A -T4 172.18.0.70

# Fast scan - scan เฉพาะ 100 พอร์ตที่พบบ่อยที่สุด
sudo nmap -F 172.18.0.70

# Stealth SYN scan - ไม่สร้าง full TCP connection (ยากต่อการตรวจจับ)
sudo nmap -sS 172.18.0.70

# UDP scan - scan พอร์ต UDP
sudo nmap -sU --top-ports 20 172.18.0.70

# Scan multiple targets จาก file
nmap -iL targets.txt -oN scan_results.txt
```

> [!tip] Nmap Output Formats
> - `-oN` - Normal output (readable)
> - `-oX` - XML output (for tools)
> - `-oG` - Grepable output (for parsing)
> - `-oA` - All formats at once

### Common Services to Check
```bash
# HTTP/HTTPS (80, 443, 8080, 8443)
nmap -p 80,443,8080,8443 -sV 172.18.0.70

# SSH (22)
nmap -p 22 --script ssh-auth-methods 172.18.0.70

# FTP (21)
nmap -p 21 --script ftp-anon 172.18.0.70

# SMB (445, 139)
nmap -p 445,139 --script smb-enum-shares 172.18.0.70

# MySQL (3306)
nmap -p 3306 --script mysql-info 172.18.0.70
```

[[Nmap]]

---

## Subdomain Enumeration

> [!warning] Subdomain Discovery
> Subdomain มักมีการรักษาความปลอดภัยน้อยกว่า main domain และอาจเป็นจุดเข้าสู่ระบบที่ดี

### Passive Subdomain Enumeration
ค้นหา subdomain โดยไม่ต้องโต้ตอบกับเป้าหมายโดยตรง

```bash
# Subfinder - Fast passive subdomain enumeration
subfinder -d example.com -o subdomains.txt

# Amass - Comprehensive OSINT subdomain discovery
amass enum -d example.com -o amass_results.txt

# Assetfinder - ค้นหา domains และ subdomains ที่เกี่ยวข้อง
assetfinder --subs-only example.com > assetfinder_results.txt

# Sublist3r - ใช้ search engines และ DNS records
sublist3r -d example.com -o sublist3r_results.txt

# crt.sh - Certificate Transparency logs
curl -s "https://crt.sh/?q=%25.example.com&output=json" | jq -r '.[].name_value' | sort -u

# SecurityTrails API
curl -s "https://api.securitytrails.com/v1/domain/example.com/subdomains" \
  -H "APIKEY: YOUR_API_KEY" | jq -r '.subdomains[]'
```

### Active Subdomain Enumeration
ใช้ brute force เพื่อค้นหา subdomains

```bash
# Gobuster DNS mode - brute force subdomains
gobuster dns -d example.com -w /usr/share/wordlists/SecLists/Discovery/DNS/subdomains-top1million-5000.txt -o gobuster_dns.txt

# FFuF - Fast web fuzzer (DNS mode)
ffuf -w /usr/share/wordlists/SecLists/Discovery/DNS/subdomains-top1million-5000.txt \
  -u http://FUZZ.example.com -o ffuf_subdomains.json

# DNSRecon - DNS enumeration tool
dnsrecon -d example.com -t brt -D /usr/share/wordlists/dnsmap.txt

# Fierce - DNS reconnaissance tool
fierce --domain example.com --subdomains accounts admin api dev staging

# Massdns - High-performance DNS resolver
massdns -r resolvers.txt -t A -o S subdomains.txt
```

### Subdomain Validation
ตรวจสอบว่า subdomain ที่พบมีจริงและใช้งานได้

```bash
# HTTPx - Probe HTTP/HTTPS servers
cat subdomains.txt | httpx -silent -o live_subdomains.txt

# ตรวจสอบ subdomain ที่มี HTTP service
cat subdomains.txt | httpx -status-code -title -tech-detect -o detailed_subdomains.txt

# Screenshot subdomains ด้วย Aquatone
cat live_subdomains.txt | aquatone -out aquatone_screenshots
```

---

## Directory and File Discovery

### Gobuster - Directory Brute Force
```bash
# Basic directory enumeration
sudo gobuster dir -w /usr/share/wordlists/dirbuster/directory-list-2.3-medium.txt \
  --url http://172.18.0.33/ -o gobuster_dirs.txt

# Search for specific file extensions
sudo gobuster dir -w /usr/share/wordlists/dirbuster/directory-list-2.3-medium.txt \
  --url http://172.18.0.33/database/backup/secret/ \
  -x json,php,txt,bak,zip,tar.gz \
  --random-agent \
  --exclude-length 1591 \
  -b 403,404

# Advanced options
gobuster dir -u http://target.com \
  -w /usr/share/wordlists/dirbuster/directory-list-2.3-medium.txt \
  -x php,html,txt,bak \
  -t 50 \
  --random-agent \
  -k \
  -o results.txt
```

#### Gobuster Parameters
- `-x` - กำหนดนามสกุลไฟล์ที่ต้องการค้นหา
- `-b` - ข้าม HTTP status codes ที่ระบุ (403, 404)
- `--random-agent` - สุ่ม User-Agent เพื่อหลีกเลี่ยงการตรวจจับ
- `--exclude-length` - ข้าม response ที่มี content length ตามที่กำหนด
- `-t` - จำนวน threads (default: 10)
- `-k` - ข้ามการตรวจสอบ SSL certificate

[[Gobuster]]

### Dirb - Web Content Scanner
```bash
# Basic scan
dirb http://172.18.0.64/

# Scan with custom wordlist
dirb http://172.18.0.64/ /usr/share/wordlists/dirb/big.txt

# Scan with extensions
dirb http://172.18.0.64/ -X .php,.html,.txt,.bak

# Scan with authentication
dirb http://172.18.0.64/ -u username:password
```

### FFuF - Fast Web Fuzzer
```bash
# Directory fuzzing
ffuf -w /usr/share/wordlists/dirbuster/directory-list-2.3-medium.txt \
  -u http://target.com/FUZZ -o ffuf_dirs.json

# File fuzzing with multiple extensions
ffuf -w /usr/share/wordlists/dirbuster/directory-list-2.3-medium.txt \
  -u http://target.com/FUZZ \
  -e .php,.html,.txt,.bak,.zip

# Recursive fuzzing
ffuf -w wordlist.txt -u http://target.com/FUZZ -recursion -recursion-depth 2

# Filter responses by size
ffuf -w wordlist.txt -u http://target.com/FUZZ -fs 4242

# Match specific status codes
ffuf -w wordlist.txt -u http://target.com/FUZZ -mc 200,301,302
```

### Feroxbuster - Fast, Simple, Recursive Content Discovery
```bash
# Basic recursive scan
feroxbuster -u http://target.com -w /usr/share/wordlists/dirbuster/directory-list-2.3-medium.txt

# Deep recursive scan
feroxbuster -u http://target.com -w wordlist.txt --depth 4 -t 200

# Scan with extensions and filter
feroxbuster -u http://target.com -w wordlist.txt -x php,html,txt -C 404
```

### Backup File Discovery
```bash
# ลองเติม .bak หรือ backup extensions
# Common backup file patterns:
# - file.php.bak
# - file.php~
# - file.php.old
# - file.php.backup
# - .file.php.swp

# FFuF backup file fuzzing
ffuf -w filenames.txt -u http://target.com/FUZZ -e .bak,.old,.backup,.swp,~

# Manual testing
curl http://target.com/config.php.bak
curl http://target.com/database.php~
curl http://target.com/.index.php.swp
```

---

## Technology Fingerprinting

> [!note] Technology Stack Detection
> การระบุเทคโนโลยีที่ใช้ช่วยให้เราหา exploits และ vulnerabilities ที่เฉพาะเจาะจง

### WhatWeb - Web Technology Identifier
```bash
# Basic scan
whatweb http://target.com

# Aggressive scan (level 3)
whatweb -a 3 http://target.com

# Scan multiple URLs
whatweb -i urls.txt -a 3 --log-verbose=whatweb_results.txt

# Show detailed information
whatweb -v http://target.com
```

### Wappalyzer (CLI)
```bash
# ติดตั้ง wappalyzer-cli
npm install -g wappalyzer-cli

# Scan single URL
wappalyzer http://target.com

# Scan multiple URLs
wappalyzer http://target.com http://target2.com
```

### Manual Fingerprinting
```bash
# ตรวจสอบ HTTP headers
curl -I http://target.com

# ดู Server header
curl -s -I http://target.com | grep -i "server:"

# ดู X-Powered-By header
curl -s -I http://target.com | grep -i "x-powered-by:"

# ดู cookies
curl -I http://target.com | grep -i "set-cookie:"

# ดู response ด้วย verbose mode
curl -v http://target.com

# ตรวจสอบ SSL/TLS certificate
openssl s_client -connect target.com:443 </dev/null 2>/dev/null | openssl x509 -noout -text
```

### Nikto - Web Server Scanner
```bash
# Basic scan
nikto -h http://target.com

# Scan with tuning options
nikto -h http://target.com -Tuning 123bde

# Save output to file
nikto -h http://target.com -o nikto_results.txt -Format txt

# Scan multiple hosts
nikto -h hosts.txt
```

---

## DNS Enumeration

### Basic DNS Queries
```bash
# DNS lookup
nslookup target.com

# DNS records enumeration
dig target.com ANY

# Get A records
dig target.com A

# Get MX records (Mail servers)
dig target.com MX

# Get TXT records (SPF, DKIM)
dig target.com TXT

# Get NS records (Name servers)
dig target.com NS

# Reverse DNS lookup
dig -x 1.2.3.4

# Zone transfer attempt (ถ้า misconfigured)
dig axfr @ns1.target.com target.com
```

### DNSEnum - DNS Enumeration Tool
```bash
# Comprehensive DNS enumeration
dnsenum target.com

# With specific options
dnsenum --enum target.com -f /usr/share/wordlists/dnsmap.txt -o dnsenum_results.txt

# Attempt zone transfer
dnsenum --noreverse -o dnsenum.txt target.com
```

---

## Information Gathering

### Google Dorking
ใช้ Google search operators เพื่อหาข้อมูลที่ sensitive

```bash
# หา subdomains
site:target.com -www

# หาไฟล์ที่ sensitive
site:target.com filetype:pdf
site:target.com filetype:sql
site:target.com filetype:env
site:target.com filetype:log

# หา login pages
site:target.com inurl:login
site:target.com inurl:admin

# หา directory listing
site:target.com intitle:"index of"

# หา exposed files
site:target.com ext:sql | ext:db | ext:log | ext:bak

# หา error messages
site:target.com intext:"error" | intext:"warning"
```

### Shodan - Search Engine for Internet-connected Devices
```bash
# ติดตั้ง Shodan CLI
pip install shodan

# ค้นหาข้อมูลเกี่ยวกับ IP
shodan host 1.2.3.4

# ค้นหาด้วย query
shodan search "hostname:target.com"

# ค้นหา exposed services
shodan search "org:\"Company Name\" port:22"
```

### WHOIS Lookup
```bash
# WHOIS information
whois target.com

# Extract email addresses
whois target.com | grep -i "email"

# Extract nameservers
whois target.com | grep -i "name server"
```

### TheHarvester - OSINT Tool
```bash
# Gather emails, subdomains, hosts, employees
theHarvester -d target.com -b all -l 500 -f theharvester_results.html

# Use specific sources
theHarvester -d target.com -b google,bing,linkedin

# Search for emails only
theHarvester -d target.com -b all -l 200 | grep @
```

---

## Web Application Enumeration

### Robots.txt and Sitemap.xml
```bash
# ตรวจสอบ robots.txt
curl http://target.com/robots.txt

# ตรวจสอบ sitemap.xml
curl http://target.com/sitemap.xml

# Common sitemap locations
curl http://target.com/sitemap.xml
curl http://target.com/sitemap_index.xml
curl http://target.com/sitemap/sitemap.xml
```

### Common Configuration Files
```bash
# Configuration files
curl http://target.com/.env
curl http://target.com/config.php
curl http://target.com/web.config
curl http://target.com/.htaccess

# Git repository (exposed)
curl http://target.com/.git/config
curl http://target.com/.git/HEAD

# SVN repository
curl http://target.com/.svn/entries

# Docker files
curl http://target.com/.dockerignore
curl http://target.com/docker-compose.yml
```

### API Endpoint Discovery
```bash
# Common API paths
curl http://target.com/api/
curl http://target.com/api/v1/
curl http://target.com/api/v2/
curl http://target.com/graphql
curl http://target.com/swagger.json
curl http://target.com/api-docs

# API documentation
curl http://target.com/docs
curl http://target.com/api/docs
curl http://target.com/swagger-ui
```

---

## Automated Reconnaissance Tools

### Recon-ng - Web Reconnaissance Framework
```bash
# เริ่มต้น recon-ng
recon-ng

# สร้าง workspace
[recon-ng][default] > workspaces create target_recon

# เพิ่ม domain
[recon-ng][target_recon] > db insert domains target.com

# โหลด modules และรัน
[recon-ng][target_recon] > marketplace search
[recon-ng][target_recon] > marketplace install all
[recon-ng][target_recon] > modules load recon/domains-hosts/bing_domain_web
[recon-ng][target_recon] > run
```

### Photon - Fast Web Crawler
```bash
# Crawl website
python3 photon.py -u http://target.com

# Crawl with depth
python3 photon.py -u http://target.com -l 3

# Extract specific data
python3 photon.py -u http://target.com --keys --dns --export json
```

---

## Complete Reconnaissance Workflow

```bash
#!/bin/bash
# comprehensive_recon.sh - Complete web reconnaissance script
# การทำ reconnaissance อย่างครบวงจรสำหรับเป้าหมาย

TARGET="target.com"
OUTPUT_DIR="recon_$TARGET"

# สร้าง output directory
mkdir -p $OUTPUT_DIR

echo "[+] Starting reconnaissance for $TARGET"

# 1. Subdomain Enumeration
echo "[+] Enumerating subdomains..."
subfinder -d $TARGET -o $OUTPUT_DIR/subdomains_subfinder.txt
amass enum -d $TARGET -o $OUTPUT_DIR/subdomains_amass.txt

# รวมผลลัพธ์และลบ duplicates
cat $OUTPUT_DIR/subdomains_*.txt | sort -u > $OUTPUT_DIR/all_subdomains.txt

# 2. Check Live Hosts
echo "[+] Checking live hosts..."
cat $OUTPUT_DIR/all_subdomains.txt | httpx -silent -o $OUTPUT_DIR/live_hosts.txt

# 3. Port Scanning
echo "[+] Port scanning..."
nmap -iL $OUTPUT_DIR/live_hosts.txt -oN $OUTPUT_DIR/nmap_scan.txt

# 4. Technology Detection
echo "[+] Detecting technologies..."
while read host; do
    whatweb -a 3 $host >> $OUTPUT_DIR/technology_stack.txt
done < $OUTPUT_DIR/live_hosts.txt

# 5. Directory Enumeration
echo "[+] Directory enumeration..."
while read host; do
    gobuster dir -u $host -w /usr/share/wordlists/dirbuster/directory-list-2.3-medium.txt \
      -x php,html,txt,bak -o $OUTPUT_DIR/dirs_$(echo $host | tr '/:' '_').txt
done < $OUTPUT_DIR/live_hosts.txt

echo "[+] Reconnaissance complete! Results saved to $OUTPUT_DIR/"
```

---

## Related Techniques
- [[Nmap]]
- [[Gobuster]]
- [[Web Exploitation]]
- [[OSINT]]
- [[Google Dorking]]

## References
- OWASP Testing Guide: Information Gathering
- HackTricks: External Recon Methodology
- Pentesting Book: Web Application Reconnaissance
