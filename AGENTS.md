# AGENTS.md

## Repository Purpose

Repository นี้ใช้พัฒนาและควบคุม Workflow วิเคราะห์แปลงปลูกเสริมจากภาพโดรนหรือ Orthomosaic ขนาดใหญ่

ผลลัพธ์เป็น Candidate Evidence จากภาพ ต้องผ่าน Human Review และไม่ใช่ขอบเขตตามกฎหมาย

## Start Here

ก่อนทำงานทุกครั้งให้อ่าน:

1. `README.md`
2. `skills/README.md`
3. Skill เฉพาะงานที่ถูกเรียก

งานครบทั้งกระบวนการให้เริ่มจาก:

`skills/00-enrichment-analysis-orchestrator/SKILL.md`

## Skill Separation Rule

ห้ามรวมหน้าที่ของ Skill โดยไม่จำเป็น:

- Skill 01 อ่าน Raster และแบ่ง Tile แต่ไม่จำแนกพืช
- Skill 02 จำแนกสิ่งปกคลุมแต่ไม่สร้างขอบเขต
- Skill 03 ตรวจโกงกาง Candidate แต่ไม่ยืนยันว่าเกิดจากการปลูก
- Skill 04 ตรวจวัชพืชและการบดบัง
- Skill 05 ตรวจต้นจาก
- Skill 06 ตรวจป่าเดิม เงา และเรือนยอดปิด
- Skill 07 วิเคราะห์ช่องว่างปลูกเสริม
- Skill 08 สร้าง Candidate Boundary จากหลักฐานที่มีแล้ว
- Skill 09 ตรวจ QA และเตรียม Human Review

หากต้องใช้หลาย Skill ให้เรียกผ่าน Skill 00 และรายงาน Intermediate Output ทุกขั้น

## Mandatory Interpretation

- ภาพใน `examples/` เป็น Crop จาก Orthomosaic ใช้สอนการตีความ ไม่ใช่พิกัดจริง
- งานจริงต้องอ่าน GeoTIFF, COG หรือ VRT ที่มี CRS และ Affine Transform
- ห้ามโหลด Orthomosaic ขนาดใหญ่ทั้งหมดเข้า RAM
- ต้องใช้ Overview และ Tile/Window ที่มี Overlap
- ห้ามใช้สีเพียงอย่างเดียว ต้องใช้ Color, Shape, Texture, Scale และ Context
- พุ่มสีเข้มอาจเป็นโกงกาง เงา หรือป่าเดิม ต้องแยกด้วยหลักฐานอื่น
- พื้นสีเขียวอ่อนละเอียดต่อเนื่องอาจเป็นวัชพืช แต่ต้องตรวจ Object Scale และ Texture
- ต้นจากมีใบยาวแผ่เป็นรัศมี ต้องแยกเป็น `nypa_palm`
- พื้นที่ใต้เรือนยอดปิดใช้ `closed_canopy_unknown` ห้ามใช้ `not_planted`
- ภาพช่วงเวลาเดียวไม่สามารถยืนยันได้ว่าต้นเกิดจากการปลูก

## Required Core Classes

```text
target_rhizophora_candidate
weed_groundcover
nypa_palm
existing_woody_canopy
natural_regeneration
dead_dry_vegetation
bare_mud
open_water
canal_or_tidal_channel
shadow_unknown
closed_canopy_unknown
unknown_object
```

## Required Gap Classes

```text
confirmed_enrichment_gap
probable_enrichment_gap
uncertain_enrichment_gap
natural_regeneration_gap
disturbed_open_area
closed_canopy_unknown_gap
excluded_water_gap
```

`confirmed_enrichment_gap` ต้องมี Supporting Evidence มากกว่าลักษณะภาพเพียงอย่างเดียว เช่น Ground Truth, Planting Point หรือภาพก่อน–หลัง

## Required Boundary Outputs

```text
enrichment_planting_core
enrichment_evidence_boundary
uncertain_enrichment_boundary
excluded_area
boundary_segment_confidence
```

## Full Pipeline

```text
Large georeferenced orthomosaic
→ Skill 01: inspect and tile
→ Skill 02: classify land cover
→ Skill 03: detect Rhizophora candidates
→ Skill 04: classify weed and obstruction
→ Skill 05: detect Nypa palm
→ Skill 06: classify existing canopy and occlusion
→ Skill 07: analyze enrichment gaps
→ Skill 08: delineate candidate boundaries
→ Skill 09: QA and human-review package
```

## Required Checkpoints

หยุดให้ผู้ใช้ตรวจอย่างน้อย:

1. Classification Preview
2. Target vs Weed vs Nypa Preview
3. Gap Classification Preview
4. Candidate Boundary and Warning Preview

ห้ามรันผ่าน Checkpoint โดยไม่แสดงผลหรือรายงานเหตุผลที่ไม่สามารถสร้างได้

## Large Raster Rules

- อ่าน Metadata ก่อนเสมอ
- Tile เริ่มต้น 2048 หรือ 4096 Pixel
- Overlap เริ่มต้น 15% และปรับได้ช่วง 10–20%
- บันทึก Tile ID, Window, Transform, Bounds และ Source Raster
- Prediction ใกล้ Tile Edge ต้องลดน้ำหนักหรือรวมกับ Tile ซ้อน
- รวมผลใน CRS ต้นฉบับ ไม่รวมด้วย Local Pixel Coordinate
- ห้ามสร้างเส้นตาม Tile Edge เป็นขอบเขตจริง

## Human Review

AI ใช้สถานะได้สูงสุด:

```text
draft
needs_human_review
rework
```

AI ห้ามตั้ง:

```text
reviewed
rejected
approved
```

ทุกผลลัพธ์ต้องระบุข้อจำกัด ข้อมูลที่ขาด และจุดที่ต้องตรวจภาคสนาม
