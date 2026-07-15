---
name: model-calibration-evaluator
description: ประเมินความแม่นยำของ Detection, Spacing Validation, Missing Position และ Survival จาก Ground Truth พร้อมเสนอ Threshold/Config แบบมีหลักฐาน โดยไม่ปรับโมเดลหรืออนุมัติค่าใหม่เอง
---

# Model Calibration and Evaluation

## หน้าที่

ตอบให้ได้ว่า Workflow นับต้นแม่นแค่ไหน ผิดในบริบทใด และควรปรับอะไร ก่อนใช้กับแปลงอื่นหรือรายงานผลอย่างเป็นทางการ

## Inputs

```text
ground_truth_observations.gpkg
ground_truth_tree_points.gpkg
ground_truth_planting_positions.gpkg
validated_planted_tree_points.gpkg
planting_point_status.gpkg
false_positive_detections.gpkg
merged_canopy_clusters.gpkg
survival_mortality_metrics.json
surface_condition.tif เมื่อมี
planting_pattern_blocks.gpkg
analysis_manifest.json
config/defaults.yaml หรือ config ที่ใช้จริง
```

## Evaluation Split Rules

- แยก Train/Calibration/Test ตาม Plot, Pattern Block หรือวันที่ ห้ามสุ่มจุดใกล้กันปะปนจนเกิด Spatial Leakage
- รายงานจำนวน Ground Truth และ Coverage ต่อ Class
- ห้ามสรุปค่ารวมเพียงค่าเดียวเมื่อบริบทต่างกันมาก
- แยกผลอย่างน้อยตาม:
  - isolated crown
  - touching/merged canopy
  - water context
  - bare ground / road / shadow
  - existing large tree
  - planting pattern class
  - crown size class
  - image quality / tile seam เมื่อมี

## Detection Metrics

```text
precision
recall
f1_score
false_positive_rate
false_negative_rate
position_error_mean_m
position_error_p95_m
duplicate_rate
```

## Counting Metrics

```text
count_error
absolute_count_error
percentage_count_error
estimated_range_coverage
merged_canopy_count_error
block_level_count_bias
```

## Position and Survival Metrics

```text
surviving_precision
surviving_recall
missing_precision
missing_recall
not_observable_accuracy
survival_rate_bias
survival_rate_absolute_error
mortality_rate_bias
```

## Context Error Metrics

```text
false_positive_on_water_rate
false_positive_on_bare_ground_rate
false_positive_on_road_rate
false_positive_on_shadow_rate
existing_large_tree_inclusion_rate
grid_only_surviving_rate
random_scatter_acceptance_rate
merged_canopy_unresolved_rate
```

## Calibration Rules

1. ประเมิน Threshold ทีละกลุ่ม เช่น Canopy Support, Spacing Match, Large-tree Ratio, Random Scatter และ Matching Distance
2. ห้ามเลือก Threshold จาก Test Set แล้วรายงานผลบน Test Set เดิม
3. แสดง Trade-off Precision vs Recall
4. Threshold ที่ลด False Positive แต่อาจเพิ่ม False Negative ต้องมี Warning
5. แนะนำค่าแยกตาม Pattern Block หรือ Image Quality ได้เมื่อมีหลักฐานเพียงพอ
6. หาก Ground Truth ไม่ครอบคลุม Class สำคัญ ให้รายงาน `insufficient_ground_truth`
7. ห้ามปรับ Config Production โดยอัตโนมัติ

## Required Outputs

```text
model_evaluation.json
model_evaluation_summary.csv
context_error_breakdown.csv
threshold_recommendations.yaml
calibration_curves.csv
error_samples.gpkg
false_positive_samples.gpkg
false_negative_samples.gpkg
calibration_report.md
calibration_preview.png
```

## Required Recommendation Fields

```text
parameter_name
current_value
recommended_value
valid_context
supporting_sample_count
expected_precision_change
expected_recall_change
risk
confidence
requires_human_approval
```

## Acceptance Guidance

ค่าเป้าหมายต้องกำหนดตาม Use Case เช่น:

- สำรวจเพื่อคัดจุดเข้าตรวจ สามารถยอมรับ Recall สูงและ False Positive บางส่วน
- ใช้คำนวณอัตรารอด ต้องควบคุม False Positive และ Missing/Not-observable Confusion เข้มกว่า
- ใช้ตรวจรับผู้รับเหมา ต้องมี Ground Truth, Version และ Human Review ที่เข้มงวด

ห้ามฝังค่า Precision/Recall เป้าหมายเดียวเป็นมาตรฐานทุกโครงการ

## Outputs to Workflow

เมื่อผู้ใช้อนุมัติ Recommendation แล้ว จึงสร้าง Config Version ใหม่ เช่น:

```text
config/calibrated/<project_or_model_version>.yaml
```

ต้องบันทึก:

```text
parent_config
approval_by
approval_date
ground_truth_version
evaluation_version
change_summary
```

## Restrictions

- ห้ามรายงาน Accuracy จาก Sample ที่ใช้ปรับ Threshold ชุดเดียวกันโดยไม่เปิดเผย
- ห้ามใช้จำนวนต้นตามสัญญาเป็น Ground Truth
- ห้ามปรับผลให้ตรงจำนวนที่คาด
- ห้ามซ่อน Class ที่ไม่มีตัวอย่าง
- ห้ามอ้างว่าโมเดลพร้อม Production หากยังไม่มี Independent Test Set
- AI เสนอค่าได้ แต่ผู้รับผิดชอบต้องอนุมัติ Config ใหม่