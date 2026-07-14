# train-plant

ชุดตัวอย่างและ Skill สำหรับช่วยเสนอขอบเขตพื้นที่ที่พบร่องรอยการปลูกจากภาพโดรนหรือ Orthomosaic ขนาดใหญ่

> ผลลัพธ์เป็น Candidate Boundary จากหลักฐานในภาพ ต้องผ่าน Human Review ก่อนนำไปใช้งานจริง และไม่ใช่ขอบเขตทางกฎหมาย

## ข้อเท็จจริงสำคัญ

ภาพใน `examples/example_01` เป็นเพียงภาพแคปบางส่วนจาก Orthomosaic โดรนขนาดใหญ่ ใช้สอนการตีความ Include / Exclude / Uncertain เท่านั้น

ระบบจริงต้องประมวลผล GeoTIFF, COG หรือ VRT แบบ Overview + Tile/Window และคืน Polygon ใน CRS ของภาพต้นฉบับ ห้ามใช้ Pixel Coordinate จากภาพแคปเป็นพิกัดจริง

## โครงสร้าง

```text
AGENTS.md
planting-evidence-boundary/
├─ SKILL.md
├─ LARGE_ORTHOMOSAIC_WORKFLOW.md
├─ PROMPT.md
├─ PROMPT_LARGE_ORTHOMOSAIC.md
└─ examples/
   └─ example_01/
      ├─ input.jpg
      ├─ expected_overlay.jpg
      ├─ expected_boundary_pixel.json
      ├─ example_manifest.json
      └─ notes.md
```

## วิธีเริ่มใช้

เปิด Repository ด้วย Codex แล้วให้ทำตาม `AGENTS.md`

สำหรับการพัฒนาระบบกับ Orthomosaic ขนาดใหญ่ ให้ใช้ Prompt ใน:

`planting-evidence-boundary/PROMPT_LARGE_ORTHOMOSAIC.md`

## แนวทางประมวลผลจริง

```text
Large georeferenced orthomosaic
→ read metadata and overviews
→ optional AOI clipping
→ overlapping tile/window processing
→ planting evidence analysis
→ tile-edge suppression
→ transform results to source CRS
→ merge/dissolve polygons
→ geometry QA
→ human review
→ export GPKG/GeoJSON/KML
```

## หมายเหตุเรื่องไฟล์ตัวอย่าง

ไฟล์ภาพตัวอย่างควรเป็น Crop ขนาดพอให้เห็นร่องรอยการปลูกชัด ไม่จำเป็นต้องนำ Orthomosaic ขนาดใหญ่มาเก็บใน Git โดยตรง

ไฟล์ Orthomosaic จริงควรเก็บไว้นอก Repository หรือใช้ Git LFS/Object Storage เมื่อจำเป็น เนื่องจากไฟล์อาจมีขนาดหลาย GB
