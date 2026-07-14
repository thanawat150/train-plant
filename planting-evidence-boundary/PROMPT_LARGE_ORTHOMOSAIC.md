# Prompt สำหรับ Codex — Orthomosaic ขนาดใหญ่

```text
ใช้ Repository นี้เพื่อพัฒนาระบบเสนอขอบเขตพื้นที่ที่พบร่องรอยการปลูกจากภาพ Orthomosaic ขนาดใหญ่

ก่อนเริ่มงาน ให้อ่าน:

1. AGENTS.md
2. planting-evidence-boundary/SKILL.md
3. planting-evidence-boundary/LARGE_ORTHOMOSAIC_WORKFLOW.md
4. planting-evidence-boundary/examples/example_01/notes.md
5. planting-evidence-boundary/examples/example_01/example_manifest.json
6. planting-evidence-boundary/examples/example_01/expected_boundary_pixel.json

ข้อเท็จจริงสำคัญ:

- ภาพใน example_01 เป็นเพียงภาพแคปบางส่วนจาก Orthomosaic ขนาดใหญ่
- ใช้เพื่อสอนหลักการ Include / Exclude / Uncertain เท่านั้น
- พิกัดใน expected_boundary_pixel.json เป็นพิกัด Pixel ของภาพแคป ไม่ใช่พิกัดจริง
- Input ที่ใช้งานจริงเป็น GeoTIFF, COG หรือ VRT ที่มี CRS และ Affine Transform

ห้าม:

- โหลด Orthomosaic ทั้งไฟล์เข้า RAM
- แปลง Orthomosaic ทั้งภาพเป็น PNG ความละเอียดเต็ม
- ใช้ขนาดภาพตัวอย่างกำหนดขนาดภาพจริง
- ใช้พิกัด Pixel ของภาพตัวอย่างเป็นพิกัดภูมิศาสตร์
- ใช้ขอบ Tile เป็น Candidate Boundary
- ใช้ PDD เป็นผลลัพธ์แทนการวิเคราะห์ภาพ
- ใช้ Convex Hull รวมพื้นที่ที่ถูกคลอง ถนน หรือคันดินแบ่งออก
- ตั้งสถานะ approved

ให้สร้าง Workflow ดังนี้:

Large georeferenced orthomosaic
→ inspect raster metadata
→ use overviews for whole-image understanding
→ optional AOI clip with buffer
→ generate 2048/4096 px overlapping windows
→ analyze planting evidence per tile
→ create candidate mask and confidence per tile
→ suppress tile-edge artifacts
→ transform tile results into source CRS
→ merge and dissolve connected evidence
→ preserve barriers, holes and concavities
→ geometry QA
→ create overview and detail previews
→ wait for Human Review
→ export GPKG, GeoJSON and KML

ข้อกำหนด Tile:

- Tile Size กำหนดผ่าน Config
- Overlap เริ่มต้น 15%
- ทุก Tile ต้องมี tile_id, raster window, window transform, CRS และ bounds
- รองรับ Resume และ Retry ราย Tile
- ผลลัพธ์ทุก Polygon ต้องสืบย้อนกลับไปยัง Source Raster และ Tile ได้

ก่อนเขียนระบบ ให้รายงานก่อนว่า:

1. Raster จะถูกอ่านแบบใด
2. จะเลือก Overview และ Detail Resolution อย่างไร
3. Tile Size และ Overlap เท่าไร พร้อมเหตุผล
4. จะป้องกัน Tile Edge Artifact อย่างไร
5. จะแปลง Pixel กลับเป็น Map Coordinate อย่างไร
6. จะรวม Polygon ข้าม Tile อย่างไร
7. จะรักษาคลอง ถนน คันดิน Hole และรอยเว้าอย่างไร
8. จะจัดการพื้นที่ปลูกหลายกลุ่มอย่างไร
9. จะ Resume หลังหยุดกลางทางอย่างไร
10. Output และ QA Evidence มีอะไรบ้าง

จากนั้นจึงลงมือสร้างระบบจริง

Acceptance Criteria:

- รันกับ Raster ที่ใหญ่กว่า RAM ได้
- ไม่สูญเสีย Georeferencing
- ไม่มีเส้นขอบ Tile ปลอม
- รองรับ MultiPolygon
- มี Overview Preview และ Detail Preview
- มี Uncertain Zone
- Geometry Valid
- คำนวณพื้นที่จาก Projected CRS
- มี Manifest และ Review Checklist
- Human Reviewer เป็นผู้อนุมัติเท่านั้น
```
