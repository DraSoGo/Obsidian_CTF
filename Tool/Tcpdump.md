#network 
- สำหรับดักจับและวิเคราะห์แพ็กเก็ตข้อมูลที่วิ่งอยู่บนเครือข่าย
- command
  tcpdump -i INTERFACE : อ่าน packet จาก network
  tcpdump -r FILE : อ่าน packet จาก ไฟล์
  tcpdump -w FILE : เขียน packet ลงบนไฟล์
  tcpdump -c COUNT : จำกัดจำนวน packet ที่ output ออกมา
  tcpdump -n/-nn : ไม่แก้ ip ก่อน output
- filter
  src filter : ต้นทาง
  dst filter : ปลายทาง
  tcpdump host example.com : กรองแค่ example.com
  tcpdump port 80 : กรองแค่ port 80
  tcpdump tcp : กรองแค่ protocol
  tcp ตัวเชื่อม : or,and,not
- advance filter
  tcpdump ‘greater 1500’ : กรอง length ≥ 1500
  tcpdump ‘less 1500’ : กรอง length ≤ 1500
  tcpdump ‘tcp[tcpflags] == tcp-syn’ : tcp-syn,tcp-ack,tcp-fin,tcp-rst,tcp-push
  ตัวเชื่อม : &,|,!
- displaying
  tcpdump | wc : แสดงจำนวน packet
  tcpdump -A : ASCII tcpdump -e : MAC
  tcpdump -xx : hexadecimal
  tcpdump -X : hexadecimal and ASCII