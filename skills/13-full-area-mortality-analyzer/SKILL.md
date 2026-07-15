---
name: full-area-mortality-analyzer
description: วิเคราะห์อัตรารอดและช่องว่างต้นตายในแปลงปลูกเต็มพื้นที่จากผลนับที่ผ่าน Spacing/Canopy Validation แล้ว ห้ามใช้ Raw Detection โดยตรง
---

# Full-area Survival and Mortality Analyzer

## Scope

จัดสถานะตำแหน่งปลูกและสร้างโซนอัตรารอดจากผล Skill 21 ห้ามใช้จุดแดงหรือ Raw Detection จาก Skill 11 เป็นจำนวนต้น และห้ามวงขอบเขตแปลงสุดท้าย

## Required inputs

```text
validated_planted_tree_points.gpkg
planting_point_status.gpkg
planting_grid.gpkg
grid_blocks.gpkg
spacing_model.json
```

Optional:

```text
merged_canopy_clusters.gpkg
existing_large_trees_validated.gpkg
surface_condition.tif
hydrology_features.gpkg
planned_planting_points.gpkg
```

## Position classes

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

- `confirmed_missing` และ `probable_missing` ต้องอยู่ใน Pattern/Grid Block ที่เชื่อถือได้
- ต้องไม่มี Canopy Support ที่เพียงพอ ณ ตำแหน่ง Missing
- จุดใต้ต้นใหญ่ เงา ภาพเบลอ หรือเรือนยอดปิดเป็น `not_observable` ไม่ใช่ต้นตาย
- Background เป็นน้ำไม่ได้ทำให้ตำแหน่งเป็น `not_observable` โดยอัตโนมัติ หากเห็นเรือนยอดชัดยังนับเป็นต้นรอดได้
- แยก Confirmed, Spacing-supported, Probable และ Estimated ออกจากกันในรายงาน
- คำนวณแยกตาม Grid/Pattern Block และ Zone
- ต้นเดิมขนาดใหญ่ไม่รวมเป็นต้นปลูกที่รอดและไม่รวมในตัวหาร
- จุดที่ถูกกำหนดจาก Grid อย่างเดียวโดยไม่มี Canopy ต้องไม่ถูกนับเป็น Surviving
- โซน `unreliable_random_scatter_zone` ห้ามใช้คำนวณอัตรารอดจนกว่าจะ Recalibrate

## Zone classes

```text
high_survival_zone
medium_survival_zone
low_survival_zone
mortality_gap_zone
merged_canopy_estimation_zone
uncertain_observation_zone
validation_failed_zone
```

Threshold ต้องตั้งใน Config และรายงานค่าที่ใช้

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

## Rate formulas

Confirmed-only:

```text
survival_rate_confirmed =
confirmed_surviving_count
/
(confirmed_surviving_count + confirmed_missing_count)
```

Operational estimate:

```text
survival_rate_including_spacing_supported =
(confirmed_surviving_count + spacing_supported_surviving_count)
/
(confirmed_surviving_count + spacing_supported_surviving_count + confirmed_missing_count + probable_missing_count)
```

`not_observable` ต้องไม่อยู่ในตัวหาร เว้นแต่ผู้ใช้กำหนดวิธีอื่นอย่างชัดเจน

กรณี Merged Canopy ยังไม่แน่นอน ให้รายงานเป็นช่วง ไม่บังคับตัวเลขเดียว

## Outputs

```text
planting_position_status_final.gpkg
survival_mortality_zones.gpkg
survival_mortality_metrics.json
survival_mortality_summary.csv
survival_mortality_preview.png
```

## QA

- แจ้งเตือนเมื่อยังใช้ Raw Detection เป็น Input
- แจ้งเตือนเมื่อ False Positive บนน้ำ/ดินยังอยู่ในจำนวนต้น
- แจ้งเตือนเมื่อต้นใหญ่เดิมยังอยู่ในตัวนับ
- แจ้งเตือนเมื่อพื้นที่ตรวจไม่ได้มีสัดส่วนสูง
- แยก Missing Tree ออกจาก Not Observable
- แสดงตัวหารและสูตรทุกครั้ง
- แสดง Confirmed Count และ Estimated Range แยกกัน
- ห้ามเรียก `probable_missing` ว่าต้นตายยืนยันแล้วโดยไม่มี Ground Truth
