#forensics 
# What
เครื่องมือสำหรับวิเคราะห์ภาพที่มีข้อมูลซ่อนอยู่
# Option
- แยกมุมมองและลองดูแต่ละ bit-plane ของภาพ (เช่น การดู channel ต่าง ๆ หรือระดับ bit ชั้นต่าง ๆ)
- ทำการแปลงภาพต่าง ๆ เช่น grayscale, color channels, inversion เพื่อดูว่าอะไรซ่อนอยู่
- รองรับการแยกข้อมูลจาก plane แบบ row หรือ column และแปลงเป็น bytes ได้
- มีโหมด stereogram solver ช่วยปรับ offset จนเห็นภาพซ่อน (เหมือนการมองภาพ Magic Eye)
- รองรับการดู frame สำหรับภาพเคลื่อนไหว (เช่น GIF) หรือภาพ PNG ที่มีหลาย layer
- รวมภาพ (combine image) เพื่อดูผลลัพธ์ที่ซ้อนกันหลายแบบ
# Run
```bash
java -jar ./bin/stegsolve.jar
```