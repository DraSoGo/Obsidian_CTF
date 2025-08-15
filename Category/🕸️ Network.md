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
# 🛠️ Tool
- [[WireShark]] : จับและวิเคราะห์แพ็กเก็ต
- [[Tcpdump]] : จับแพ็กเก็ตผ่าน CLI
- [[Nmap]] : สแกนพอร์ต/บริการ
- [[Tshark]] : วิเคราะห์แพ็กเก็ตเชิงลึกแบบเดียวกับ Wireshark
- [[Metasploit]] : Framework สำหรับโจมตีและทดสอบช่องโหว่ พร้อม payload สำเร็จรูป