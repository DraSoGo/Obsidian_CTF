#web
![[web_application.png]]
# ❓What
- **Web Application** คือ โปรแกรมหรือระบบที่ทำงานบนเว็บเบราว์เซอร์ โดยผู้ใช้เข้าถึงผ่าน **HTTP/HTTPS**
- แตกต่างจากเว็บไซต์ธรรมดาตรงที่ Web Application มี **การโต้ตอบ (Interactive)** และ **ประมวลผลฝั่งเซิร์ฟเวอร์** เช่น ระบบชำระเงินออนไลน์, ระบบจองตั๋ว, Email
# 🔧 Architecture
## Client Side (Front-end)
- คือ ส่วนที่ผู้ใช้มองเห็นและโต้ตอบผ่านเบราว์เซอร์
- ภาษาที่ใช้
	- HTML → โครงสร้างเนื้อหา
	-CSS → การจัดรูปแบบและสไตล์
	- JavaScript → เพิ่มความโต้ตอบ เช่น ฟอร์มตรวจสอบข้อมูล
- Framework/Library: React, Vue.js, Angular
## Server Side (Back-end)
- คือ ส่วนที่ประมวลผลข้อมูลและตอบกลับไปยัง Client
- ภาษาที่ใช้: PHP, Python (Django/Flask), Node.js, Java, Ruby
- ทำหน้าที่
	- รับ Request
	- ประมวลผล Logic
	- ติดต่อฐานข้อมูล
	- ส่ง Response กลับไป
## Database
- เก็บข้อมูลที่เว็บต้องใช้ เช่น ข้อมูลผู้ใช้, รายการสินค้า
- ประเภท
	- Relational (MySQL, PostgreSQL, SQL Server)
	- NoSQL (MongoDB, Redis)
# 📡 Communication
ใช้ HTTP หรือ HTTPS
## Request
- จาก Client → Server
- ประกอบด้วย
	- Method: GET, POST, PUT, DELETE
	- Headers: ข้อมูลเสริม เช่น Cookie, Content-Type
	- Body: ข้อมูลที่ส่งไป (เช่น ฟอร์ม)
## Response
- จาก Server → Client
- ประกอบด้วย
	- Status Code: เช่น 200 OK, 404 Not Found, 500 Internal Server Error
	- Headers
	- Body: เนื้อหาที่ส่งกลับ (HTML, JSON, ไฟล์ ฯลฯ)
# 🦾 Core Mechanisms
## Session
- เก็บข้อมูลสถานะของผู้ใช้ เช่น หลังล็อกอิน
- ใช้ Session ID ที่เก็บใน Cookie หรือ URL
## Cookies
- ข้อมูลขนาดเล็กที่เก็บในเครื่องผู้ใช้ เพื่อจดจำสถานะ เช่น การเข้าสู่ระบบ, การตั้งค่าภาษา
## Authentication
- กระบวนการยืนยันตัวตน (เช่น การเข้าสู่ระบบด้วยรหัสผ่าน, OTP)
## Authorization
- การกำหนดสิทธิ์เข้าถึงข้อมูลหรือฟังก์ชัน
## API
- ช่องทางให้ระบบอื่นเรียกใช้งานข้อมูลหรือฟังก์ชัน เช่น REST API, GraphQL
# 🟦 Type
- Static Web App: เนื้อหาไม่เปลี่ยน เช่น เว็บไซต์ Portfolio
- Dynamic Web App: มีการดึงข้อมูลจากฐานข้อมูล เช่น E-Commerce
- Single Page Application (SPA): โหลดหน้าเดียวแล้วเปลี่ยนเนื้อหาด้วย JavaScript
- Progressive Web App (PWA): ทำงานเหมือนแอปมือถือ ใช้งาน Offline ได้บางส่วน

# 🛠️ Tool
- [[Online Web]]
- [[Burp Suite]] : Proxy สำหรับดักและแก้ไข HTTP/HTTPS Request–Response ใช้ทดสอบความปลอดภัยเว็บ
- [[HHTrack]] : โปรแกรมดาวน์โหลดเว็บไซต์ทั้งหมดมาเก็บไว้แบบ Offline
- [[Hydra]] : เครื่องมือ Brute-force รหัสผ่านสำหรับหลายโปรโตคอล
- [[Gobuster]] : เครื่องมือค้นหา directory หรือไฟล์ที่ซ่อนอยู่ในเว็บ
- [[SQLMap]] : เครื่องมือเจาะช่องโหว่ SQL Injection และดึงข้อมูลจากฐานข้อมูล
- [[Nmap]] : เครื่องมือสแกนพอร์ตและค้นหาบริการที่รันอยู่บนเซิร์ฟเวอร์
- [[Metasploit]] : Framework สำหรับโจมตีและทดสอบช่องโหว่ พร้อม payload สำเร็จรูป
# 💡Technic
- [[Authentication]] : กระบวนการยืนยันตัวตนผู้ใช้ (เช่น Login)
- [[Web Reconnaissance]] : การเก็บข้อมูลเว็บเป้าหมายก่อนโจมตี
- [[SQL Injection]] : - เทคนิคใส่คำสั่ง SQL ผ่านช่องทางรับข้อมูลเพื่อควบคุมฐานข้อมูล