---
name: planting-grid-inference
description: อนุมานแนวแถว ทิศทาง ระยะต้น ระยะแถว และตำแหน่งต้นที่คาดว่าหายจากผลตรวจต้นในแปลงปลูกเต็มพื้นที่
---

# Planting Grid Inference

## Scope

รับผลต้นจาก Skill 11 แล้วสร้างโครงสร้างแถวและกริด ห้ามตรวจต้นใหม่ ห้ามวงขอบเขตสุดท้าย

## Inputs

```text
planted_tree_candidates.gpkg
existing_large_trees.gpkg
optional AOI
```

## Method rules

1. ตัด `existing_large_tree`, `shadow` และ Low-confidence outlier ก่อน Fit Grid
2. ตรวจทิศทางหลักด้วย Neighbor Vectors, Hough/RANSAC หรือวิธีที่ทนต่อ Outlier
3. คำนวณระยะต้นและระยะแถวด้วยค่ามัธยฐาน ไม่ใช้ค่าเฉลี่ยอย่างเดียว
4. รองรับหลาย Grid Block หากทิศทางหรือระยะเปลี่ยน
5. สร้างตำแหน่งคาดหมายเฉพาะภายในบริเวณที่มีแถวรองรับ
6. ห้ามขยายกริดผ่านคลอง ถนน ขอบภาพ หรือพื้นที่ไม่มีหลักฐาน

## Required metrics

```text
grid_block_id
row_orientation_deg
cross_row_orientation_deg
median_tree_spacing_m
median_row_spacing_m
spacing_cv
spacing_consistency
observed_tree_count
expected_position_count
missing_position_count
row_coverage_ratio
confidence
```

## Missing-position classes

```text
probable_missing_tree
uncertain_missing_position
not_evaluated
```

ตำแหน่งหายต้องมีต้นหรือแนวแถวสนับสนุนทั้งสองด้าน และไม่มี Barrier ถาวรขวาง

## Outputs

```text
planting_rows.gpkg
planting_grid.gpkg
expected_missing_positions.gpkg
grid_blocks.gpkg
grid_metrics.json
grid_missing_preview.png
```

## QA

- แจ้งเตือนเมื่อ Fit ด้วยต้นน้อยเกินไป
- แจ้งเตือนเมื่อมีหลายทิศทางแต่ระบบสร้าง Grid เดียว
- แจ้งเตือนเมื่อ Grid ข้าม Barrier
- ห้ามสร้าง Missing Position ในพื้นที่ที่ไม่มี Coverage ของแถว
