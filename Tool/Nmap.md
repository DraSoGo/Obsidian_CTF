#network #web 
# What
- สำหรับ scan network ว่ามี ip อะไร port ไหนบ้าง
# Command
## Scan
ดูว่าจะสแกนอะไรบ้าง / คือ subnet -  ipa - ipb
```bash
nmap -sL 192.168.0.1
```
แสกนด้วย ping
```bash
nmap -sn 192.168.0.1
```
แสกนด้วย TCP
```bash
nmap -sT 192.168.0.1
```
แสกนด้วย TCP SYN
```bash
nmap -sS 192.168.0.1
```
แสกนด้วย UDP
```bash
nmap -sU 192.168.0.1
```
แสกนด้วย 100 port ยอดนิยม
```bash
nmap -F 192.168.0.1
```
แสกนด้วย port 10-101
```bash
nmap -p10-101  192.168.0.1
```
แสกนด้วย host ที่หยุดการทำงานไปแล้ว
```bash
nmap -Pn 192.168.0.1
```
## Service detection
detect OS
```bash
nmap -O 192.168.0.1
```
detect version
```bash
nmap -sV 192.168.0.1
```
detect All
```bash
nmap -A 192.168.0.1
```
## Timing
- -T<0-5> : ใช้เวลาจากมากไปน้อย
- --min-parallelism <n> , --max-parallelism <n> : minimum & maximum number of parallel probes
- --min-rate <n> , --max-rate <n> : minimum & maximum อัตราเร็วการ ping
- --host-timeout : ระยะเวลาสูงสุดในการรอ host เป้าหมาย
## Debug
- -v<0-4> : ระดับความละเอียดข้อมูล
- -d<0-9> : ระดับการแก้ไขข้อบกพร่อง
## Report
- -oN <filename> : Normal output
- -oX <filename> : XML output
- -oG <filename> : grep-able output
- -oA <filename> : All output
## Script
```bash
nmap -sV --script vuln -v 10.10.136.62
```
https://nmap.org/nsedoc/categories/default.html