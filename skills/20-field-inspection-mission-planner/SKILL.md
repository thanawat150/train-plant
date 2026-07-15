---
name: field-inspection-mission-planner
description: รวมคะแนนความจำเป็นต้องตรวจจาก Skill 18 กับภาระการเข้าถึงทางบก/ทางเรือจาก Skill 19 เพื่อเลือกจุดตรวจ จัดกลุ่มภารกิจ วางแผนวัน ทีม เส้นทาง และรายการข้อมูลที่ต้องเก็บ
---

# Field Inspection Mission Planner

## หน้าที่

เปลี่ยน Candidate Inspection Points และ Route Options ให้เป็นแผนปฏิบัติงานภาคสนามที่ทำได้จริง โดยรักษาสองแกนแยกกัน:

1. `evidence_priority_score` — จำเป็นต้องตรวจมากเพียงใด
2. `access_burden_score` — เข้าถึงยากเพียงใด

ห้ามใช้สูตรเดียวที่ทำให้จุดสำคัญแต่ไกลถูกตัดออกโดยอัตโนมัติ

## Inputs

- `inspection_candidate_points.gpkg`
- `inspection_priority_summary.csv`
- `inspection_route_options.gpkg`
- `inspection_access_assessment.csv`
- จุดเริ่มต้นทีม ฐานปฏิบัติงาน ที่จอดรถ และท่าเรือ
- จำนวนทีม คนต่อทีม เวลาทำงานต่อวัน และพาหนะ
- เวลาเฉลี่ยในการตรวจต่อจุด
- Tide Window, Permission และ Safety Constraint เมื่อมี

## Decision Matrix

| Evidence Priority | Access Burden | Action |
|---|---|---|
| สูง | ต่ำ/กลาง | `inspect_now` |
| สูง | สูงมาก/ไม่ทราบ | `special_mission` หรือ `route_scout_first` |
| ปานกลาง | ต่ำ | `bundle_opportunistically` |
| ปานกลาง | สูง | `remote_cluster_mission` เมื่อรวมหลายจุดคุ้มค่า |
| ต่ำ | ต่ำ | `qa_control_or_opportunistic` |
| ต่ำ | สูง | `defer_or_remote_monitor` เว้นแต่ขาดตัวแทนพื้นที่ |

ระยะไกลต้องเพิ่มการพิจารณาเรื่อง Logistics, Safety, Tide และการรวมหลายจุด ไม่ใช่ลดความสำคัญทางหลักฐานทันที

## Mission Selection Rules

1. รักษาจุด P1 ทุกจุดไว้ในแผนหรือ Backlog พร้อมเหตุผล
2. จุด P1/P2 ที่ไกลให้รวมกับจุดใกล้เคียงเพื่อเฉลี่ยต้นทุนการเดินทาง
3. สร้าง Mission แยกสำหรับทางบก ทางเรือ และแบบผสมเมื่อเหมาะสม
4. จำกัดเวลาเดินทาง + เวลาตรวจ + Buffer ไม่เกินเวลาทำงานต่อวัน
5. เพิ่ม Buffer สำหรับน้ำขึ้นน้ำลง เลนลึก การเปลี่ยนพาหนะ และสภาพอากาศ
6. เลือกจุดตัวแทนของแต่ละ Stratum เช่น Survival Class, Surface Zone และ Gap Class
7. ห้ามเลือกแต่จุดผิดปกติ ต้องมี High-confidence QA Control ตามสัดส่วนที่กำหนด
8. ลดจุดซ้ำที่ตรวจคำถามเดียวกันในบริเวณใกล้กัน แต่รักษาจุดคนละสาเหตุ
9. จุดเข้าถึงไม่ทราบให้สร้าง Mission สำรวจทางเข้าแยกก่อน
10. จุดที่ต้องขออนุญาตต้องไม่ถูกจัดวันจนกว่าสถานะ Permission พร้อม
11. Route ทางเรือต้องอยู่ใน Tide Window และมีแผนกลับออก
12. ระบุ Alternate Route หรือ Abort Condition สำหรับ Mission เสี่ยงสูง

## Mission Classes

```text
inspect_now
special_mission
remote_cluster_mission
route_scout_first
bundle_opportunistically
qa_control_mission
defer_or_remote_monitor
blocked_pending_permission
```

## Mission Value

ใช้คะแนนเพื่อจัดลำดับภารกิจ ไม่ใช้ตัดสินสถานะต้นไม้:

```text
mission_value =
  evidence_value
+ coverage_bonus
+ remote_cluster_bonus
+ multi_issue_bonus
- redundancy_penalty
```

`access_burden` ใช้คำนวณทรัพยากร เวลา และชนิด Mission แยกจาก `mission_value`

## Remote-area Rules

พื้นที่ไกลต้องพิจารณาเพิ่มดังนี้:

- มีจุดสำคัญกี่จุดในรัศมีเดียวกัน
- สามารถรวมตรวจหลายสาเหตุในเที่ยวเดียวหรือไม่
- มีทางเรือที่ลดเวลาได้หรือไม่
- ต้องค้างพื้นที่หรือไม่
- มี Tide Window จำกัดหรือไม่
- มีจุดขึ้นฝั่งและทางเดินช่วงสุดท้ายหรือไม่
- มีพื้นที่ที่ไม่เคยถูกตรวจจนเกิด Sampling Bias หรือไม่
- ควรบินโดรนซ้ำก่อนส่งคนหรือไม่

หากจุดไกลมีความสำคัญต่ำและไม่มี Coverage Gap สามารถเลื่อนได้ แต่ต้องเก็บใน Backlog ไม่ลบทิ้ง

## Field Package per Point

ทุกจุดต้องระบุ:

```text
inspection_id
mission_id
priority_class
inspection_reason
question_to_answer
expected_observations
primary_access_mode
route_distance_m
travel_time_min
tide_window
safety_note
photo_requirements
gps_accuracy_requirement
field_form_id
```

## Minimum Field Checklist

- ยืนยันตำแหน่งและ GPS Accuracy
- ถ่ายภาพภาพรวมและอย่างน้อย 4 ทิศเมื่อทำได้
- ยืนยันชนิดพืชหรือสิ่งปกคลุม
- ยืนยันรอด ตาย หาย หรือมองไม่เห็น
- บันทึกจำนวนต้นในรัศมีตรวจ
- บันทึกน้ำ เลน วัชพืช การกัดเซาะ และสิ่งกีดขวาง
- ตรวจความถูกต้องของขอบเขตหรือ Grid เมื่อเกี่ยวข้อง
- บันทึกทางเข้าจริง จุดขึ้นฝั่ง และเวลาที่ใช้
- บันทึกเหตุผลเมื่อไม่สามารถตรวจได้

## Required Outputs

- `selected_inspection_points.gpkg`
- `field_missions.gpkg`
- `mission_routes.gpkg`
- `mission_day_plan.csv`
- `inspection_backlog.csv`
- `field_checklist.csv`
- `mission_summary.json`
- `field_mission_preview.png`
- `route_cards/`

## Required Mission Attributes

```text
mission_id
mission_class
access_mode
start_point_id
point_count
p1_point_count
estimated_distance_m
estimated_travel_time_min
estimated_inspection_time_min
estimated_total_time_min
team_size
vehicle_or_boat_required
tide_window
permission_status
safety_level
alternate_route_available
abort_conditions
mission_value
access_burden_mean
status
```

## Checkpoints

หยุดให้ผู้ใช้ตรวจ:

1. Candidate Points และเหตุผล
2. Access Route Options
3. Mission Grouping และจุดที่ถูกเลื่อน
4. Day Plan ก่อน Export ขั้นสุดท้าย

## Warnings

```text
mission_over_capacity
high_priority_point_deferred
remote_sampling_gap
route_scout_required
missing_tide_information
permission_not_ready
no_alternative_route
unsafe_return_window
insufficient_qa_control_points
```

## Restrictions

- ห้ามให้ AI รับรองความปลอดภัยของเส้นทาง
- ห้ามถือ Route Candidate ว่าใช้งานได้จริงจนทีมพื้นที่ยืนยัน
- ห้ามลบจุดสำคัญเพียงเพราะไกล
- ห้ามจัดตารางโดยไม่รวมเวลาไป–กลับและ Buffer
- ห้ามตั้ง Mission เป็น `approved`; มนุษย์ต้องยืนยันแผนปฏิบัติงาน
