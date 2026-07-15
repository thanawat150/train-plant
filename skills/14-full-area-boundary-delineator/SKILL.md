---
name: full-area-boundary-delineator
description: สร้าง Candidate Boundary ของแปลงปลูกเต็มพื้นที่จากต้นที่ผ่าน Skill 21 แนวแถว Pattern Blocks และโซนอัตรารอด โดยรักษาพื้นที่ที่รูปแบบปลูกยังต่อเนื่อง
---

# Full-area Planting Boundary Delineator

## Scope

สร้าง Polygon หลังจาก Skill 12, 21 และ 13 มีผลแล้ว ห้ามตรวจต้น Fit Grid หรือแก้จำนวนต้นใหม่ในขั้นตอนนี้

## Inputs

```text
validated_planted_tree_points.gpkg
planting_point_status.gpkg
planting_rows.gpkg
planting_grid.gpkg
grid_blocks.gpkg
planting_pattern_blocks.gpkg
survival_mortality_zones.gpkg
existing_large_trees_validated.gpkg
optional barrier layers
```

ห้ามใช้ Raw Detection จาก Skill 11 เป็นหลักฐานขอบเขตโดยตรง

## Boundary outputs

```text
full_area_planting_core
full_area_planting_evidence_boundary
full_area_uncertain_edge
excluded_area
boundary_segment_confidence
```

## Rules

- `core` ครอบบริเวณที่มีต้น Validated และ Pattern/Spacing ชัด
- `evidence_boundary` รวม Sparse Survival และ Mortality Gap เมื่อ Pattern Block ยังต่อเนื่อง
- ตามแนวแถวหรือ Pattern Block ด้านนอกสุดที่มีหลักฐานรองรับ
- False Positive, Existing Large Tree และ Random Scatter Zone ห้ามขยายขอบเขต
- จุด `off_grid_tree_candidate` ไม่ใช้ขยาย Boundary จนกว่าจะ Review
- จุดใต้ต้นใหญ่ที่เป็น Not Observable ไม่ได้ทำให้เกิดช่องเว้าหรือตัดพื้นที่โดยอัตโนมัติ
- สีพื้นน้ำ เลน และความชื้นไม่ใช่ขอบเขตอัตโนมัติ
- คลอง ถนน คันดิน และ AOI เป็น Candidate Barrier ต้องตรวจร่วมกับ Pattern
- แยกหลาย Pattern Block เมื่อไม่ต่อเนื่อง
- ห้ามใช้ Convex Hull เป็นค่าเริ่มต้น
- ห้าม Smooth จนเส้นเคลื่อนออกจากแนวปลูกด้านนอก

## Segment confidence

```text
segment_id
boundary_type
confidence
support_type
outer_pattern_distance_m
validated_tree_support_count
spacing_supported_count
false_positive_near_edge_count
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

- ขอบเขตห่างจากแนวนอกสุดผิดปกติหรือไม่
- ต้น Validated ที่สัมพันธ์กับ Pattern หลักตกอยู่นอก Polygon มากหรือไม่
- Polygon ครอบ Random Scatter หรือ False-positive Zone หรือไม่
- ต้นใหญ่เดิมทำให้ Boundary พองออกหรือไม่
- Polygon ครอบพื้นที่ที่ไม่มี Pattern รองรับมากหรือไม่
- เส้นเกาะ Tile Edge หรือไม่
- Geometry valid หรือไม่
