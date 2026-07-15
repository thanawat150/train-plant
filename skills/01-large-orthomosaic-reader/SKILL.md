---
name: large-orthomosaic-reader
description: อ่านและเตรียม Orthomosaic, GeoTIFF, COG หรือ VRT ขนาดใหญ่แบบไม่โหลดทั้งไฟล์เข้า RAM สร้าง Overview, Tile/Window และ Metadata ที่รักษาพิกัด
---

# Large Orthomosaic Reader

## หน้าที่

เตรียม Raster ขนาดใหญ่ให้ Skill อื่นใช้ ไม่วิเคราะห์พืช ไม่สร้างจำนวนต้น และไม่วงขอบเขต

## Required Metadata

- source path
- width, height, band count, dtype
- CRS และ Affine Transform
- pixel size และ bounds
- nodata/alpha
- internal overviews
- block size และ file size
- acquisition date เมื่อมี

## Processing

1. เปิด Raster แบบ Read-only
2. ตรวจ CRS, Transform และ NoData
3. สร้าง Overview Cache/VRT แยกเมื่อจำเป็น โดยไม่แก้ต้นฉบับ
4. ใช้ AOI ลดพื้นที่ค้นหาได้ แต่ AOI ไม่ใช่ผลขอบเขต
5. ใช้ Tile 2048 หรือ 4096 Pixel ตาม Config
6. ใช้ Overlap 10–20% ค่าเริ่มต้น 15%
7. บันทึก Window Transform, Bounds และ Tile ID
8. สร้าง Overview Preview และ Tile Index

## Tile Metadata

```text
tile_id
source_raster
col_off
row_off
width
height
window_transform
crs
bounds
valid_pixel_ratio
overlap_percent
```

## Outputs

```text
raster_metadata.json
overview_preview.png
tile_index.gpkg
tile_manifest.jsonl
optional VRT/overview cache
```

## QA

- Tile Pixel ต้องแปลงกลับพิกัดต้นฉบับได้
- ห้ามสลับ Band/สีโดยไม่บันทึก
- ตรวจ Valid Pixel ที่ขอบภาพ
- ห้ามรวม Local Pixel ข้าม Tile โดยตรง
- ห้ามโหลด Raster ใหญ่ทั้งหมดเข้า RAM

## Stop Conditions

- CRS หรือ Transform หาย
- Raster เสียหรืออ่านไม่ได้
- Pixel Size ไม่สมเหตุสมผล
- JPG/PNG ไม่มีพิกัดแต่ผู้ใช้ต้องการพื้นที่จริง

JPG/PNG ใช้ได้เฉพาะ `image_demo_only`

## Downstream Contract

งาน Production ต้องส่ง `raster_metadata.json` และ Input Layers ไป Skill 01b เพื่อสร้าง `project_manifest.json` ก่อนเริ่ม Analysis

งานทดลองภาพอย่างเดียวสามารถข้าม 01b ได้ แต่ห้ามรายงานระยะ พื้นที่ หรืออัตรารอดจริง
