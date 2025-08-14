#forensics 
# What
- The Sleuth Kit เป็นชุดคำสั่ง CLI สำหรับทำ Disk & File System Forensics
- ใช้วิเคราะห์ image เช่น .dd, .img, .E01 เพื่อดูโครงสร้างพาร์ทิชัน, ไฟล์ที่ลบ, metadata ฯลฯ
# Tool
## Img Stat
ดูข้อมูลพื้นฐานของ image
### Syntax
```
img_stat image.dd
```
### Sample
```bash
img_stat disk.img
```
## MMLS
แสดง Partition Table
### Syntax
```
mmls image.dd
```
### Sample
```bash
mmls disk.img
```
## FS Stat
แสดงข้อมูล Filesystem
### Syntax
```
fsstat -o <sector> image.dd
```
### Sample
```bash
fsstat -o 2048 disk.img
```
## FLS
List ไฟล์และไดเรกทอรี (รวมไฟล์ลบ)
### Syntax
```
fls [options] -o <sector> image.dd
```
- -r = recursive  
- -m path = ใส่ path prefix
### Sample
```bash
fls -o 2048 disk.img
fls -o 2048 -r disk.img | grep -i deleted
```
## ICat
Extract ไฟล์จาก inode
### Syntax
```
icat -o <sector> image.dd <inode>
```
### Sample
```bash
icat -o 2048 disk.img 42 > secret.txt
```
## IStat
ดู metadata ของ inode
### Syntax
```
istat -o <sector> image.dd <inode>
```
### Sample
```bash
istat -o 2048 disk.img 42
```
## BLKLS
Dump ข้อมูล block (รวม unallocated space)
### Syntax
```
blkls -o <sector> image.dd
```
### Sample
```bash
blkls -o 2048 disk.img > unallocated.raw
```
## Tsk Recover
กู้ไฟล์ทั้งหมดออกมา
### Syntax
```
tsk_recover -o <sector> image.dd output_dir
```
### Sample
```bash
tsk_recover -o 2048 disk.img ./recovered
```
## FFind
หา inode ของไฟล์ที่มีชื่อกำหนด
### Syntax
```
ffind image.dd "filename"
```
### Sample
```bash
ffind disk.img "flag.txt"
```
## Srch Strings
ค้นหา string (คล้าย strings)
### Syntax
```
srch_strings image.dd | grep keyword
```
### Sample
```bash
srch_strings disk.img | grep -i flag
```
# Step
1. ดูโครงสร้างพาร์ทิชัน
```bash
mmls disk.img
```
2. ดูข้อมูล Filesystem
```bash
fsstat -o 2048 disk.img
```
3. List ไฟล์
```bash
fls -o 2048 disk.img
```
4. Extract ไฟล์
```bash
icat -o 2048 disk.img 15 > secret.txt
```
- 15 คือ inode number ของไฟล์
5. ดู metadata ของไฟล์
```bash
istat -o 2048 disk.img 15
```
6. กู้ไฟล์ทั้งหมด (รวดเร็ว)
```bash
tsk_recover -o 2048 disk.img ./output_dir
```
7. ค้นหาคีย์เวิร์ด / flag
```bash
blkls -o 2048 disk.img | strings | grep -i flag
```