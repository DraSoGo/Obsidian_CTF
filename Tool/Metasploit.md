#web #network #privesc
# Basic knowledge
Metasploit เป็นเครื่องมือที่มีประสิทธิภาพสูงสำหรับการทดสอบการเจาะระบบ (penetration testing) การพัฒนา exploit และการวิจัยช่องโหว่ต่างๆ โดยมีส่วนประกอบหลักๆ ดังนี้
- Exploit: โค้ดที่ใช้โจมตีช่องโหว่ของระบบ
- Payload: โค้ดที่จะทำงานบนเครื่องเป้าหมายหลังจากที่ exploit ทำงานสำเร็จ ซึ่งมีตั้งแต่ command shell ธรรมดาไปจนถึง Meterpreter ที่มีความสามารถสูง
- Module: ส่วนประกอบต่างๆ ของ Metasploit เช่น exploit, payload, หรือ auxiliary scanner
- Listener: โปรเซสที่รอการเชื่อมต่อจากเครื่องเป้าหมาย
# Basic commands

| **Command**            | **Mean**                                                                          | **Example**                    |
| ---------------------- | --------------------------------------------------------------------------------- | ------------------------------ |
| msfconsole             | เป็นหน้าต่างหลักสำหรับใช้งาน Metasploit Framework                                 |                                |
| help                   | แสดงรายการคำสั่งทั้งหมดที่มีให้ใช้                                                |                                |
| search [keyword]       | ดูพาทชค้นหาโมดูลต่างๆ เช่น exploit, auxiliary scanner, และ payload                | search `scanner`               |
| use [module_name]      | เลือกใช้งานโมดูลที่ต้องการ                                                        | use `windows/smb/psexec`       |
| info                   | แสดงข้อมูลรายละเอียดของโมดูลที่กำลังใช้งานอยู่                                    |                                |
| show options           | แสดงตัวเลือกที่สามารถตั้งค่าได้สำหรับโมดูลปัจจุบัน                                |                                |
| set [options] [value]  | ตั้งค่าตัวเลือกสำหรับโมดูลปัจจุบัน                                                | set ==`rhost`== `192.12.10.3`  |
| setg [options] [value] | ตั้งค่าตัวเลือกแบบ global ซึ่งจะมีผลกับทุกโมดูล                                   | setg ==`rhost`== `192.12.10.3` |
| unset                  | ยกเลิกตั้งค่าปัจจุบัน                                                             | unset `rhost`                  |
| unsetg                 | ยกเลิกตั้งค่า global                                                              | unsetg `all`                   |
| exploit/run            | เริ่มการทำงานของโมดูลปัจจุบัน                                                     |                                |
| back                   | ออกจากการใช้งานโมดูลปัจจุบันและกลับไปที่หน้าหลักของ msfconsole                    |                                |
| sessions               | จัดการ session ที่กำลังเชื่อมต่ออยู่ สามารถแสดงรายการ, โต้ตอบ, และปิด session ได้ |                                |
# Database commands

| **Command**                                | **Mean**                                           |
| ------------------------------------------ | -------------------------------------------------- |
| db_connect [user]:[pass]@[host]/[database] | เชื่อมต่อกับฐานข้อมูล                              |
| db_import [file]                           | นำเข้าผลการสแกนจากเครื่องมืออย่าง Nmap หรือ Nessus |
| db_export [format] [file]                  | ส่งออกข้อมูลจากฐานข้อมูล                           |
| hosts                                      | แสดงรายการ host ทั้งหมดในฐานข้อมูล                 |
| vulns                                      | แสดงรายการช่องโหว่ทั้งหมดในฐานข้อมูล               |
# Meterpreter commands
Meterpreter เป็น payload ที่มีความสามารถสูงมาก ช่วยให้สามารถควบคุมเครื่องเป้าหมายได้อย่างเต็มที่

| **Command**                         | **Mean**                                           |
| ----------------------------------- | -------------------------------------------------- |
| sysinfo                             | แสดงข้อมูลระบบของเครื่องเป้าหมาย                   |
| getuid                              | แสดง user ID ที่ Meterpreter กำลังทำงานอยู่        |
| ps                                  | แสดงรายการโปรเซสที่กำลังทำงานอยู่บนเครื่องเป้าหมาย |
| migrate [pid]                       | ย้ายโปรเซสของ Meterpreter ไปยังโปรเซสอื่น          |
| shell                               | เข้าสู่ command shell ของเครื่องเป้าหมาย           |
| upload [local_file] [remote_path]   | อัปโหลดไฟล์ไปยังเครื่องเป้าหมาย                    |
| download [remote_file] [local_path] | ดาวน์โหลดไฟล์จากเครื่องเป้าหมาย                    |
| screenshot                          | ถ่ายภาพหน้าจอของเครื่องเป้าหมาย                    |
| webcam_snap                         | ถ่ายภาพจากเว็บแคมของเครื่องเป้าหมาย                |
| keyscan_start/keyscan_dump          | เริ่มและดึงข้อมูลจาก keylogger                     |
# MSFVenom commands
MSFVenom เป็นเครื่องมือที่ใช้สร้าง payload แบบ standalone

| **Command**                                                  | **Mean**                                                         |
| ------------------------------------------------------------ | ---------------------------------------------------------------- |
| msfvenom -p [payload] [options] -f [format] -o [output_file] | รูปแบบคำสั่งพื้นฐาน                                              |
| -p                                                           | ระบุ payload ที่จะใช้                                            |
| LHOST                                                        | ตั้งค่า IP ของเครื่องเรา (listening host) สำหรับ reverse shell   |
| LPORT                                                        | ตั้งค่า port ของเครื่องเรา (listening port) สำหรับ reverse shell |
| -f                                                           | ระบุรูปแบบของไฟล์ผลลัพธ์ (เช่น exe, py, raw)                     |
| -e                                                           | ระบุ encoder ที่จะใช้เพื่อเข้ารหัส payload                       |
| -i                                                           | กำหนดจำนวนรอบในการเข้ารหัส                                       |
| -o                                                           | ระบุชื่อไฟล์ผลลัพธ์                                              |