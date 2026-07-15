# AGENTS.md

## Purpose

ควบคุม Workflow วิเคราะห์แปลงปลูกเสริมและแปลงปลูกเต็มพื้นที่จาก Orthomosaic ขนาดใหญ่

ผลลัพธ์เป็น Candidate Evidence ต้องผ่าน Human Review ไม่ใช่ขอบเขตตามกฎหมาย

## Mandatory token routing

ห้ามอ่านทุก Skill พร้อมกัน

อ่านตามลำดับเท่านั้น:

1. `README.md`
2. `skills/README.md`
3. Orchestrator หรือ Skill ที่ผู้ใช้เรียก
4. Skill ก่อนหน้าเฉพาะ Output Contract เมื่อจำเป็น

เมื่อจบแต่ละขั้น ให้สรุป Output Path, Schema และ Warning แบบสั้น แล้วเริ่ม Session/Prompt ถัดไปด้วยไฟล์ผลลัพธ์แทนการส่งบริบทเดิมทั้งหมด

## Choose one workflow

### Enrichment planting

ใช้เมื่อปลูกแทรกในช่องว่างของป่าเดิม มีวัชพืช ต้นจาก เรือนยอดปิด และรูปแบบไม่เป็นกริดต่อเนื่อง

เริ่มที่:

`skills/00-enrichment-analysis-orchestrator/SKILL.md`

### Full-area planting

ใช้เมื่อต้นปลูกครอบคลุมพื้นที่กว้าง เห็นแถว กริด หรือระยะปลูกซ้ำ และช่องว่างส่วนใหญ่มีแนวโน้มเป็นต้นหายหรือต้นตาย

เริ่มที่:

`skills/10-full-area-planting-orchestrator/SKILL.md`

ห้ามผสมกฎของสอง Workflow โดยไม่มีเหตุผล

## Shared raster rules

- งานจริงต้องอ่าน GeoTIFF, COG หรือ VRT ที่มี CRS และ Affine Transform
- ห้ามโหลด Orthomosaic ขนาดใหญ่ทั้งหมดเข้า RAM
- ใช้ Overview และ Tile/Window พร้อม Overlap 10–20%
- บันทึก Tile ID, Window, Transform, Bounds และ Source Raster
- รวมผลใน CRS ต้นฉบับ ไม่รวมด้วย Local Pixel Coordinate
- ห้ามใช้ Tile Edge เป็นขอบเขตจริง

## Full-area pipeline

```text
Skill 01: inspect and tile
→ Skill 11: detect planted-tree candidates
→ Skill 12: infer rows and grid
→ Skill 13: analyze survival and mortality
→ Skill 14: delineate full-area boundary
→ Skill 15: QA and human review
```

กฎสำคัญ:

- แยกเงาและต้นเดิมขนาดใหญ่ออกจากต้นปลูก
- ตำแหน่งต้นหายต้องมี Grid รองรับ
- `not_observable` ไม่ใช่ต้นตาย
- พื้นที่ต้นรอดต่ำยังอยู่ในขอบเขตได้เมื่อ Grid ต่อเนื่อง
- สีพื้นน้ำหรือเลนไม่ใช่แนวแบ่งอัตโนมัติ

## Enrichment pipeline

```text
Skill 01: inspect and tile
→ Skill 02: land-cover classification
→ Skill 03: Rhizophora candidates
→ Skill 04: weed and obstruction
→ Skill 05: Nypa palm
→ Skill 06: existing canopy and occlusion
→ Skill 07: enrichment gaps
→ Skill 08: candidate boundary
→ Skill 09: QA and human review
```

กฎสำคัญ:

- ห้ามใช้สีเพียงอย่างเดียว
- แยกวัชพืช ต้นจาก ป่าเดิม น้ำ เลน เงา และเรือนยอดปิด
- `closed_canopy_unknown` ไม่ใช่ `not_planted`
- วิเคราะห์แบบ Gap-by-gap ไม่บังคับ Grid
- ภาพช่วงเวลาเดียวไม่ยืนยันว่าต้นเกิดจากการปลูก

## Required checkpoints

### Full-area

1. Tree Detection Preview
2. Grid and Missing-position Preview
3. Survival/Mortality Preview
4. Candidate Boundary Preview

### Enrichment

1. Classification Preview
2. Target vs Weed vs Nypa Preview
3. Gap Classification Preview
4. Candidate Boundary Preview

ห้ามข้าม Checkpoint โดยไม่แสดงผลหรือรายงานเหตุผล

## Status control

AI ใช้ได้สูงสุด:

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
