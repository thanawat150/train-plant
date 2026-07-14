# train-plant

ชุดตัวอย่างและ Skill สำหรับช่วยเสนอขอบเขตพื้นที่ที่พบร่องรอยการปลูกจากภาพโดรนหรือ Orthomosaic

> ผลลัพธ์เป็น Candidate Boundary จากหลักฐานในภาพ ต้องผ่าน Human Review ก่อนนำไปใช้งานจริง และไม่ใช่ขอบเขตทางกฎหมาย

## โครงสร้าง

```text
planting-evidence-boundary/
├─ SKILL.md
├─ PROMPT.md
└─ examples/
   └─ example_01/
      ├─ input.jpg
      ├─ expected_overlay.jpg
      ├─ expected_boundary_pixel.json
      ├─ example_manifest.json
      └─ notes.md
```

## วิธีเริ่มใช้

ให้ Codex อ่าน `planting-evidence-boundary/SKILL.md` และไฟล์ทั้งหมดใน `examples/example_01` ก่อนเขียนโค้ดหรือวิเคราะห์ภาพใหม่
