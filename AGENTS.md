# AGENTS.md

## Purpose

ควบคุม Workflow วิเคราะห์แปลงปลูกเสริมและแปลงปลูกเต็มพื้นที่จาก Orthomosaic ขนาดใหญ่ รวมถึงการแปลงผลเป็นแผนเข้าตรวจภาคสนามทางบกและทางเรือ

ผลลัพธ์เป็น Candidate Evidence, Candidate Count, Candidate Route และ Candidate Mission ต้องผ่าน Human Review

## Mandatory token routing

ห้ามอ่านทุก Skill พร้อมกัน

อ่านตามลำดับเท่านั้น:

1. `README.md`
2. `skills/README.md`
3. Orchestrator หรือ Skill ที่ผู้ใช้เรียก
4. Skill ก่อนหน้าเฉพาะ Output Contract เมื่อจำเป็น

เมื่อจบแต่ละขั้น ส่งต่อเพียง Output Path, Schema, Metrics และ Warnings ไม่ส่งบริบทเดิมซ้ำทั้งหมด

## Choose one analysis workflow

### Enrichment planting

ใช้เมื่อปลูกแทรกตามช่องว่างป่าเดิม รูปแบบไม่เป็นกริดต่อเนื่อง

เริ่มที่:

`skills/00-enrichment-analysis-orchestrator/SKILL.md`

### Full-area planting

ใช้เมื่อต้นปลูกครอบคลุมพื้นที่กว้าง เห็นแถว กริด ระยะปลูกซ้ำ หรือ Planting Block ในนากุ้ง

เริ่มที่:

`skills/10-full-area-planting-orchestrator/SKILL.md`

ห้ามผสมกฎสอง Workflow โดยไม่มีเหตุผล

## Shared raster rules

- งานจริงต้องอ่าน GeoTIFF, COG หรือ VRT ที่มี CRS และ Affine Transform
- ห้ามโหลด Orthomosaic ขนาดใหญ่ทั้งหมดเข้า RAM
- ใช้ Overview และ Tile/Window พร้อม Overlap 10–20%
- บันทึก Tile ID, Window, Transform, Bounds และ Source Raster
- รวมผลใน CRS ต้นฉบับ ไม่รวมด้วย Local Pixel Coordinate
- ห้ามใช้ Tile Edge เป็นขอบเขตจริง

## Full-area mandatory pipeline

```text
Skill 01: inspect and tile
→ Skill 11: raw crown/tree detection
→ Skill 12: preliminary rows, spacing and planting-pattern blocks
→ Skill 21: spacing-guided tree count validation
→ Skill 13: survival and mortality
→ Skill 14: full-area boundary
→ Skill 15: QA and human review
```

ห้ามข้าม Skill 21 และห้ามใช้ Raw Detection จาก Skill 11 คำนวณอัตรารอด

## Full-area interpretation rules

### Raw detection

- จุดแดงหรือ Detection Point เป็น Raw Candidate ไม่ใช่จำนวนต้นสุดท้าย
- Candidate ต้องมีเรือนยอดจริงรองรับ
- จุดบนดิน น้ำเปล่า ถนน คันดิน วัชพืช หรือ Shadow-only ต้องถูกตรวจเป็น False Positive

### Water context

- ห้ามใช้ Water Mask ตัดต้นทั้งหมด เพราะต้นป่าชายเลนสามารถขึ้นในน้ำตื้นได้
- จุดบนผิวน้ำที่ไม่มีพุ่ม = `false_positive_on_water`
- จุดในบริบทน้ำที่มีเรือนยอดจริง = Valid Candidate

### Planting pattern

- ต้นปลูกต้องมีทั้ง Canopy Evidence และ Pattern/Spacing Support
- Pattern ที่รองรับ: Row/Grid, Staggered Grid, Locally Regular Full-area Block และ Shrimp-pond Planting Block
- จุดฟุ้งที่ไม่มีแนว ไม่มี Local Spacing Consistency และไม่มีพุ่มจริง = Detection Failure
- รองรับหลาย Pattern Block เมื่อแนวหรือระยะเปลี่ยน

### Touching/merged canopy

- ห้ามใช้ `1 canopy blob = 1 tree`
- เรือนยอดชิดกันให้นับจาก Crown Center, Reference Spacing, แถว/กริด และ Planned Point
- Spacing เป็นโครงสร้างสนับสนุน แต่ห้ามสร้าง Surviving Tree เมื่อไม่มี Canopy Evidence
- กลุ่มที่ยังแยกไม่ได้ต้องรายงาน Min–Max หรือ `unresolved_merged_canopy`

### Existing large trees

- ต้นเดิมขนาดใหญ่ไม่รวมเป็นต้นปลูก
- ไม่ใช้ Fit Grid หรือ Reference Spacing
- จุดปลูกใต้พุ่มใหญ่ = `not_observable_under_existing_tree`
- ห้ามเรียกจุดใต้พุ่มใหญ่ว่าต้นตายโดยอัตโนมัติ

### Survival

- Skill 13 ใช้ `validated_planted_tree_points.gpkg` และ `planting_point_status.gpkg` จาก Skill 21
- แยก Confirmed, Spacing-supported, Probable, Missing และ Not Observable
- `not_observable` ไม่อยู่ในตัวหารหลัก
- รายงาน Confirmed Rate และ Estimated Range แยกกัน

## Optional analysis context

### Skill 16 — Surface/Hydrology

เรียกเมื่อมีดิน เลน น้ำขัง แอ่งน้ำ ร่องน้ำ หรือ Surface Zone ต่างกันชัด

- ใช้ Class ที่มองเห็นจากภาพเท่านั้น
- `possible_salt_crust` ไม่ใช่ผลยืนยันดินเค็ม
- ห้ามสรุป pH, EC หรือ Acid Sulfate Soil จากสีภาพ

### Skill 17 — Palm/Coconut

เรียกเมื่อตรวจพบต้นเดี่ยวทรงดาวหรือรัศมีที่อาจปะปนกับต้นปลูก

- ต้นจากมักเป็นกอ/ผืน ศูนย์กลางไม่ชัด
- ห้ามเรียกทุก Crown แบบรัศมีว่า Nypa

Optional Skill ต้องไม่ถูกอ่านเมื่อไม่มี Trigger

## Enrichment pipeline

```text
Skill 01: inspect and tile
→ Skill 02: land-cover classification
→ Skill 03: Rhizophora candidates
→ Skill 04: weed and obstruction
→ Skill 05: Nypa patch detection
→ Skill 06: existing canopy and occlusion
→ Skill 07: enrichment gaps
→ Skill 08: candidate boundary
→ Skill 09: QA and human review
```

Optional:

```text
Skill 16: Surface/Hydrology
Skill 17: Palm/Coconut
หลัง Skill 09 → Skill 18 → 19 → 20 เมื่อขอ Field Plan
```

กฎปลูกเสริม:

- วิเคราะห์แบบ Gap-by-gap ไม่บังคับ Grid
- แยกวัชพืช ต้นจาก ปาล์ม/มะพร้าว ป่าเดิม น้ำ เลน เงา และเรือนยอดปิด
- `closed_canopy_unknown` ไม่ใช่ `not_planted`
- ภาพช่วงเวลาเดียวไม่ยืนยันว่าต้นเกิดจากการปลูก

## Optional field planning

```text
Skill 18: inspection priority
→ Skill 19: land/boat access routes
→ Skill 20: missions and day plan
```

### Decision rules

รักษาสองคะแนนแยกกัน:

```text
evidence_priority_score
access_burden_score
```

- Evidence Priority ตอบว่า “จำเป็นต้องตรวจหรือไม่”
- Access Burden ตอบว่า “ต้องใช้ทรัพยากรมากเท่าไร”
- ระยะไกลไม่ใช่เหตุผลลบจุดสำคัญ
- จุดไกลหลายจุดให้รวม Mission เมื่อคุ้มค่า
- ต้องมีจุดตัวแทนทุก Stratum และ High-confidence QA Control

### Access rules

- ใช้เส้นทางบก ทางเดิน คันดิน คลอง ทางเรือ ท่าเรือ จุดขึ้นฝั่ง และ Transfer Point ที่มี Source
- Route จาก Orthomosaic ใช้ได้สูงสุดเป็น Candidate
- ห้ามถือคลองทุกเส้นว่าเรือผ่านได้
- ห้ามถือคันดินทุกเส้นว่าเดินหรือขับรถได้
- หากไม่มี Network ให้ใช้ `access_network_missing`
- ต้องรวมเวลาไป–กลับ Off-network Walk, Transfer, Tide Wait, Permission และ Safety Buffer
- AI ห้ามรับรองความปลอดภัยของ Route

## Required checkpoints

### Full-area

1. Raw Detection Preview
2. Reference Spacing และ Pattern Block Preview
3. Skill 21 Exclusion / False-positive Preview
4. Merged-canopy Center Preview
5. Final Count Status Preview
6. Survival/Mortality Preview
7. Candidate Boundary Preview

### Enrichment

1. Classification Preview
2. Optional Surface Preview
3. Target vs Weed vs Nypa vs Other Palm Preview
4. Gap Classification Preview
5. Candidate Boundary Preview

### Field Planning

1. Candidate Inspection Points พร้อมเหตุผล
2. Land/Boat Route Options
3. Mission Grouping และจุดที่เลื่อน
4. Day Plan และ Field Checklist

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

ทุกผลลัพธ์ต้องระบุข้อจำกัด ข้อมูลที่ขาด จุดต้องตรวจภาคสนาม และ Version ของ Skill/Config
