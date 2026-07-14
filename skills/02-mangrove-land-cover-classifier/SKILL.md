---
name: mangrove-land-cover-classifier
description: จำแนกสิ่งปกคลุมจากภาพโดรนในพื้นที่ป่าชายเลน โดยใช้สี รูปทรง Texture ขนาด และบริบทเชิงพื้นที่ร่วมกัน ใช้สร้าง Class Map ก่อนตรวจโกงกางหรือวิเคราะห์แปลงปลูกเสริม
---

# Mangrove Land-cover Classifier

## หน้าที่

สร้างแผนที่ Class ขั้นกลาง ไม่สร้าง Candidate Boundary และไม่ยืนยันว่าต้นใดเกิดจากการปลูก

## Required Classes

```text
target_rhizophora_candidate
existing_woody_canopy
weed_groundcover
nypa_palm
natural_regeneration
dead_dry_vegetation
bare_mud
open_water
canal_or_tidal_channel
shadow_unknown
closed_canopy_unknown
unknown_object
```

## Evidence Sources

ใช้ร่วมกันอย่างน้อย 2 กลุ่ม:

- Color: RGB, HSV, Lab, ExG, GLI, VARI
- Shape: crown compactness, radial leaves, object size
- Texture: local variance, entropy, GLCM หรือ feature ที่เทียบเท่า
- Context: อยู่ในช่องว่าง ใกล้คลอง เกาะขอบป่า หรืออยู่เป็นผืนต่อเนื่อง
- Height: DSM/DTM/CHM เมื่อมี

ห้ามใช้สีหรือ Threshold ค่าเดียวเป็นคำตอบสุดท้าย

## Image-based Interpretation

- โกงกางเป้าหมายมักเป็นพุ่มค่อนข้างเข้มและแยกเป็นก้อน แต่สีเปลี่ยนได้ตามแสงและอายุ
- วัชพืชมักเป็นผืนละเอียด สีเขียวอ่อนหรือเขียวเหลือง ไม่มี Crown Center ชัด
- จากมีใบยาวแผ่รัศมีคล้ายพัดหรือดาว และอาจมีใบเหลืองหรือน้ำตาลปะปน
- ป่าเดิมมีพุ่มใหญ่ หลายขนาด หลายเฉด และ Texture หยาบ
- เงาต้องแยกจากพุ่มสีเข้มด้วย Shape และ Texture

## Workflow

1. อ่าน Tile พร้อม Metadata จาก Skill 01
2. Normalize สีแบบบันทึก Parameter และไม่แก้ Source
3. สร้าง Feature Stack
4. Segment เป็น Object/Superpixel หรือ Pixel Class ตามความเหมาะสม
5. จำแนก Class พร้อม Confidence
6. ทำ Edge Suppression บริเวณขอบ Tile
7. รวมผลกลับสู่ CRS ต้นฉบับ
8. สร้าง Confusion/Review Samples เมื่อมี Ground Truth

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
- Flag Tile ที่มีแสงหรือสีผิดปกติ
