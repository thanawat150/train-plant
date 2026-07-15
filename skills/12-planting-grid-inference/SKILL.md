---
name: planting-grid-inference
description: อนุมานแนวแถว ระยะปลูก และ Planting Pattern Block เบื้องต้นจากต้นเดี่ยวที่เชื่อถือได้หรือจุดปลูกเดิม เพื่อส่งให้ Skill 12b ตรวจจำนวนและขอ Refit เมื่อจำเป็น
---

# Planting Grid Inference

## Scope

สร้าง Preliminary Spacing/Pattern จากผล Skill 11 หรือจุดปลูกเดิม ห้ามนับต้นสุดท้าย สร้างต้นจากกริด หรือวงขอบเขต

## Inputs

```text
planted_tree_candidates.gpkg
raw_crown_objects.gpkg
existing_large_trees.gpkg
project_manifest.json
optional planned_planting_points.gpkg
optional validated_reference_samples.gpkg
optional AOI
optional barriers.gpkg
```

หากมี `validated_reference_samples.gpkg` จาก Skill 12b ให้ใช้ Refit แทน Reference เดิม

## Anchor Priority

```text
surveyed_planting_point
approved_plan_point
human_digitized_candidate
isolated_planted_crown_candidate_high_confidence
model_inferred_position
```

ต้องเก็บ `point_source` และ Reliability แยกกัน

## Reference Sample Rules

ใช้ Fit เฉพาะต้นเดี่ยว High-confidence หรือจุดปลูกที่มี Provenance เชื่อถือได้

ตัดออกก่อน Fit:

- existing large tree
- merged/touching cluster ที่ยังไม่รู้ศูนย์กลาง
- palm/coconut
- จุดบนดิน น้ำเปล่า ถนน หรือเงา
- Low-confidence outlier
- Duplicate Detection
- Random Scatter

## Pattern Classes

```text
row_grid_block
staggered_grid_block
locally_regular_full_area_block
shrimp_pond_planting_block
mixed_spacing_block
unreliable_pattern_zone
```

แปลงนากุ้งไม่จำเป็นต้องมีแนวเดียวทั้งแปลง แต่ภายใน Block ต้องมีระยะใกล้เคียงกันและต้นจริงรองรับ

## Method Rules

1. ใช้ Neighbor Vectors, Hough/RANSAC หรือวิธีทน Outlier
2. ใช้ Median และ MAD สำหรับระยะ
3. คำนวณ Nearest-neighbor Distribution และ Local Spacing CV
4. รองรับหลาย Block เมื่อแนว ระยะ หรือสภาพพื้นที่เปลี่ยน
5. ขอบ Blockสัมพันธ์กับคันบ่อ ร่องน้ำ ถนน หรือ Pattern Change
6. สร้าง Expected Position เฉพาะบริเวณที่มี Pattern Support
7. ห้ามขยาย Grid ผ่านคลอง ถนน NoData หรือพื้นที่ไม่มีหลักฐาน
8. Grid เป็นตำแหน่งคาดหมาย ไม่ใช่หลักฐานว่าต้นรอด
9. Refit ได้สูงสุดตาม `processing.max_grid_refit_iterations`
10. ทุก Refit ต้องเพิ่ม `grid_model_version` และเก็บ Parent Version

## Required Metrics

```text
grid_block_id
grid_model_version
parent_grid_model_version
pattern_class
row_orientation_deg
cross_row_orientation_deg
median_tree_spacing_m
median_row_spacing_m
spacing_mad_m
spacing_cv
spacing_consistency
reference_tree_count
expected_position_count
row_coverage_ratio
pattern_support_ratio
confidence
```

## Candidate Position Classes

```text
observed_reference_position
expected_occupied_candidate
expected_missing_candidate
uncertain_expected_position
not_evaluated
```

Skill นี้ยังไม่ตัดสิน Surviving/Missing สุดท้าย

## Outputs

```text
planting_rows.gpkg
planting_grid.gpkg
expected_missing_positions.gpkg
grid_blocks.gpkg
spacing_reference_samples.gpkg
grid_metrics.json
grid_missing_preview.png
```

## QA

- Reference Trees น้อยเกินไป
- หลายทิศทางแต่สร้าง Grid เดียว
- Grid ข้าม Barrier
- Random Scatter ถูกใช้เป็น Reference
- Spacing CV สูงเกิน Config
- Expected Position อยู่นอก Pattern Coverage
- Refit เกินจำนวนรอบ

## Downstream

ส่งผลให้ Skill 12b ตรวจร่วมกับ Canopy, Planned Points, Surface และต้นใหญ่

หาก Skill 12b ส่ง `grid_refit_required=true` ให้ Refit ด้วย Validated Reference แล้วส่งกลับ 12b ห้ามส่งต่อ Skill 13 ก่อน Validation ผ่าน
