#network 
# What
Responder คือเครื่องมือ “ตอบปลอม” โปรโตคอลชื่อโฮสต์/บริการในเครือข่ายวินโดวส์ เพื่อหลอกไคลเอนต์ให้เชื่อมต่อมายังเรา แล้วเก็บ NTLM challenge/response (hash) จากการพิสูจน์ตัวตน
# Syntax
```bash
responder -I [interface] [options]
```
- [interface] = ชื่อ network interface เช่น eth0, wlan0
# Option
| Option        | Mean                              |
| ------------- | --------------------------------- |
| -I eth0       | ระบุ interface                    |
| -w            | ตอบ Web request                   |
| -r            | ตอบ NetBIOS Name Service (NBT-NS) |
| -f            | ตอบ File Server (SMB)             |
| --disable SMB | ปิด module SMB                    |
| -v            | verbose mode                      
# Command
```bash
# 1. เริ่ม Responder บน interface eth0
sudo responder -I eth0

# 2. เริ่มและเปิดโหมด web/SMB/FTP response
sudo responder -I eth0 -wrf

# 3. ปิดบาง service เช่น SMB
sudo responder -I eth0 -wrf --disable SMB
```