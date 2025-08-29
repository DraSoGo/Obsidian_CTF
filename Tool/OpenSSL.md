#cryptography 
# What
- เครื่องมือ และไลบรารี สำหรับการเข้ารหัส และการจัดการ SSL/TLS
- ใช้กันอย่างกว้างขวางในโลกไอที ตั้งแต่ การเข้ารหัสไฟล์, การสร้างใบรับรองดิจิทัล (X.509), จัดการคีย์ RSA/EC, การทำ Hash, การเชื่อมต่อ HTTPS ไปจนถึง การวิเคราะห์โปรโตคอล TLS/SSL
- ส่วนใหญ่ติดตั้งมาใน Linux/Unix อยู่แล้ว (เช่น Ubuntu, Debian, macOS) และสามารถใช้ผ่าน command line ได้ทันที
# Use Case
## Hash
```bash
openssl dgst -sha256 myfile.txt
```
แสดงค่า SHA-256 ของไฟล์
## Symmetric Encryption
### Encryption
```bash
openssl enc -aes-256-cbc -salt -pbkdf2 -in secret.txt -out secret.txt.enc -k mypassword
```
### Decryption
```bash
openssl enc -d -aes-256-cbc -pbkdf2 -in secret.txt.enc -out secret.txt -k mypassword
```
## Key Pair RSA
```bash
# สร้าง private key 2048-bit
openssl genrsa -out private.pem 2048

# สร้าง public key
openssl rsa -in private.pem -pubout -out public.pem
```
## Certificate Signing Request
ใช้เวลาขอใบรับรอง SSL/TLS สำหรับเว็บไซต์
```bash
openssl req -new -key private.pem -out mycsr.csr
```
## ตรวจสอบใบรับรอง SSL ของเว็บไซต์
```bash
openssl s_client -connect example.com:443
```
## ดูข้อมูลใบรับรอง .crt
```bash
openssl x509 -in cert.crt -noout -text
```
# Command
```bash
openssl <command> [options]
openssl list -commands #ดูคำสั่งที่มีทั้งหมด
openssl list -cipher-commands #ดูรายการ cipher ที่ใช้กับ enc ได้
```
- enc : เข้ารหัส/ถอดรหัส
- dgst : ทำ hash
- genrsa : สร้างกุญแจ RSA
- req : ทำ CSR
- x509 : จัดการใบรับรอง X.509
- s_client : เชื่อมต่อ TLS/SSL