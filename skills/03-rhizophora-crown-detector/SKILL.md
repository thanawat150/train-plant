---
name: rhizophora-crown-detector
description: ตรวจวัตถุหรือพุ่มที่มีลักษณะคล้ายโกงกางเป้าหมายจากภาพโดรนเป็นรายต้นหรือรายพุ่ม ใช้เมื่อต้องการนับต้น สร้างจุดเป้าหมาย หรือเตรียม Density Surface สำหรับวิเคราะห์แปลงปลูกเสริม
---

# Rhizophora Crown Detector

## หน้าที่

ตรวจ `target_rhizophora_candidate` เป็นราย Object แต่ไม่ยืนยันว่าเป็นต้นที่มนุษย์ปลูก และไม่สร้างขอบเขตแปลงเอง

## Visual Evidence

จากตัวอย่างของผู้ใช้ โกงกางเป้าหมายมักมี:

- พุ่มสีเขียวค่อนข้างเข้มกว่าวัชพืชรอบข้าง
- รูปทรงเป็นก้อนหรือ Crown แยกได้
- Texture หยาบกว่าพืชคลุมดิน
- ขนาดพุ่มบางกลุ่มใกล้เคียงกัน
- กระจายอยู่ในช่องว่างของป่าเดิมหรือบนพื้นวัชพืช

สีเข้มเพียงอย่างเดียวไม่เพียงพอ เพราะอาจเป็นเงา ป่าเดิม หรือพืชชนิดอื่น

## Inputs

- Orthomosaic Tile พร้อมพิกัด
- Land-cover Class และ Confidence จาก Skill 02
- Weed, Nypa และ Existing Canopy Masks เมื่อมี
- ตัวอย่าง Positive/Negative Label เมื่อมี

## Detection Options

- Object Detection
- Instance Segmentation
- Crown Segmentation
- Template/Shape + Texture สำหรับ MVP

เลือกวิธีตามขนาดพุ่มในหน่วย Pixel และจำนวนตัวอย่างฝึก

## Rules

1. หนึ่งพุ่มที่แยกได้ให้เป็นหนึ่ง Object
2. พุ่มติดกันที่แยกไม่ได้ต้องระบุ `crown_cluster`
3. ตัดพื้นที่ `nypa_palm`, `weed_groundcover`, `shadow_unknown` และ `existing_woody_canopy` ตาม Confidence
4. ห้ามนับพุ่มซ้ำจาก Tile Overlap
5. เก็บ Low-confidence Object ให้ผู้ตรวจ ไม่ลบทิ้งเงียบ ๆ
6. ไม่ใช้ผลตรวจต้นเพื่อสรุปขอบเขตโดยตรง

## Required Attributes

```text
object_id
class
confidence
crown_area_sqm
crown_diameter_m
source_tile
source_raster
review_status
notes
```

## Outputs

- `rhizophora_candidates.gpkg`
- `rhizophora_crown_mask.tif`
- `rhizophora_confidence.tif`
- `rhizophora_detection_preview.png`

## QA

- ตรวจ False Positive บนเงา จาก วัชพืช และพุ่มป่าเดิม
- ตรวจ False Negative ในวัชพืชหนาแน่นและบริเวณแสงต่างกัน
- ตรวจ Duplicate Detection ระหว่าง Tile
- รายงาน Precision/Recall เมื่อมี Ground Truth
- ชื่อผลต้องใช้คำว่า Candidate จนกว่าจะผ่านการตรวจภาคสนามหรือ Label ที่เชื่อถือได้
