---
name: full-area-planting-orchestrator
description: ควบคุม Workflow วิเคราะห์แปลงปลูกเต็มพื้นที่จาก Orthomosaic โดยเรียกเฉพาะ Skill ที่จำเป็น ลด Token และสามารถต่อยอดผลต้น กริด และอัตรารอดเป็นแผนเข้าตรวจภาคสนามทางบก/ทางเรือ
---

# Full-area Planting Orchestrator

## ใช้เมื่อ

- ต้นปลูกครอบคลุมพื้นที่กว้าง
- เห็นแถว กริด หรือระยะปลูกซ้ำต่อเนื่อง
- ช่องว่างส่วนใหญ่มีแนวโน้มเป็นต้นตายหรือต้นหาย
- ต้องการนับต้น วิเคราะห์อัตรารอด วงขอบเขต หรือวางแผนเข้าตรวจ

ห้ามใช้กับพื้นที่ปลูกแทรกตามช่องว่างป่าเดิม ให้ใช้ Skill 00–09 แทน

## Token-efficient reading

อ่านเฉพาะ:

1. `AGENTS.md`
2. `skills/README.md`
3. Skill นี้
4. Skill ขั้นตอนที่กำลังรันเพียงตัวเดียว

ห้ามอ่าน Skill ปลูกเสริมทั้งหมด เว้นแต่งานร้องขอ Classifier เฉพาะ และห้ามอ่าน Optional Skill หากไม่มี Trigger

## Core Pipeline

```text
Skill 01: inspect and tile large raster
→ Skill 11: detect planted-tree crowns
→ Skill 12: infer planting rows and grid
→ Skill 13: analyze survival and mortality gaps
→ Skill 14: delineate full-area boundary
→ Skill 15: QA and human review
```

## Optional Context

- Skill 16 เมื่อพื้นที่มีดิน เลน น้ำขัง ร่องน้ำ หรือ Surface Zone ต่างกันชัด
- Skill 17 เมื่อมีต้นเดี่ยวทรงรัศมีที่อาจปะปนกับต้นปลูก
- Optional Context ใช้เปรียบเทียบหรือ Exclusion ไม่ใช้แทนหลักฐาน Grid

## Optional Field Inspection Pipeline

เรียกเมื่อผู้ใช้ต้องการวางแผนเข้าตรวจ:

```text
หลัง Skill 15
→ Skill 18: prioritize inspection points from mortality, uncertainty and QA
→ Skill 19: assess land/boat access and route burden
→ Skill 20: group missions, teams, days and field checklist
```

Input เพิ่มเติมเมื่อวางแผนภาคสนาม:

- ถนน ทางเดิน คันดิน และเส้นเข้าแปลงทางบก
- คลอง เส้นทางเรือ ท่าเรือ และจุดขึ้นฝั่ง
- จุดจอดรถ จุดเข้าแปลง และฐานทีม
- เขตห้ามเข้า พื้นที่เอกชน และพื้นที่เสี่ยง
- Tide Window, เวลาเดินทาง, จำนวนทีม และเวลาทำงานต่อวัน

## Field Planning Rules

- แยก `evidence_priority_score` ออกจาก `access_burden_score`
- ห้ามลดความสำคัญของจุดเพียงเพราะไกล
- จุดสำคัญแต่ไกลให้เป็น `special_mission` หรือรวมกับจุดใกล้เคียงเป็น `remote_cluster_mission`
- เส้นทางต้องใช้ Network Distance และเวลาเดินทาง ไม่ใช้ระยะเส้นตรง
- ทางบกและทางเรือต้องประเมินแยกก่อนเลือก Route
- พื้นที่ไม่มี Route ที่ยืนยันแล้วให้ใช้ `route_scout_required`

## Checkpoints

หยุดให้ตรวจหลัง:

1. Tree Detection Preview
2. Grid and Missing-position Preview
3. Survival/Mortality Preview
4. Optional Surface/Palm Preview เมื่อเรียกใช้
5. Candidate Boundary Preview
6. Candidate Inspection Points และเหตุผล เมื่อเรียก Skill 18
7. Land/Boat Route Options เมื่อเรียก Skill 19
8. Mission Grouping และ Day Plan เมื่อเรียก Skill 20

## Required Outputs

```text
planted_tree_candidates.gpkg
planting_rows.gpkg
planting_grid.gpkg
expected_missing_positions.gpkg
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

## Restrictions

- ห้ามตีความเงาเป็นต้น
- ห้ามให้ต้นเดิมหรือปาล์ม/มะพร้าวขนาดใหญ่บิดระยะกริด
- ห้ามตัดพื้นที่ออกเพียงเพราะต้นตาย หากกริดยังต่อเนื่อง
- ห้ามใช้สีพื้นน้ำหรือเลนเป็นแนวแบ่งโดยอัตโนมัติ
- ห้ามสรุปสาเหตุการตายจาก Surface Class เพียงอย่างเดียว
- ห้ามเลือกเฉพาะจุดตรวจที่อยู่ใกล้ทางเข้า
- ห้ามใช้ Candidate Route เป็นเส้นทางรับรองความปลอดภัย
- ห้ามตั้งสถานะ `approved`
