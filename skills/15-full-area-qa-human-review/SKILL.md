---
name: full-area-qa-human-review
description: ตรวจผลแปลงปลูกเต็มพื้นที่ สร้าง QA Metrics, Warning Layers และชุดตรวจทานสำหรับมนุษย์ โดยไม่แก้ผลให้ผ่านเอง
---

# Full-area QA and Human Review

## Scope

ตรวจผลจาก Skill 11–14 เท่านั้น ห้ามตรวจต้นใหม่ Fit Grid ใหม่ หรือเปลี่ยน Candidate Boundary โดยไม่สร้าง Rework Request

## Required checks

```text
duplicate_tree_warning
shadow_as_tree_warning
large_tree_inclusion_warning
weak_grid_warning
multi_grid_forced_warning
grid_crosses_barrier_warning
missing_vs_unobservable_warning
boundary_too_far_warning
under_boundary_warning
large_unsupported_area_warning
tile_edge_warning
geometry_invalid_warning
low_segment_confidence_warning
```

## Required metrics

```text
detected_tree_count
probable_tree_count
existing_large_tree_count
expected_position_count
probable_missing_count
not_observable_count
observable_survival_rate
grid_block_count
spacing_consistency
unsupported_area_ratio
outside_grid_tree_count
uncertain_boundary_ratio
geometry_valid
```

## Review package

```text
01_tree_detection_preview.png
02_grid_missing_preview.png
03_survival_mortality_preview.png
04_boundary_confidence_preview.png
05_warning_preview.png
qa_report.json
warning_layers.gpkg
review_checklist.csv
analysis_manifest.json
```

## Review decisions

AI ใช้สถานะได้เพียง:

```text
draft
needs_human_review
rework
```

AI ห้ามตั้ง:

```text
reviewed
rejected
approved
```

## Acceptance before human decision

- Source Raster, CRS และ Config ถูกบันทึก
- ทุกผลย้อนกลับไปยัง Tile/Window ได้
- สูตรอัตรารอดและตัวหารถูกแสดง
- Missing Tree แยกจาก Unobservable
- Boundary มี Segment Confidence
- Warning ทุกข้อมีตำแหน่งและเหตุผล
- ไม่มีการแก้ผลอัตโนมัติเพียงเพื่อให้ผ่าน QA
