---
name: full-area-qa-human-review
description: ตรวจผลแปลงปลูกเต็มตั้งแต่ Raw Detection, Skill 12b Validation, Survival และ Boundary พร้อมสร้าง Warning Layers และ Human Review Package
---

# Full-area QA and Human Review

## Scope

ตรวจผลจาก Skill 11, 12, 12b, 13 และ 14 ห้ามตรวจต้นใหม่ Fit Grid ใหม่ หรือแก้ผลให้ผ่านเองโดยไม่สร้าง Rework Request

## Required Inputs

```text
project_manifest.json
planted_tree_candidates.gpkg
validated_planted_tree_points.gpkg
planting_point_status.gpkg
false_positive_detections.gpkg
existing_large_trees_validated.gpkg
merged_canopy_clusters.gpkg
planting_pattern_blocks.gpkg
spacing_model.json
tree_count_validation_metrics.json
survival_mortality_zones.gpkg
full_area_planting_evidence_boundary.gpkg
```

## Required Checks

```text
raw_detection_used_as_final_warning
preflight_warning
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
grid_refit_unresolved_warning
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

## Required Metrics

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
grid_refit_iterations
unsupported_area_ratio
off_grid_tree_count
uncertain_boundary_ratio
geometry_valid
```

## Mandatory Comparison

แสดง:

```text
Raw Detection Count
vs Validated Confirmed Count
vs Validated + Spacing-supported Count
vs Estimated Count Range
```

อธิบายส่วนต่างจาก False Positive, Existing Large Tree, Duplicate, Merged Canopy และ Not Observable

## Review Package

```text
01_preflight_summary.png หรือ md
02_raw_tree_detection_preview.png
03_reference_spacing_pattern_preview.png
04_false_positive_large_tree_preview.png
05_merged_canopy_validation_preview.png
06_final_tree_count_status_preview.png
07_survival_mortality_preview.png
08_boundary_confidence_preview.png
09_warning_preview.png
qa_report.json
warning_layers.gpkg
review_checklist.csv
analysis_manifest.json
```

## Review Status

AI:

```text
draft
needs_human_review
rework
```

Human only:

```text
reviewed
rejected
approved
```

## Acceptance Before Human Decision

- Preflight ผ่านหรือ Warning ได้รับการรับทราบ
- Source Raster, CRS, Config และ Skill Version ถูกบันทึก
- ทุกผลย้อนกลับ Tile/Window ได้
- Raw Detection ไม่ถูกใช้เป็นจำนวนสุดท้าย
- จุดบนดิน น้ำ ถนน และเงาถูกตรวจ
- ต้นใหญ่เดิมไม่อยู่ในจำนวนและไม่ใช้ Fit Grid
- Merged Canopy แยก Confirmed, Probable หรือ Min–Max
- Grid-only Position ไม่เป็น Surviving
- Grid Refit จบอย่างมีสถานะชัดเจน
- สูตรอัตรารอดและตัวหารถูกแสดง
- Missing แยกจาก Not Observable
- Boundary ใช้ผล Validated และมี Segment Confidence
- Warning ทุกข้อมีตำแหน่งและเหตุผล
