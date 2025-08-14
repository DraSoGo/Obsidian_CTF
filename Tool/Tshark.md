#network 
# Basic Command
```bash
tshark -r file.pcap          # อ่านไฟล์ pcap
tshark -i eth0               # ดักฟัง live interface
```
# Protocol Overview
เริ่มต้นให้ดูว่ามี Protocol อะไรบ้างในไฟล์
```bash
tshark -r file.pcap -z io,phs
```
# Filter
## Syntax

| Filter                  | Mean                      |
| ----------------------- | ------------------------- |
| http                    | แพ็กเก็ต HTTP ทั้งหมด     |
| ftp                     | FTP Traffic               |
| tftp                    | TFTP Traffic              |
| ip.addr == 192.168.1.10 | IP ที่กำหนด               |
| tcp.port == 80          | TCP Port 80               |
| tftp.opcode == 3        | TFTP Data Packet เท่านั้น |
| frame contains "flag"   | หาแพ็กเก็ตที่มีคำว่า flag |
## Command
```bash
tshark -r file.pcap -Y "http"
tshark -r file.pcap -Y "frame contains \"flag\""
```
# Payload / Data
การดู Payload / Data
```bash
tshark -r file.pcap -Y "http" -T fields -e http.file_data
```
- -T fields → แสดงเฉพาะฟิลด์
- -e http.file_data → เลือกฟิลด์เนื้อหาไฟล์ HTTP
# Extract Files
HTTP
```bash
tshark -r file.pcap --export-objects "http,./out_dir"
```
TFTP
```bash
tshark -r file.pcap --export-objects "tftp,./out_dir"
```
FTP
```bash
tshark -r file.pcap --export-objects "ftp,./out_dir"
```