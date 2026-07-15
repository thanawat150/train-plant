---
name: surface-hydrology-condition-classifier
description: จำแนกสภาพผิวดิน เลน น้ำขัง และร่องน้ำที่มองเห็นจาก Orthomosaic โดยไม่ยืนยันคุณสมบัติดินทางห้องปฏิบัติการ ใช้ร่วมได้ทั้งงานปลูกเสริมและปลูกเต็มพื้นที่
---

# Surface and Hydrology Condition Classifier

## หน้าที่

สร้างแผนที่สภาพผิวและน้ำที่มองเห็นจากภาพ เพื่อใช้เป็นบริบทในการวิเคราะห์การรอด การปลูกซ่อม และ Candidate Boundary

Skill นี้ไม่ตรวจชนิดพืช ไม่ยืนยันความเค็ม pH ดินกรดกำมะถัน หรือความเหมาะสมปลูกจากสีภาพเพียงอย่างเดียว

## Required Classes

```text
shallow_water_pool
waterlogged_depression
tidal_microchannel
erosion_rill
wet_mud
dry_pale_mud
dry_oxidized_soil
transitional_mud
disturbed_fill_soil
possible_salt_crust
vegetated_stable_surface
shadow_unknown
unknown_surface
```

## Interpretation Rules

1. แยกน้ำจริงออกจากเงาด้วยรูปร่าง ขอบ ความต่อเนื่อง การสะท้อนแสง และความสัมพันธ์กับเรือนยอด
2. สีขาวหรือเทาใช้ได้สูงสุดเป็น `dry_pale_mud` หรือ `possible_salt_crust` ห้ามยืนยันว่าดินเค็ม
3. สีส้ม–น้ำตาลอาจเป็นดินออกซิไดซ์ ดินถม หรือพื้นที่ถูกรบกวน ต้องใช้ `dry_oxidized_soil` หรือ `disturbed_fill_soil` ตามหลักฐาน
4. ร่องน้ำและแอ่งน้ำต้องเก็บ Geometry แยก เพราะอาจเป็น Barrier หรือพื้นที่เสี่ยงกัดเซาะ
5. สีพื้นน้ำหรือเลนไม่ใช่ขอบเขตปลูกโดยอัตโนมัติ
6. ความหนาแน่นต้นต่ำในพื้นผิวชนิดหนึ่งเป็นเพียงความสัมพันธ์ ไม่ใช่ข้อสรุปเชิงสาเหตุ
7. เมื่อมี DEM/DTM/DSM ให้ใช้ Relative Elevation, Depression และ Flow Context ร่วมกับ RGB

## Optional Inputs

- DSM, DTM หรือ DEM
- ระดับน้ำหรือเวลาน้ำขึ้นน้ำลงขณะบิน
- จุดตรวจ pH, EC, ความเค็ม เนื้อดิน และความลึกชั้นเลน
- ภาพหลายช่วงเวลา

## Required Metrics

```text
class_area_sqm
class_area_ratio
water_ratio
waterlogged_ratio
wet_mud_ratio
dry_pale_mud_ratio
dry_oxidized_soil_ratio
disturbed_surface_ratio
possible_salt_crust_ratio
microchannel_length_m
surface_confidence_mean
```

## Outputs

- `surface_condition.tif`
- `surface_condition_confidence.tif`
- `hydrology_features.gpkg`
- `surface_objects.gpkg`
- `surface_condition_preview.png`
- `surface_condition_metrics.csv`

## Warnings

```text
shadow_water_confusion_warning
possible_salt_crust_unverified
surface_color_normalization_warning
orthomosaic_seam_warning
low_surface_confidence_warning
missing_elevation_data_warning
```

## Downstream Use

- ปลูกเสริม: ส่งสัดส่วนน้ำ เลน พื้นถูกรบกวน และแอ่งน้ำให้ Skill 07
- ปลูกเต็ม: ใช้แบ่ง Survival/Mortality Zone แต่ห้ามถือสภาพผิวเป็นสาเหตุโดยอัตโนมัติ
- วงขอบเขต: ใช้ร่องน้ำเป็น Candidate Barrier เมื่อมีหลักฐานสอดคล้อง
- วางแผนภาคสนาม: เสนอจุดตรวจดินและระดับพื้นที่ในโซนที่แตกต่างกัน

## Human Review

พื้นที่ `possible_salt_crust`, `unknown_surface` และบริเวณที่แยกน้ำออกจากเงาไม่ได้ ต้องเป็น `needs_human_review`
