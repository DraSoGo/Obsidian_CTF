#forensics 
![[forensics.png]]
# ❓ What
Digital Forensics คือ กระบวนการ เก็บ รักษา วิเคราะห์ และนำเสนอ ข้อมูลดิจิทัลที่เป็นหลักฐานได้อย่างถูกต้องตามหลักกฎหมาย
# 🟦 Category
- Computer Forensics : วิเคราะห์คอมพิวเตอร์, ดิสก์, ไฟล์, ระบบปฏิบัติการ
- Mobile Device Forensics : วิเคราะห์โทรศัพท์และแท็บเล็ต
- Network Forensics : วิเคราะห์ข้อมูลการสื่อสารผ่านเครือข่าย
- Memory Forensics : วิเคราะห์ข้อมูลใน RAM
- Cloud Forensics : เก็บและวิเคราะห์ข้อมูลในระบบ Cloud
- Malware Forensics : วิเคราะห์ไฟล์หรือโปรแกรมที่เป็นอันตราย
# ⭕ Core Principles
1. Preservation : เก็บรักษาหลักฐานโดยไม่เปลี่ยนแปลงข้อมูลต้นฉบับ
2. Acquisition : ทำสำเนาหลักฐาน (Forensic Image) พร้อมสร้างค่า hash ยืนยันความถูกต้อง
3. Analysis : วิเคราะห์เนื้อหา, โครงสร้าง, metadata, timeline
4. Documentation : บันทึกขั้นตอนและผลการตรวจสอบอย่างละเอียด
5. Presentation : นำเสนอข้อมูลที่เข้าใจง่ายและน่าเชื่อถือในรายงานหรือชั้นศาล
# 🥾 Forensic Process Model
1. Identification : ระบุและหาตำแหน่งของหลักฐานที่เกี่ยวข้อง
2. Collection : เก็บข้อมูลด้วยวิธีที่ปลอดภัยและถูกต้อง
3. Examination : ตรวจสอบไฟล์, โฟลเดอร์, artifact
4. Analysis : ตีความและเชื่อมโยงเหตุการณ์
5. Reporting : จัดทำรายงานสรุป
# 📚 Data
- Metadata : ข้อมูลประกอบไฟล์ (วันที่สร้าง, แก้ไข, ผู้สร้าง)
- Timestamps : เวลา MACE (Modified, Accessed, Created, Entry Modified)
- Log Files : บันทึกเหตุการณ์จากระบบและแอปพลิเคชัน
- Artifacts : ร่องรอยที่เหลือจากการใช้งาน (Recycle Bin, Browser History, Prefetch)
- Deleted Data : ข้อมูลที่ถูกลบแต่สามารถกู้คืนได้
- File Signatures : ตรวจสอบชนิดไฟล์จริง
# 🛠️ Tool
- [[PDF info]] : ใช้ตรวจสอบ metadata และโครงสร้างภายในไฟล์ PDF
- [[Exif Tool]] : ใช้ค้นหาร่องรอยการแก้ไขหรือที่มาของไฟล์
- [[Aperi resolve]] : เครื่องมือ ซ่อมหรืออ่านไฟล์ภาพที่เสียหาย
- [[Binwalk]] : สามารถ extract ไฟล์ย่อยที่ฝังอยู่
- [[Ewfmount]] : ใช้ mount ไฟล์ forensic image ที่เป็นฟอร์แมต EWF (เช่น .E01)
- [[Stylesuxx]] : Steganography Online
- [[Hexed]] : โปรแกรม Hex Editor สำหรับเปิดและแก้ไขไฟล์ในรูปแบบฐาน 16
- [[Sleuth Kit]] : ชุดเครื่องมือ command-line สำหรับ วิเคราะห์ระบบไฟล์ disk image
- [[Zsteg]] : เครื่องมือ Steganography analysis สำหรับไฟล์ภาพ PNG และ BMP
- [[Decompiler]] : ใช้แปลงโค้ดคอมไพล์ (binary/executable) กลับเป็นโค้ดที่อ่านง่ายขึ้น
- [[Steghide]] : ใช้สำหรับซ่อนข้อมูล และดึงข้อมูล ที่ถูกซ่อนเอาไว้ในไฟล์ .jpg, .bmp, .wav, .au
- [[Git]] : Git
- [[XXD]] : จัดการไฟล์ heximal
- [[Stegsolve]] : เครื่องมือสำหรับวิเคราะห์ภาพที่มีข้อมูลซ่อนอยู่
# 🗒️ Resource
- [[Hex]] : Hexadecimal Representation