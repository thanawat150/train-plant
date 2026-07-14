# Large Orthomosaic Workflow

## Scope

เอกสารนี้ใช้เมื่อข้อมูลจริงเป็นภาพ Orthomosaic หรือ GeoTIFF ขนาดใหญ่ ซึ่งไม่สามารถส่งให้โมเดลมองทั้งภาพในความละเอียดเต็มได้ในครั้งเดียว

ภาพใน `examples/example_01` เป็นเพียง Crop สำหรับสอนการตีความว่าอะไรควรรวมและอะไรควรตัดออก ไม่ใช่ขนาดภาพจริงและไม่ใช่หน่วยวิเคราะห์หลักของระบบ

## เป้าหมาย

สร้าง Candidate Boundary จาก Orthomosaic ขนาดใหญ่โดย:

- รักษาพิกัดภูมิศาสตร์เดิม
- ไม่โหลด Raster ทั้งไฟล์เข้า RAM
- ตรวจภาพทั้งระดับ Overview และระดับรายละเอียด
- ลดรอยต่อระหว่าง Tile
- รวมผลเป็น Polygon ใน CRS จริง
- รองรับพื้นที่ปลูกหลายกลุ่ม
- สร้างหลักฐานสำหรับ Human Review

## Supported Inputs

- GeoTIFF
- Cloud Optimized GeoTIFF หรือ COG
- VRT
- Raster ที่ GDAL/Rasterio อ่านได้

Input ควรมี:

- CRS
- Affine Transform
- Pixel Size
- NoData
- Band Information

หาก Input เป็น JPG/PNG ที่ไม่มีพิกัด ให้ใช้ได้เฉพาะสร้างตัวอย่างหรือทดลอง Algorithm ห้ามคำนวณพื้นที่จริง

## Processing Modes

### Mode A — Known AOI

ใช้เมื่อมี PDD, ขอบเขตแปลง หรือ AOI สำหรับจำกัดพื้นที่ค้นหา

- ใช้ AOI เป็นกรอบค้นหาและลดปริมาณข้อมูล
- เพิ่ม Buffer รอบ AOI เพื่อไม่ตัดร่องรอยปลูกที่อยู่ใกล้ขอบ
- ห้ามคัดลอก AOI มาเป็น Candidate Boundary โดยตรง

### Mode B — Full Orthomosaic Scan

ใช้เมื่อยังไม่มี AOI ที่เชื่อถือได้

- สร้าง Overview ระดับต่ำเพื่อค้นหากลุ่มที่น่าจะมีร่องรอยปลูก
- แบ่ง Candidate Region เป็นหน้าต่างย่อย
- ประมวลผล Detail Tile เฉพาะบริเวณที่มีโอกาสพบร่องรอยปลูก

## Phase 1 — Raster Inspection

อ่านและบันทึก:

```text
source_path
width_px
height_px
band_count
dtype
nodata
crs
transform
pixel_size_x
pixel_size_y
bounds
overviews
block_size
file_size
```

ตรวจ:

- CRS หายหรือไม่
- Pixel Size สมเหตุสมผลหรือไม่
- มี Alpha/NoData หรือไม่
- มีขอบดำ ขอบขาว หรือช่องว่างจากการต่อภาพหรือไม่
- มี Internal Overviews หรือไม่

หาก Raster ใหญ่มากและไม่มี Overviews ให้สร้าง VRT หรือ Overview Cache แยก โดยห้ามแก้ไฟล์ต้นฉบับ

## Phase 2 — Multi-Scale Review

### Overview Level

สร้างภาพ Preview ลดความละเอียดสำหรับ:

- ดูขอบเขตภาพทั้งหมด
- ระบุคลอง ถนน คันดิน และแปลงข้างเคียง
- ระบุ Candidate Region เบื้องต้น
- ตรวจพื้นที่ที่ไม่ควรประมวลผล

Overview ห้ามใช้วงแนวขอบสุดท้ายถ้ามองไม่เห็นต้นหรือรูปแบบการปลูก

### Detail Level

ใช้ความละเอียดจริงหรือระดับที่ยังเห็น:

- ต้นกล้า
- แนวแถว
- ระยะปลูก
- ช่องว่าง
- แนวสิ่งกีดขวาง

## Phase 3 — Tile Generation

ค่าเริ่มต้นแนะนำ:

```yaml
tile_size_px: 2048
alternate_tile_size_px: 4096
overlap_percent: 15
minimum_overlap_percent: 10
maximum_overlap_percent: 20
```

เลือก Tile Size ตาม:

- ขนาดต้นในหน่วย Pixel
- RAM/VRAM
- ขนาด Context ที่ Algorithm ต้องใช้
- ความกว้างของคลองหรือคันดิน

ทุก Tile ต้องมี Metadata:

```text
tile_id
source_raster
window_col_off
window_row_off
window_width
window_height
window_transform
crs
bounds
valid_pixel_ratio
```

ห้ามบันทึกเฉพาะชื่อ `tile_001.png` โดยไม่มี Window และ Transform

## Phase 4 — Tile Analysis

สำหรับแต่ละ Tile:

1. ตัด NoData และขอบภาพ
2. ปรับภาพเพื่อวิเคราะห์โดยไม่แก้ Source Raster
3. ตรวจจุดต้นหรือทรงพุ่มขนาดเล็ก
4. ตรวจรูปแบบแถวหรือกริด
5. สร้าง Density Surface
6. ตรวจคลอง ถนน คันดิน น้ำเปิด และพื้นที่ว่าง
7. สร้าง Candidate Mask
8. สร้าง Confidence Raster หรือ Confidence Attribute
9. บันทึก Debug Preview เฉพาะเมื่อจำเป็น

ใช้หลายหลักฐานร่วมกัน เช่น:

- Tree/Object Detection
- RGB Vegetation Indices
- Texture
- Row Pattern
- Spacing Consistency
- Density
- Segmentation

ห้ามใช้ Color Threshold ค่าเดียวตัดสินผลสุดท้าย

## Phase 5 — Tile Edge Handling

Overlap มีไว้ลดผลกระทบจากการตัดบริบท

กฎ:

- Prediction ในเขตกลาง Tile มีน้ำหนักสูงกว่า
- Prediction ใกล้ขอบ Tile ต้องลดน้ำหนัก
- หาก Feature ปรากฏใน Tile ซ้อนกัน ให้รวมด้วย Confidence หรือ Majority Agreement
- ห้ามเก็บเส้นตรงตามขอบ Tile เป็นขอบเขตจริง เว้นแต่มีสิ่งกีดขวางจริงตรงตำแหน่งนั้น
- Flag Polygon ที่แนวขอบจำนวนมากตรงกับ Tile Boundary เป็น `tile_edge_warning`

## Phase 6 — Convert to Map Coordinates

ทุก Pixel Coordinate ต้องแปลงผ่าน Window Transform ของ Tile กลับไปยัง Map Coordinate

```text
local tile pixel
→ source raster pixel
→ affine transform
→ map coordinate in source CRS
```

ห้ามรวม Polygon จาก Tile ด้วยพิกัด Pixel Local

## Phase 7 — Merge Evidence

หลังแปลงเป็น Map Coordinate:

1. รวม Candidate Polygon จาก Tile ซ้อนกัน
2. Dissolve กลุ่มที่ต่อเนื่อง
3. เชื่อมช่องว่างเล็กตามระยะปลูกจริง
4. แยกกลุ่มที่ถูกคลอง ถนน หรือคันดินแบ่ง
5. ตัดพื้นที่น้ำเปิดหรือพื้นที่ไม่มีหลักฐาน
6. รักษารอยเว้าที่มีเหตุผล
7. สร้าง Hole เมื่อมีพื้นที่ยกเว้นขนาดใหญ่
8. สร้าง MultiPolygon เมื่อกลุ่มปลูกไม่ต่อกัน

ห้ามใช้ Convex Hull ปิดทุกกลุ่มเข้าด้วยกัน

## Phase 8 — Boundary Refinement

ใช้ภาพความละเอียดจริงตรวจแนวขอบรอบ Candidate Polygon

แบ่งขอบเป็น Segment และให้แต่ละ Segment มี:

```text
segment_id
confidence
boundary_evidence
nearest_barrier
review_status
remarks
```

Candidate Boundary ต้องตาม:

- จุดปลูกด้านนอกสุดที่ต่อเนื่อง
- แนวคลอง ถนน หรือคันดินเมื่อเป็น Barrier จริง
- แนวเปลี่ยนผ่านระหว่างรูปแบบปลูกกับพื้นที่ไม่มีหลักฐาน

## Phase 9 — Geometry QA

ตรวจอย่างน้อย:

- Geometry Validity
- Self Intersection
- Spike
- Sliver
- Duplicate Vertex
- Tiny Hole
- Tile Edge Artifact
- Polygon Overlap
- Unexpected Multipart
- CRS
- Area Unit

คำนวณพื้นที่ใน Projected CRS ที่เหมาะสม

## Phase 10 — Review Package

สร้าง:

```text
outputs/<job_id>/
├─ candidate_boundary.gpkg
├─ candidate_boundary.geojson
├─ candidate_boundary.kml
├─ overview_preview.jpg
├─ detail_previews/
├─ uncertain_zones.gpkg
├─ excluded_areas.gpkg
├─ tile_index.gpkg
├─ manifest.json
└─ review_checklist.csv
```

Overview Preview แสดงภาพทั้งหมดพร้อม Candidate Boundary

Detail Preview ต้องสร้างเฉพาะ:

- แนวขอบความมั่นใจต่ำ
- จุดเชื่อม Tile
- พื้นที่ติดคลอง ถนน หรือคันดิน
- จุดที่ Polygon ต่างจาก PDD มาก

## Resume and Fault Tolerance

ระบบต้อง:

- บันทึกผลต่อ Tile
- ข้าม Tile ที่ประมวลผลสำเร็จแล้ว
- Retry เฉพาะ Tile ที่ล้มเหลว
- ไม่ทำให้ผลเดิมเสียเมื่อหยุดกลางทาง
- เขียน Manifest แบบ Atomic
- เก็บ Hash หรือ Modified Time ของ Source Raster เพื่อป้องกัน Resume ผิดไฟล์

## Performance Rules

- ห้ามอ่าน Raster ทั้งไฟล์ด้วย `read()` โดยไม่ระบุ Window เมื่อไฟล์ใหญ่
- ห้ามสร้าง PNG ของ Orthomosaic ทั้งภาพที่ความละเอียดเต็ม
- ใช้ Overviews, Windowed Reading และ Block Processing
- จำกัดจำนวน Worker ตาม RAM และ Disk Throughput
- ห้ามสันนิษฐานว่าเพิ่ม Worker แล้วเร็วขึ้นเสมอ
- เก็บ Temporary Files แยกและลบเมื่อ Job ผ่าน QA

## Acceptance Criteria

1. รองรับ Orthomosaic ที่ใหญ่กว่าหน่วยความจำเครื่อง
2. ผลลัพธ์อยู่ใน CRS เดิมหรือ CRS ที่กำหนดอย่างชัดเจน
3. ไม่มีรอยต่อ Tile ที่ถูกตีความเป็นขอบเขตจริง
4. สามารถ Resume และ Retry ราย Tile ได้
5. มี Overview และ Detail Review
6. Candidate Polygon อ้างย้อนกลับไปยัง Tile และ Source Raster ได้
7. AI ไม่อนุมัติผลเอง
8. ไม่ใช้ Screenshot Pixel Coordinate เป็นพิกัดจริง
