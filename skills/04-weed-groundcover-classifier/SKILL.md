---
name: weed-groundcover-classifier
description: จำแนกวัชพืชและพืชคลุมดินจากภาพโดรน เพื่อแยกออกจากพุ่มโกงกางและใช้ระบุพื้นที่ที่อาจบดบังต้นปลูก ใช้เมื่อพื้นที่เปิดมีสีเขียวอ่อนละเอียดต่อเนื่องเป็นผืน
---

# Weed Groundcover Classifier

## หน้าที่

สร้าง Mask ของวัชพืชและพืชคลุมดิน ไม่ตรวจต้นโกงกาง ไม่สร้างขอบเขตแปลง และไม่สรุปว่าพื้นที่มีหรือไม่มีการปลูก

## Visual Evidence

จากตัวอย่างของผู้ใช้ วัชพืชมักมี:

- สีเขียวอ่อน เขียวเหลือง หรือเขียวมะกอก
- Texture ละเอียดและค่อนข้างสม่ำเสมอ
- ปกคลุมต่อเนื่องเป็นผืน
- ไม่มี Crown Center หรือขอบพุ่มรายต้นชัด
- อยู่ในช่องเปิด พื้นเลน หรือบริเวณที่ถูกรบกวน

## Classes

```text
weed_groundcover
mixed_weed_and_target
weed_obstruction
non_weed_low_vegetation
unknown_low_vegetation
```

## Rules

1. ใช้ Color + Texture + Object Scale ร่วมกัน
2. ห้ามตีความพื้นที่สีเขียวอ่อนทุกแห่งเป็นวัชพืช
3. หากพบพุ่มเป้าหมายแทรกในวัชพืช ให้ใช้ `mixed_weed_and_target`
4. พื้นที่วัชพืชหนาแน่นที่อาจบังต้นใช้ `weed_obstruction`
5. `weed_obstruction` ต้องลด Confidence ของการตรวจต้น แต่ไม่ถือว่าไม่มีต้นปลูก
6. แยกเงา น้ำ และพื้นเลนออกก่อนคำนวณ Texture

## Inputs

- RGB Orthomosaic Tile
- Land-cover Feature Stack
- Target Crown Candidates เมื่อมี
- CHM หรือ Height Layer เมื่อมี

## Outputs

- `weed_groundcover.tif`
- `weed_obstruction.gpkg`
- `weed_confidence.tif`
- `weed_preview.png`

## QA

- ตรวจการสับสนกับโกงกางพุ่มเล็ก
- ตรวจการสับสนกับ Natural Regeneration
- ตรวจพื้นที่แสงจัดหรือสีผิดจากการต่อภาพ
- รายงานสัดส่วนวัชพืชภายใน Candidate Gap
- เก็บบริเวณกำกวมเป็น `unknown_low_vegetation`

## Downstream Use

ผลจาก Skill นี้ใช้เพื่อ:

- ตัด False Positive ของการตรวจโกงกาง
- ระบุพื้นที่ที่ต้นกล้าอาจถูกบดบัง
- ช่วยอธิบายเหตุผลของ `uncertain_enrichment_gap`
- สร้าง QA Metric เรื่อง Weed Obstruction
