---
name: field-feedback-ground-truth-ingestor
description: รับผลตรวจภาคสนามที่มนุษย์ยืนยันแล้ว เชื่อมกลับกับจุดต้น จุดปลูก ความผิดปกติ และเส้นทาง เพื่อสร้าง Ground Truth แบบมี Version โดยไม่ทับผล AI เดิม
---

# Field Feedback and Ground Truth Ingestor

## หน้าที่

ปิดวงจรจาก Candidate Inspection Plan กลับสู่ข้อมูลที่ใช้ตรวจความแม่นยำและปรับ Workflow

Skill นี้ไม่ฝึกโมเดลและไม่ปรับ Threshold เอง ทำหน้าที่ตรวจ รับเข้า และจัด Version ของข้อมูลที่มนุษย์ยืนยัน

## Inputs

```text
selected_inspection_points.gpkg หรือ inspection_candidate_points.gpkg
field_checklist.csv
field_observation_form.csv / gpkg / geojson
field_photos หรือ photo references
GPS tracks เมื่อมี
validated_planted_tree_points.gpkg
planting_point_status.gpkg
false_positive_detections.gpkg
existing_large_trees_validated.gpkg
analysis_manifest.json
project_manifest.json
```

## Required Human Provenance

ทุก Observation ต้องมี:

```text
observer_name
observation_datetime
source_device_or_form
gps_accuracy_m
inspection_id หรือ source_position_id
human_review_status
```

สถานะ Human:

```text
reviewed_confirmed
reviewed_corrected
reviewed_unresolved
rejected_observation
```

AI ห้ามสร้างสถานะ Human เหล่านี้แทนผู้ตรวจ

## Observation Classes

```text
planted_tree_alive
planted_tree_dead
planting_position_empty
planting_not_performed
natural_tree
existing_large_tree
nypa_palm
other_palm_or_coconut
weed_or_groundcover
open_water
bare_ground
not_observable
wrong_location
unknown
```

`planted_tree_dead`, `planting_position_empty` และ `planting_not_performed` ต้องแยกกันเมื่อภาคสนามยืนยันได้

## Matching Rules

1. จับคู่ Observation กับ `inspection_id`, `position_id`, `planned_point_id` หรือ Geometry ใกล้สุดตามลำดับ
2. ใช้ระยะ Matching ตาม GPS Accuracy และ Config ไม่ใช้ค่าคงที่เดียวทุกอุปกรณ์
3. หากมีหลาย Candidate ในระยะเดียวกัน ให้เป็น `ambiguous_match`
4. ห้ามย้ายจุด AI ไปตรงจุดภาคสนามโดยไม่เก็บ Original Geometry
5. เก็บค่าก่อนแก้และหลังแก้ทุก Field
6. ภาพประกอบต้องเก็บ Reference, เวลา และตำแหน่งเมื่อมี
7. ข้อมูลคนละวัน/คนละรอบสำรวจต้องเป็นคนละ Observation Version
8. จุดที่ตรวจไม่ถึงต้องเก็บเป็น `field_not_reached` พร้อมเหตุผล ไม่ถือเป็น Ground Truth ของสถานะต้น

## Correction Types

```text
confirm_ai_class
change_class
add_false_negative
remove_false_positive
resolve_merged_canopy_count
confirm_existing_large_tree
correct_planting_point
confirm_missing_position
mark_not_observable
correct_access_route
unresolved
```

## Outputs

```text
ground_truth_observations.gpkg
ground_truth_tree_points.gpkg
ground_truth_planting_positions.gpkg
field_route_feedback.gpkg
label_change_log.csv
ambiguous_feedback.gpkg
unresolved_feedback.gpkg
feedback_manifest.json
field_feedback_summary.csv
```

## Required Attributes

```text
observation_id
inspection_id
position_id
plot_code
observation_class
previous_ai_class
correction_type
human_review_status
observer_name
observation_datetime
gps_accuracy_m
photo_refs
notes
source_analysis_version
feedback_version
```

## QA

- ตรวจ Duplicate Observation
- ตรวจ GPS Point อยู่นอกแปลงผิดปกติ
- ตรวจ Missing Observer/Date
- ตรวจภาพกับ Point/Time ไม่สอดคล้อง
- ตรวจ Human Class ที่อยู่นอก Taxonomy
- ตรวจ Observation ที่จับคู่กับ AI ไม่ได้
- ตรวจว่าการแก้ไขไม่ทับ Raw Output

## Downstream Contract

ส่ง Ground Truth ให้ Skill 22 ประเมิน Precision/Recall, Count Error, Survival Bias และ Threshold Recommendation

## Restrictions

- ห้ามเปลี่ยนผล AI เดิมแบบ In-place
- ห้ามถือจุดที่ทีมเข้าไม่ถึงเป็นต้นตายหรือไม่มีต้น
- ห้ามใช้ Observation ที่ไม่มีผู้ตรวจและวันที่เป็น Ground Truth ระดับสูง
- ห้ามนำภาพภาคสนามที่ไม่ทราบตำแหน่งไปจับคู่แบบบังคับ
- ทุกการแก้ต้องย้อนกลับได้ผ่าน Change Log