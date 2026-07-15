---
name: full-area-boundary-delineator
description: สร้าง Candidate Boundary จากต้นที่ผ่าน Skill 12b, Pattern Blocks และโซนอัตรารอด โดยรักษาพื้นที่ที่รูปแบบปลูกยังต่อเนื่อง
---

# Full-area Planting Boundary Delineator

## Scope

สร้าง Polygon หลัง Skill 12, 12b และ 13 ผ่านแล้ว ห้ามตรวจต้น Fit Grid หรือแก้จำนวนใหม่

## Inputs

```text
validated_planted_tree_points.gpkg
planting_point_status.gpkg
planting_position_status_final.gpkg
planting_rows.gpkg
planting_grid.gpkg
grid_blocks.gpkg
planting_pattern_blocks.gpkg
survival_mortality_zones.gpkg
existing_large_trees_validated.gpkg
optional barrier layers
project_manifest.json
```

ห้ามใช้ Raw Detection จาก Skill 11 เป็นหลักฐานขอบเขตโดยตรง

## Boundary Outputs

```text
full_area_planting_core
full_area_planting_evidence_boundary
full_area_uncertain_edge
excluded_area
boundary_segment_confidence
```

## Rules

- Core ครอบพื้นที่ที่มีต้น Validated และ Pattern/Spacing ชัด
- Evidence Boundary รวม Sparse Survival และ Mortality Gap เมื่อ Pattern ยังต่อเนื่อง
- ตามแนวแถวหรือ Pattern Block ด้านนอกสุดที่มีหลักฐาน
- False Positive, Existing Large Tree และ Random Scatter ห้ามขยายขอบเขต
- `off_grid_tree_candidate` ห้ามขยาย Boundary จนกว่า Review
- จุดใต้ต้นใหญ่ที่เป็น Not Observable ไม่ทำให้เกิดช่องเว้าอัตโนมัติ
- สีพื้นน้ำ เลน และความชื้นไม่ใช่ขอบเขตอัตโนมัติ
- คลอง ถนน คันดิน และ AOI เป็น Candidate Barrier ต้องตรวจร่วมกับ Pattern
- แยกหลาย Pattern Block เมื่อไม่ต่อเนื่อง
- ห้ามใช้ Convex Hull เป็นค่าเริ่มต้น
- ห้าม Smooth จนเส้นออกจากแนวปลูกด้านนอก

## Segment Attributes

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

## QA Precheck

- ขอบห่างจากแนวนอกสุดผิดปกติ
- ต้น Validated ใน Pattern หลักอยู่นอก Polygon มาก
- Polygon ครอบ Random Scatter/False-positive Zone
- ต้นใหญ่ทำให้ Boundary พอง
- Polygon ครอบพื้นที่ไม่มี Pattern Support
- เส้นเกาะ Tile Edge
- Geometry Invalid
