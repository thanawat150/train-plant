---
name: large-orthomosaic-reader
description: อ่านและเตรียม Orthomosaic, GeoTIFF, COG หรือ VRT ขนาดใหญ่แบบไม่โหลดทั้งไฟล์เข้า RAM สร้าง Overview, Tile/Window และ Metadata ที่รักษาพิกัด ใช้ก่อนการจำแนกหรือ Detection ทุกประเภท
---

# Large Orthomosaic Reader

## หน้าที่

เตรียมข้อมูล Raster ขนาดใหญ่ให้ Skill อื่นใช้งาน โดยไม่วิเคราะห์ชนิดพืชและไม่สร้างขอบเขตปลูก

## Required Metadata

- source path
- width, height, band count และ dtype
- CRS และ Affine Transform
- pixel size และ bounds
- nodata/alpha
- internal overviews
- block size และ file size
- acquisition date ถ้ามี

## Processing

1. เปิด Raster แบบ read-only
2. ตรวจ CRS, Transform และ NoData
3. สร้าง Overview Cache หรือ VRT แยกเมื่อจำเป็น โดยไม่แก้ต้นฉบับ
4. ใช้ AOI เพื่อลดพื้นที่ค้นหาเมื่อมี แต่ห้ามใช้ AOI เป็นคำตอบของขอบเขต
5. สร้าง Tile/Window ขนาดเริ่มต้น 2048 หรือ 4096 Pixel
6. ใช้ Overlap 10–20% ค่าเริ่มต้น 15%
7. บันทึก Window Transform, Bounds และ Tile ID ทุก Tile
8. สร้าง Overview Preview และ Tile Index

## Required Tile Metadata

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

- `raster_metadata.json`
- `overview_preview.png`
- `tile_index.gpkg`
- `tile_manifest.jsonl`
- VRT/Overview Cache เมื่อจำเป็น

## QA

- Pixel ของ Tile ต้องแปลงกลับเป็นพิกัดต้นฉบับได้
- Tile ต้องไม่มีการสลับ Band หรือเปลี่ยนสีโดยไม่บันทึก
- ตรวจว่า Tile ขอบภาพมี Valid Pixel เพียงพอ
- ห้ามใช้ Local Pixel จากคนละ Tile รวมกันโดยตรง
- ห้ามโหลด Raster ทั้งไฟล์เข้า RAM เมื่อไฟล์ใหญ่

## Stop Conditions

หยุดและรายงานเมื่อ:

- CRS หรือ Transform หาย
- Raster เสียหรืออ่านไม่ได้
- Pixel Size ไม่สมเหตุสมผล
- Input เป็น JPG/PNG ไม่มีข้อมูลตำแหน่ง แต่ผู้ใช้ต้องการพื้นที่จริง

JPG/PNG ใช้ได้เฉพาะทดลองหรือทำตัวอย่างเชิงภาพ
