# train-plant

ชุด Skill สำหรับช่วยวิเคราะห์ **แปลงปลูกเสริมในพื้นที่ป่าชายเลน** จากภาพโดรนหรือ Orthomosaic ขนาดใหญ่ ตั้งแต่การอ่านภาพ จำแนกสิ่งปกคลุม ตรวจโกงกาง แยกวัชพืชและต้นจาก วิเคราะห์ช่องว่าง วง Candidate Boundary และตรวจ QA

> ผลลัพธ์เป็นหลักฐานจากภาพและ Candidate Result ต้องผ่าน Human Review ก่อนใช้งานจริง ไม่ใช่ขอบเขตตามกฎหมาย และภาพช่วงเวลาเดียวไม่สามารถยืนยันได้ว่าต้นไม้เกิดจากการปลูก

## ความเข้าใจจากภาพตัวอย่าง

ระบบต้องแยกอย่างน้อย:

- `target_rhizophora_candidate` — พุ่มที่มีลักษณะคล้ายโกงกางเป้าหมาย มักเป็นก้อนค่อนข้างเข้มกว่า Groundcover
- `weed_groundcover` — วัชพืชหรือพืชคลุมดิน มักเป็นผืนละเอียด สีเขียวอ่อนหรือเขียวเหลือง
- `nypa_palm` — ต้นจาก มีใบยาวแผ่เป็นรัศมีคล้ายพัดหรือดาว
- `existing_woody_canopy` — ป่าเดิม พุ่มใหญ่ หลายขนาด หลายเฉด และ Texture หยาบ
- `closed_canopy_unknown` — พื้นที่ใต้เรือนยอดปิดที่มองไม่เห็น ห้ามสรุปว่าไม่มีการปลูก
- `dead_dry_vegetation`, `bare_mud`, `water`, `shadow_unknown` และ Class อื่นที่อาจรบกวน

ห้ามใช้สีเพียงอย่างเดียว ต้องพิจารณา **Color + Shape + Texture + Scale + Spatial Context** ร่วมกัน

## โครงสร้าง Skill

```text
AGENTS.md
README.md
skills/
├─ README.md
├─ 00-enrichment-analysis-orchestrator/
│  └─ SKILL.md
├─ 01-large-orthomosaic-reader/
│  └─ SKILL.md
├─ 02-mangrove-land-cover-classifier/
│  └─ SKILL.md
├─ 03-rhizophora-crown-detector/
│  └─ SKILL.md
├─ 04-weed-groundcover-classifier/
│  └─ SKILL.md
├─ 05-nypa-palm-detector/
│  └─ SKILL.md
├─ 06-existing-canopy-occlusion/
│  └─ SKILL.md
├─ 07-enrichment-gap-analyzer/
│  └─ SKILL.md
├─ 08-enrichment-boundary-delineator/
│  └─ SKILL.md
└─ 09-enrichment-qa-human-review/
   └─ SKILL.md
```

โฟลเดอร์ `planting-evidence-boundary/` เป็นเอกสารและตัวอย่างเดิม ใช้เป็น Reference และ Compatibility Router เท่านั้น งานใหม่ให้เริ่มจาก `skills/`

# เริ่มใช้งาน

## 1. เตรียม Orthomosaic

แนะนำให้เก็บภาพจริงไว้นอก Git Repository เช่น:

```text
D:\Drone\Enrichment_Project\Input\plot_001_orthomosaic.tif
D:\Drone\Enrichment_Project\Outputs\plot_001\
```

รองรับข้อมูลที่ GDAL/Rasterio อ่านได้ เช่น GeoTIFF, COG และ VRT โดยควรมี CRS และ Affine Transform

อย่านำ Orthomosaic หลาย GB เข้า Git โดยตรง ภาพใน `examples/` เป็นเพียง Crop สำหรับสอนการตีความ

## 2. เปิด Repository ใน Codex

ให้ Codex อ่าน `AGENTS.md` ก่อนทุกครั้ง

## 3. ใช้คำสั่งตามงาน

### A. วิเคราะห์ครบทั้งระบบ

```text
อ่าน AGENTS.md และใช้ Skill
skills/00-enrichment-analysis-orchestrator/SKILL.md

Input Orthomosaic:
D:\Drone\Enrichment_Project\Input\plot_001_orthomosaic.tif

Output:
D:\Drone\Enrichment_Project\Outputs\plot_001

ให้รัน Workflow ครบตั้งแต่ตรวจ Raster, แบ่ง Tile, จำแนกสิ่งปกคลุม,
ตรวจโกงกาง, แยกวัชพืชและต้นจาก, วิเคราะห์เรือนยอดปิด,
สร้าง Enrichment Gap, Candidate Boundary และ QA Package

หยุดให้ตรวจหลัง Classification, หลัง Target Detection และหลัง Candidate Boundary
ห้ามตั้งสถานะ approved
```

### B. ตรวจภาพและแบ่ง Tile เท่านั้น

```text
ใช้ skills/01-large-orthomosaic-reader/SKILL.md
ตรวจ Metadata ของ Orthomosaic และสร้าง Overview, Tile Index และ Tile Manifest
ห้ามวิเคราะห์ชนิดพืชและห้ามโหลดภาพทั้งหมดเข้า RAM

Input: <ORTHOMOSAIC_PATH>
Output: <OUTPUT_PATH>
```

### C. จำแนกสิ่งปกคลุม

```text
ใช้ skills/02-mangrove-land-cover-classifier/SKILL.md
จำแนกโกงกางเป้าหมายเบื้องต้น วัชพืช ต้นจาก ป่าเดิม น้ำ เลน เงา
เรือนยอดปิด และ Class ที่ไม่มั่นใจ
สร้าง Class Raster, Confidence Raster และ Preview

Input: <ORTHOMOSAIC_OR_TILE_MANIFEST>
Output: <OUTPUT_PATH>
```

### D. ตรวจและนับพุ่มโกงกาง

```text
ใช้ skills/03-rhizophora-crown-detector/SKILL.md
ตรวจ target_rhizophora_candidate เป็นรายพุ่ม
ใช้ Weed, Nypa, Shadow และ Existing Canopy เป็น Exclusion Context
ป้องกันการนับซ้ำระหว่าง Tile และสร้าง Detection Preview

Input: <ORTHOMOSAIC_OR_TILE_MANIFEST>
Output: <OUTPUT_PATH>
```

### E. จำแนกวัชพืช

```text
ใช้ skills/04-weed-groundcover-classifier/SKILL.md
แยก weed_groundcover, mixed_weed_and_target และ weed_obstruction
ห้ามสรุปว่าพื้นที่วัชพืชไม่มีต้นปลูก

Input: <ORTHOMOSAIC_OR_TILE_MANIFEST>
Output: <OUTPUT_PATH>
```

### F. ตรวจต้นจาก

```text
ใช้ skills/05-nypa-palm-detector/SKILL.md
ตรวจต้นจากจากรูปทรงใบแบบรัศมีและ Texture ของใบยาว
แยกออกจากโกงกาง ป่าเดิม และ Crown ชนิดอื่น

Input: <ORTHOMOSAIC_OR_TILE_MANIFEST>
Output: <OUTPUT_PATH>
```

### G. วิเคราะห์ช่องว่างปลูกเสริม

```text
ใช้ skills/07-enrichment-gap-analyzer/SKILL.md
ใช้ผลโกงกาง วัชพืช ต้นจาก ป่าเดิม น้ำ เลน เงา และเรือนยอดปิด
วิเคราะห์ทีละ Gap และสร้าง confirmed, probable, uncertain,
natural_regeneration และ disturbed_open_area
ยังไม่ต้องสร้าง Polygon ขอบเขตสุดท้าย

Input Layers: <CLASS_AND_DETECTION_OUTPUTS>
Output: <OUTPUT_PATH>
```

### H. สร้าง Candidate Boundary

```text
ใช้ skills/08-enrichment-boundary-delineator/SKILL.md
สร้าง enrichment_planting_core, enrichment_evidence_boundary
และ uncertain_enrichment_boundary จาก Gap Candidates ที่มีอยู่
รักษาคลอง น้ำ ป่าเดิม และรอยเว้าที่มีเหตุผล
ห้ามใช้ Convex Hull เป็นค่าเริ่มต้นและห้ามตั้งสถานะ approved

Input Layers: <GAP_AND_EVIDENCE_LAYERS>
Output: <OUTPUT_PATH>
```

### I. ตรวจ QA อย่างเดียว

```text
ใช้ skills/09-enrichment-qa-human-review/SKILL.md
ตรวจ Candidate Outputs ที่มีอยู่ สร้าง QA Metrics, Warning Layers,
Debug Previews และ Review Checklist
ห้ามแก้ผลให้ผ่านเอง

Input Directory: <ANALYSIS_OUTPUT_PATH>
Output: <QA_OUTPUT_PATH>
```

### J. ให้ Codex อธิบายภาพ Crop ก่อนพัฒนา

```text
อ่านภาพ Crop และไฟล์ตัวอย่างที่เกี่ยวข้องก่อน
ยังห้ามเขียน Code และห้ามสร้าง Polygon

ให้รายงาน:
1. ส่วนที่น่าจะเป็นโกงกางเป้าหมาย
2. ส่วนที่เป็นวัชพืช
3. ส่วนที่เป็นต้นจาก
4. ป่าเดิมและเรือนยอดปิด
5. น้ำ เลน เงา และสิ่งรบกวน
6. ส่วนที่มั่นใจและไม่มั่นใจ
7. Class ที่ควรใช้ Label
8. ข้อมูลเพิ่มเติมที่ต้องมีเพื่อยืนยันว่าเป็นการปลูก
```

# ผลลัพธ์หลักของ Full Pipeline

```text
raster_metadata.json
tile_index.gpkg
land_cover_class.tif
rhizophora_candidates.gpkg
weed_groundcover.tif
nypa_palm_candidates.gpkg
closed_canopy_unknown.tif
enrichment_gap_candidates.gpkg
enrichment_planting_core.gpkg
enrichment_evidence_boundary.gpkg
uncertain_enrichment_boundary.gpkg
boundary_segment_confidence.gpkg
qa_report.json
review_package/
analysis_manifest.json
```

# สถานะงาน

AI ใช้ได้สูงสุด:

```text
draft
needs_human_review
rework
```

สถานะต่อไปนี้ต้องมาจากมนุษย์:

```text
reviewed
rejected
approved
```

# ข้อจำกัด

Repository นี้เป็นชุด Skill, กฎ, ตัวอย่าง และคำสั่งสำหรับให้ Codex พัฒนาหรือควบคุม Workflow หาก Repository ยังไม่มีโปรแกรมประมวลผลที่ทำงานครบ Codex ต้องสร้างและทดสอบ Code ตาม Skill ก่อน จึงจะสามารถรัน Orthomosaic จริงได้
