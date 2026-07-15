# Skill Index

เลือกใช้ Skill ตามงานและอ่านเฉพาะไฟล์ที่จำเป็น เพื่อลด Token

## Shared

| Skill | หน้าที่ |
|---|---|
| `01-large-orthomosaic-reader` | อ่าน GeoTIFF/COG/VRT ขนาดใหญ่ สร้าง Overview, Tile Index และ Manifest |

## ปลูกเสริม

| Skill | หน้าที่ |
|---|---|
| `00-enrichment-analysis-orchestrator` | เรียก Workflow ปลูกเสริมทั้งชุด |
| `02-mangrove-land-cover-classifier` | จำแนกสิ่งปกคลุมหลัก |
| `03-rhizophora-crown-detector` | ตรวจพุ่มโกงกาง Candidate |
| `04-weed-groundcover-classifier` | แยกวัชพืชและการบดบัง |
| `05-nypa-palm-detector` | ตรวจต้นจาก |
| `06-existing-canopy-occlusion` | ป่าเดิม เงา และเรือนยอดปิด |
| `07-enrichment-gap-analyzer` | วิเคราะห์ช่องว่างแบบ Gap-by-gap |
| `08-enrichment-boundary-delineator` | สร้างขอบเขตปลูกเสริม Candidate |
| `09-enrichment-qa-human-review` | QA และชุด Human Review |

## ปลูกเต็มพื้นที่

| Skill | หน้าที่ |
|---|---|
| `10-full-area-planting-orchestrator` | เรียก Workflow ปลูกเต็มทั้งชุด |
| `11-full-area-tree-detector` | ตรวจต้นปลูกเป็นรายพุ่ม แยกเงาและต้นเดิม |
| `12-planting-grid-inference` | หาแถว กริด ระยะปลูก และตำแหน่งต้นหาย |
| `13-full-area-mortality-analyzer` | วิเคราะห์อัตรารอด ต้นหาย และพื้นที่ตรวจไม่ได้ |
| `14-full-area-boundary-delineator` | สร้าง Core, Evidence Boundary และ Uncertain Edge |
| `15-full-area-qa-human-review` | QA และชุด Human Review สำหรับปลูกเต็ม |

# Token-efficient load sets

| งาน | อ่านเต็ม | อ่านเฉพาะ Output Contract |
|---|---|---|
| อ่านภาพ/แบ่ง Tile | 01 | ไม่มี |
| นับต้นปลูกเต็ม | 01, 11 | ไม่มี |
| หา Grid | 12 | 11 |
| วิเคราะห์อัตรารอด | 13 | 11, 12 |
| วงขอบเขตปลูกเต็ม | 14 | 11, 12, 13 |
| QA ปลูกเต็ม | 15 | 11–14 |
| จำแนกสิ่งปกคลุมปลูกเสริม | 01, 02 | ไม่มี |
| หาโกงกางในปลูกเสริม | 03 | 01, 02 |
| แยกวัชพืช | 04 | 01, 02 |
| แยกต้นจาก | 05 | 01, 02 |
| วิเคราะห์ Gap ปลูกเสริม | 07 | 02–06 |
| วงขอบเขตปลูกเสริม | 08 | 07 |
| QA ปลูกเสริม | 09 | 02–08 |

## วิธีอ่าน Output Contract

เมื่อไม่ต้องอ่าน Skill ก่อนหน้าทั้งไฟล์ ให้อ่านเฉพาะหัวข้อ:

```text
Inputs
Outputs
Required attributes
Required metrics
Warnings
```

ห้ามส่ง Prompt และคำอธิบายจากขั้นก่อนหน้าซ้ำทั้งหมด ให้ส่งเพียง:

```text
source_file_paths
schema/version
crs
config_used
summary_metrics
warnings
```

# Routing rules

- งานครบปลูกเสริม: Skill 00
- งานครบปลูกเต็ม: Skill 10
- ห้ามอ่าน Skill 00–09 เมื่องานเป็นปลูกเต็ม เว้นแต่ต้องใช้ Classifier เฉพาะจริง
- ห้ามอ่าน Skill 10–15 เมื่องานเป็นปลูกเสริม
- ภาพ Crop ใช้สอนการตีความ ไม่ใช่พิกัดจริง
- งานจริงต้องรักษา CRS และ Affine Transform
- AI สร้างได้เฉพาะ Candidate Result และห้ามตั้งสถานะ `approved`
