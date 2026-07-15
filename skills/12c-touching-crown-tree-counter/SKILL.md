---
name: touching-crown-tree-counter
description: Use when planted-tree crowns touch, overlap, merge into continuous canopy, or are under-counted because connected canopy objects cannot be separated reliably; also use when linear vegetation on roads or pond dikes may be counted as planted trees.
---

# Touching Crown Tree Counter

## หน้าที่

แยกและนับต้นปลูกในบริเวณเรือนยอดชิด ซ้อน หรือรวมเป็นผืน โดยใช้ `Crown Center + Canopy Evidence + Local Spacing + 2D Row Support` ร่วมกัน

```text
1 connected canopy blob != 1 tree
1 grid position != 1 tree
visible crown center + local planting support = tree candidate
```

ใช้หลัง Skill 11 และ Skill 12 ก่อนส่งผลให้ Skill 12b ตัดสินสถานะสุดท้าย

## Token Rule

อ่านเต็มเฉพาะ Skill นี้ และอ่าน Output Contract ของ Skill 11, 12 และ 01b รวมถึง 16/17 เมื่อมี Trigger

## Inputs

```text
source orthomosaic
raw_crown_objects.gpkg
touching_crown_clusters.gpkg
existing_large_trees.gpkg
planted_tree_candidates.gpkg
planting_rows.gpkg
grid_blocks.gpkg
spacing_reference_samples.gpkg
grid_metrics.json
project_manifest.json
optional canopy_mask.tif
optional canopy_probability.tif
optional road_or_dike_mask.tif
optional shadow_mask.tif
optional surface_condition.tif
```

ห้ามใช้ Point Plan หรือ Grid เป็นหลักฐานเรือนยอด

หากไม่มี `canopy_mask.tif` หรือ `canopy_probability.tif` ให้ใช้ Crown Objects หรือสร้าง Temporary Canopy Evidence จาก Orthomosaic พร้อมบันทึกวิธีและลด Confidence

## Resolution Gate

ก่อนแยก Crown Center ให้ตรวจ:

```text
minimum_resolvable_crown_diameter_px
minimum_center_separation_px
image_blur_score
orthomosaic_seam_conflict
```

หากขนาดพุ่มหรือระยะศูนย์กลางต่ำกว่าความสามารถของภาพ ห้ามบังคับแยกต้น ให้เป็น `unresolved_merged_canopy` หรือ `closed_canopy_unresolved`

## Required Workflow

### 1. ตรวจ Cluster ทุกขนาด

จัดแต่ละ connected object เป็น:

```text
touching_crown_cluster
merged_planted_canopy_candidate
closed_canopy_unresolved
existing_large_tree
groundcover_or_dike_cluster
```

- ห้ามข้าม Cluster เพราะมีพื้นที่มากกว่าเกณฑ์
- Cluster ใหญ่ต้องได้รับการค้นหา Crown Center หรือมีสถานะ Unresolved
- ห้ามถือ Texture หลายยอดภายในต้นใหญ่เดิมว่าเป็นต้นปลูกหลายต้น

### 2. สร้าง Crown Center จากภาพ

ใช้หลักฐานอย่างน้อยหนึ่งวิธี:

```text
local maximum of canopy probability
distance-transform peak
multiscale LoG/blob center
watershed basin center
crown-center model response
```

กฎ:

- ทุก Center ต้องทับ Local Canopy Maximum หรือ Crown-center Evidence
- ใช้หลาย Scale เมื่อเรือนยอดมีหลายขนาด
- Minimum Peak Distance ต้องมาจาก Config และ Local Spacing
- ห้ามใช้ Grid Seed สร้าง Center ในตำแหน่งที่ไม่มี Canopy Evidence
- เก็บ `center_method`, `center_score`, `source_cluster_id` และ `source_tile`

### 3. ตรวจ Local Spacing และ 2D Row Support

ใช้ Spacing ของ `grid_block_id` ที่ Center อยู่จริง ห้ามใช้ค่าเดียวทั้งแปลง

Center ที่นับได้ต้องผ่าน Config:

```text
spacing residual <= max(3 × local spacing MAD, configured tolerance)
row-angle residual <= configured angle tolerance
canopy support >= configured minimum
```

Row Support ต้องมี:

- เพื่อนบ้านตามแนวแถว และ
- เพื่อนบ้านจากแถวข้างเคียงหรือ Cross-row Support

แนวเส้นเดียวแม้ระยะสม่ำเสมอ ห้ามถือเป็น Planting Grid ที่เพียงพอ

### 4. กันถนน คันดิน และวัชพืช

ใช้หลักฐานอย่างน้อยสองประเภทก่อนตัดเป็น Road/Dike Conflict:

```text
road_or_dike_mask overlap
narrow linear corridor geometry
single-axis-only neighbor support
surface ring เป็นดิน ถนน หรือพืชคลุมดิน
อยู่นอก planting-pattern interior
แนว center ต่อเนื่องตามขอบบ่อหรือคันดิน
```

- Surface Ring ต้องสัมพันธ์กับ Local Crown Radius หรือ Local Spacing และตั้งผ่าน Config
- Dense-green Ring อย่างเดียวห้ามตัดทิ้ง
- น้ำไม่ใช่ Exclusion อัตโนมัติ หากมี Crown Center และ Pattern Support ให้เป็น `valid_tree_in_water_context`

### 5. แยกต้นใหญ่เดิม

ต้นใหญ่เดิมต้อง:

- ไม่ใช้ Fit Grid
- ไม่รวมจำนวนต้นปลูก
- ไม่แตกเป็นหลายต้นจาก Texture ภายในพุ่ม
- ทำตำแหน่งใต้พุ่มเป็น `not_observable_under_existing_tree` เมื่อประเมินไม่ได้

### 6. Deduplicate

- Deduplicate ที่ Tile Overlap ก่อน
- ภายใน Cluster เดียวกัน เก็บ Center คะแนนสูงกว่าเมื่อระยะต่ำกว่า Minimum Peak Distance
- ห้ามรวม Center คนละแถวเพียงเพราะเรือนยอดสัมผัสกัน

## Decision Classes

```text
touching_crown_confirmed
touching_crown_spacing_supported
touching_crown_probable
unresolved_merged_canopy
closed_canopy_unresolved
existing_large_tree_excluded
false_positive_on_road_or_dike
false_positive_on_groundcover
false_positive_on_shadow
duplicate_crown_center
```

การตัดสิน:

```text
Crown Center ชัด + Canopy ชัด + 2D Row Support
= touching_crown_confirmed

Crown Center ชัด + Local Spacing/Row สอดคล้อง แต่ Cross-row Support อ่อน
= touching_crown_spacing_supported

Crown Center มีหลักฐาน แต่แยกพุ่มหรือ Pattern ยังไม่แน่นอน
= touching_crown_probable

มี Merged Canopy แต่หา Center ที่มีหลักฐานไม่ได้
= unresolved_merged_canopy หรือ closed_canopy_unresolved
```

Unresolved ห้ามนับเป็น Confirmed ให้รายงาน Min–Max แยกต่างหาก

## Center Attributes

```text
center_id
plot_code
grid_block_id
source_cluster_id
center_method
final_class
count_weight
confidence
canopy_support_score
crown_center_score
local_spacing_m
spacing_residual_m
row_alignment_score
row_angle_residual_deg
cross_row_support_count
single_axis_only
surface_ring_class
water_context
road_or_dike_conflict
existing_large_tree_overlap
image_quality_status
source_tile
source_raster
review_status
```

## Cluster Attributes

```text
cluster_id
plot_code
cluster_class
cluster_area_sqm
cluster_width_m
cluster_length_m
candidate_center_count
confirmed_center_count
probable_center_count
estimated_count_min
estimated_count_max
center_coverage_ratio
unresolved_reason
grid_block_id
image_quality_status
review_status
```

## Outputs

```text
touching_crown_centers.gpkg
touching_crown_clusters_validated.gpkg
touching_crown_false_positives.gpkg
touching_crown_unresolved.gpkg
touching_crown_metrics.json
touching_crown_preview.png
```

`touching_crown_metrics.json` ต้องผ่าน `schemas/touching_crown_metrics.schema.json` และมี:

```text
input_cluster_count
large_connected_cluster_count
candidate_center_count
confirmed_count
spacing_supported_count
probable_count
unresolved_cluster_count
existing_large_tree_excluded_count
road_or_dike_excluded_count
groundcover_excluded_count
duplicate_center_removed_count
grid_created_tree_count
estimated_count_min
estimated_count_max
center_on_canopy_ratio
center_without_canopy_count
cluster_processed_ratio
unresolved_area_ratio
single_axis_rejection_count
low_resolution_cluster_count
center_spacing_residual_median
center_spacing_residual_p95
validation_status
```

ต้องมี:

```text
grid_created_tree_count = 0
center_without_canopy_count = 0
cluster_processed_ratio = 1.0
```

## QA Gate

ก่อนส่ง Skill 12b:

1. แสดง Crop ของ Cluster เรือนยอดชิดอย่างน้อย 3 ระดับความหนาแน่น
2. แสดง Crop ถนนหรือคันดินอย่างน้อย 2 จุด
3. ตรวจว่า Center อยู่บนเรือนยอดจริง ไม่อยู่ในช่องว่าง
4. ตรวจว่า Single-axis Corridor ไม่ผ่านเป็น Planting Block
5. ตรวจ Cluster ทุกก้อนว่าไม่ถูกข้ามโดยไม่มีสถานะ
6. ตรวจต้นใหญ่เดิม ความละเอียดภาพ และ Tile Overlap
7. รายงาน Confirmed, Probable, Unresolved และ Estimated Range แยกกัน

หากไม่มี Ground Truth ให้ใช้ `needs_human_review` ห้ามตั้ง `approved`

## Failure Conditions

ตั้ง `rework` เมื่อ:

- Cluster ถูกข้ามเพราะ Area Threshold
- Crown Center ถูกสร้างจาก Grid โดยไม่มี Canopy Evidence
- แนวคันดินผ่านเพราะมี Row Alignment แกนเดียว
- ต้นใหญ่เดิมถูกแตกเป็นหลายต้น
- ภาพละเอียดไม่พอแต่ยังบังคับแยก Center
- ไม่แยก Confirmed, Probable และ Unresolved
- ไม่มี Preview สำหรับ Touching Crown และ Dike False Positive
- `grid_created_tree_count` หรือ `center_without_canopy_count` มากกว่า 0

## Downstream Mapping to Skill 12b

```text
touching_crown_confirmed         → surviving_confirmed candidate
touching_crown_spacing_supported → surviving_spacing_supported candidate
touching_crown_probable          → surviving_in_merged_canopy_probable candidate
unresolved_merged_canopy         → unresolved_merged_canopy
closed_canopy_unresolved         → not_observable_closed_canopy
existing_large_tree_excluded     → existing_large_tree_excluded
false_positive_*                 → matching false_positive class
duplicate_crown_center           → duplicate_detection
```

Skill 12b เป็นผู้ตัดสิน Final Class และต้องใช้ `touching_crown_centers.gpkg` แทนการเดาจำนวนจากพื้นที่ Blob โดยคง `source_cluster_id` เพื่อย้อนตรวจได้

ตำแหน่ง Grid ที่ไม่มี Crown Center ใช้ได้เฉพาะ Missing หรือ Not Observable ห้ามเป็น Surviving

## Config Keys

อ่านค่าจาก `config/defaults.yaml` หมวด `touching_crown` ห้ามฝัง Threshold ใน Code