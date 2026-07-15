---
name: mangrove-land-cover-classifier
description: จำแนกสิ่งปกคลุมจากภาพโดรนในพื้นที่ป่าชายเลน โดยใช้สี รูปทรง Texture ขนาด และบริบทเชิงพื้นที่ร่วมกัน ใช้สร้าง Class Map ก่อนตรวจโกงกางหรือวิเคราะห์แปลงปลูกเสริม
---

# Mangrove Land-cover Classifier

## หน้าที่

สร้างแผนที่ Class ขั้นกลาง ไม่สร้าง Candidate Boundary ไม่ยืนยันว่าต้นใดเกิดจากการปลูก และไม่ยืนยันคุณสมบัติดินจากสีภาพ

## Required Classes

```text
target_rhizophora_candidate
existing_woody_canopy
weed_groundcover
nypa_palm
coconut_palm_candidate
other_palm_candidate
palm_unknown
natural_regeneration
dead_dry_vegetation
bare_mud
open_water
canal_or_tidal_channel
shadow_unknown
closed_canopy_unknown
unknown_object
```

## Optional Surface Classes

เมื่อภาพมีดิน เลน น้ำขัง หรือพื้นผิวแตกต่างชัด ให้สร้าง Class เบื้องต้นหรือ Route ไป Skill 16:

```text
shallow_water_pool
waterlogged_depression
wet_mud
dry_pale_mud
dry_oxidized_soil
transitional_mud
disturbed_fill_soil
possible_salt_crust
unknown_surface
```

## Evidence Sources

ใช้ร่วมกันอย่างน้อย 2 กลุ่ม:

- Color: RGB, HSV, Lab, ExG, GLI, VARI
- Shape: crown compactness, radial leaves, object size, crown center
- Texture: local variance, entropy, GLCM หรือ feature ที่เทียบเท่า
- Context: อยู่ในช่องว่าง ใกล้คลอง เกาะขอบป่า อยู่เป็นผืนต่อเนื่อง หรือเป็นต้นเดี่ยว
- Height: DSM/DTM/CHM เมื่อมี
- Surface/Hydrology: Skill 16 เมื่อมีความแตกต่างของเลน น้ำขัง หรือพื้นถูกรบกวน

ห้ามใช้สีหรือ Threshold ค่าเดียวเป็นคำตอบสุดท้าย

## Image-based Interpretation

- โกงกางเป้าหมายมักเป็นพุ่มค่อนข้างเข้มและแยกเป็นก้อน แต่สีเปลี่ยนได้ตามแสงและอายุ
- วัชพืชมักเป็นผืนละเอียด สีเขียวอ่อนหรือเขียวเหลือง ไม่มี Crown Center ชัด
- ต้นจากมักขึ้นเป็นกอหรือผืน ใบยาวซ้อนกัน ศูนย์กลางไม่ชัด และสัมพันธ์กับพื้นที่ชุ่มน้ำ
- ปาล์มหรือมะพร้าวมักเห็นเป็นต้นเดี่ยว มี Crown Center ชัด และทรงรัศมีจากยอดเดียว ให้ Route ไป Skill 17
- ป่าเดิมมีพุ่มใหญ่ หลายขนาด หลายเฉด และ Texture หยาบ
- เงาต้องแยกจากพุ่มสีเข้มและแอ่งน้ำด้วย Shape, Texture และความสัมพันธ์กับเรือนยอด
- สีขาว–เทาใช้ได้สูงสุดเป็น Surface Candidate ห้ามยืนยันเกลือ ความเค็ม หรือชนิดดิน

## Workflow

1. อ่าน Tile พร้อม Metadata จาก Skill 01
2. Normalize สีแบบบันทึก Parameter และไม่แก้ Source
3. สร้าง Feature Stack
4. Segment เป็น Object/Superpixel หรือ Pixel Class ตามความเหมาะสม
5. จำแนก Class พร้อม Confidence
6. Route กลุ่มต้นจากไป Skill 05 และต้นเดี่ยวทรงรัศมีไป Skill 17
7. Route พื้นผิวดิน–น้ำที่ซับซ้อนไป Skill 16
8. ทำ Edge Suppression บริเวณขอบ Tile
9. รวมผลกลับสู่ CRS ต้นฉบับ
10. สร้าง Confusion/Review Samples เมื่อมี Ground Truth

## Outputs

- `land_cover_class.tif`
- `land_cover_confidence.tif`
- `land_cover_objects.gpkg`
- `class_legend.json`
- `classification_preview.png`

## QA

- Class ทุกชนิดต้องมี Confidence
- เก็บ `unknown` แทนการบังคับ Class
- พื้นที่ใต้เรือนยอดปิดใช้ `closed_canopy_unknown`
- ตรวจ Class Balance และพื้นที่ที่สีคล้ายกันแต่ Texture ต่างกัน
- ตรวจ Nypa vs Palm/Coconut Confusion
- ตรวจ Water vs Shadow Confusion
- Flag Tile ที่มีแสง สี หรือ Orthomosaic Seam ผิดปกติ
