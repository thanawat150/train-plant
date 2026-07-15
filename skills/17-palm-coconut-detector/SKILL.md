---
name: palm-coconut-detector
description: ตรวจและแยกปาล์ม มะพร้าว และเรือนยอดปาล์มไม่ทราบชนิดออกจากต้นจากและโกงกาง ใช้เมื่อภาพมีเรือนยอดทรงดาวหรือรัศมีเป็นรายต้น
---

# Palm and Coconut Detector

## หน้าที่

ตรวจเรือนยอดปาล์ม/มะพร้าวที่เป็นต้นเดี่ยวหรือมีกลางพุ่มชัด เพื่อป้องกันการจำแนกผิดเป็น `nypa_palm` หรือ `target_rhizophora_candidate`

## Classes

```text
coconut_palm_candidate
other_palm_candidate
palm_unknown
mixed_palm_and_other_canopy
```

## Visual Evidence

ลักษณะที่สนับสนุนปาล์มหรือมะพร้าว:

- เห็นเป็นต้นเดี่ยวหรือแยก Crown ได้
- มีศูนย์กลางเรือนยอดชัด
- ใบแผ่ออกจากยอดเดียวเป็นดาวหรือรัศมีค่อนข้างสมมาตร
- อาจเห็นลำต้นหรือเงาลำต้นยาว
- ระยะห่างระหว่างต้นชัดกว่ากอจาก
- อาจอยู่บนพื้นที่สูง แห้ง หรือพื้นที่มนุษย์ใช้ประโยชน์

ลักษณะที่สนับสนุนต้นจากแทน:

- ขึ้นเป็นกอหรือผืนหนาแน่นต่อเนื่อง
- ใบหลายกอซ้อนกันและศูนย์กลางไม่ชัด
- ไม่เห็นลำต้นตั้งเด่น
- มีใบเขียว เหลือง และแห้งปะปนจำนวนมาก
- เชื่อมต่อกับเลน คลอง หรือพื้นที่น้ำกร่อย

## Rules

1. ห้ามเรียกทุก Crown แบบรัศมีว่า `nypa_palm`
2. ใช้ `palm_unknown` เมื่อแยกมะพร้าวกับปาล์มชนิดอื่นไม่ได้
3. ห้ามยืนยันชนิดพฤกษศาสตร์จาก RGB มุมบนเพียงอย่างเดียว
4. ส่ง Crown ที่ขึ้นเป็นผืนและไม่มีลำต้นชัดกลับให้ Skill 05 ตรวจ Nypa
5. ส่ง Crown ทรงก้อนที่ไม่มีลักษณะใบแบบรัศมีให้ Skill 03 หรือ Skill 06
6. ป้องกันการนับซ้ำข้าม Tile ด้วยตำแหน่ง Crown Center และ Overlap Agreement

## Detection Features

- Single crown center
- Radial symmetry
- Long-frond orientation
- Visible trunk or trunk shadow
- Crown diameter and aspect
- Isolation from neighboring crowns
- Surface and hydrology context from Skill 16 เมื่อมี

## Outputs

- `palm_coconut_candidates.gpkg`
- `palm_coconut_mask.tif`
- `palm_coconut_confidence.tif`
- `palm_coconut_preview.png`

## Required Attributes

```text
object_id
class
confidence
crown_diameter_m
center_clarity
radial_score
trunk_visible
trunk_shadow_score
source_tile
source_raster
review_status
```

## Warnings

```text
nypa_palm_confusion_warning
large_tree_radial_confusion_warning
shadow_radial_confusion_warning
species_unverified_warning
```

## Downstream Use

ใช้เป็น Exclusion Context ในงานปลูกเสริมและปลูกเต็ม เมื่อปาล์ม/มะพร้าวไม่ใช่ชนิดเป้าหมาย ห้ามตัดพื้นที่รอบต้นทั้งหมดออกโดยอัตโนมัติ ให้ตัดเฉพาะ Crown หรือ Mixed Zone ที่มีหลักฐาน
