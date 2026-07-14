---
name: enrichment-qa-human-review
description: ตรวจคุณภาพผลจำแนก ตรวจโกงกาง วิเคราะห์ Gap และ Candidate Boundary พร้อมสร้าง Warning, Debug Layer และชุดภาพสำหรับ Human Review ใช้ก่อนส่งออกหรือใช้ในรายงานทุกครั้ง
---

# Enrichment QA and Human Review

## หน้าที่

ตรวจผลจาก Skill อื่น อธิบายสาเหตุของข้อผิดพลาด และจัดเตรียมงานให้คนตรวจ ไม่แก้ผลให้ผ่านเองและไม่ตั้งสถานะ Approved

## Required QA Metrics

```text
target_count_inside
target_count_outside_near_boundary
target_density_per_rai
weed_ratio
nypa_ratio
existing_canopy_ratio
closed_canopy_unknown_ratio
water_ratio
bare_mud_ratio
large_empty_area_ratio
boundary_to_nearest_target_mean_m
boundary_to_nearest_target_p95_m
tile_edge_contact_ratio
uncertain_boundary_ratio
low_confidence_segment_ratio
geometry_valid
```

## Warning Rules

```text
large_empty_area_warning
under_boundary_warning
weed_obstruction_warning
nypa_confusion_warning
natural_regeneration_warning
closed_canopy_unknown_warning
boundary_too_far_warning
tile_edge_warning
barrier_mismatch_warning
low_segment_confidence_warning
invalid_geometry_warning
```

## Required Debug Layers

- Target Rhizophora Candidates
- Weed Groundcover and Obstruction
- Nypa Palm Candidates
- Existing Canopy and Closed-canopy Unknown
- Gap Candidates and Metrics
- Core, Evidence and Uncertain Boundary
- Boundary Segment Confidence
- Tile Index and Tile-edge Zone
- Excluded Water, Canal, Bare Mud and Dead/Dry Vegetation

## Preview Set

1. Overview Classification
2. Target vs Weed vs Nypa
3. Gap Classification
4. Core vs Evidence vs Uncertain Boundary
5. Segment Confidence
6. Warning Map
7. Detail Crops ของจุด Confidence ต่ำ

## Review Checklist

- โกงกางเป้าหมายถูกแยกจากวัชพืชและจากหรือไม่
- เงาและป่าเดิมถูกนับเป็น Target หรือไม่
- มี Target อยู่ภายนอกขอบเขตมากหรือไม่
- Polygon ครอบพื้นที่ว่างหรือน้ำมากเกินไปหรือไม่
- พื้นที่เรือนยอดปิดถูกสรุปว่าไม่มีการปลูกหรือไม่
- Gap ที่ไม่ต่อกันถูกบังคับรวมกันหรือไม่
- แนวขอบเกาะ Tile หรือ Candidate Barrier โดยไม่มีหลักฐานหรือไม่
- จุดที่ต้องลงภาคสนามถูกระบุหรือไม่

## Review Status

```text
draft
needs_human_review
rework
reviewed
rejected
approved
```

AI ใช้ได้สูงสุด `needs_human_review` หรือ `rework`

`reviewed`, `rejected` และ `approved` ต้องมาจากผู้ตรวจที่ระบุชื่อและวันที่

## Outputs

- `qa_report.json`
- `qa_summary.csv`
- `warning_layers.gpkg`
- `review_checklist.csv`
- `review_package/` พร้อม Preview ทั้งหมด
- `analysis_manifest.json`

## Acceptance for Handoff

- มี Output และ Debug Layer ครบตามงานที่รัน
- Geometry Valid
- Warning ทุกข้อมีสถานะและคำอธิบาย
- จุดไม่มั่นใจไม่ถูกซ่อน
- ระบุข้อจำกัดของภาพและข้อมูลที่ไม่มี
- ระบุชัดว่าเป็น Candidate Enrichment Boundary ไม่ใช่ขอบเขตกฎหมายหรือผลยืนยันภาคสนาม
