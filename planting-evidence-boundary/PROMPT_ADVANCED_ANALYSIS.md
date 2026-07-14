# Prompt — Advanced Planting Evidence Analysis

ใช้ข้อความด้านล่างกับ Codex หลังเปิด Repository นี้

```text
อ่าน AGENTS.md และเอกสารทั้งหมดตาม Required Reading Order ก่อน

งานนี้ต้องวิเคราะห์ Orthomosaic ขนาดใหญ่เพื่อสร้าง Candidate Planting Evidence Boundary

Input raster:
<ใส่ path ของ GeoTIFF / COG / VRT>

Optional AOI:
<ใส่ path หรือ NONE>

Project ID:
<ใส่รหัสโครงการ>

Plot Code:
<ใส่รหัสแปลง>

ข้อบังคับ:

1. ห้ามโหลด Orthomosaic ทั้งไฟล์เข้า RAM
2. อ่าน Metadata, CRS, Transform, Pixel Size, Bounds, NoData และ Overviews ก่อน
3. ใช้ Overview ค้นหา Candidate Region และใช้ Tile/Window แบบมี Overlap สำหรับรายละเอียด
4. ห้ามใช้สีเขียวหรือ Color Threshold ค่าเดียวตัดสินพื้นที่ปลูก
5. ตรวจต้นที่พบ แนวแถว ทิศทาง ระยะต้น ระยะแถว ความสม่ำเสมอ และ Missing Positions
6. แยก Natural Vegetation, Open Water, Canal, Road/Dike และ Unknown Object
7. คลอง ถนน และคันดินเป็น Candidate Barrier ไม่ใช่ขอบเขตอัตโนมัติ
8. สร้างผลแยกเป็น planting_core, planting_evidence_boundary และ uncertain_boundary
9. ช่องว่างที่สัมพันธ์กับกริดให้พิจารณาเป็น mortality_gap ห้ามตัดออกอัตโนมัติ
10. แบ่งขอบเขตเป็น Segment พร้อม Confidence และเหตุผลรายช่วง
11. ตรวจ Large Empty Area, Under-Boundary, Natural Vegetation Inclusion, Boundary Too Far, Tile Edge Artifact และ Barrier Mismatch
12. สร้าง Debug Layers และ Preview ตาม QA_DEBUG_OUTPUTS.md
13. ผลทั้งหมดต้องอยู่ใน CRS ของ Source Raster
14. ห้ามตั้งสถานะ approved
15. ห้ามแก้หรือเขียนทับ Source Raster

ก่อนเขียนระบบหรือรันเต็ม ให้ตอบก่อนว่า:

A. เข้าใจความแตกต่างระหว่าง surviving_tree, missing_tree_position, planting_row, planting_grid และ natural_vegetation อย่างไร
B. จะสร้าง planting_core กับ planting_evidence_boundary ต่างกันอย่างไร
C. จะตรวจ mortality_gap อย่างไร
D. จะป้องกันการรวมแปลงข้างเคียงและการข้ามคลองอย่างไร
E. จะลด Tile Edge Artifact อย่างไร
F. จะใช้ Metrics ใดตรวจว่า Polygon กว้างหรือแคบเกินไป

จากนั้นดำเนินงานเป็น Phase:

Phase 1 — Environment and raster inspection
Phase 2 — Overview and candidate region detection
Phase 3 — Tile generation with provenance
Phase 4 — Tree, row, spacing and missing-position analysis
Phase 5 — Natural vegetation and barrier classification
Phase 6 — Core, evidence envelope and uncertain boundary creation
Phase 7 — Merge in source CRS and boundary refinement
Phase 8 — Segment confidence, geometry QA and warnings
Phase 9 — Debug layers and previews
Phase 10 — Human review package and export

ทุก Phase ต้องบันทึก:

- สิ่งที่ทำ
- Input
- Output
- Warning
- สิ่งที่ยังไม่แน่ใจ
- ไฟล์ที่สร้าง

หากหลักฐานจากภาพไม่เพียงพอ ให้หยุดที่ needs_review หรือ field_check_required ห้ามสร้างผลให้ดูสมบูรณ์โดยการเดา
```
