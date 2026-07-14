# AGENTS.md

## Repository Purpose

Repository นี้ใช้พัฒนา Workflow สำหรับเสนอขอบเขตพื้นที่ที่พบร่องรอยการปลูกจากภาพโดรนหรือ Orthomosaic ขนาดใหญ่

## Mandatory Interpretation

- ภาพใน `examples/` เป็นเพียงภาพแคปบางส่วนจาก Orthomosaic ขนาดใหญ่ เพื่อสอนหลักการตีความ
- ห้ามถือว่าขนาดภาพตัวอย่างคือขนาดข้อมูลจริง
- ห้ามใช้พิกัด Pixel ของภาพตัวอย่างเป็นพิกัดภูมิศาสตร์
- งานจริงต้องอ่าน GeoTIFF, COG, VRT หรือ Raster ที่มี GeoTransform และ CRS
- ห้ามโหลด Orthomosaic ทั้งไฟล์เข้า RAM เมื่อไฟล์มีขนาดใหญ่
- ต้องประมวลผลแบบ Window/Tile และแปลงผลกลับเป็นพิกัดจริงของ Raster

## Required Reading Order

ก่อนเขียน Code หรือวิเคราะห์ภาพใหม่ ให้อ่านตามลำดับ:

1. `planting-evidence-boundary/SKILL.md`
2. `planting-evidence-boundary/LARGE_ORTHOMOSAIC_WORKFLOW.md`
3. `planting-evidence-boundary/examples/example_01/notes.md`
4. `planting-evidence-boundary/examples/example_01/example_manifest.json`
5. `planting-evidence-boundary/examples/example_01/expected_boundary_pixel.json`

## Large Raster Rules

1. อ่าน Metadata ก่อนเสมอ: width, height, band count, dtype, nodata, CRS, transform, pixel size และ overviews
2. ใช้ Rasterio Window, GDAL block processing หรือ COG range reading
3. ใช้ภาพ Overview สำหรับค้นหา Candidate Area ระดับภาพรวม
4. ใช้ Detail Tile สำหรับตรวจร่องรอยต้นปลูกและแนวขอบ
5. Tile แนะนำ 2048 หรือ 4096 Pixel พร้อม Overlap 10–20%
6. Prediction ใกล้ขอบ Tile ต้องถูกลดน้ำหนักหรือตัดออกก่อนรวมผล
7. รวม Mask/Polygon ใน Map CRS ไม่ใช่ต่อภาพด้วย Pixel จาก Tile คนละตำแหน่ง
8. ใช้ Tile ID, Window, Affine Transform และ Source Raster เป็น Provenance ทุกครั้ง
9. หากพื้นที่ปลูกไม่ต่อกัน ให้สร้าง MultiPolygon หรือหลาย Feature ห้ามบังคับรวมเป็น Polygon เดียว
10. PDD ใช้เป็น AOI หรือข้อมูลเปรียบเทียบได้ แต่ห้ามใช้แทนหลักฐานจากภาพ

## Required Pipeline

```text
Large georeferenced orthomosaic
→ read metadata and overviews
→ optional AOI clipping
→ generate overlapping tiles/windows
→ analyze planting evidence per tile
→ create tile masks and confidence
→ suppress tile-edge artifacts
→ transform results to map coordinates
→ merge and dissolve connected evidence
→ remove water/road/empty-area exclusions
→ geometry QA
→ overview and detail previews
→ human review
→ export GPKG/GeoJSON/KML
```

## Human Review

AI สร้างได้เฉพาะ Candidate Boundary

ห้ามตั้งสถานะ `approved`

ผลลัพธ์ต้องระบุชัดว่าเป็น `planting_evidence_boundary` และไม่ใช่ Legal Boundary
