#reverse 
# What
- ชุดเครื่องมือ Reverse Engineering แบบ command-line
- ครอบคลุม disassembly, debugging, patching, file analysis
# Example Command
```bash
# เปิดไฟล์พร้อมวิเคราะห์อัตโนมัติ
r2 -A ./a.out

# ดูฟังก์ชันทั้งหมด
afl

# ไปที่ main แล้วดูดิสแอสเซมบลีของฟังก์ชัน
s main
pdf

# โหมดกราฟ (visual graph)
VV     # กด q ออก
```
# Basic Command
## ข้อมูลไฟล์ (i*)
```bash
i         # summary ของไบนารี
iI        # info แบบละเอียด (arch/bits/os)
is        # สัญลักษณ์ (symbols)
iS        # sections/segments
iz        # strings ในไบนารี (จากตารางสัญลักษณ์)
izz       # ค้นหา strings แบบสแกนดิบ
ie; iM    # entrypoints; imports
```
## การวิเคราะห์โค้ด (a*)
```bash
aaa       # วิเคราะห์ทุกอย่าง (functions, xrefs, cfg) *ใช้บ่อย*
afl       # list ฟังก์ชันที่พบ
af        # สร้างฟังก์ชันที่ตำแหน่งปัจจุบัน (ถ้า r2 ยังไม่รู้จัก)
afi       # รายละเอียดฟังก์ชันปัจจุบัน
afn name  # เปลี่ยนชื่อฟังก์ชัน
axt @ addr_or_sym   # cross-references ถึงตำแหน่ง/สตริงนั้น
agf @ fcn           # วาดกราฟฟังก์ชัน (โหมดปกติ)
```
## การนำทางและธง (seek/flags)
```bash
s 0x401000    # seek ไปแอดเดรส
s main        # seek ไปสัญลักษณ์ชื่อ main
s-; s+        # ถอย/เดินหน้าตาม history
f name @ addr # ตั้ง "flag" ให้ตำแหน่ง (บุ๊กมาร์ก)
f~name        # ค้นหาธงด้วย regex
```
## แสดงผล/ดิสแอสเซมบลี (p*)
```bash
pd 20        # disassemble 20 ไลน์จากตำแหน่งปัจจุบัน
pdf @ main   # disassemble ทั้งฟังก์ชัน
px 64        # hexdump 64 ไบต์
ps @ addr    # พิมพ์สตริงที่ addr (หรือ ps 64 เพื่ออ่าน 64 ไบต์เป็นสตริง)
pc           # พิมพ์เป็น C-like pseudo (ถ้ารองรับในบางเวอร์ชัน/ปลั๊กอิน)
```
## ค้นหา (/*)
```bash
/x 9090          # ค้นหา pattern hex
/c "password"    # ค้นหาสตริง
/a "mov eax, 1"  # ค้นหา opcode/assembly
/!               # ดูผลลัพธ์ล่าสุดอีกครั้ง
```
## โหมด Visual (อินเทอร์แอคทีฟ)
```bash
V          # เข้า visual
p          # สลับมุมมอง (hex/asm/graph)
?          # ช่วยเหลือภายในโหมด
g          # ไปแอดเดรส
q          # ออก
VV         # เข้าโหมด graph view ตรงๆ
```
# Debugging
## เปิดแบบดีบัก
```bash
r2 -d ./prog           # เปิดโปรแกรมแบบดีบัก
ood ./prog arg1 arg2   # (ภายใน r2) เปิด/รันใหม่พร้อมอาร์กิวเมนต์
doo                    # run (exec) ปัจจุบันอีกครั้ง
```
## เบรกพอยน์ต์/รัน
```bash
db sym.main      # ตั้ง BP ที่ main (db @@ sym.* ตั้งหลายตัว)
db 0x401234      # ตั้ง BP ที่แอดเดรส
dbc 0x401234     # ลบ BP ที่แอดเดรส
dc               # continue
dcu 0x401080     # continue จนถึงแอดเดรส
ds               # step into (คำสั่งเดียว)
dso              # step over
dsi              # step instruction
```
## รีจิสเตอร์/เมมโมรี/แมป
```bash
dr               # ดู/แก้รีจิสเตอร์ (dr eax=0x1)
dm               # memory maps (ของโปรเซส)
pxw 32 @ rsp     # hexdump 32 ไบต์ที่ stack
dpt              # thread list (ถ้ามี)
```