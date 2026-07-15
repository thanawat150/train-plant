# AGENTS.md

## Purpose

ควบคุม Workflow วิเคราะห์แปลงปลูกเสริมและแปลงปลูกเต็มพื้นที่จาก Orthomosaic ขนาดใหญ่ รวมถึงการแปลงผลวิเคราะห์เป็นแผนเข้าตรวจภาคสนามทางบกและทางเรือ

ผลลัพธ์เป็น Candidate Evidence, Candidate Route และ Candidate Mission ต้องผ่าน Human Review ไม่ใช่ขอบเขตตามกฎหมาย ไม่ใช่ผลตรวจดินทางห้องปฏิบัติการ และไม่ใช่การรับรองความปลอดภัยของเส้นทาง

## Mandatory token routing

ห้ามอ่านทุก Skill พร้อมกัน

อ่านตามลำดับเท่านั้น:

1. `README.md`
2. `skills/README.md`
3. Orchestrator หรือ Skill ที่ผู้ใช้เรียก
4. Skill ก่อนหน้าเฉพาะ Output Contract เมื่อจำเป็น

เมื่อจบแต่ละขั้น ให้สรุป Output Path, Schema และ Warning แบบสั้น แล้วเริ่มขั้นถัดไปด้วยไฟล์ผลลัพธ์แทนการส่งบริบทเดิมทั้งหมด

## Choose one analysis workflow

### Enrichment planting

ใช้เมื่อปลูกแทรกในช่องว่างของป่าเดิม มีวัชพืช ต้นจาก เรือนยอดปิด และรูปแบบไม่เป็นกริดต่อเนื่อง

เริ่มที่:

`skills/00-enrichment-analysis-orchestrator/SKILL.md`

### Full-area planting

ใช้เมื่อต้นปลูกครอบคลุมพื้นที่กว้าง เห็นแถว กริด หรือระยะปลูกซ้ำ และช่องว่างส่วนใหญ่มีแนวโน้มเป็นต้นหายหรือต้นตาย

เริ่มที่:

`skills/10-full-area-planting-orchestrator/SKILL.md`

ห้ามผสมกฎของสอง Workflow โดยไม่มีเหตุผล

## Optional shared skills

### Skill 16 — Surface/Hydrology

เรียกเมื่อภาพมีดิน เลน น้ำขัง แอ่งน้ำ ร่องน้ำ หรือ Surface Zone ต่างกันชัด

- ใช้ Class ที่มองเห็นจากภาพเท่านั้น
- `possible_salt_crust` ไม่ใช่ผลยืนยันดินเค็ม
- ห้ามสรุป pH, EC, Acid Sulfate Soil หรือความเหมาะสมปลูกจากสีภาพ

### Skill 17 — Palm/Coconut

เรียกเมื่อตรวจพบต้นเดี่ยวทรงดาวหรือรัศมี มี Crown Center ชัด หรืออาจเห็นลำต้น/เงาลำต้น

- ต้นจากมักเป็นกอหรือผืน ใบหลายกอซ้อน ศูนย์กลางไม่ชัด และไม่มีลำต้นเด่น
- ห้ามเรียกทุก Crown แบบรัศมีว่า `nypa_palm`
- เมื่อแยกชนิดไม่ได้ให้ใช้ `palm_unknown`

### Skill 18 — Inspection Priority

เรียกเมื่อมีผลวิเคราะห์แล้วและต้องการเลือกจุดที่ควรเข้าตรวจ

- ใช้ความไม่แน่นอน ความเสียหาย ผลกระทบ การเปลี่ยนแปลง และความเป็นตัวแทน
- ต้องมีทั้งจุดปัญหาและ High-confidence QA Control
- ห้ามเลือกเฉพาะจุดใกล้ทางเข้า

### Skill 19 — Land/Boat Access

เรียกเมื่อมี Candidate Inspection Points และต้องวางเส้นทางเข้าทางบก/ทางเรือ

- ใช้ Network Distance และเวลาเดินทาง ไม่ใช้ระยะเส้นตรง
- แยก Land, Boat, Walk และ Transfer Cost
- Candidate Route จากภาพต้องให้ทีมพื้นที่ยืนยัน

### Skill 20 — Field Mission

เรียกเมื่อมี Priority และ Access Assessment แล้ว

- รวมจุดเป็น Mission, วัน, ทีม และ Checklist
- จุดสำคัญแต่ไกลต้องเป็น `special_mission` หรือ `remote_cluster_mission`
- ห้ามตัดจุด P1 ทิ้งเพียงเพราะไกล

Optional Skill ต้องไม่ถูกอ่านเมื่อไม่มี Trigger

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

Optional Analysis Context:

```text
Skill 16: surface/hydrology context
Skill 17: palm/coconut exclusion
```

Optional Field Planning:

```text
Skill 18: inspection priority
→ Skill 19: land/boat access routes
→ Skill 20: missions and day plan
```

กฎสำคัญ:

- แยกเงา ต้นเดิม และปาล์ม/มะพร้าวออกจากต้นปลูก
- ตำแหน่งต้นหายต้องมี Grid รองรับ
- `not_observable` ไม่ใช่ต้นตาย
- พื้นที่ต้นรอดต่ำยังอยู่ในขอบเขตได้เมื่อ Grid ต่อเนื่อง
- สีพื้นน้ำหรือเลนไม่ใช่แนวแบ่งอัตโนมัติ
- Surface Class ใช้เปรียบเทียบกับอัตรารอดได้ แต่ห้ามสรุปสาเหตุโดยอัตโนมัติ

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

Optional routing:

```text
หลัง Skill 02 → Skill 16 เมื่อมี Surface/Hydrology Trigger
หลัง Skill 05 → Skill 17 เมื่อมี Single Radial Crown Trigger
หลัง Skill 09 → Skill 18 → 19 → 20 เมื่อขอ Field Plan
```

กฎสำคัญ:

- ห้ามใช้สีเพียงอย่างเดียว
- แยกวัชพืช ต้นจาก ปาล์ม/มะพร้าว ป่าเดิม น้ำ เลน เงา และเรือนยอดปิด
- `closed_canopy_unknown` ไม่ใช่ `not_planted`
- วิเคราะห์แบบ Gap-by-gap ไม่บังคับ Grid
- ภาพช่วงเวลาเดียวไม่ยืนยันว่าต้นเกิดจากการปลูก
- Surface Class ไม่ใช่ผลตรวจดิน

## Field inspection decision rules

ต้องรักษาสองคะแนนแยกกัน:

```text
evidence_priority_score
access_burden_score
```

- Evidence Priority ตอบว่า “จำเป็นต้องตรวจหรือไม่”
- Access Burden ตอบว่า “ต้องใช้ทรัพยากรมากเท่าไร”
- ระยะไกลไม่ใช่เหตุผลลบจุดสำคัญ
- จุดไกลหลายจุดให้พิจารณารวม Mission
- จุดไกลความสำคัญต่ำสามารถเลื่อน แต่ต้องอยู่ใน Backlog
- ต้องมีจุดตัวแทนทุก Stratum สำคัญและจุด QA ในพื้นที่ Confidence สูง

## Access rules

- ใช้เส้นทางบก ทางเดิน คันดิน คลอง ทางเรือ ท่าเรือ จุดขึ้นฝั่ง และจุด Transfer ที่มี Source ชัดเจน
- Route จาก Orthomosaic ใช้ได้สูงสุดเป็น Candidate
- ห้ามถือคลองทุกเส้นว่าเรือผ่านได้
- ห้ามถือคันดินทุกเส้นว่าเดินหรือขับรถได้
- หากไม่มี Network ให้ใช้ `access_network_missing` ไม่สร้างเส้นตรงเป็น Route
- ต้องรวมเวลาไป–กลับ, Off-network Walk, Mode Transfer, Tide Wait, Permission และ Safety Buffer
- AI ห้ามรับรองความปลอดภัยของ Route

## Required checkpoints

### Full-area

1. Tree Detection Preview
2. Grid and Missing-position Preview
3. Survival/Mortality Preview
4. Optional Surface/Palm Preview
5. Candidate Boundary Preview

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

ทุกผลลัพธ์ต้องระบุข้อจำกัด ข้อมูลที่ขาด จุดที่ต้องตรวจภาคสนาม และ Route ที่ยังไม่ยืนยัน
