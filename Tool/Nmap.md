#network #web 
- สำหรับ scan network ว่ามี ip อะไร port ไหนบ้าง
- scan
	nmap -sL 192.168.0.1 : ดูว่าจะสแกนอะไรบ้าง / คือ subnet -  ipa - ipb
	nmap -sn 192.168.0.1 : แสกนด้วย ping
	nmap -sT 192.168.0.1 : แสกนด้วย TCP
	nmap -sS 192.168.0.1 : แสกนด้วย TCP SYN
	nmap -sU 192.168.0.1 : แสกนด้วย UDP
	nmap -F 192.168.0.1 : แสกนด้วย 100 port ยอดนิยม
	nmap -p10-101  192.168.0.1 : แสกนด้วย port 10-101
	nmap -Pn 192.168.0.1 : แสกนด้วย host ที่หยุดการทำงานไปแล้ว
- service detection
	nmap -O 192.168.0.1 : detect OS
	nmap -sV 192.168.0.1 : detect version
	nmap -A 192.168.0.1 : detect All
- timing
	-T<0-5> : ใช้เวลาจากมากไปน้อย
	--min-parallelism <n> , --max-parallelism <n> : minimum & maximum number of parallel probes
	--min-rate <n> , --max-rate <n> : minimum & maximum อัตราเร็วการ ping
	--host-timeout : ระยะเวลาสูงสุดในการรอ host เป้าหมาย
- debug
	-v<0-4> : ระดับความละเอียดข้อมูล
	-d<0-9> : ระดับการแก้ไขข้อบกพร่อง
- report
	-oN <filename> : Normal output
	-oX <filename> : XML output
	-oG <filename> : grep-able output
	-oA <filename> : All output
- script
	nmap -sV --script vuln -v 10.10.136.62
	https://nmap.org/nsedoc/categories/default.html