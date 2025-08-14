#network 
# What
- สำหรับดักจับและวิเคราะห์แพ็กเก็ตข้อมูลที่วิ่งอยู่บนเครือข่าย
# Command
อ่าน packet จาก network
```bash
tcpdump -i INTERFACE
```
อ่าน packet จาก ไฟล์
```bash
tcpdump -r FILE
```
เขียน packet ลงบนไฟล์
```bash
tcpdump -w FILE
```
จำกัดจำนวน packet ที่ output ออกมา
```bash
tcpdump -c COUNT
```
ไม่แก้ ip ก่อน output
```bash
tcpdump -n/-nn
```
# Filter
กรองแค่ example.com
```bash
tcpdump host example.com
```
กรองแค่ port 80
```bash
tcpdump port 80
```
กรองแค่ protocol
```bash
tcpdump tcp
```
src filter : ต้นทาง
dst filter : ปลายทาง
tcp ตัวเชื่อม : or,and,not
# Advance filter
กรอง length ≥ 1500
```bash
tcpdump greater 1500
```
กรอง length ≤ 1500
```bash
tcpdump less 1500
```
tcp-syn,tcp-ack,tcp-fin,tcp-rst,tcp-push
```bash
 tcpdump tcp[tcpflags] == tcp-syn
```
  ตัวเชื่อม : &,|,!
# Displaying
แสดงจำนวน packet
```bash
tcpdump | wc
```
ASCII
```bash
tcpdump -A
```
MAC
```bash
tcpdump -e
```
hexadecimal
```bash
tcpdump -xx
```
hexadecimal and ASCII
```bash
tcpdump -X
```