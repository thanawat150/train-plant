---
name: existing-canopy-occlusion
description: จำแนกป่าเดิม เรือนยอดไม้ขนาดใหญ่ เงา และพื้นที่ที่มองไม่เห็นใต้เรือนยอด เพื่อป้องกันการสรุปผิดว่าไม่มีการปลูกเสริม ใช้เมื่อภาพมีเรือนยอดปิดและต้นปลูกอาจถูกบดบัง
---

# Existing Canopy and Occlusion

## หน้าที่

แยกบริบทของป่าเดิมและพื้นที่มองไม่เห็น ไม่ตรวจต้นเป้าหมายใต้เรือนยอดจากสิ่งที่ภาพไม่ได้แสดง และไม่ใช้เรือนยอดปิดเป็นหลักฐานว่าไม่มีการปลูก

## Classes

```text
existing_woody_canopy
closed_canopy_unknown
partial_canopy_occlusion
shadow_unknown
dead_dry_canopy
natural_regeneration
```

## Visual Evidence

ป่าเดิมมักมี:

- พุ่มใหญ่และขนาดหลากหลาย
- สีเขียวหลายเฉด
- รูปทรงไม่สม่ำเสมอ
- Texture หยาบและมีเงาระหว่างพุ่ม
- ไม่มี Pattern การปลูกที่ชัดเจน

## Rules

1. พื้นที่ที่มองไม่เห็นพื้นด้านล่างให้เป็น `closed_canopy_unknown`
2. ห้ามแปลง `closed_canopy_unknown` เป็น `not_planted`
3. เงาสีเข้มต้องแยกจากพุ่มโกงกางด้วย Shape, Texture และขอบเรือนยอด
4. ต้นแห้งหรือเรือนยอดสีเทาขาวต้องแยกเป็น `dead_dry_canopy`
5. Natural Regeneration ต้องใช้เมื่อมีต้นเล็กกระจายแบบไม่สม่ำเสมอและไม่มีหลักฐานการปลูก
6. เมื่อมี CHM ให้ใช้ Height และ Canopy Structure ช่วยแยกชั้นเรือนยอด

## Inputs

- Orthomosaic Tile
- Land-cover Class
- DSM/DTM/CHM เมื่อมี
- Shadow Mask
- ภาพต่างช่วงเวลาเมื่อมี

## Outputs

- `existing_canopy.gpkg`
- `closed_canopy_unknown.tif`
- `shadow_unknown.tif`
- `dead_dry_canopy.gpkg`
- `canopy_occlusion_preview.png`

## QA

- ตรวจว่าเงาไม่ถูกนับเป็นโกงกาง
- ตรวจว่าป่าเดิมไม่ถูกตัดเป็นพื้นที่ว่างจากความต่างของสี
- รายงานสัดส่วน `closed_canopy_unknown` ในแต่ละ Candidate Gap
- Flag พื้นที่ที่ต้องใช้ข้อมูลภาคสนามหรือภาพมุมเอียง

## Downstream Use

- ลด Confidence ของการสรุปพื้นที่ปลูกเสริมในบริเวณบดบัง
- สร้าง `uncertain_enrichment_gap`
- แยกพื้นที่ที่ประเมินได้จากภาพออกจากพื้นที่ที่ต้องสำรวจเพิ่ม
