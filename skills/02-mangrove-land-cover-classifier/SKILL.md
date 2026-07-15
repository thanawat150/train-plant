---
name: mangrove-land-cover-classifier
description: สร้าง Land-cover Context แบบหยาบจากภาพโดรนเพื่อ Route งานไป Skill ตรวจพืช สิ่งบดบัง และ Surface เฉพาะทาง ไม่ยืนยันชนิดพืชหรือคุณสมบัติดินขั้นสุดท้าย
---

# Mangrove Land-cover Classifier

## หน้าที่

สร้าง Context Map ก่อนตรวจ Target และวิเคราะห์ Gap

Skill นี้ไม่สร้าง Boundary ไม่ยืนยันว่าต้นเกิดจากการปลูก และไม่เป็นเจ้าของ Class รายละเอียดที่ Skill 03–06, 16 และ 17 รับผิดชอบ

## Core Context Classes

```text
woody_crown_candidate
low_vegetation_candidate
radial_frond_patch_candidate
existing_canopy_candidate
natural_regeneration_candidate
dead_or_dry_vegetation
bare_surface
water_surface
channel_or_tidal_feature
shadow_unknown
closed_canopy_unknown
unknown_object
```

## Specialist Ownership

```text
03 = target_rhizophora_candidate
04 = weed/groundcover
05 = nypa_palm patch
06 = existing canopy/occlusion
16 = detailed surface/hydrology
17 = palm/coconut candidate
```

ห้ามให้ Skill 02 สร้างผล Final แทน Specialist เมื่อมีการเรียก Specialist แล้ว

## Evidence Sources

ใช้ร่วมกันอย่างน้อย 2 กลุ่ม:

- Color: RGB, HSV, Lab, ExG, GLI, VARI
- Shape: compactness, radial leaves, object size, crown center
- Texture: local variance, entropy, GLCM หรือเทียบเท่า
- Context: gap, canal, forest edge, continuous patch, isolated crown
- Height: DSM/DTM/CHM เมื่อมี

ห้ามใช้สีหรือ Threshold เดี่ยวเป็นคำตอบสุดท้าย

## Routing Rules

- Woody Crown → Skill 03 หรือ 06 ตาม Context
- Low Vegetation → Skill 04
- Radial Patch แบบกอ/ผืน → Skill 05
- Radial Single Crown → Skill 17
- Surface/Hydrology ซับซ้อน → Skill 16
- Closed Canopy → Skill 06
- Unknown ต้องคงไว้ ไม่บังคับ Class

## Workflow

1. อ่าน Tile จาก Skill 01 และ `project_manifest.json` จาก 01b
2. Normalize สีโดยบันทึก Parameter
3. สร้าง Feature Stack
4. Segment เป็น Object/Superpixel หรือ Pixel Context
5. สร้าง Core Context พร้อม Confidence
6. Route Object ไป Specialist
7. ทำ Tile-edge Suppression
8. รวมผลใน CRS ต้นฉบับ
9. สร้าง Review Samples เมื่อมี Ground Truth

## Outputs

```text
land_cover_context.tif
land_cover_context_confidence.tif
land_cover_context_objects.gpkg
routing_candidates.gpkg
context_legend.json
classification_preview.png
```

เพื่อ Compatibility สามารถ Export Alias เดิมได้:

```text
land_cover_class.tif
land_cover_confidence.tif
land_cover_objects.gpkg
```

แต่ต้องระบุว่าเป็น Coarse Context ไม่ใช่ Specialist Final Class

## QA

- ทุก Class มี Confidence
- Unknown ไม่ถูกบังคับ
- Closed Canopy ไม่ถูกแปลงเป็นพื้นที่ไม่มีการปลูก
- ตรวจ Water vs Shadow
- ตรวจ Nypa-like Patch vs Single Palm Crown
- ตรวจ Seam/Lighting Artifact
- ตรวจ Routing Coverage ว่า Object สำคัญถูกส่ง Specialist หรือไม่
