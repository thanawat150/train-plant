---
name: full-area-qa-human-review
description: ตรวจผลแปลงปลูกเต็มพื้นที่ตั้งแต่ Raw Detection, Spacing Validation, อัตรารอด และ Boundary พร้อมสร้าง Warning Layers และชุดตรวจทานสำหรับมนุษย์
---

# Full-area QA and Human Review

## Scope

ตรวจผลจาก Skill 11, 12, 21, 13 และ 14 ห้ามตรวจต้นใหม่ Fit Grid ใหม่ หรือเปลี่ยนผลให้ผ่านเองโดยไม่สร้าง Rework Request

## Required inputs

```text
planted_tree_candidates.gpkg
validated_planted_tree_points.gpkg
planting_point_status.gpkg
false_positive_detections.gpkg
existing_large_trees_validated.gpkg
merged_canopy_clusters.gpkg
planting_pattern_blocks.gpkg
spacing_model.json
survival_mortality_zones.gpkg
full_area_planting_evidence_boundary.gpkg
```

## Required checks

```text
raw_detection_used_as_final_warning
false_positive_on_bare_ground_warning
false_positive_on_water_warning
false_positive_on_road_warning
shadow_as_tree_warning
duplicate_tree_warning
large_tree_inclusion_warning
large_tree_used_for_grid_warning
merged_canopy_forced_count_warning
grid_only_surviving_warning
random_scatter_detection_warning
weak_pattern_warning
multi_pattern_forced_warning
grid_crosses_barrier_warning
missing_vs_unobservable_warning
boundary_expanded_by_false_positive_warning
boundary_too_far_warning
under_boundary_warning
large_unsupported_area_warning
tile_edge_warning
geometry_invalid_warning
low_segment_confidence_warning
```

## Required metrics

```text
raw_detection_count
validated_confirmed_count
validated_spacing_supported_count
probable_merged_canopy_count
estimated_tree_count_min
estimated_tree_count_max
false_positive_bare_ground_count
false_positive_water_count
false_positive_road_count
false_positive_shadow_count
duplicate_detection_count
existing_large_tree_count
grid_only_detection_count
random_scatter_ratio
crown_support_ratio
pattern_support_ratio
unresolved_merged_canopy_cluster_count
confirmed_missing_count
probable_missing_count
not_observable_count
survival_rate_confirmed
survival_rate_estimated_min
survival_rate_estimated_max
pattern_block_count
spacing_consistency
unsupported_area_ratio
off_grid_tree_count
uncertain_boundary_ratio
geometry_valid
```

## Mandatory comparison

แสดงอย่างน้อย:

```text
Raw Detection Count
เทียบกับ
Validated Confirmed Count
เทียบกับ
Estimated Count Range
```

หากตัวเลขต่างกันมาก ต้องอธิบายว่าเกิดจาก False Positive, Existing Large Tree, Merged Canopy, Duplicate หรือ Not Observable เท่าไร

## Review package

```text
01_raw_tree_detection_preview.png
02_reference_spacing_pattern_preview.png
03_false_positive_large_tree_preview.png
04_merged_canopy_validation_preview.png
05_final_tree_count_status_preview.png
06_survival_mortality_preview.png
07_boundary_confidence_preview.png
08_warning_preview.png
qa_report.json
warning_layers.gpkg
review_checklist.csv
analysis_manifest.json
```

## Review decisions

AI ใช้สถานะได้เพียง:

```text
draft
needs_human_review
rework
```

AI ห้ามตั้ง:

```text
reviewed
rejected
approved
```

## Acceptance before human decision

- Source Raster, CRS, Config และ Skill Version ถูกบันทึก
- ทุกผลย้อนกลับไปยัง Tile/Window ได้
- Raw Detection ไม่ถูกใช้เป็นจำนวนสุดท้าย
- จุดบนดิน น้ำ ถนน และเงาถูกตรวจ
- ต้นใหญ่เดิมไม่อยู่ในจำนวนต้นปลูกและไม่ใช้ Fit Grid
- Merged Canopy แสดง Confirmed, Probable หรือ Min–Max อย่างโปร่งใส
- Grid-only Position ไม่ถูกนับเป็น Surviving
- สูตรอัตรารอดและตัวหารถูกแสดง
- Missing Tree แยกจาก Not Observable
- Boundary ใช้ผล Validated และมี Segment Confidence
- Warning ทุกข้อมีตำแหน่งและเหตุผล
- ไม่มีการแก้ผลอัตโนมัติเพียงเพื่อให้ผ่าน QA
