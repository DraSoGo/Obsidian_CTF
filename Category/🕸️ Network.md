#network
# 🏛️ OSI
![[OSI.png]]
# 🎮 Protocol
![[network_protocol.png]]
# 🩻 Structure
## IP Address
- IPv4 (32-bit): 192.168.1.1
- IPv6 (128-bit): 2001:db8::1
- แบ่งเป็น Public IP และ Private IP
## MAC Address
- รหัสประจำการ์ดเครือข่าย
## Port
- Range: 0–65535
- Well-known ports: 80 (HTTP), 443 (HTTPS), 22 (SSH), 21 (FTP)
## Subnet & Netmask
- ใช้แบ่งเครือข่าย เช่น /24 = 255.255.255.0
## Gateway & Routing
- Gateway คือ จุดออกสู่เครือข่ายอื่น
- Routing คือ การเลือกเส้นทางส่งข้อมูล
# 📃 Command

| **Command** | **Mean**                           | **Note**                                                                                                            |
| ----------- | ---------------------------------- | ------------------------------------------------------------------------------------------------------------------- |
| ifconfig    | ตรวจสอบสถานะอุปกรณ์เครือข่าย       |                                                                                                                     |
| ping        | ตรวจสอบการเชื่อมต่อเครือข่าย       | ping google.com                                                                                                     |
| nc          | ใช้เปิดการเชื่อมต่อ                | nc google.com 80                                                                                                    |
| ncat        | เหมือน nc แต่ดีกว่า                | ncat --ssl localhost 30000                                                                                          |
| nmap        | แสกน ip ว่าเปิด port อะไรบ้าง      |                                                                                                                     |
| nslookup    | ถามข้อมูลจาก DNS                   | หา ip address                                                                                                       |
| whois       | ถามหา Domain register              | หาเจ้าของ                                                                                                           |
| telnet      | ส่ง packet เข้าไปเพื่อตรวจสอบ port | telnet 10.10.105.127 80<br>GET /flag.html HTTP/1.1<br>Host: anything                                                |
| ftp         | เชื่อมต่อผ่าน ftp                  | fftp 10.10.105.127<br>Connected to 10.10.105.127.<br>220 (vsFTPd 3.0.5)<br>Name (10.10.105.127:drasogun): anonymous |
# 🛠️ Tool
- [[WireShark]] : จับและวิเคราะห์แพ็กเก็ต
- [[Tcpdump]] : จับแพ็กเก็ตผ่าน CLI
- [[Nmap]] : สแกนพอร์ต/บริการ
- [[Tshark]] : วิเคราะห์แพ็กเก็ตเชิงลึกแบบเดียวกับ Wireshark
- [[Metasploit]] : Framework สำหรับโจมตีและทดสอบช่องโหว่ พร้อม payload สำเร็จรูป
- [[Hydra]] : เครื่องมือ Brute-force รหัสผ่านสำหรับหลายโปรโตคอล