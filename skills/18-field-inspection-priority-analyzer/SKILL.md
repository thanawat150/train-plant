---
name: field-inspection-priority-analyzer
description: เลือกจุดที่ควรเข้าตรวจจากผล Validated ของ Skill 12b, Survival, Boundary และ QA พร้อมเหตุผลและคะแนนความจำเป็น โดยยังไม่วางเส้นทาง
---

# Field Inspection Priority Analyzer

## หน้าที่

สร้าง `จุดที่ควรเข้าตรวจ` และ `โซนที่ควรเข้าตรวจ` โดยตอบว่า:

- ตรวจเพราะอะไร
- ต้องยืนยันคำถามใด
- มีผลกระทบหรือความไม่แน่นอนเท่าไร
- เป็นจุดปัญหา จุดตัวแทน หรือ QA Control

Skill นี้ไม่สร้าง Route และไม่ตัดจุดเพราะอยู่ไกล

## Inputs

Full-area ใช้ผล Validated:

```text
validated_planted_tree_points.gpkg
planting_point_status.gpkg
false_positive_detections.gpkg
existing_large_trees_validated.gpkg
merged_canopy_clusters.gpkg
planting_pattern_blocks.gpkg
survival_mortality_zones.gpkg
boundary_segment_confidence.gpkg
qa_report.json
project_manifest.json
```

ห้ามใช้ `planted_tree_candidates.gpkg` จาก Skill 11 เป็นจำนวนสุดท้าย

Enrichment ใช้:

```text
enrichment_gap_candidates.gpkg
uncertain_enrichment_boundary.gpkg
qa_report.json
```

Optional: Surface/Hydrology, Planned Points, ภาพหลายช่วงเวลา และ Ground Truth เดิม

## Inspection Triggers

1. กลุ่ม Confirmed/Probable Missing ต่อเนื่อง
2. อัตรารอดต่ำ
3. Not Observable จำนวนมาก
4. Pattern กับต้น Validated ไม่สอดคล้อง
5. Random Scatter หรือ Validation Failed
6. False Positive บนดิน น้ำ ถนน เงา หรือวัชพืชสูง
7. Merged Canopy ที่ช่วงประมาณกว้าง
8. ต้นใหญ่กับต้นปลูกแยกไม่ชัด
9. Off-grid Tree หลายจุด
10. Boundary Confidence ต่ำหรืออาจถูก False Positive ขยาย
11. Surface/Hydrology สัมพันธ์กับการรอดต่ำ
12. ผลต่างตามเวลาผิดปกติ
13. พื้นที่ผลกระทบสูง
14. โซนไม่มี Ground Truth
15. High-confidence QA Control
16. พื้นที่ไกลที่ยังไม่มีตัวแทนตรวจ

## Point Types

```text
mandatory_issue_check
mortality_cluster_check
missing_tree_validation
low_survival_zone_check
false_positive_validation
random_scatter_recalibration_check
merged_canopy_count_check
existing_large_tree_exclusion_check
off_grid_tree_check
boundary_validation
class_confusion_check
surface_hydrology_check
closed_canopy_check
temporal_change_check
representative_stratum_sample
high_confidence_qa_control
remote_coverage_sample
```

## Evidence Priority Score

คำนวณ 0–100 แยกจาก Access Burden:

```text
uncertainty_or_low_confidence   25
anomaly_or_model_disagreement  20
impact_area_or_tree_count      20
survival_or_mortality_risk     15
temporal_change                10
sampling_representativeness    10
```

Priority:

```text
P1_critical        80–100
P2_high            60–79
P3_representative  40–59
P4_opportunistic   0–39
```

จุด P1 ห้ามตัดเพราะไกล

## Placement Rules

- วางจุดที่ตอบคำถามได้จริง ไม่ใช้ Centroid อัตโนมัติ
- กลุ่มต้นหาย: จุดในปัญหา + รอยต่อ + Control พื้นที่รอดดี
- Random Scatter: จุด False Positive + ต้นจริงที่ระบบตกหล่น
- Merged Canopy: จุด Expected Centers + Reference ที่แยกพุ่มชัด
- ต้นใหญ่: จุดขอบพุ่มเพื่อตรวจต้นปลูกใต้เรือนยอด
- Boundary: จุดบน Segment Confidence ต่ำ
- Surface: ตัวแทนแต่ละ Stratum และ Transition
- เก็บ `source_zone_id` และ `source_issue_id`

## Required Attributes

```text
inspection_id
plot_code
source_zone_id
source_issue_id
point_type
inspection_reason
question_to_answer
evidence_priority_score
priority_class
uncertainty_score
impact_score
mortality_risk_score
model_disagreement_score
representativeness_score
estimated_tree_impact
estimated_area_impact_sqm
required_observations
recommended_field_action
special_logistics_candidate
source_files
analysis_version
review_status
```

## Field Observations Requested

- มีต้น/ไม่มีต้นและ Crown Center จริง
- False Positive หรือ False Negative
- จำนวนต้นใน Merged Canopy
- ต้นใหญ่เดิมกับต้นปลูก
- ชนิดพืชและสถานะรอด/ตาย/ไม่เคยปลูก/มองไม่เห็น
- ภาพรวม ภาพ 4 ทิศ จุด GPS และ GPS Accuracy
- น้ำ เลน วัชพืช การกัดเซาะ และสิ่งกีดขวาง
- แนว/ระยะปลูก Boundary และการเข้าถึงจริง

## Outputs

```text
inspection_priority_zones.gpkg
inspection_candidate_points.gpkg
inspection_priority_summary.csv
inspection_reasons.json
inspection_priority_preview.png
```

## Warnings

```text
insufficient_analysis_input
raw_detection_used_as_final_warning
no_ground_truth_zone
high_priority_remote_unrouted
sampling_bias_warning
redundant_points_warning
large_unobservable_area_warning
merged_canopy_ground_truth_needed
random_scatter_recalibration_needed
low_confidence_priority_warning
```

## Restrictions

- ห้ามเลือกเฉพาะจุดใกล้ทางเข้า
- ห้ามใช้เส้นตรงแทน Route
- ห้ามสรุปสาเหตุการตายจากภาพ
- ห้ามใช้ Raw Detection จัดลำดับผลกระทบ
- ผลเป็น Candidate Inspection Plan ต้องผ่าน Human Review
