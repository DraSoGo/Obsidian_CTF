#forensics 
# What
แปลงไฟล์หรือสตริงให้เป็น hexadecimal dump (มองเห็นข้อมูลดิบ) หรือแปลงกลับจาก hex → ไฟล์ปกติก็ได้
# Command
## Hex dump
```bash
xxd file.bin
```
```yaml
00000000: 8950 4e47 0d0a 1a0a 0000 000d 4948 4452  .PNG........IHDR
```
- ด้านซ้าย = offset (ตำแหน่ง byte)
- ตรงกลาง = hex
- ด้านขวา = ASCII ที่อ่านออก
## Plain hex
```bash
xxd -p file.bin
```
```yaml
89504e470d0a1a0a0000000d49484452...
```
## Hex to file
```bash
xxd -r -p hex.txt out.bin
```
## Hex dump rule
```bash
xxd -s 0x100 -l 64 file.bin
```
- -s = เริ่มที่ offset
- -l = กำหนดจำนวน byte ที่จะแสดง
## Config byte of line
แสดงทีละ 8 byte ต่อบรรทัด (ค่า default = 16)
```
xxd -c 8 file.bin
```