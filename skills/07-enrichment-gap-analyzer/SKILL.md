---
name: enrichment-gap-analyzer
description: วิเคราะห์ช่องว่างในป่าเดิมแบบ Gap-by-gap เพื่อหาพื้นที่ที่มีหลักฐานปลูกเสริมจากโกงกางเป้าหมาย พร้อมแยกวัชพืช จาก ป่าเดิม น้ำ เลน เงา และพื้นที่ไม่มั่นใจ ใช้ก่อนสร้างขอบเขต
---

# Enrichment Gap Analyzer

## หน้าที่

รวมผลจาก Detection และ Classification เพื่อประเมินช่องว่างแต่ละแห่ง ไม่สร้าง Polygon ขอบเขตสุดท้ายและไม่ยืนยันว่าโกงกางเกิดจากการปลูก

## Gap Classes

```text
confirmed_enrichment_gap
probable_enrichment_gap
uncertain_enrichment_gap
natural_regeneration_gap
disturbed_open_area
closed_canopy_unknown_gap
excluded_water_gap
```

## Core Principle

แปลงปลูกเสริมต้องวิเคราะห์เป็นช่องว่างหรือกลุ่มย่อย ไม่วงป่าทั้งผืนและไม่วงพื้นที่เปิดทุกแห่ง

## Evidence for Enrichment

ใช้หลักฐานร่วมกัน:

- จำนวนและความหนาแน่นของ `target_rhizophora_candidate`
- ขนาดพุ่มที่ใกล้เคียงกัน
- การกระจายภายในช่องว่างของป่าเดิม
- ระยะระหว่างพุ่มหรือกลุ่มที่สัมพันธ์กัน
- ภาพก่อน–หลัง จุดปลูก หรือข้อมูลภาคสนาม
- สัดส่วนวัชพืช จาก ป่าเดิม น้ำ และพื้นที่บดบัง

## Rules

1. พื้นที่เปิดไม่มีต้นเป้าหมายให้เป็น `disturbed_open_area`
2. พื้นที่มี Target ชัดและสิ่งรบกวนน้อยให้เป็น `confirmed_enrichment_gap` เฉพาะเมื่อมี Supporting Evidence เพียงพอ
3. พบ Target หลายต้นแต่ยังแยกจาก Natural Regeneration ไม่ได้ให้เป็น `probable_enrichment_gap`
4. วัชพืชหนาแน่น เงา หรือเรือนยอดปิดให้เป็น `uncertain_enrichment_gap`
5. จากและพืชธรรมชาติไม่ใช่ Target แต่สามารถอยู่ร่วมใน Gap เดียวกันได้
6. น้ำหรือคลองต้องตัดเป็น Exclusion เว้นแต่ต้นเป้าหมายเกิดอยู่ในพื้นที่เลนที่มีน้ำตื้นและเห็นได้จริง
7. ห้ามรวม Gap ที่ถูกคลองหรือป่าเดิมหนาแน่นแบ่งออกเพียงเพราะอยู่ใกล้กัน

## Required Metrics per Gap

```text
gap_id
gap_area_sqm
target_count
target_density_per_rai
weed_ratio
nypa_ratio
existing_canopy_ratio
closed_canopy_unknown_ratio
water_ratio
bare_mud_ratio
target_confidence_mean
supporting_evidence_count
gap_confidence
review_status
```

## Outputs

- `enrichment_gap_candidates.gpkg`
- `gap_metrics.csv`
- `gap_confidence.tif`
- `gap_classification_preview.png`

## QA

- ตรวจ Gap ที่มีพื้นที่ใหญ่แต่ Target น้อย
- ตรวจ Gap ที่ Target กระจายเฉพาะขอบ
- ตรวจการรวม Natural Regeneration เป็นการปลูก
- ตรวจพื้นที่ที่ถูกวัชพืชหรือเรือนยอดปิดบดบัง
- ต้องแสดงเหตุผลของ Class แต่ละ Gap
