---
name: spacing-guided-tree-count-validator
description: ตรวจสอบและแก้ผลนับต้นปลูกเต็มพื้นที่โดยใช้เรือนยอดจริงร่วมกับระยะปลูก แนวแถว กริดหรือจุดปลูก แยกจุดฟุ้งบนดิน/น้ำ ต้นเดิมขนาดใหญ่ และเรือนยอดชิดกัน ก่อนส่งผลไปคำนวณอัตรารอด
---

# Spacing-guided Tree Count Validator

## หน้าที่

เปลี่ยน Raw Detection จาก Skill 11 และแบบจำลองแนวปลูกจาก Skill 12 ให้เป็นจำนวนต้นปลูกที่ผ่านการตรวจเชิงพื้นที่

หลักสำคัญ:

```text
spacing/row/grid = โครงสร้างสนับสนุน
visible canopy   = หลักฐานว่ามีต้น
```

ห้ามนับจากกริดอย่างเดียว และห้ามนับจาก Texture หรือจุดแดงที่กระจายฟุ้งโดยไม่มีเรือนยอดรองรับ

## ใช้เมื่อ

- จุดตรวจต้นกระจายบนดินโล่ง น้ำ ถนน วัชพืช หรือซากพืช
- พุ่มปลูกมีระยะใกล้เคียงกัน แต่บางบริเวณเรือนยอดเริ่มชิดกัน
- แปลงปลูกเต็มในนากุ้งมีหลายแถว หลายกริด หรือหลาย Planting Block
- มีต้นเดิมขนาดใหญ่ปะปนและไม่ต้องการนับ
- ต้องจับคู่จุดปลูกเดิมกับต้นที่เห็นในภาพ
- ต้องการผลนับที่พร้อมส่งให้ Skill 13 วิเคราะห์อัตรารอด

## Token-efficient inputs

อ่านเต็มเฉพาะ Skill นี้ แล้วอ่าน Output Contract ของ Skill 11, 12 และ 16 เมื่อมี

Inputs ที่รองรับ:

```text
planted_tree_candidates.gpkg        # Raw candidates จาก Skill 11
existing_large_trees.gpkg
planting_rows.gpkg
planting_grid.gpkg
grid_blocks.gpkg
expected_missing_positions.gpkg
optional planned_planting_points.gpkg
optional canopy_mask.tif
optional surface_condition.tif
optional hydrology_features.gpkg
optional road_or_dike_mask.tif
optional shadow_mask.tif
```

หากมี `planned_planting_points.gpkg` ให้ใช้เป็น Anchor หลักก่อน Grid ที่อนุมานจากภาพ

## ความเข้าใจหลัก

### 1. จุดแดงไม่ใช่จำนวนต้นสุดท้าย

จุดจาก Detector เป็นเพียง Raw Candidate ต้องตรวจว่า:

- มีเรือนยอดจริงรองรับหรือไม่
- อยู่ใกล้ศูนย์กลางพุ่มหรือไม่
- สอดคล้องกับระยะปลูกหรือแนวปลูกหรือไม่
- เป็นจุดซ้ำในพุ่มเดียวหรือไม่
- ตกบนพื้นที่ผิดประเภทหรือไม่

### 2. น้ำไม่ใช่ Exclusion ทั้งหมด

ต้นป่าชายเลนสามารถขึ้นในน้ำตื้นหรือเลนที่มีน้ำได้

```text
จุดบนผิวน้ำ + ไม่มีเรือนยอด = false_positive_on_water
จุดในบริบทน้ำ + มีเรือนยอดจริง = valid_tree_in_water_context
```

ห้ามตัดต้นจริงเพียงเพราะ Background เป็นน้ำ

### 3. เรือนยอดชิดกันยังนับได้

ถ้าศูนย์กลางต้นยังห่างใกล้เคียงกับ Reference Spacing ให้ใช้ระยะปลูก แนวแถว จุดปลูก และ Canopy Support ช่วยวางศูนย์กลางต้น

ห้ามใช้กฎ:

```text
1 canopy blob = 1 tree
```

### 4. ต้นเดิมขนาดใหญ่ไม่ต้องนับ

พุ่มที่ใหญ่ผิดจากประชากรต้นปลูกรอบข้าง ไม่สอดคล้องกับแถว หรือครอบหลายตำแหน่งปลูก ให้เป็น `existing_large_tree`

- ไม่นับเป็นต้นปลูก
- ไม่ใช้ Fit Grid
- จุดปลูกที่ถูกบังให้เป็น `not_observable_under_existing_tree`
- ห้ามเรียกจุดใต้พุ่มใหญ่ว่าต้นตายโดยอัตโนมัติ

## Planting pattern classes

```text
row_grid_block
staggered_grid_block
locally_regular_full_area_block
shrimp_pond_planting_block
mixed_spacing_block
unreliable_random_scatter_zone
```

`locally_regular_full_area_block` และ `shrimp_pond_planting_block` ไม่จำเป็นต้องเป็นเส้นตรงสมบูรณ์ทั้งแปลง แต่ต้องมีระยะใกล้เคียงกันภายใน Block และมีเรือนยอดจริงรองรับ

จุดฟุ้งที่ไม่มีแนว ไม่มี Local Spacing Consistency และไม่มี Canopy Support ให้เป็น `unreliable_random_scatter_zone`

## Reference spacing model

1. เลือกต้นเดี่ยว High-confidence ที่เห็นขอบพุ่มชัด
2. ตัดต้นใหญ่ เงา Palm/Coconut และ Outlier
3. คำนวณเป็นราย Block:
   - nearest-neighbor spacing
   - row orientation
   - cross-row spacing
   - spacing median และ MAD
   - local spacing CV
4. รองรับหลาย Block เมื่อระยะหรือแนวเปลี่ยน
5. ห้ามใช้ค่าเดียวครอบทั้งแปลงเมื่อมีคันบ่อ ร่องน้ำ ทาง หรือสภาพปลูกต่างกัน

## Two-stage validation

### Stage A — Canopy support

คำนวณ:

```text
canopy_support_score
crown_center_score
texture_support_score
visible_green_crown_ratio
```

พื้นที่ต่อไปนี้เป็น Negative Context เมื่อไม่มีพุ่มจริง:

```text
bare_ground
open_water_without_crown
road_or_dike
dead_dry_surface
shadow_only
non_target_groundcover
```

### Stage B — Planting-pattern support

คำนวณ:

```text
spacing_match_score
row_alignment_score
grid_position_score
local_neighbor_consistency
planned_point_match_score
```

จุดจะเป็นต้นปลูกได้ต้องมี Canopy Support และ Pattern Support ตาม Config

ข้อยกเว้น: `off_grid_tree_candidate` อาจมีพุ่มจริงแต่ไม่ตรงกริด ต้องแยกตรวจ ไม่บังคับเป็นต้นปลูกหรือตัดทิ้งทันที

## Merged-canopy rules

แบ่งเป็น:

```text
isolated_crown
touching_crown_cluster
merged_planted_canopy
closed_canopy_unresolved
```

### isolated_crown

เห็นศูนย์กลางและขอบพุ่มชัด นับได้โดยตรง

### touching_crown_cluster

พุ่มแตะกัน แต่ยังเห็น Local Peaks หรือจุดปลูก/ระยะสนับสนุน ให้นับศูนย์กลางหลายต้นได้

### merged_planted_canopy

ขอบพุ่มรวมเป็นผืน แต่ตำแหน่งศูนย์กลางตามระยะปลูกมี Canopy Support ต่อเนื่อง ให้ใช้:

```text
confirmed_center_count
probable_spacing_supported_count
estimated_min_count
estimated_max_count
```

### closed_canopy_unresolved

แยกศูนย์กลางและระยะไม่ได้อย่างน่าเชื่อถือ ให้รายงานเป็น Unresolved Area และห้ามบังคับจำนวนเดียว

## Existing large tree rules

ใช้หลักฐานร่วมกัน:

```text
crown_size_ratio_to_local_median
row_alignment_score
internal_peak_spacing
texture_difference
position_relative_to_plot_edge_or_existing_forest
```

Candidate ตั้งต้นเมื่อ Crown Diameter หรือ Crown Area ใหญ่กว่าค่ากลางต้นปลูกรอบข้างอย่างชัดเจน แต่ห้ามใช้ขนาดเพียงอย่างเดียว

Classes:

```text
existing_large_tree
large_tree_uncertain
merged_planted_canopy_not_large_tree
```

## Final position classes

```text
surviving_confirmed
surviving_spacing_supported
surviving_in_merged_canopy_probable
probable_surviving_tree
confirmed_missing
probable_missing
not_observable_under_existing_tree
not_observable_shadow_or_blur
not_observable_closed_canopy
off_grid_tree_candidate
existing_large_tree_excluded
false_positive_on_bare_ground
false_positive_on_water
false_positive_on_road_or_dike
false_positive_on_shadow
false_positive_on_groundcover
duplicate_detection
grid_only_detection
unresolved_merged_canopy
```

## Decision rules

```text
มี Canopy ชัด + ตรงจุดปลูก/กริด
= surviving_confirmed

Canopy ชิดกัน + ระยะ/แนวสอดคล้อง + มี Canopy Support
= surviving_spacing_supported หรือ surviving_in_merged_canopy_probable

มีกริด แต่ไม่มี Canopy และพื้นมองเห็นชัด
= confirmed_missing หรือ probable_missing

มีกริด แต่ถูกต้นใหญ่/เงา/เรือนยอดปิดบัง
= not_observable

มีจุด Detection แต่ไม่มี Canopy รองรับ
= false_positive

มี Canopy จริง แต่ไม่มี Pattern รองรับ
= off_grid_tree_candidate
```

ห้ามใช้ Grid สร้าง `surviving` เมื่อไม่มี Canopy Evidence

## Required attributes

```text
position_id
plot_code
grid_block_id
planned_point_id
raw_detection_id
final_class
count_weight
confidence
canopy_support_score
crown_center_score
spacing_match_score
row_alignment_score
planned_point_match_score
local_spacing_m
spacing_residual_m
water_context
bare_ground_conflict
road_conflict
shadow_conflict
existing_large_tree_overlap
merged_canopy_cluster_id
count_source
review_status
source_tile
source_raster
```

`count_source`:

```text
direct_isolated_crown
planned_point_plus_canopy
spacing_guided_center
merged_canopy_estimate
human_review
```

`count_weight` ค่าเริ่มต้น:

```text
surviving_confirmed                     1.0
surviving_spacing_supported             1.0
surviving_in_merged_canopy_probable      configurable, default 1.0 but separate in report
probable_surviving_tree                  configurable
all false_positive / existing_large_tree 0.0
not_observable                           excluded from survival denominator
```

ห้ามรวม Probable กับ Confirmed โดยไม่แสดงแยก

## Required metrics

```text
raw_detection_count
validated_confirmed_count
validated_spacing_supported_count
probable_merged_canopy_count
estimated_tree_count_min
estimated_tree_count_max
existing_large_tree_count
false_positive_bare_ground_count
false_positive_water_count
false_positive_road_count
false_positive_shadow_count
duplicate_detection_count
grid_only_detection_count
off_grid_tree_count
unresolved_merged_canopy_cluster_count
crown_support_ratio
pattern_support_ratio
random_scatter_ratio
not_observable_count
```

## Outputs

```text
validated_planted_tree_points.gpkg
planting_point_status.gpkg
false_positive_detections.gpkg
existing_large_trees_validated.gpkg
merged_canopy_clusters.gpkg
planting_pattern_blocks.gpkg
spacing_model.json
tree_count_validation_metrics.json
tree_count_validation_preview.png
false_positive_preview.png
merged_canopy_preview.png
```

## QA failure conditions

Flag `tree_count_validation_failed` เมื่อเกิดอย่างใดอย่างหนึ่ง:

- Crown Support Ratio ต่ำกว่า Config
- จุดบนดิน/น้ำ/ถนนเกินเกณฑ์
- Random Scatter Ratio สูงและไม่มี Pattern Block รองรับ
- Grid-only Detection จำนวนมาก
- ต้นใหญ่ถูกใช้ Fit Grid
- Merged Canopy Unresolved มีสัดส่วนสูง
- ใช้ Spacing เดียวทั้งแปลงทั้งที่มีหลาย Pattern
- ไม่สามารถแยก Confirmed, Probable และ Not Observable ได้

## Checkpoints

หยุดให้ผู้ใช้ตรวจ:

1. Exclusion และ Existing Large Tree Preview
2. Reference Spacing / Planting Pattern Blocks
3. False-positive Removal Preview
4. Merged-canopy Center Preview
5. Final Confirmed / Probable / Missing / Not-observable Summary

## Downstream contract

Skill 13 ต้องใช้ `validated_planted_tree_points.gpkg` และ `planting_point_status.gpkg` เป็น Input หลัก ห้ามใช้นับอัตรารอดจาก Raw Detection ของ Skill 11 โดยตรง

## Restrictions

- ห้ามถือจุดแดงทุกจุดเป็นต้นไม้
- ห้ามถือผิวน้ำทั้งหมดเป็นพื้นที่ห้ามมีต้น
- ห้ามนับต้นใหญ่เดิมเป็นต้นปลูก
- ห้ามใช้พุ่มใหญ่หนึ่งพุ่มเป็นหลายต้นเพียงเพราะมี Texture หลายยอด
- ห้ามใช้ `1 canopy blob = 1 tree` ในพื้นที่เรือนยอดชิด
- ห้ามสร้างต้นจาก Grid อย่างเดียว
- ห้ามใช้จำนวนคาดหมายเพื่อปรับผลให้ตรงเป้าหมาย
- ผลยังเป็น Candidate Count ต้องผ่าน Human Review
