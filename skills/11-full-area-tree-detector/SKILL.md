---
name: full-area-tree-detector
description: สร้าง Raw Crown/Tree Candidates ในแปลงปลูกเต็มพื้นที่จากภาพจริง ผลยังไม่ใช่จำนวนต้นสุดท้ายและต้องผ่าน Skill 12, 12c และ 12b
---

# Full-area Tree Detector

## Scope

ทำเฉพาะ Raw Detection จากเรือนยอดที่มองเห็น ห้ามอนุมานต้นจาก Grid สรุปจำนวนสุดท้าย วงขอบเขต หรือคำนวณอัตรารอด

ผลต้องผ่าน Skill 12, 12c และ 12b

## Inputs

- Tile Manifest จาก Skill 01
- `project_manifest.json` จาก Skill 01b
- CRS และ Pixel Size
- Optional AOI
- Optional Surface/Hydrology จาก Skill 16
- Optional Palm/Coconut Exclusion จาก Skill 17

## Detection Rules

- ตรวจจาก Canopy Evidence ไม่ใช้สีหรือ Texture เดี่ยว
- สร้าง Crown Polygon และ Candidate Center เมื่อทำได้
- จุดบนดิน น้ำเปล่า ถนน คันดิน หรือ Shadow-only ไม่ใช่ต้น
- น้ำไม่ใช่ Exclusion ทั้งหมด: ต้นในน้ำตื้นนับได้เมื่อมีเรือนยอดจริง
- แยก `existing_large_tree_candidate` เมื่อขนาด รูปทรง หรือบริบทต่างจากต้นปลูกรอบข้าง
- แยก `touching_crown_cluster`, `merged_planted_canopy_candidate` และ `closed_canopy_unresolved`
- ทุก Touching/Merged Cluster ต้องมี `touching_cluster_id`
- รวม Detection ซ้ำใน Tile Overlap
- ห้ามใช้ Expected Count หรือ Grid เติม Detection

## Classes

```text
isolated_planted_crown_candidate
probable_planted_crown_candidate
touching_crown_cluster
merged_planted_canopy_candidate
closed_canopy_unresolved
existing_large_tree_candidate
palm_or_other_large_crown_candidate
shadow_only
bare_ground_false_candidate
water_surface_false_candidate
unknown_crown
```

## Required Attributes

```text
object_id
class
confidence
crown_area_sqm
crown_diameter_m
crown_center_x
crown_center_y
canopy_support_score
center_clarity
touching_cluster_id
source_tile
source_raster
edge_distance_px
duplicate_group
review_status
```

## Outputs

```text
planted_tree_candidates.gpkg
raw_crown_objects.gpkg
touching_crown_clusters.gpkg
existing_large_trees.gpkg
shadow_mask.tif
optional canopy_mask.tif
optional canopy_probability.tif
tree_detection_preview.png
tree_detection_metrics.json
```

`touching_crown_clusters.gpkg` ต้องมีทุก Cluster ที่ชิด รวมเป็นผืน หรือยังแยกไม่ได้ ห้ามตัด Cluster ใหญ่ทิ้งด้วย Area Threshold

## QA

- จุด Candidate ไม่มีพุ่มรองรับ
- Candidate บนดิน น้ำ ถนน หรือ Shadow-only
- Duplicate ใน Tile Overlap
- ต้นเดิมขนาดใหญ่ถูกเสนอเป็นต้นปลูก
- พุ่มเดียวถูกแบ่งหลายจุด
- เรือนยอดชิดถูกบังคับเป็นหนึ่งต้น
- Cluster ใหญ่ถูกข้ามโดยไม่มีสถานะ
- รายงาน High/Medium/Low Confidence และ Cluster ที่ยังแยกไม่ได้

ห้ามปรับ Threshold เพื่อให้จำนวนตรงค่าที่คาด

## Downstream

- Skill 12 ใช้เฉพาะต้นเดี่ยว High-confidence หา Preliminary Spacing/Pattern
- Skill 12c ใช้ `touching_crown_clusters.gpkg` ร่วมกับ Pattern จาก Skill 12 เพื่อสร้าง Image-supported Crown Centers
- Skill 12b รวม Raw Detection และผล 12c เพื่อตัดสิน Final Class
- Skill 13–15 ห้ามใช้ผล Skill 11 เป็นจำนวนสุดท้ายโดยตรง