---
name: field-inspection-priority-analyzer
description: วิเคราะห์ผลนับที่ผ่าน Skill 21 กริด อัตรารอด ขอบเขต ความมั่นใจ สภาพผิว และ QA เพื่อเลือกจุดที่ควรเข้าตรวจ พร้อมเหตุผลและคะแนนความจำเป็น โดยยังไม่วางเส้นทาง
---

# Field Inspection Priority Analyzer

## หน้าที่

เปลี่ยนผลวิเคราะห์เป็น `จุดที่ควรเข้าตรวจ` และ `โซนที่ควรเข้าตรวจ` โดยตอบว่า:

- จุดนี้ต้องตรวจเพราะอะไร
- ต้องยืนยันข้อมูลอะไร
- ความสำคัญสูงเพราะผลกระทบหรือความไม่แน่นอนด้านใด
- เป็นจุดบังคับตรวจ จุดตัวแทน หรือจุด QA ควบคุม

Skill นี้ไม่สร้างเส้นทางและไม่ตัดจุดทิ้งเพียงเพราะอยู่ไกล ให้ Skill 19 ประเมินการเข้าถึงภายหลัง

## Inputs

สำหรับปลูกเต็ม ให้ใช้ผล Validated ก่อน Raw Detection:

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
```

Optional:

```text
surface_condition.tif
hydrology_features.gpkg
land_cover_confidence.tif
planned_planting_points.gpkg
ภาพหรือผลหลายช่วงเวลา
```

ปลูกเสริมยังใช้ `enrichment_gap_candidates.gpkg` และ `uncertain_enrichment_boundary.gpkg` ได้

ห้ามใช้ `planted_tree_candidates.gpkg` จาก Skill 11 เป็นผลนับสุดท้าย

## Inspection triggers

สร้าง Candidate Point/Zone เมื่อพบอย่างน้อยหนึ่งเงื่อนไข:

1. กลุ่ม `confirmed_missing` หรือ `probable_missing` ต่อเนื่อง
2. อัตรารอดต่ำกว่าค่าที่ผู้ใช้กำหนด
3. `not_observable` จำนวนมาก โดยเฉพาะใต้ต้นใหญ่หรือเรือนยอดปิด
4. Grid/Pattern กับต้น Validated ไม่สอดคล้องกัน
5. `unreliable_random_scatter_zone` หรือ `tree_count_validation_failed`
6. False Positive บนดิน น้ำ ถนน เงา หรือวัชพืชจำนวนมาก
7. Merged Canopy ที่ Estimated Range กว้างหรือยัง Unresolved
8. ต้นใหญ่เดิมกับกลุ่มต้นปลูกชิดกันจนแยก Class ไม่แน่ใจ
9. `off_grid_tree_candidate` หลายจุด
10. ขอบเขตมี Confidence ต่ำหรืออาจถูก False Positive ขยาย
11. พื้นที่น้ำขัง ร่องน้ำ เลน หรือ Surface Zone สัมพันธ์กับการรอดต่ำ
12. ผลต่างจากภาพครั้งก่อนอย่างผิดปกติ
13. พื้นที่มีผลกระทบสูง เช่น ครอบคลุมต้นจำนวนมากหรือพื้นที่กว้าง
14. โซนที่ยังไม่มี Ground Truth
15. High-confidence QA Control เพื่อตรวจ False Positive/False Negative
16. พื้นที่ห่างไกลที่ยังไม่มีตัวแทนตรวจและอาจเกิด Sampling Bias

## Point types

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

## Evidence priority score

คำนวณ `evidence_priority_score` ช่วง 0–100 แยกจากภาระการเดินทาง

ค่าเริ่มต้นที่ปรับได้:

```text
uncertainty_or_low_confidence   25
anomaly_or_model_disagreement  20
impact_area_or_tree_count      20
survival_or_mortality_risk     15
temporal_change                10
sampling_representativeness    10
```

กฎ:

- คะแนนสูงหมายถึงจำเป็นต้องตรวจมาก ไม่ได้แปลว่าเข้าถึงง่าย
- ห้ามลดคะแนนเพียงเพราะจุดอยู่ไกล
- จุดไกลและจำเป็นสูงให้เป็น `special_logistics_candidate`
- ลดจุดซ้ำด้วย Minimum Spacing แต่ห้ามรวมคนละสาเหตุโดยไม่เก็บเหตุผล

## Priority classes

```text
P1_critical        80–100
P2_high            60–79
P3_representative  40–59
P4_opportunistic   0–39
```

จุด P1 ต้องไม่ถูกตัดออกอัตโนมัติจากข้อจำกัดระยะทาง

## Point placement rules

- วางจุดในตำแหน่งที่ตอบคำถามได้จริง ไม่วางกลาง Polygon โดยอัตโนมัติ
- กลุ่มต้นหายให้วางจุดในพื้นที่หาย จุดรอยต่อ และ Control ในพื้นที่รอดดี
- Random Scatter ให้เลือกทั้งจุดที่นับผิดบนพื้นโล่งและต้นจริงที่ระบบตกหล่น
- Merged Canopy ให้เลือกจุดที่มี Expected Centers หลายจุดและพื้นที่ที่แยกได้ชัดเป็น Reference
- ต้นใหญ่ให้วางจุดขอบพุ่มเพื่อยืนยันชนิด/ขนาดและดูต้นปลูกใต้เรือนยอด
- ขอบเขตไม่มั่นใจให้วางบน Segment ที่ Confidence ต่ำ
- Surface/Hydrology ให้เลือกแต่ละ Stratum และจุดเปลี่ยนผ่าน
- เพิ่ม High-confidence Control ตามสัดส่วนที่กำหนด
- เก็บ `source_zone_id` และ `source_issue_id` ทุกจุด

## Required attributes

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

## Field observations to request

- ยืนยันมีต้น/ไม่มีต้นและตำแหน่งศูนย์กลางจริง
- ยืนยันว่า Raw Detection เป็น False Positive หรือไม่
- ยืนยันจำนวนต้นใน Merged Canopy
- ยืนยันต้นใหญ่เดิมกับต้นปลูก
- ชนิดพืช
- สถานะรอด ตาย หาย หรือมองไม่เห็น
- ความสูงและขนาดต้นโดยประมาณ
- ภาพรวม ภาพ 4 ทิศ และภาพจุด GPS
- สภาพน้ำ เลน วัชพืช การกัดเซาะ และสิ่งกีดขวาง
- ความถูกต้องของแนว/ระยะปลูกและขอบเขต
- การเข้าถึงจริงและข้อจำกัดหน้างาน

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
- ห้ามใช้ระยะเส้นตรงเป็นตัวแทนการเข้าถึง
- ห้ามเรียกจุดว่าเข้าถึงไม่ได้จนกว่า Skill 19 จะตรวจ Network
- ห้ามสรุปสาเหตุการตายจากภาพเพียงอย่างเดียว
- ห้ามใช้ Raw Detection เป็นจำนวนต้นเพื่อจัดลำดับผลกระทบ
- ผลทั้งหมดเป็น Candidate Inspection Plan ต้องผ่าน Human Review
