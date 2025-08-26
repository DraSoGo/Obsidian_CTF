#cryptography #forensics 
# What
Steghide เป็นเครื่องมือยอดนิยมสำหรับซ่อน (embed) และดึงข้อมูล (extract) จากไฟล์รูปภาพหรือไฟล์เสียง เช่น JPG, BMP, WAV, AU
# Command
## Embed data
ซ่อนไฟล์
```bash
steghide embed -cf cover.jpg -ef secret.txt
```
## Extract data
ดึงไฟล์
```bash
steghide extract -sf cover.jpg
```
## Info
ดู metadata ภายในไฟล์
```bash
steghide info cover.jpg
```