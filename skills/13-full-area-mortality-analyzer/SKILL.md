---
name: full-area-mortality-analyzer
description: วิเคราะห์อัตรารอดและตำแหน่งต้นหายจากผลนับที่ผ่าน Skill 12b แล้ว ห้ามใช้ Raw Detection โดยตรง
---

# Full-area Survival and Mortality Analyzer

## Scope

จัดสถานะตำแหน่งปลูกและสร้างโซนอัตรารอดจากผล Skill 12b ห้ามใช้จุดแดงดิบจาก Skill 11 และห้ามวงขอบเขตสุดท้าย

## Required Inputs

```text
validated_planted_tree_points.gpkg
planting_point_status.gpkg
planting_grid.gpkg
grid_blocks.gpkg
spacing_model.json
tree_count_validation_metrics.json
```

เริ่มได้เมื่อ:

```text
validation_status = passed หรือ passed_with_warnings
grid_refit_required = false
```

Optional:

```text
merged_canopy_clusters.gpkg
existing_large_trees_validated.gpkg
surface_condition.tif
hydrology_features.gpkg
planned_planting_points.gpkg
```

## Position Classes

```text
surviving_confirmed
surviving_spacing_supported
surviving_in_merged_canopy_probable
probable_surviving_tree
confirmed_missing
probable_missing
uncertain_position
not_observable_under_existing_tree
not_observable_shadow_or_blur
not_observable_closed_canopy
off_grid_tree_candidate
```

False Positive และ Existing Large Tree ต้องไม่เข้าสู่จำนวนต้นปลูก

## Rules

- Missing ต้องอยู่ใน Pattern/Grid Block ที่เชื่อถือได้
- ต้องไม่มี Canopy Support เพียงพอ ณ ตำแหน่ง Missing
- จุดใต้ต้นใหญ่ เงา ภาพเบลอ หรือเรือนยอดปิดเป็น Not Observable
- Background เป็นน้ำไม่ทำให้ Not Observable โดยอัตโนมัติ หากเห็นเรือนยอดยังนับได้
- แยก Confirmed, Spacing-supported, Probable และ Estimated
- คำนวณแยกตาม Pattern Block และ Zone
- ต้นใหญ่เดิมไม่รวมตัวนับหรือตัวหาร
- Grid-only Position ห้ามเป็น Surviving
- `unreliable_random_scatter_zone` ห้ามคำนวณอัตรารอด

## Zone Classes

```text
high_survival_zone
medium_survival_zone
low_survival_zone
mortality_gap_zone
merged_canopy_estimation_zone
uncertain_observation_zone
validation_failed_zone
```

Threshold ต้องอยู่ใน Config และรายงานค่าที่ใช้

## Metrics

```text
confirmed_surviving_count
spacing_supported_surviving_count
probable_merged_canopy_count
estimated_surviving_min
estimated_surviving_max
confirmed_missing_count
probable_missing_count
observable_expected_count
not_observable_count
existing_large_tree_excluded_count
false_positive_excluded_count
survival_rate_confirmed
survival_rate_including_spacing_supported
survival_rate_estimated_min
survival_rate_estimated_max
mortality_rate_confirmed
uncertain_ratio
validation_failed_area_sqm
```

## Rate Formulas

Confirmed-only:

```text
confirmed_surviving
/
(confirmed_surviving + confirmed_missing)
```

Operational estimate:

```text
(confirmed_surviving + spacing_supported_surviving)
/
(confirmed_surviving + spacing_supported_surviving + confirmed_missing + probable_missing)
```

Not Observable ไม่อยู่ในตัวหาร เว้นแต่ผู้ใช้กำหนดวิธีอื่นอย่างชัดเจน

Merged Canopy ที่ยังไม่แน่นอนต้องรายงานเป็นช่วง

## Outputs

```text
planting_position_status_final.gpkg
survival_mortality_zones.gpkg
survival_mortality_metrics.json
survival_mortality_summary.csv
survival_mortality_preview.png
```

## QA

- แจ้งเมื่อยังใช้ Raw Detection
- ตรวจ False Positive บนน้ำ/ดิน/ถนน
- ตรวจต้นใหญ่เดิมในตัวนับ
- แยก Missing จาก Not Observable
- แสดงตัวหารและสูตร
- แสดง Confirmed Count กับ Estimated Range แยกกัน
- ห้ามเรียก `probable_missing` ว่าต้นตายยืนยันโดยไม่มี Ground Truth
