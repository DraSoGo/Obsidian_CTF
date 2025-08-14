#forensics 
# What
เครื่องมือนี้เหมาะกับการหาข้อมูลซ่อน (steganography) ในไฟล์ภาพ **PNG** และ **BMP** โดยเฉพาะ
**zsteg** เป็นเครื่องมือ CLI สำหรับตรวจหา **hidden data** ในไฟล์ภาพ โดยดูจาก
- การซ่อนข้อมูลใน **Least Significant Bit (LSB)** ของช่องสี (Red, Green, Blue, Alpha)
- การซ่อนใน **palette** (สำหรับภาพที่มี palette table)
- การซ่อนใน **metadata / chunks** ของ PNG
- การ encode เป็น text, base64, zlib-compressed ฯลฯ
# Syntax
```bash
zsteg [options] file.png
```

| Option    | Mean                                         |
| --------- | -------------------------------------------- |
|           | สแกนทุกแบบอัตโนมัติ                          |
| -a        | Auto-detect ทั้งหมด                          |
| -E <expr> | Extract ตาม expression ที่กำหนด เช่น -E b1,r |
| -l        | List รูปแบบที่ตรวจได้                        |
# Expression
**format**
- b1 = ใช้ 1 bit (LSB)
- r, g, b, a = เลือก channel (red, green, blue, alpha)
- msb, lsb, br = bit order หรือ byte order

**example**
- b1,r = LSB ของช่องสีแดง
- b2,g = bit ที่ 2 ของช่องสีเขียว
- b1,rgba = รวมทุก channel