---
name: full-area-tree-detector
description: สร้าง Raw Crown/Tree Candidates ในแปลงปลูกเต็มพื้นที่จากภาพจริง โดยแยกเงา ต้นเดิมขนาดใหญ่ และสิ่งรบกวนเบื้องต้น ผลยังไม่ใช่จำนวนต้นสุดท้ายและต้องผ่าน Skill 21
---

# Full-area Tree Detector

## Scope

ทำเฉพาะ Raw Detection จากเรือนยอดที่มองเห็น ห้ามอนุมานต้นจากกริด ห้ามสรุปจำนวนสุดท้าย ห้ามวงขอบเขต และห้ามคำนวณอัตรารอด

ผลจาก Skill นี้ต้องผ่าน Skill 12 และ Skill 21 ก่อนใช้เป็นจำนวนต้น

## Inputs

- Orthomosaic หรือ Tile Manifest จาก Skill 01
- CRS และ Pixel Size
- Optional AOI
- Optional Surface/Hydrology Mask จาก Skill 16
- Optional Palm/Coconut Exclusion จาก Skill 17

## Detection rules

- ตรวจจาก Canopy Evidence ไม่ตรวจจากจุดสีหรือ Texture เดี่ยว
- สร้างทั้ง Crown Polygon และ Candidate Center เมื่อทำได้
- จุดบนดินโล่ง น้ำเปล่า ถนน คันดิน หรือ Shadow-only ต้องไม่ถูกถือเป็นต้น
- น้ำไม่ใช่ Exclusion ทั้งหมด: ต้นในน้ำตื้นนับได้เมื่อมีเรือนยอดจริงรองรับ
- แยก `existing_large_tree_candidate` เมื่อขนาด รูปทรง หรือบริบทต่างจากประชากรต้นปลูกอย่างชัดเจน
- แยก `touching_crown_cluster` เมื่อหลายพุ่มแตะกัน
- แยก `merged_canopy_candidate` เมื่อขอบพุ่มรวมเป็นผืนและยังไม่ทราบจำนวนต้น
- รวม Detection ซ้ำใน Tile Overlap ด้วยตำแหน่งและ Crown Overlap
- ห้ามใช้ Expected Count หรือ Grid เพื่อเติม Detection ใน Skill นี้

## Classes

```text
isolated_planted_crown_candidate
probable_planted_crown_candidate
touching_crown_cluster
merged_canopy_candidate
existing_large_tree_candidate
palm_or_other_large_crown_candidate
shadow_only
bare_ground_false_candidate
water_surface_false_candidate
unknown_crown
```

## Required attributes

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
tree_detection_preview.png
tree_detection_metrics.json
```

## QA

- ตรวจจุด Candidate ที่ไม่มีพุ่มจริงรองรับ
- ตรวจ Candidate บนดิน น้ำ ถนน และ Shadow-only
- ตรวจการนับซ้ำบริเวณ Tile Overlap
- ตรวจต้นเดิมขนาดใหญ่ที่ถูกเสนอเป็นต้นปลูก
- ตรวจพุ่มเดียวที่ถูกแบ่งเป็นหลายจุด
- ตรวจเรือนยอดชิดที่ถูกบังคับเป็นหนึ่งต้น
- รายงานจำนวน High/Medium/Low Confidence และ Cluster ที่ยังแยกไม่ได้
- ห้ามปรับ Threshold เพื่อให้จำนวนตรงกับค่าที่คาดไว้

## Downstream

- Skill 12 ใช้เฉพาะต้นเดี่ยว High-confidence เพื่อหา Reference Spacing และ Grid เบื้องต้น
- Skill 21 ตรวจ Raw Detection ทั้งหมดร่วมกับ Grid/จุดปลูก แล้วสร้างจำนวนต้นที่ผ่านการ Validation
- Skill 13 ห้ามใช้ผล Skill 11 โดยตรง
