# train-plant

ชุดตัวอย่างและ Skill สำหรับช่วยเสนอขอบเขตพื้นที่ที่พบร่องรอยการปลูกจากภาพโดรนหรือ Orthomosaic ขนาดใหญ่

> ผลลัพธ์เป็น Candidate Boundary จากหลักฐานในภาพ ต้องผ่าน Human Review ก่อนนำไปใช้งานจริง และไม่ใช่ขอบเขตทางกฎหมาย

## สิ่งที่ระบบนี้ออกแบบให้ทำได้

- อ่าน Orthomosaic, GeoTIFF, COG หรือ VRT ขนาดใหญ่แบบ Tile/Window
- ตรวจต้นที่ยังมองเห็นและกลุ่มต้นปลูก
- วิเคราะห์แนวแถว ทิศทางแถว ระยะต้น และระยะแถว
- คาดตำแหน่งที่เคยปลูกแต่ไม่พบต้น เพื่อไม่ให้ขอบเขตหดตามอัตรารอด
- แยกพื้นที่ปลูกออกจากพืชธรรมชาติ น้ำเปิด คลอง ถนน และคันดิน
- สร้าง `planting_core` สำหรับพื้นที่มั่นใจสูง
- สร้าง `planting_evidence_boundary` สำหรับขอบเขตจากหลักฐานรวม
- สร้าง `uncertain_boundary` สำหรับส่วนที่ต้องให้คนตรวจ
- ให้ Confidence แยกรายช่วงแนวขอบ
- ตรวจปัญหา Polygon กว้างเกิน แคบเกิน ข้ามคลอง รวมแปลงข้างเคียง และเกาะขอบ Tile
- สร้าง Debug Layers เพื่อย้อนดูว่าวงขอบเขตจากหลักฐานใด
- ส่งออก GPKG, GeoJSON, KML, Preview, Manifest และ QA Report

## ข้อเท็จจริงสำคัญ

ภาพใน `examples/example_01` เป็นเพียงภาพแคปบางส่วนจาก Orthomosaic โดรนขนาดใหญ่ ใช้สอนการตีความ Include / Exclude / Uncertain เท่านั้น

ระบบจริงต้องประมวลผล GeoTIFF, COG หรือ VRT แบบ Overview + Tile/Window และคืน Polygon ใน CRS ของภาพต้นฉบับ ห้ามใช้ Pixel Coordinate จากภาพแคปเป็นพิกัดจริง

ระบบห้ามสร้างขอบเขตจากสีเขียวหรือต้นที่ยังรอดเพียงอย่างเดียว ต้องพิจารณาแนวแถว ระยะปลูก ช่องว่างต้นตาย พืชธรรมชาติ และ Candidate Barrier ร่วมกัน

## โครงสร้าง

```text
AGENTS.md
planting-evidence-boundary/
├─ SKILL.md
├─ SKILL_ADDENDUM_ADVANCED.md
├─ LARGE_ORTHOMOSAIC_WORKFLOW.md
├─ QA_DEBUG_OUTPUTS.md
├─ PROMPT.md
├─ PROMPT_LARGE_ORTHOMOSAIC.md
├─ PROMPT_ADVANCED_ANALYSIS.md
├─ config/
│  └─ defaults.yaml
├─ schemas/
│  └─ analysis_manifest.schema.json
└─ examples/
   ├─ example_crop_metadata.template.json
   ├─ negative_examples/
   │  └─ README.md
   └─ example_01/
      ├─ input.jpg
      ├─ expected_overlay.jpg
      ├─ expected_boundary_pixel.json
      ├─ example_manifest.json
      └─ notes.md
```

## วิธีเริ่มใช้

เปิด Repository ด้วย Codex แล้วให้ทำตาม `AGENTS.md`

Prompt พร้อมรันสำหรับระบบขั้นสูง:

`planting-evidence-boundary/PROMPT_ADVANCED_ANALYSIS.md`

สำหรับ Workflow Orthomosaic ขนาดใหญ่แบบพื้นฐาน:

`planting-evidence-boundary/PROMPT_LARGE_ORTHOMOSAIC.md`

## แนวทางประมวลผลจริง

```text
Large georeferenced orthomosaic
→ read metadata and overviews
→ optional AOI clipping
→ overlapping tile/window processing
→ detect trees and planting patterns
→ infer rows, spacing and missing positions
→ classify natural vegetation and barriers
→ create planting core
→ create evidence envelope
→ create uncertain boundary
→ tile-edge suppression
→ transform results to source CRS
→ merge/dissolve polygons
→ segment confidence and QA metrics
→ overview/detail/debug previews
→ human review
→ export GPKG/GeoJSON/KML
```

## ผลลัพธ์หลัก

```text
planting_core.gpkg
planting_evidence_boundary.gpkg
uncertain_boundary.gpkg
segment_confidence.gpkg
analysis_manifest.json
qa_report.json
overview_boundary_preview.png
core_vs_envelope_preview.png
evidence_preview.png
warning_preview.png
```

เมื่อเปิด Debug Mode จะสามารถส่งออก Detected Trees, Missing Positions, Planting Rows, Planting Grid, Candidate Barriers, Natural Vegetation, Density Raster และ Confidence Raster ได้

## หมายเหตุเรื่องไฟล์ตัวอย่าง

ไฟล์ภาพตัวอย่างควรเป็น Crop ขนาดพอให้เห็นร่องรอยการปลูกชัด ไม่จำเป็นต้องนำ Orthomosaic ขนาดใหญ่มาเก็บใน Git โดยตรง

ไฟล์ Orthomosaic จริงควรเก็บไว้นอก Repository หรือใช้ Git LFS/Object Storage เมื่อจำเป็น เนื่องจากไฟล์อาจมีขนาดหลาย GB
