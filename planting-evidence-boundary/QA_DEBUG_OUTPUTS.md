# QA Metrics and Debug Outputs

เอกสารนี้กำหนดหลักฐานที่ระบบต้องสร้าง เพื่อให้ผู้ตรวจทราบว่า Candidate Boundary ถูกสร้างจากอะไรและผิดตรงไหนได้

## 1. Required QA Metrics

รายงานอย่างน้อย:

```text
detected_tree_count_inside
detected_tree_count_outside_near_boundary
expected_missing_position_count
planting_row_count
planting_row_coverage
median_tree_spacing_m
median_row_spacing_m
spacing_consistency
empty_area_ratio
mortality_gap_ratio
natural_vegetation_ratio
open_water_without_evidence_ratio
boundary_to_outer_row_distance_mean_m
boundary_to_outer_row_distance_p95_m
tile_edge_contact_ratio
uncertain_boundary_length_m
uncertain_boundary_ratio
geometry_valid
```

## 2. Warning Rules

### `large_empty_area_warning`

แจ้งเตือนเมื่อพื้นที่ภายใน Polygon จำนวนมากไม่มีต้น แนวแถว หรือ Historical Evidence รองรับ

ค่าเริ่มต้นแนะนำ:

```text
empty_area_ratio > 0.30
```

ต้องตรวจว่าพื้นที่ดังกล่าวเป็น Mortality Gap จริงหรือเป็นพื้นที่ที่ไม่เคยปลูก

### `under_boundary_warning`

แจ้งเตือนเมื่อพบต้นหรือแนวปลูกที่สัมพันธ์กับกลุ่มหลักอยู่นอก Candidate Boundary จำนวนมาก

### `natural_vegetation_warning`

แจ้งเตือนเมื่อ Polygon ครอบพืชธรรมชาติที่ไม่มี Pattern ปลูกในสัดส่วนสูง

### `boundary_too_far_warning`

แจ้งเตือนเมื่อแนวขอบอยู่ห่างจากแถวปลูกด้านนอกสุดมากเกินไป

ค่าเริ่มต้นแนะนำ:

```text
outer_row_distance > 3 * median_tree_spacing
```

อย่าใช้กฎนี้ตัดสินอัตโนมัติ หากมีหลักฐาน Historical Planting หรือ AOI ที่ผ่านการตรวจ

### `tile_edge_warning`

แจ้งเตือนเมื่อแนวขอบจำนวนมากตรงกับขอบ Tile ซึ่งอาจเป็น Artifact จากการแบ่งภาพ

### `barrier_mismatch_warning`

แจ้งเตือนเมื่อระบบใช้คลอง ถนน หรือคันดินเป็นขอบ แต่แนวปลูกสิ้นสุดห่างจาก Barrier มาก หรือมี Pattern ต่อเนื่องข้าม Barrier

### `low_segment_confidence_warning`

แจ้งเตือนเมื่อช่วงแนวขอบ Confidence ต่ำกว่า 50

## 3. Required Debug Layers

ระบบต้องสามารถส่งออก Layer ต่อไปนี้เมื่อเปิด Debug Mode:

```text
detected_trees.gpkg
expected_missing_positions.gpkg
planting_rows.gpkg
planting_grid.gpkg
candidate_barriers.gpkg
natural_vegetation.gpkg
planting_density.tif
planting_confidence.tif
tile_confidence.tif
uncertain_segments.gpkg
candidate_boundary.gpkg
planting_core.gpkg
planting_evidence_boundary.gpkg
excluded_areas.gpkg
```

## 4. Required Attributes

### Detected trees

```text
object_id
class
confidence
crown_size_px
source_tile
source_raster
```

### Missing positions

```text
position_id
row_id
expected_x
expected_y
confidence
reason
```

### Planting rows

```text
row_id
orientation_deg
length_m
point_count
missing_count
spacing_m
confidence
```

### Candidate barriers

```text
barrier_id
barrier_type
confidence
source
used_as_boundary
review_status
```

### Boundary segments

```text
segment_id
polygon_id
confidence
confidence_class
evidence
barrier_type
outer_row_distance_m
requires_review
review_note
```

## 5. Preview Set

ต้องสร้างอย่างน้อย:

1. `overview_boundary_preview.png` แสดงภาพเต็มระดับ Overview และ Candidate Boundary
2. `core_vs_envelope_preview.png` แยก Core, Evidence Envelope และ Uncertain Zone
3. `segment_confidence_preview.png` แสดง Confidence รายช่วงแนวขอบ
4. `evidence_preview.png` แสดงต้นที่พบ Missing Position แนวแถว และ Barrier
5. `warning_preview.png` ซูมบริเวณที่มี Warning

ภาพ Preview ต้องมี:

- Source raster name
- CRS
- Scale
- North arrow เมื่อเหมาะสม
- Legend
- Boundary version
- Review status
- Warning count

## 6. QA Decision

สถานะผลตรวจ:

- `pass` หลักฐานชัด ไม่มี Warning สำคัญ
- `pass_with_warning` ใช้งานตรวจต่อได้แต่มีส่วนต้องระวัง
- `rework` Algorithm หรือ Polygon ต้องแก้
- `field_check_required` ภาพไม่เพียงพอ ต้องตรวจภาคสนาม
- `rejected` ไม่พบหลักฐานเพียงพอสำหรับสร้างขอบเขต

AI ห้ามเปลี่ยนเป็น `approved`

## 7. Minimum Acceptance

Candidate Boundary พร้อมให้คนตรวจเมื่อ:

- Geometry valid
- มี Core และ Evidence Boundary
- มี Confidence ราย Segment
- มี Row/Spacing Metrics หรือระบุเหตุผลว่าทำไม่ได้
- มี Warning Report
- มี Overview และ Detail Preview
- มี Provenance ย้อนกลับไปยัง Source Raster และ Tile ได้
