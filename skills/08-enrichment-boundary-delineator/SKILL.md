---
name: enrichment-boundary-delineator
description: สร้าง Candidate Polygon สำหรับแปลงปลูกเสริมจาก Gap Candidates และหลักฐานพืชเป้าหมาย โดยแยก Core, Evidence Boundary และ Uncertain Boundary ใช้หลังผ่านการจำแนกและ Gap Analysis แล้ว
---

# Enrichment Boundary Delineator

## หน้าที่

สร้าง Geometry จากหลักฐานที่มีอยู่ ไม่ตรวจชนิดพืชใหม่และไม่อนุมัติขอบเขต

## Required Outputs

```text
enrichment_planting_core
enrichment_evidence_boundary
uncertain_enrichment_boundary
excluded_area
boundary_segment_confidence
```

## Boundary Logic

### Enrichment Planting Core

รวมเฉพาะพื้นที่ที่:

- มี Target Rhizophora Candidates ความมั่นใจสูง
- มี Density หรือกลุ่มต่อเนื่องเพียงพอ
- สิ่งรบกวนและพื้นที่ Unknown อยู่ในระดับยอมรับได้

### Enrichment Evidence Boundary

รวม Core และพื้นที่ต่อเนื่องที่มีหลักฐานสนับสนุน เช่น:

- Target กระจายบางแต่สัมพันธ์กับกลุ่มหลัก
- พื้นที่วัชพืชที่มี Target แทรกอยู่
- ช่องว่างที่ข้อมูลภาคสนามหรือภาพก่อน–หลังสนับสนุน

### Uncertain Enrichment Boundary

ใช้เมื่อ:

- วัชพืชบดบัง
- เรือนยอดเดิมปิด
- เงาหรือภาพผิดปกติ
- แยก Target จาก Natural Regeneration ไม่ได้

## Rules

1. วงแบบ Gap-by-gap ก่อนพิจารณารวม
2. ใช้ Concave Hull, Alpha Shape หรือ Density Contour ตามหลักฐาน ห้ามใช้ Convex Hull เป็นค่าเริ่มต้น
3. รักษาคลอง น้ำเปิด คันดิน ป่าเดิม และรอยเว้าที่มีเหตุผล
4. จากไม่ใช่ Target แต่ไม่จำเป็นต้องตัดทั้ง Gap หาก Target อยู่ร่วมกัน
5. แนวขอบต้องอยู่สัมพันธ์กับ Target ชั้นนอกสุดและ Gap Geometry
6. ห้าม Smooth จนขอบเขตเคลื่อนออกจากหลักฐาน
7. หากกลุ่มไม่ต่อกัน ให้สร้างหลาย Feature หรือ MultiPolygon
8. Candidate Barrier เป็นข้อมูลช่วย ไม่ใช่ขอบอัตโนมัติ

## Segment Confidence

แบ่งเส้นเป็น Segment และเก็บ:

```text
segment_id
polygon_id
confidence
confidence_class
evidence
barrier_type
nearest_target_distance_m
requires_review
review_note
```

## Outputs

- `enrichment_planting_core.gpkg`
- `enrichment_evidence_boundary.gpkg`
- `uncertain_enrichment_boundary.gpkg`
- `excluded_areas.gpkg`
- `boundary_segment_confidence.gpkg`
- `boundary_preview.png`

## QA before Handoff

- Geometry valid
- ไม่มี Tile-edge artifact
- ไม่ข้ามคลองหรือรวม Gap ที่ไม่ต่อกันโดยไม่มีเหตุผล
- ไม่ครอบพื้นที่ว่างขนาดใหญ่ที่ไม่มีหลักฐาน
- ไม่ตัด Target กลุ่มหลักออก
- ทุก Segment มี Confidence และ Evidence

สถานะสูงสุดที่ AI ตั้งได้คือ `needs_human_review`
