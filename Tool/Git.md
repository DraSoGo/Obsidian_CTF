#forensics 
# What
คือ ระบบควบคุมเวอร์ชัน (Version Control System – VCS) แบบกระจาย (Distributed)
# Command
## Start
Make project
```bash
git init
```
Clone
```bash
git clone https://github.com/user/repo.git
# หรือแบบ SSH (แนะนำภายหลังตั้งกุญแจ)
git clone git@github.com:user/repo.git
```
## Default Command
```bash
git status                # ดูไฟล์ที่เปลี่ยน
git add file1 file2       # เลือกไฟล์ที่จะ commit
git add .                 # เพิ่มทั้งหมดที่เปลี่ยน
git commit -m "อธิบายสิ่งที่เปลี่ยน"
git log --oneline --graph # ดูประวัติสั้นๆ
```
## Branch && Merge
```bash
git branch                # ดูสาขา
git checkout -b feature/x # สร้างและย้ายไปสาขาใหม่
# แก้โค้ด → add → commit ตามปกติ
git checkout main
git merge feature/x       # รวมเข้ากับ main
git branch -d feature/x   # ลบสาขา (หลัง merge แล้ว)
```
## Remote
Connect
```bash
git remote add origin https://github.com/user/repo.git
# หรือ SSH
git remote add origin git@github.com:user/repo.git
```
Push
```bash
git push -u origin main     # ครั้งแรก
git push                    # ครั้งต่อๆ ไป
```
Pull
```
git pull                    # ดึงและผสานอัตโนมัติ
git fetch                   # แค่ดึงข้อมูล ไม่ผสาน
```
## Reverse
```bash
git restore file.txt                 # ย้อนไฟล์ (ยังไม่ add)
git restore --staged file.txt        # เอาไฟล์ออกจาก stage
git checkout -- file.txt             # (สไตล์เก่า) ย้อนไฟล์

git reset --soft HEAD~1              # ย้อน commit ล่าสุด แต่เก็บไฟล์ไว้ใน stage
git reset --mixed HEAD~1             # ย้อน commit ล่าสุด ไฟล์กลับไปเป็น untracked/modified
git reset --hard HEAD~1              # ย้อนแบบล้างหมด (ระวัง!)

git revert <commit-hash>             # สร้าง commit แก้ย้อนอย่างปลอดภัย
```
## Stash
```bash
git stash           # เก็บการเปลี่ยนไว้ชั่วคราว
git stash list
git stash apply     # ดึงกลับ (ยังคงใน list)
git stash pop       # ดึงกลับและลบจาก list
```
## SSH Key
```bash
ssh-keygen -t ed25519 -C "you@example.com"
# กด Enter ยาวๆ → เอา public key ไปใส่ใน GitHub: Settings → SSH and GPG keys
cat ~/.ssh/id_ed25519.pub
```