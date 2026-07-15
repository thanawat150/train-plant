---
name: field-inspection-priority-analyzer
description: วิเคราะห์ผลจากการนับต้น กริด อัตรารอด ขอบเขต ความมั่นใจ สภาพผิว และ QA เพื่อเลือกจุดที่ควรเข้าตรวจภาคสนาม พร้อมเหตุผลและคะแนนความจำเป็น โดยยังไม่วางเส้นทาง
---

# Field Inspection Priority Analyzer

## หน้าที่

เปลี่ยนผลวิเคราะห์ภาพให้เป็น `จุดที่ควรเข้าตรวจ` และ `โซนที่ควรเข้าตรวจ` โดยตอบให้ได้ว่า:

- จุดนี้ต้องตรวจเพราะอะไร
- ต้องยืนยันข้อมูลอะไร
- ความสำคัญสูงเพราะผลกระทบหรือความไม่แน่นอนด้านใด
- เป็นจุดบังคับตรวจ จุดตัวแทน หรือจุด QA ควบคุม

Skill นี้ไม่สร้างเส้นทางและไม่ตัดจุดทิ้งเพียงเพราะอยู่ไกล ให้ Skill 19 ประเมินภาระการเข้าถึงภายหลัง

## Inputs

อ่านเฉพาะผลที่มีอยู่ เช่น:

- `planted_tree_candidates.gpkg`
- `planting_grid.gpkg`
- `expected_missing_positions.gpkg`
- `survival_mortality_zones.gpkg`
- `enrichment_gap_candidates.gpkg`
- `boundary_segment_confidence.gpkg`
- `uncertain_enrichment_boundary.gpkg`
- `surface_condition.tif`
- `hydrology_features.gpkg`
- `land_cover_confidence.tif`
- `qa_report.json`
- ภาพหรือผลหลายช่วงเวลา เมื่อมี

## Inspection Triggers

สร้าง Candidate Point/Zone เมื่อพบอย่างน้อยหนึ่งเงื่อนไข:

1. กลุ่มต้นหายหรือต้นตายต่อเนื่อง
2. อัตรารอดต่ำกว่าค่าที่ผู้ใช้กำหนด
3. `probable_missing`, `uncertain` หรือ `not_observable` จำนวนมาก
4. Grid กับตำแหน่งต้นที่ตรวจพบไม่สอดคล้องกัน
5. ขอบเขตมี Confidence ต่ำหรือมี Warning
6. โกงกางสับสนกับวัชพืช ต้นจาก ปาล์ม/มะพร้าว หรือพืชธรรมชาติ
7. พื้นที่น้ำขัง ร่องน้ำ เลน หรือ Surface Zone สัมพันธ์กับการรอดต่ำ
8. ผลต่างจากภาพครั้งก่อนอย่างผิดปกติ
9. พื้นที่มีผลกระทบสูง เช่น ครอบคลุมต้นจำนวนมากหรือพื้นที่กว้าง
10. โซนที่ยังไม่มี Ground Truth หรือตัวอย่างภาคสนาม
11. จุด QA ตัวแทนในพื้นที่ Confidence สูง เพื่อประเมิน False Positive/False Negative
12. พื้นที่ห่างไกลที่ยังไม่มีตัวแทนตรวจและอาจเกิด Sampling Bias

## Point Types

```text
mandatory_issue_check
mortality_cluster_check
missing_tree_validation
low_survival_zone_check
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

คำนวณคะแนน `evidence_priority_score` ช่วง 0–100 แยกจากภาระการเดินทาง

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
- จุดที่ไกลแต่มีความจำเป็นสูงต้องคงไว้และส่งต่อเป็น `special_logistics_candidate`
- ลดความซ้ำซ้อนด้วย Minimum Spacing แต่ห้ามรวมคนละสาเหตุเข้าด้วยกันโดยไม่เก็บเหตุผล

## Priority Classes

```text
P1_critical        80–100
P2_high            60–79
P3_representative  40–59
P4_opportunistic   0–39
```

จุด P1 ต้องไม่ถูกตัดออกอัตโนมัติจากข้อจำกัดระยะทาง

## Point Placement Rules

- วางจุดในตำแหน่งที่ตรวจสาเหตุได้จริง ไม่วางกลาง Polygon โดยอัตโนมัติ
- กลุ่มต้นตายให้วางทั้งจุดกึ่งกลางกลุ่มและจุดรอยต่อกับพื้นที่รอดดีเมื่อจำเป็น
- ขอบเขตไม่มั่นใจให้วางบน Segment ที่ Confidence ต่ำ
- Surface/Hydrology ให้เลือกจุดตัวแทนแต่ละ Stratum และจุดเปลี่ยนผ่าน
- พื้นที่เรือนยอดปิดให้เลือกจุดเข้าถึงขอบโซน ไม่วางลึกจนไม่มีทางเข้าโดยไม่จำเป็น
- เพิ่ม Control Point ในพื้นที่ที่ระบบมั่นใจสูงอย่างน้อยตามสัดส่วนที่ผู้ใช้กำหนด
- เก็บ `source_zone_id` และ `source_issue_id` ทุกจุด

## Required Attributes

```text
inspection_id
plot_code
source_zone_id
point_type
inspection_reason
evidence_priority_score
priority_class
uncertainty_score
impact_score
mortality_risk_score
change_score
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

## Field Observations to Request

กำหนดตามเหตุผลของจุด เช่น:

- ยืนยันมีต้น/ไม่มีต้น
- ชนิดพืช
- สถานะรอด ตาย หรือมองไม่เห็น
- ความสูงและขนาดต้นโดยประมาณ
- สาเหตุผิดปกติที่สังเกตได้
- ภาพ 4 ทิศและภาพจุด GPS
- สภาพน้ำ เลน วัชพืช และการกัดเซาะ
- ความถูกต้องของขอบเขต
- การเข้าถึงจริงและข้อจำกัดหน้างาน

## Outputs

- `inspection_priority_zones.gpkg`
- `inspection_candidate_points.gpkg`
- `inspection_priority_summary.csv`
- `inspection_reasons.json`
- `inspection_priority_preview.png`

## Warnings

```text
insufficient_analysis_input
no_ground_truth_zone
high_priority_remote_unrouted
sampling_bias_warning
redundant_points_warning
large_unobservable_area_warning
low_confidence_priority_warning
```

## Restrictions

- ห้ามเลือกเฉพาะจุดใกล้ทางเข้า
- ห้ามใช้ระยะเส้นตรงเป็นตัวแทนการเข้าถึง
- ห้ามเรียกจุดว่าเข้าถึงไม่ได้จนกว่า Skill 19 จะตรวจ Network
- ห้ามสรุปสาเหตุการตายจากภาพเพียงอย่างเดียว
- ผลทั้งหมดเป็น Candidate Inspection Plan ต้องผ่าน Human Review
