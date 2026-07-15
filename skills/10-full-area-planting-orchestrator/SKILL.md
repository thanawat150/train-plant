---
name: full-area-planting-orchestrator
description: ควบคุม Workflow วิเคราะห์แปลงปลูกเต็มพื้นที่ ตั้งแต่ Raw Detection หาแนว/ระยะ ตรวจนับด้วย Canopy+Spacing วิเคราะห์อัตรารอด วงขอบเขต และต่อยอดเป็นแผนเข้าตรวจภาคสนาม
---

# Full-area Planting Orchestrator

## ใช้เมื่อ

- ต้นปลูกครอบคลุมพื้นที่กว้าง
- เห็นแถว กริด ระยะปลูกซ้ำ หรือ Planting Block ภายในนากุ้ง
- บางบริเวณเรือนยอดชิดกันแต่ระยะศูนย์กลางยังใกล้เคียงกัน
- ต้องการนับต้น หาต้นหาย วิเคราะห์อัตรารอด วงขอบเขต หรือวางแผนเข้าตรวจ

ห้ามใช้กับพื้นที่ปลูกแทรกตามช่องว่างป่าเดิม ให้ใช้ Skill 00–09 แทน

## Token-efficient reading

อ่านเฉพาะ:

1. `AGENTS.md`
2. `skills/README.md`
3. Skill นี้
4. Skill ขั้นตอนที่กำลังรันเพียงตัวเดียว
5. Skill ก่อนหน้าเฉพาะ Output Contract

ห้ามอ่านทุก Skill พร้อมกัน

## Core Pipeline

```text
Skill 01: inspect and tile large raster
→ Skill 11: create raw crown/tree candidates
→ Skill 12: infer preliminary rows, spacing and planting-pattern blocks
→ Skill 21: validate count using canopy + spacing + planned points
→ Skill 13: analyze survival and mortality
→ Skill 14: delineate full-area boundary
→ Skill 15: QA and human review
```

ห้ามส่ง Raw Detection จาก Skill 11 ไปคำนวณอัตรารอดโดยข้าม Skill 21

## Core interpretation rules

- จุดแดงจาก Detector เป็น Raw Candidate ไม่ใช่จำนวนต้นสุดท้าย
- ต้นปลูกต้องมี Canopy Evidence และ Pattern/Spacing Support
- Grid ใช้ช่วยตรวจและหาตำแหน่งหาย ห้ามสร้างต้นที่ไม่มีเรือนยอด
- น้ำไม่ใช่ Exclusion ทั้งหมด ต้นในน้ำตื้นนับได้เมื่อเห็นพุ่มจริง
- จุดบนผิวน้ำที่ไม่มีพุ่มเป็น False Positive
- จุดบนดินโล่ง ถนน คันดิน วัชพืช หรือ Shadow-only ต้องถูกตรวจและตัดออก
- เรือนยอดชิดกันให้นับจากศูนย์กลาง ระยะปลูก แนวแถว และจุดปลูก ไม่ใช้ `1 blob = 1 tree`
- ต้นเดิมขนาดใหญ่ไม่นับ ไม่ใช้ Fit Grid และจุดใต้พุ่มใหญ่เป็น Not Observable
- แบ่งหลาย Pattern Block เมื่อแนวหรือระยะเปลี่ยน
- จุดฟุ้งที่ไม่มี Canopy และไม่มี Local Spacing Consistency เป็น Detection Failure

## Optional Context

- Skill 16 เมื่อพื้นที่มีดิน เลน น้ำขัง ร่องน้ำ หรือ Surface Zone ต่างกันชัด
- Skill 17 เมื่อมีต้นเดี่ยวทรงรัศมีที่อาจปะปนกับต้นปลูก
- Optional Context ใช้เปรียบเทียบหรือ Exclusion ไม่ใช้แทน Canopy/Spacing Evidence

## Optional Field Inspection Pipeline

```text
หลัง Skill 15
→ Skill 18: prioritize inspection points
→ Skill 19: assess land/boat access
→ Skill 20: group missions, teams, days and checklist
```

## Checkpoints

หยุดให้ตรวจหลัง:

1. Raw Crown Detection Preview
2. Reference Spacing และ Planting Pattern Blocks
3. Skill 21 False-positive / Large-tree / Merged-canopy Validation
4. Final Confirmed / Probable / Missing / Not-observable Count
5. Survival/Mortality Preview
6. Optional Surface/Palm Preview
7. Candidate Boundary Preview
8. Field Planning Checkpoints เมื่อเรียก Skill 18–20

## Required Outputs

```text
planted_tree_candidates.gpkg
planting_rows.gpkg
planting_grid.gpkg
grid_blocks.gpkg
validated_planted_tree_points.gpkg
planting_point_status.gpkg
false_positive_detections.gpkg
existing_large_trees_validated.gpkg
merged_canopy_clusters.gpkg
spacing_model.json
survival_mortality_zones.gpkg
full_area_planting_core.gpkg
full_area_planting_evidence_boundary.gpkg
full_area_uncertain_edge.gpkg
qa_report.json
review_package/
```

Optional Context:

```text
surface_condition.tif
hydrology_features.gpkg
palm_coconut_candidates.gpkg
```

Optional Field Planning:

```text
inspection_candidate_points.gpkg
inspection_priority_zones.gpkg
inspection_route_options.gpkg
selected_inspection_points.gpkg
field_missions.gpkg
mission_routes.gpkg
mission_day_plan.csv
field_checklist.csv
```

## Failure conditions

หยุดก่อนคำนวณอัตรารอดเมื่อ:

- จุด Raw Detection บนดิน/น้ำ/ถนนจำนวนมาก
- Crown Support Ratio ต่ำ
- จุดกระจายฟุ้งโดยไม่มี Pattern Block
- Grid-only Detection จำนวนมาก
- ต้นใหญ่เดิมถูกนับหรือใช้ Fit Grid
- Merged Canopy ถูกบังคับเป็นจำนวนเดียวโดยไม่มีหลักฐาน
- Skill 21 รายงาน `tree_count_validation_failed`

## Restrictions

- ห้ามถือ Raw Detection เป็นจำนวนต้น
- ห้ามให้ต้นเดิมหรือปาล์ม/มะพร้าวขนาดใหญ่บิดระยะกริด
- ห้ามถือผิวน้ำทั้งหมดว่าไม่มีต้น
- ห้ามตัดพื้นที่ออกเพียงเพราะต้นตาย หากกริดยังต่อเนื่อง
- ห้ามสรุปสาเหตุการตายจาก Surface Class เพียงอย่างเดียว
- ห้ามเลือกเฉพาะจุดตรวจที่อยู่ใกล้ทางเข้า
- ห้ามใช้ Candidate Route เป็นเส้นทางรับรองความปลอดภัย
- ห้ามตั้งสถานะ `approved`
