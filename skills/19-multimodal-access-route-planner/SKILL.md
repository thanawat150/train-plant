---
name: multimodal-access-route-planner
description: ประเมินการเข้าถึงจุดตรวจภาคสนามด้วยทางบก ทางเรือ และการเดินเท้าช่วงสุดท้าย โดยใช้ Network Distance เวลาเดินทาง จุดขึ้นลงเรือ ข้อจำกัดน้ำขึ้นน้ำลง และความปลอดภัย ไม่ใช้ระยะเส้นตรงแทนเส้นทางจริง
---

# Multimodal Access Route Planner

## หน้าที่

รับจุดตรวจจาก Skill 18 แล้ววิเคราะห์ว่าแต่ละจุดควรเข้าทางใด:

- ทางบก
- ทางเรือ
- ทางบกต่อเรือ
- ทางเรือต่อเดินเท้า
- หลายรูปแบบร่วมกัน
- ยังไม่ทราบทางเข้าและต้องสำรวจเส้นทางก่อน

Skill นี้ประเมิน `access_burden` แยกจาก `evidence_priority` และห้ามตัดจุดสำคัญทิ้งเพียงเพราะอยู่ไกล

## Required Inputs

- `inspection_candidate_points.gpkg`
- ขอบเขตแปลงและ AOI
- CRS แบบ Projected ที่เหมาะสม

## Recommended Access Layers

```text
road_access_lines
trail_access_lines
dike_or_embankment_lines
boat_route_lines
canal_or_tidal_channel_lines
parking_points
pier_points
boat_landing_points
plot_entry_points
mode_transfer_points
restricted_or_no_go_areas
private_access_areas
hazard_zones
```

Optional:

- ตารางน้ำขึ้นน้ำลงหรือช่วงเวลาที่เรือผ่านได้
- ความลึกน้ำหรือข้อจำกัดเรือ
- ความเร็วเดิน รถ และเรือ
- จุดจอดรถ จุดรวมพล และฐานปฏิบัติงาน
- เวลาทำงานต่อวัน
- ข้อจำกัดสิทธิ์ผ่านพื้นที่

## Route Source Rules

กำหนด `route_source` ทุกเส้น:

```text
verified_field_route
existing_gis_route
digitized_from_orthomosaic
inferred_candidate_route
unknown
```

- เส้นที่ Digitize หรืออนุมานจากภาพต้องเป็น Candidate และต้องให้คนยืนยัน
- ห้ามสร้างเส้นทางผ่านน้ำ ป่าทึบ หรือพื้นที่เอกชนเพียงเพราะเป็นเส้นตรงสั้นที่สุด
- เมื่อไม่มี Route Layer ให้รายงาน `access_network_missing` และสร้างได้เพียง Candidate Access Corridor

## Network Model

สร้าง Network แยกตาม Mode:

```text
land_vehicle
land_walk
boat
transfer_walk_to_boat
transfer_boat_to_walk
```

Cost หลักให้ใช้เวลาเดินทาง ไม่ใช้ระยะทางเพียงอย่างเดียว

```text
network_travel_time
mode_transfer_time
off_network_approach_time
tide_wait_time
permission_delay
safety_penalty
```

## Distance Rules

ต้องรายงานอย่างน้อย:

- ระยะตาม Network เที่ยวเดียว
- ระยะเดินนอก Network ช่วงสุดท้าย
- เวลาเดินทางเที่ยวเดียวและไป–กลับ
- จำนวนครั้งที่เปลี่ยนรูปแบบการเดินทาง
- ระยะจากจุดจอด/ท่าเรือ/จุดขึ้นฝั่ง

ห้ามใช้ Euclidean Distance เป็นระยะเข้าถึงจริง ยกเว้นแสดงเป็นข้อมูลเปรียบเทียบ

ค่าตั้งต้นที่ปรับได้:

```text
off_network_walk_low       0–300 m
off_network_walk_medium    300–800 m
off_network_walk_high      >800 m
one_way_time_low           0–30 min
one_way_time_medium        30–60 min
one_way_time_high          >60 min
```

ค่าเหล่านี้เป็น Default ต้องปรับตามสภาพเลน ป่า น้ำ และความสามารถทีม

## Access Burden Score

คำนวณ `access_burden_score` ช่วง 0–100 แยกจาก Evidence Priority

ค่าเริ่มต้นที่ปรับได้:

```text
travel_time             35
off_network_distance    20
mode_transfers          15
tide_dependency         15
safety_or_permission    15
```

## Access Classes

```text
A1_easy_access
A2_moderate_access
A3_difficult_access
A4_special_logistics
A5_access_unknown
```

จุด P1/P2 ที่เป็น A4 หรือ A5 ต้องส่งต่อเป็น `special_mission` ไม่ใช่ตัดออก

## Routing Rules

1. ประเมินทางบกและทางเรือเป็นทางเลือกแยกก่อน
2. สร้าง Primary Route และ Alternative Route เมื่อมี
3. ใช้จุด Transfer ที่ยืนยันได้ เช่น ท่าเรือหรือจุดขึ้นฝั่ง
4. ตรวจคลองขาด ช่วงน้ำตื้น สิ่งกีดขวาง และเส้นทางตัน
5. หากต้องเดินบนเลน ให้ลดความเร็วและเพิ่ม Safety Penalty
6. หาก Tide-dependent ให้ระบุ Time Window และความเสี่ยงตกค้าง
7. หากเส้นทางผ่านพื้นที่ต้องขออนุญาต ให้ตั้ง `permission_required`
8. หากไม่มีทางเข้าถึงที่น่าเชื่อถือ ให้สร้าง `route_scout_required`
9. จุดไกลหลายจุดที่อยู่ใกล้กันควรส่งข้อมูลให้ Skill 20 รวมเป็น Mission เดียว
10. ห้ามลด Evidence Priority เพราะ Access Burden สูง

## Required Attributes

```text
inspection_id
primary_access_mode
alternative_access_mode
network_distance_m
off_network_distance_m
one_way_travel_time_min
round_trip_time_min
mode_transfer_count
start_access_point_id
landing_point_id
tide_dependency
tide_window
permission_required
safety_risk
route_source
route_confidence
access_burden_score
access_class
route_scout_required
special_logistics_required
```

## Outputs

- `access_network.gpkg`
- `candidate_access_points.gpkg`
- `inspection_route_options.gpkg`
- `inspection_access_assessment.csv`
- `access_burden_preview.png`
- `route_warning_zones.gpkg`

## Warnings

```text
access_network_missing
straight_line_only_warning
unverified_route_warning
route_disconnected_warning
high_off_network_distance
tide_window_required
unsafe_mud_approach
permission_required_warning
no_verified_landing_point
special_logistics_required
```

## Restrictions

- ห้ามถือคลองทุกเส้นว่าเรือผ่านได้
- ห้ามถือคันดินทุกเส้นว่าเดินหรือขับรถได้
- ห้ามใช้ภาพอย่างเดียวรับรองความปลอดภัย
- ห้ามสร้างเส้นผ่านพื้นที่ต้องห้าม
- ผลเส้นทางเป็น Candidate Route ต้องยืนยันกับทีมพื้นที่
