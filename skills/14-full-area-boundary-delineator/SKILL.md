---
name: full-area-boundary-delineator
description: สร้าง Candidate Boundary ของแปลงปลูกเต็มพื้นที่จากต้นปลูก กริด แนวแถว และช่องว่างต้นตาย โดยรักษาพื้นที่ที่กริดยังต่อเนื่อง
---

# Full-area Planting Boundary Delineator

## Scope

สร้าง Polygon หลังจาก Skill 11–13 มีผลแล้ว ห้ามตรวจต้นหรือ Fit Grid ใหม่ในขั้นตอนนี้

## Inputs

```text
planted_tree_candidates.gpkg
planting_rows.gpkg
planting_grid.gpkg
expected_missing_positions.gpkg
survival_mortality_zones.gpkg
optional barrier layers
```

## Boundary outputs

```text
full_area_planting_core
full_area_planting_evidence_boundary
full_area_uncertain_edge
excluded_area
boundary_segment_confidence
```

## Rules

- `core` ครอบบริเวณที่พบต้นและ Grid ชัด
- `evidence_boundary` รวม Sparse Survival และ Mortality Gap เมื่อ Grid ต่อเนื่อง
- ตามแนว Grid หรือแถวปลูกด้านนอกสุดที่มีหลักฐานรองรับ
- สีพื้นน้ำ เลน และความชื้นไม่ใช่ขอบเขตอัตโนมัติ
- คลอง ถนน คันดิน และ AOI เป็น Candidate Barrier ต้องตรวจร่วมกับ Grid
- รักษารอยเว้าที่มีสิ่งกีดขวางจริง
- แยกหลาย Grid Block เมื่อไม่ต่อเนื่อง
- ห้ามใช้ Convex Hull เป็นค่าเริ่มต้น
- ห้าม Smooth จนเส้นเคลื่อนออกจากแถวปลูกด้านนอก

## Segment confidence

แบ่งแนวขอบเป็น Segment และเก็บ:

```text
segment_id
boundary_type
confidence
support_type
outer_row_distance_m
barrier_type
requires_review
review_note
```

## Outputs

```text
full_area_planting_core.gpkg
full_area_planting_evidence_boundary.gpkg
full_area_uncertain_edge.gpkg
excluded_areas.gpkg
boundary_segment_confidence.gpkg
full_area_boundary_preview.png
```

## QA precheck

- ขอบเขตห่างจากแถวนอกสุดผิดปกติหรือไม่
- ต้นที่สัมพันธ์กับ Grid หลักตกอยู่นอก Polygon มากหรือไม่
- Polygon ครอบพื้นที่ที่ไม่มี Grid รองรับมากหรือไม่
- เส้นเกาะ Tile Edge หรือไม่
- Geometry valid หรือไม่
