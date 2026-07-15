---
name: planting-grid-inference
description: อนุมานแนวแถว ทิศทาง ระยะต้น ระยะแถว และ Planting Pattern Block จากต้นเดี่ยวที่เชื่อถือได้หรือจุดปลูกเดิม เพื่อส่งให้ Skill 21 ตรวจนับต้นในเรือนยอดชิดและกรองจุดฟุ้ง
---

# Planting Grid Inference

## Scope

สร้างแบบจำลองระยะและแนวปลูกเบื้องต้นจากผล Skill 11 หรือจุดปลูกเดิม ห้ามนับต้นสุดท้าย ห้ามสร้างต้นจากกริดอย่างเดียว และห้ามวงขอบเขตสุดท้าย

## Inputs

```text
planted_tree_candidates.gpkg
raw_crown_objects.gpkg
existing_large_trees.gpkg
optional planned_planting_points.gpkg
optional AOI
optional barriers.gpkg
```

หากมีจุดปลูกเดิม ให้ใช้เป็น Anchor หลักและตรวจความสอดคล้องกับภาพ

## Reference sample rules

ใช้ Fit Pattern เฉพาะ:

```text
isolated_planted_crown_candidate ที่ confidence สูง
หรือ planned_planting_points ที่เชื่อถือได้
```

ตัดออกก่อน Fit:

- existing large tree
- merged/touching cluster ที่ยังไม่ทราบศูนย์กลาง
- palm/coconut
- จุดบนดิน น้ำ ถนน หรือเงา
- Low-confidence outlier
- Detection ซ้ำ

## Pattern classes

```text
row_grid_block
staggered_grid_block
locally_regular_full_area_block
shrimp_pond_planting_block
mixed_spacing_block
unreliable_pattern_zone
```

แปลงนากุ้งสามารถเป็น Full-area Block ที่แนวไม่ตรงสมบูรณ์ทั้งแปลงได้ แต่ภายใน Block ต้องมีระยะใกล้เคียงกันและมีต้นจริงรองรับ

## Method rules

1. ตรวจทิศทางหลักด้วย Neighbor Vectors, Hough/RANSAC หรือวิธีที่ทนต่อ Outlier
2. คำนวณระยะต้นและระยะแถวด้วย Median และ MAD ไม่ใช้ค่าเฉลี่ยอย่างเดียว
3. คำนวณ Nearest-neighbor distribution และ Local Spacing CV
4. รองรับหลาย Block เมื่อทิศทาง ระยะ หรือสภาพพื้นที่เปลี่ยน
5. ขอบ Block ต้องสัมพันธ์กับหลักฐาน เช่น คันบ่อ ร่องน้ำ ถนน หรือการเปลี่ยน Pattern
6. สร้าง Expected Position เฉพาะภายในบริเวณที่มี Pattern รองรับ
7. ห้ามขยาย Grid ผ่านคลอง ถนน ขอบภาพ NoData หรือพื้นที่ไม่มีหลักฐาน
8. ห้ามใช้จุดฟุ้ง Random Scatter เพื่อ Fit Grid
9. Grid เป็นตำแหน่งคาดหมาย ไม่ใช่หลักฐานว่าต้นรอด

## Required metrics

```text
grid_block_id
pattern_class
row_orientation_deg
cross_row_orientation_deg
median_tree_spacing_m
median_row_spacing_m
spacing_mad_m
spacing_cv
spacing_consistency
reference_tree_count
observed_tree_count
expected_position_count
row_coverage_ratio
pattern_support_ratio
confidence
```

## Candidate position classes

```text
observed_reference_position
expected_occupied_candidate
expected_missing_candidate
uncertain_expected_position
not_evaluated
```

Skill นี้ยังไม่ตัดสิน `surviving` หรือ `missing` สุดท้าย การตัดสินต้องทำใน Skill 21 โดยตรวจ Canopy Support

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

- แจ้งเตือนเมื่อ Fit ด้วย Reference Trees น้อยเกินไป
- แจ้งเตือนเมื่อมีหลายทิศทางแต่ระบบสร้าง Grid เดียว
- แจ้งเตือนเมื่อ Grid ข้าม Barrier
- แจ้งเตือนเมื่อ Random Scatter ถูกใช้เป็น Reference
- แจ้งเตือนเมื่อ Spacing CV สูงเกิน Config
- ห้ามสร้าง Expected Position ในพื้นที่ที่ไม่มี Pattern Coverage
- ห้ามเรียก Expected Position ว่าต้นจริงหรือต้นตายก่อน Skill 21

## Downstream

ส่ง Pattern Blocks, Rows, Grid และ Expected Positions ให้ Skill 21 ตรวจร่วมกับเรือนยอดจริง จุดปลูกเดิม น้ำ ดิน และต้นใหญ่ ก่อนส่งผลให้ Skill 13
