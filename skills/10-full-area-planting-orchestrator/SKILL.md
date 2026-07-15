---
name: full-area-planting-orchestrator
description: ควบคุม Workflow แปลงปลูกเต็ม ตั้งแต่ Preflight, Raw Detection, Spacing Validation, Survival, Boundary, QA ไปจนถึง Field Plan และ Feedback Loop
---

# Full-area Planting Orchestrator

## ใช้เมื่อ

- ปลูกครอบคลุมพื้นที่กว้าง
- มีแถว กริด ระยะซ้ำ หรือ Planting Block ในนากุ้ง
- เรือนยอดบางส่วนชิดกันแต่ระยะศูนย์กลางยังสม่ำเสมอ
- ต้องการจำนวนต้น ต้นหาย อัตรารอด ขอบเขต หรือแผนเข้าตรวจ

พื้นที่ปลูกแทรกตามช่องว่างป่าเดิมให้ใช้ Skill 00

## Token Rules

อ่าน `AGENTS.md`, Skill นี้, Skill ขั้นที่กำลังรัน และ Output Contract ของขั้นก่อนหน้าเท่านั้น

## Production Pipeline

```text
01  inspect/tile raster
→ 01b validate project inputs and provenance
→ 11 raw crown candidates
→ 12 preliminary spacing and pattern blocks
→ 12b validate count with canopy + spacing + planting points
→ 13 survival/mortality
→ 14 candidate boundary
→ 15 QA/human review package
```

ห้ามข้าม 12b

## Bootstrap Refit

เมื่อ 12b รายงาน `grid_refit_required=true`:

```text
12b validated_reference_samples
→ rerun 12
→ rerun 12b
```

ทำได้สูงสุดตาม Config ค่าเริ่มต้น 2 รอบ หากยังไม่เสถียรให้หยุดเป็น `pattern_validation_failed`

## Core Rules

- จุดแดงจาก 11 เป็น Raw Candidate
- ต้นปลูกต้องมี Canopy Evidence และ Pattern/Spacing Support
- Grid ห้ามสร้างต้นที่ไม่มีเรือนยอด
- จุดบนผิวน้ำที่ไม่มีพุ่มเป็น False Positive แต่น้ำไม่ใช่ Exclusion ทั้งหมด
- จุดบนดิน ถนน คันดิน วัชพืช หรือ Shadow-only ต้องถูกตรวจ
- เรือนยอดชิดกันใช้ Center, Spacing, Rows และ Planting Points ไม่ใช้ `1 blob = 1 tree`
- ต้นเดิมขนาดใหญ่ไม่นับ ไม่ใช้ Fit Grid และจุดใต้พุ่มเป็น Not Observable
- แบ่งหลาย Pattern Block เมื่อแนวหรือระยะเปลี่ยน
- `inferred_grid_position` ไม่ใช่จุดปลูกจริง

## Optional Context

- Skill 16: Surface/Hydrology
- Skill 17: Palm/Coconut

เรียกเฉพาะเมื่อมี Trigger และไม่ใช้แทน Canopy/Spacing Evidence

## Optional Field Plan

```text
หลัง 15
→ 18 inspection priority
→ 19 land/boat access
→ 20 missions/day plan
```

## Optional Feedback Loop

```text
หลังภาคสนาม
→ 21 ingest field feedback/ground truth
→ 22 evaluate accuracy and recommend calibration
```

Threshold Recommendation ห้ามถูกใช้ Production อัตโนมัติ

## Checkpoints

1. Preflight Summary
2. Raw Detection Preview
3. Preliminary Spacing/Pattern
4. 12b False-positive/Large-tree/Merged-canopy Validation
5. Final Confirmed/Probable/Missing/Not-observable Count
6. Survival/Mortality
7. Boundary
8. QA
9. Field Plan/Feedback เมื่อเรียกใช้

## Required Core Outputs

```text
project_manifest.json
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

## Stop Conditions

หยุดก่อน Survival เมื่อ:

- Preflight ไม่ผ่าน
- 12b รายงาน `tree_count_validation_failed`
- Crown Support ต่ำหรือ Random Scatter สูง
- Grid-only Detection จำนวนมาก
- ต้นใหญ่ถูกใช้ Fit Grid
- Merged Canopy ถูกบังคับเป็นจำนวนเดียวโดยไม่มีหลักฐาน
- Grid Refit ครบจำนวนรอบแล้วยังไม่เสถียร

## Restrictions

- ห้ามใช้ Raw Detection เป็นจำนวนสุดท้าย
- ห้ามปรับผลให้ตรง Expected Count
- ห้ามสรุปสาเหตุการตายจากภาพเพียงอย่างเดียว
- ห้ามเลือกเฉพาะจุดตรวจใกล้ทางเข้า
- ห้ามรับรอง Candidate Route ว่าปลอดภัย
- ห้ามตั้งสถานะ `approved`
