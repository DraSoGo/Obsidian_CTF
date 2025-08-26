#privesc 
![[privesc.png]]
# ❓ What
- Privilege Escalation = การยกระดับสิทธิ์จากผู้ใช้ปัจจุบัน → สิทธิ์สูงกว่า (เช่น root)
- ใน Linux แบ่งเป็น:
    - Horizontal Escalation: ขยายสิทธิ์ไปยัง user อื่นที่มีสิทธิ์เท่ากัน
    - Vertical Escalation: ขึ้นไปสู่สิทธิ์สูงกว่า เช่น จาก www-data → root
# ➖ Remote
```bash
ssh USER@IP
```
# 🛠️ Tool
- [[Metasploit]] : Framework สำหรับทำ penetration testing และ exploit ช่องโหว่
- [[Reverse Shell]] : เทคนิคให้เครื่องเป้าหมายเชื่อมกลับมาหาเครื่องเราเพื่อควบคุมจากระยะไกล
- [[LinPEAS]] : สคริปต์สแกน Linux เพื่อหา misconfig และช่องโหว่สำหรับ privilege escalation
- [[pspy]] : เครื่องมือดู process และ cron job แบบ real-time โดยไม่ต้องใช้ root
# 💡 Technic
- [[Sudo]] : การใช้สิทธิ์ root ผ่านคำสั่ง sudo
- [[Passwd]] : การใช้รหัสผ่าน (หรือไฟล์รหัสผ่าน) เพื่อเข้าถึงสิทธิ์สูงกว่า
- [[SUID]] : ไฟล์ที่รันด้วยสิทธิ์เจ้าของไฟล์ แม้ผู้รันจะเป็น user ปกติ
- [[Race Condition]] : การโจมตีโดยใช้จังหวะเวลาที่โปรแกรมเข้าถึงไฟล์/ทรัพยากรพร้อมกัน เพื่อสอดแทรก payload
- [[Crontab Exploit]] : ใช้ประโยชน์จาก cron job ที่ root รันอยู่ โดยแก้ไข script หรือไฟล์ที่ถูกเรียก
# 🗒️ Resource
- [[GTFOBins]] : ฐานข้อมูลการใช้คำสั่ง Unix/Linux ปกติให้ทำ privilege escalation หรือ bypass security ได้