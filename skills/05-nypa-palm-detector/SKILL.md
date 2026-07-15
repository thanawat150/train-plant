---
name: nypa-palm-detector
description: ตรวจและแยกต้นจาก (Nypa palm) ที่ขึ้นเป็นกอหรือผืนในพื้นที่ชุ่มน้ำ ออกจากโกงกาง ปาล์ม และมะพร้าว ใช้ลด False Positive ในงานปลูกเสริม
---

# Nypa Palm Detector

## หน้าที่

ตรวจต้นจากเป็นรายกอหรือรายผืน เพื่อใช้เป็น Exclusion/Mixed Class ไม่สร้างขอบเขตปลูกเสริม และไม่เหมารวมพืชวงศ์ปาล์มชนิดอื่น

## Visual Evidence

จากตัวอย่างที่ผู้ใช้ยืนยัน ต้นจากมักมี:

- ขึ้นรวมกันเป็นกอหรือผืนหนาแน่นต่อเนื่อง
- ใบยาวแบบขนนกแตกจากระดับพื้นหรือเลน
- หลายกอซ้อนกันจนศูนย์กลางเรือนยอดไม่ชัด
- ไม่เห็นลำต้นตั้งตรงเด่นจากมุมบน
- มีใบเขียว เหลือง และใบแห้งสีน้ำตาลปะปน
- Texture เป็นเส้นใบยาวและซ้อนทับกันมาก
- สัมพันธ์กับเลน คลอง น้ำกร่อย หรือพื้นที่ชุ่มน้ำ

ต้นเดี่ยวทรงดาวที่มีศูนย์กลางชัด ลำต้นหรือเงาลำต้น ต้องส่งให้ `17-palm-coconut-detector` ไม่ควรเรียกเป็นต้นจากอัตโนมัติ

## Classes

```text
nypa_palm
probable_nypa_palm
nypa_dense_patch
nypa_mixed_patch
unknown_nypa_like_patch
```

## Rules

1. ใช้การขึ้นเป็นกอ ความต่อเนื่องของผืน Long-frond Texture และ Wetland Context ร่วมกัน
2. ห้ามเรียกต้นจากว่าโกงกาง แม้มีสีเขียวเข้ม
3. ห้ามเรียกทุก Crown แบบรัศมีว่า Nypa
4. ต้นเดี่ยวที่มี Crown Center ชัดให้ Route ไป Skill 17
5. พุ่มซ้อนกับต้นไม้เดิมให้ใช้ `nypa_mixed_patch`
6. กลุ่มที่แยกชนิดไม่ได้ให้ใช้ `unknown_nypa_like_patch`
7. รักษา Geometry, Source Tile และ Overlap Provenance
8. ห้ามยืนยันชนิดพฤกษศาสตร์จากสีอย่างเดียว

## Detection Features

- Patch continuity
- Multiple overlapping frond centers
- Long-frond orientation and line response
- Weak or multiple crown centers
- Absence of a visible upright trunk
- Color distribution ภายในกอ
- Proximity to water, mudflat หรือ tidal channel
- Surface context จาก Skill 16 เมื่อมี

## Outputs

- `nypa_palm_candidates.gpkg`
- `nypa_palm_mask.tif`
- `nypa_confidence.tif`
- `nypa_preview.png`

## Required Attributes

```text
object_id
class
confidence
patch_area_sqm
center_count
patch_continuity
trunk_visible
wetland_context_score
source_tile
source_raster
review_status
```

## QA

- ตรวจสับสนกับมะพร้าวและปาล์มชนิดอื่นด้วย Skill 17
- ตรวจพุ่มจากที่ใบซ้อนกันหลายกอ
- ตรวจ False Positive จากกิ่งแห้ง เงา และ Orthomosaic Artifact
- รายงานจำนวนและพื้นที่ Nypa ภายใน Candidate Enrichment Gap
- Flag ต้นเดี่ยวทรงดาวเป็น `possible_other_palm`

## Downstream Use

- ตัด Nypa ออกจาก Target Rhizophora Candidates
- ใช้อธิบายพื้นที่ที่ไม่ควรถูกรวมใน Core
- ใช้เป็น Exclusion หรือ Mixed Zone ตามบริบท ไม่ตัดทั้งพื้นที่โดยอัตโนมัติ
