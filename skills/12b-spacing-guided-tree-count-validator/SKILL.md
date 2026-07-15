---
name: spacing-guided-tree-count-validator
description: ตรวจและแก้ผลนับต้นปลูกเต็มพื้นที่ด้วยเรือนยอดจริงร่วมกับระยะปลูก แนวแถว กริดหรือจุดปลูก แยกจุดฟุ้ง ต้นเดิมขนาดใหญ่ และเรือนยอดชิดกัน ก่อนคำนวณอัตรารอด
---

# Spacing-guided Tree Count Validator

## หน้าที่

เปลี่ยน Raw Detection จาก Skill 11 และ Preliminary Pattern จาก Skill 12 ให้เป็นจำนวนต้นปลูกที่ผ่านการตรวจเชิงพื้นที่

```text
spacing / row / grid = โครงสร้างสนับสนุน
visible canopy       = หลักฐานว่ามีต้น
```

ห้ามนับจากกริดอย่างเดียว และห้ามถือจุดแดงดิบเป็นจำนวนต้นสุดท้าย

## Inputs

อ่านเต็มเฉพาะ Skill นี้ และอ่าน Output Contract ของ Skill 11, 12, 01b และ 16/17 เมื่อมี

```text
planted_tree_candidates.gpkg
raw_crown_objects.gpkg
existing_large_trees.gpkg
planting_rows.gpkg
planting_grid.gpkg
grid_blocks.gpkg
spacing_reference_samples.gpkg
optional planned_planting_points.gpkg
optional canopy_mask.tif
optional surface_condition.tif
optional hydrology_features.gpkg
optional road_or_dike_mask.tif
optional shadow_mask.tif
project_manifest.json
```

ลำดับ Anchor:

1. `surveyed_planting_point`
2. `approved_plan_point`
3. `digitized_candidate_point`
4. `inferred_grid_position`

ห้ามทำให้ข้อ 4 ดูมีความน่าเชื่อถือเท่าข้อ 1–2

## Core Interpretation

### Raw Detection

จุดจาก Skill 11 เป็น Candidate ต้องตรวจว่า:

- มีเรือนยอดรองรับ
- อยู่ใกล้ Crown Center
- สอดคล้องกับระยะหรือแนวปลูก
- ไม่ซ้ำกับ Detection อื่นในต้นเดียวกัน
- ไม่ตกบนพื้นที่ผิดประเภทโดยไม่มีพุ่มจริง

### Water Context

```text
ผิวน้ำ + ไม่มีเรือนยอด = false_positive_on_water
บริบทน้ำ + มีเรือนยอดจริง = valid_tree_in_water_context
```

น้ำไม่ใช่ Exclusion ทั้งหมด

### Touching / Merged Canopy

ห้ามใช้ `1 canopy blob = 1 tree`

```text
isolated_crown
touching_crown_cluster
merged_planted_canopy
closed_canopy_unresolved
```

เมื่อเรือนยอดชิดกัน ให้ใช้ Crown Center, Reference Spacing, Row Alignment, Planned Point และ Canopy Support ร่วมกัน

### Existing Large Tree

ต้นที่ใหญ่ผิดจากประชากรต้นปลูกรอบข้างและไม่สอดคล้องกับ Pattern ให้เป็น `existing_large_tree`

- ไม่นับเป็นต้นปลูก
- ไม่ใช้ Fit Grid
- ไม่แยก Texture ภายในพุ่มเป็นหลายต้นโดยอัตโนมัติ
- จุดปลูกใต้พุ่มเป็น `not_observable_under_existing_tree`

## Pattern Classes

```text
row_grid_block
staggered_grid_block
locally_regular_full_area_block
shrimp_pond_planting_block
mixed_spacing_block
unreliable_random_scatter_zone
```

แปลงนากุ้งไม่จำเป็นต้องมีกริดเดียวทั้งแปลง แต่ภายในแต่ละ Block ต้องมี Local Spacing Consistency และ Canopy Evidence

## Two-stage Validation

### Stage A — Canopy Support

```text
canopy_support_score
crown_center_score
texture_support_score
visible_green_crown_ratio
```

Negative Context เมื่อไม่มีพุ่มจริง:

```text
bare_ground
open_water_without_crown
road_or_dike
dead_dry_surface
shadow_only
non_target_groundcover
```

### Stage B — Pattern Support

```text
spacing_match_score
row_alignment_score
grid_position_score
local_neighbor_consistency
planned_point_match_score
```

ต้นปลูกต้องผ่าน Canopy Support และ Pattern Support ตาม Config

`off_grid_tree_candidate` คือมีพุ่มจริงแต่ไม่ตรง Pattern ต้อง Review แยก ไม่บังคับเป็นต้นปลูกหรือตัดทิ้งทันที

## Final Position Classes

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

## Decision Rules

```text
Canopy ชัด + ตรงจุดปลูก/กริด
= surviving_confirmed

Canopy ชิด + ระยะ/แนวสอดคล้อง + มี Canopy Support
= surviving_spacing_supported หรือ surviving_in_merged_canopy_probable

มีกริด + ไม่มี Canopy + พื้นมองเห็นชัด
= confirmed_missing หรือ probable_missing

มีกริด + ถูกต้นใหญ่/เงา/เรือนยอดปิดบัง
= not_observable

มี Detection + ไม่มี Canopy
= false_positive

มี Canopy จริง + ไม่มี Pattern รองรับ
= off_grid_tree_candidate
```

ห้ามใช้ Grid สร้าง `surviving` เมื่อไม่มี Canopy Evidence

## Bootstrap and Refit Loop

Skill 12 สร้าง Preliminary Pattern จากต้นเดี่ยว High-confidence ก่อน แล้ว Skill 12b ตรวจจุดทั้งหมด

เมื่อพบว่า Reference Set ปน False Positive หรือต้นใหญ่ หรือ Pattern เปลี่ยนชัด ให้สร้าง:

```text
grid_refit_required: true
validated_reference_samples.gpkg
grid_refit_request.json
```

จากนั้นให้ Skill 12 Refit ด้วย `validated_reference_samples.gpkg` ได้สูงสุด 2 รอบ

ห้ามวนซ้ำไม่สิ้นสุด หากยังไม่เสถียรให้เป็น `pattern_validation_failed`

Skill 13 เริ่มได้เมื่อ:

```text
validation_status = passed หรือ passed_with_warnings
grid_refit_required = false
```

## Required Attributes

```text
position_id
plot_code
grid_block_id
planned_point_id
planned_point_source
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
surveyed_point_plus_canopy
approved_plan_plus_canopy
digitized_point_plus_canopy
spacing_guided_center
merged_canopy_estimate
human_review
```

## Required Metrics

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
validation_status
grid_refit_required
```

## Outputs

```text
validated_planted_tree_points.gpkg
planting_point_status.gpkg
false_positive_detections.gpkg
existing_large_trees_validated.gpkg
merged_canopy_clusters.gpkg
planting_pattern_blocks.gpkg
validated_reference_samples.gpkg
spacing_model.json
tree_count_validation_metrics.json
grid_refit_request.json
tree_count_validation_preview.png
false_positive_preview.png
merged_canopy_preview.png
```

## Failure Conditions

Flag `tree_count_validation_failed` เมื่อ:

- Crown Support Ratio ต่ำ
- จุดบนดิน/น้ำ/ถนนเกิน Config
- Random Scatter สูงและไม่มี Pattern Block
- Grid-only Detection จำนวนมาก
- ต้นใหญ่ถูกใช้ Fit Grid
- Merged Canopy Unresolved สูง
- ใช้ Spacing เดียวทั้งแปลงทั้งที่มีหลาย Pattern
- แยก Confirmed, Probable และ Not Observable ไม่ได้

## Downstream Contract

Skill 13, 14, 15 และ 18 ต้องใช้ผล Validated จาก Skill นี้ ห้ามใช้ Raw Detection เป็นจำนวนสุดท้าย

## Restrictions

- ห้ามถือจุดแดงทุกจุดเป็นต้นไม้
- ห้ามถือผิวน้ำทั้งหมดเป็นพื้นที่ห้ามมีต้น
- ห้ามนับต้นใหญ่เดิมเป็นต้นปลูก
- ห้ามใช้พุ่มใหญ่หนึ่งพุ่มเป็นหลายต้นเพราะมี Texture หลายยอด
- ห้ามสร้างต้นจาก Grid อย่างเดียว
- ห้ามปรับผลให้ตรง Expected Count
- ผลยังเป็น Candidate Count ต้องผ่าน Human Review