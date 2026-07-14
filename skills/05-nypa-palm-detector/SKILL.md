---
name: nypa-palm-detector
description: ตรวจและแยกต้นจาก (Nypa palm) ออกจากโกงกางเป้าหมายด้วยรูปทรงใบแบบรัศมี ใบยาว Texture และบริบทพื้นที่ชุ่มน้ำ ใช้ลด False Positive ในงานปลูกเสริม
---

# Nypa Palm Detector

## หน้าที่

ตรวจ `nypa_palm` เป็นรายพุ่มหรือรายกลุ่ม เพื่อใช้เป็น Exclusion Class ไม่สร้างขอบเขตปลูกเสริมและไม่เหมารวมพืชวงศ์ปาล์มชนิดอื่น

## Visual Evidence

จากตัวอย่างของผู้ใช้ ต้นจากมักมี:

- ใบยาวแผ่ออกจากศูนย์กลางเป็นรัศมี
- รูปทรงคล้ายพัด ดาว หรือ Rosette
- สีเขียว เหลือง และน้ำตาลปะปนในพุ่มเดียว
- Texture เป็นเส้นยาวต่างจากพุ่มโกงกางแบบก้อน
- เกิดเป็นกลุ่มในพื้นที่ชุ่มน้ำ ใกล้คลองหรือเลน

## Classes

```text
nypa_palm
probable_nypa_palm
mixed_nypa_and_other_canopy
unknown_radial_crown
```

## Rules

1. ใช้ Shape และ Radial Texture เป็นหลัก สีเป็นข้อมูลเสริม
2. ห้ามเรียกต้นจากว่าโกงกาง แม้มีสีเขียวเข้ม
3. ห้ามเรียกทุก Crown แบบรัศมีว่า Nypa หากบริบทไม่สอดคล้อง
4. พุ่มซ้อนกับต้นไม้เดิมให้ใช้ `mixed_nypa_and_other_canopy`
5. กลุ่มที่แยกชนิดไม่ได้ให้ใช้ `unknown_radial_crown`
6. รักษา Geometry และ Source Tile เพื่อป้องกันการนับซ้ำ

## Detection Features

- Radial symmetry
- Long-frond orientation
- Center-to-edge line response
- Crown aspect and compactness
- Color distribution ภายใน Crown
- Proximity to water, mudflat หรือ tidal channel

## Outputs

- `nypa_palm_candidates.gpkg`
- `nypa_palm_mask.tif`
- `nypa_confidence.tif`
- `nypa_preview.png`

## QA

- ตรวจสับสนกับมะพร้าว ปาล์มชนิดอื่น และเรือนยอดแห้ง
- ตรวจพุ่มจากที่ใบซ้อนกันหลายต้น
- ตรวจ False Positive จากเส้นกิ่งหรือ Orthomosaic artifact
- รายงานจำนวนและพื้นที่ Nypa ภายใน Candidate Enrichment Gap

## Downstream Use

- ตัด Nypa ออกจาก Target Rhizophora Candidates
- ใช้อธิบายพื้นที่ที่ไม่ควรถูกรวมใน Core
- ใช้เป็น Exclusion หรือ Mixed Zone ตามบริบท ไม่ตัดทั้งพื้นที่โดยอัตโนมัติ
