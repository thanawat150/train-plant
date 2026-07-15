---
name: full-area-tree-detector
description: ตรวจต้นปลูกเป็นรายพุ่มในแปลงปลูกเต็มพื้นที่ แยกเงา ต้นเดิมขนาดใหญ่ และวัตถุรบกวน พร้อมป้องกันการนับซ้ำระหว่าง Tile
---

# Full-area Tree Detector

## Scope

ทำเฉพาะการตรวจต้นหรือพุ่ม Candidate ห้ามอนุมานกริด ห้ามวงขอบเขต และห้ามสรุปอัตรารอดใน Skill นี้

## Inputs

- Orthomosaic หรือ Tile Manifest จาก Skill 01
- CRS และ Pixel Size
- Optional AOI

## Detection rules

- หนึ่งพุ่มที่แยกได้ = หนึ่ง Candidate Object
- ใช้ Color + Shape + Texture + Object Scale
- แยกเงาด้วยทิศทางเงาที่สอดคล้องกันและตำแหน่งติดกับเรือนยอด
- แยก `existing_large_tree` เมื่อขนาดพุ่มผิดจากประชากรต้นปลูกอย่างชัดเจน
- รวม Detection ซ้ำใน Tile Overlap ด้วยตำแหน่งและ Crown Overlap
- พุ่มที่ไม่ชัดให้เป็น `probable_planted_tree` หรือ `unknown_crown`

## Classes

```text
planted_tree_candidate
probable_planted_tree
existing_large_tree
shadow
unknown_crown
```

## Required attributes

```text
object_id
class
confidence
crown_area_sqm
crown_diameter_m
source_tile
source_raster
edge_distance_px
duplicate_group
review_status
```

## Outputs

```text
planted_tree_candidates.gpkg
existing_large_trees.gpkg
shadow_mask.tif
tree_detection_preview.png
tree_detection_metrics.json
```

## QA

- ตรวจการนับซ้ำบริเวณ Tile Overlap
- ตรวจเงาที่ถูกนับเป็นต้น
- ตรวจต้นเดิมขนาดใหญ่ที่ถูกนับเป็นต้นปลูก
- รายงานจำนวน High/Medium/Low Confidence
- ห้ามปรับ Threshold เพื่อให้จำนวนตรงกับค่าที่คาดไว้โดยไม่มีหลักฐาน
