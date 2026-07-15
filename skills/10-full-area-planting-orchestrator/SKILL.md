---
name: full-area-planting-orchestrator
description: ควบคุม Workflow วิเคราะห์แปลงปลูกเต็มพื้นที่จาก Orthomosaic โดยเรียกเฉพาะ Skill ที่จำเป็น เพื่อลด Token และแยกจาก Workflow ปลูกเสริมอย่างชัดเจน
---

# Full-area Planting Orchestrator

## ใช้เมื่อ

- ต้นปลูกครอบคลุมพื้นที่กว้าง
- เห็นแถว กริด หรือระยะปลูกซ้ำต่อเนื่อง
- ช่องว่างส่วนใหญ่มีแนวโน้มเป็นต้นตายหรือต้นหาย
- ต้องการนับต้น วิเคราะห์อัตรารอด หรือวงขอบเขตปลูกเต็มพื้นที่

ห้ามใช้กับพื้นที่ปลูกแทรกตามช่องว่างป่าเดิม ให้ใช้ Skill 00–09 แทน

## Token-efficient reading

อ่านเฉพาะ:

1. `AGENTS.md`
2. `skills/README.md`
3. Skill นี้
4. Skill ขั้นตอนที่กำลังรันเพียงตัวเดียว

ห้ามอ่าน Skill ปลูกเสริมทั้งหมด เว้นแต่งานร้องขอ Classifier เฉพาะ และห้ามอ่าน Skill 16/17 หากภาพไม่มี Trigger

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

- เรียก Skill 16 เมื่อพื้นที่มีดิน เลน น้ำขัง ร่องน้ำ หรือ Surface Zone ต่างกันชัด และต้องการเปรียบเทียบกับ Survival/Mortality
- เรียก Skill 17 เมื่อมีต้นเดี่ยวทรงรัศมีที่อาจปะปนกับต้นปลูก
- Optional Skill ใช้เป็น Context/Exclusion ไม่ใช้แทนหลักฐาน Grid

## Checkpoints

หยุดให้ตรวจหลัง:

1. Tree Detection Preview
2. Grid and Missing-position Preview
3. Survival/Mortality Preview
4. Optional Surface/Palm Preview เมื่อเรียกใช้
5. Candidate Boundary Preview

## Required outputs

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

Optional:

```text
surface_condition.tif
hydrology_features.gpkg
palm_coconut_candidates.gpkg
```

## Restrictions

- ห้ามตีความเงาเป็นต้น
- ห้ามให้ต้นเดิมหรือปาล์ม/มะพร้าวขนาดใหญ่บิดระยะกริด
- ห้ามตัดพื้นที่ออกเพียงเพราะต้นตาย หากกริดยังต่อเนื่อง
- ห้ามใช้สีพื้นน้ำหรือเลนเป็นแนวแบ่งโดยอัตโนมัติ
- ห้ามสรุปสาเหตุการตายจาก Surface Class เพียงอย่างเดียว
- ห้ามตั้งสถานะ `approved`
