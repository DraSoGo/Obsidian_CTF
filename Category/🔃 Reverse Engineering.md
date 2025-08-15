#reverse
![[reverse_engineering.png]]
# ❓ What
- Reverse Engineering คือการวิเคราะห์โปรแกรมหรือระบบที่ถูกคอมไพล์แล้ว เพื่อทำความเข้าใจโครงสร้างและการทำงานภายใน
- ใช้ใน
    - การหาช่องโหว่ในโปรแกรม
    - วิเคราะห์ Malware
    - ทำ Keygen/Crack (ในแล็บที่ได้รับอนุญาต)
    - ทำ Patch/Bypass การตรวจสอบ
# 🥾 Workflow
## Gather Info
- เก็บข้อมูล binary (ไฟล์ EXE, ELF, APK)
- ดู metadata (file type, compiler, architecture)
## Static Recon
- ใช้ strings ดูข้อความฝังใน binary
- ใช้ Disassembler (Ghidra, IDA Free, Radare2)
- อ่านโครงสร้าง function, flow
## Dynamic Debug
- ใช้ Debugger (GDB, x64dbg, OllyDbg)
- วาง breakpoint → step-by-step → ตรวจ register/memory
## Modification
- แก้ assembly, patch binary, เปลี่ยน logic
## Testing
- รันโปรแกรมที่แก้ → ยืนยันผล
## Documentation
- บันทึก offset, function สำคัญ, พฤติกรรมที่สังเกตได้
# 💡 Basic Knowledge
- Architecture: x86, x86_64, ARM
- Assembly Language: mov, cmp, jmp, call, push, pop
- Calling Conventions: cdecl, stdcall, fastcall
- Binary Formats
    - ELF (Linux)
    - PE (Windows)
    - Mach-O (macOS)
- Memory Layout
    - Stack, Heap, Data, Text segments
- Compiler & Optimization: มีผลต่อโค้ด assembly
# 🛠️ Tool
- [[Exif Tool]] : ใช้ค้นหาร่องรอยการแก้ไขหรือที่มาของไฟล์
- [[Decompiler]] : ใช้แปลงโค้ดคอมไพล์ (binary/executable) กลับเป็นโค้ดที่อ่านง่ายขึ้น
- [[Radare2]] : ชุดเครื่องมือ Reverse Engineering แบบ command-line