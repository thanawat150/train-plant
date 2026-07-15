# Skill Index

เลือกใช้ Skill ตามงานและอ่านเฉพาะไฟล์ที่จำเป็น เพื่อลด Token

## Shared

| Skill | หน้าที่ |
|---|---|
| `01-large-orthomosaic-reader` | อ่าน GeoTIFF/COG/VRT ขนาดใหญ่ สร้าง Overview, Tile Index และ Manifest |
| `16-surface-hydrology-condition-classifier` | จำแนกสภาพผิว ดิน เลน น้ำขัง แอ่งน้ำ และร่องน้ำที่มองเห็น |
| `17-palm-coconut-detector` | แยกปาล์ม/มะพร้าว Candidate ออกจากต้นจากและโกงกาง |
| `18-field-inspection-priority-analyzer` | เลือกจุดและโซนที่ควรเข้าตรวจจากความไม่แน่นอน ความเสียหาย ผลกระทบ และ QA |
| `19-multimodal-access-route-planner` | ประเมินเส้นทางเข้าจุดตรวจทางบก ทางเรือ และการเดินช่วงสุดท้าย |
| `20-field-inspection-mission-planner` | รวมจุดตรวจและ Route เป็นภารกิจ วัน ทีม และ Field Checklist |

## ปลูกเสริม

| Skill | หน้าที่ |
|---|---|
| `00-enrichment-analysis-orchestrator` | เรียก Workflow ปลูกเสริมทั้งชุด |
| `02-mangrove-land-cover-classifier` | จำแนกสิ่งปกคลุมหลัก |
| `03-rhizophora-crown-detector` | ตรวจพุ่มโกงกาง Candidate |
| `04-weed-groundcover-classifier` | แยกวัชพืชและการบดบัง |
| `05-nypa-palm-detector` | ตรวจต้นจากที่ขึ้นเป็นกอหรือผืน |
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

# Field Inspection Planning

ใช้หลังมีผลวิเคราะห์แล้ว:

```text
Analysis/QA Outputs
→ Skill 18: จุดไหนต้องตรวจและเพราะอะไร
→ Skill 19: เข้าทางบกหรือทางเรือ ใช้เวลาและระยะเท่าไร
→ Skill 20: จัดกลุ่มเป็น Mission, วัน, ทีม และ Checklist
```

## หลักการสำคัญ

- `evidence_priority_score` = จุดนี้จำเป็นต้องตรวจมากเพียงใด
- `access_burden_score` = จุดนี้เข้าถึงยากเพียงใด
- ห้ามรวมสองคะแนนจนจุดสำคัญแต่ไกลหายออกจากแผน
- จุดสำคัญและไกลให้เป็น `special_mission` หรือ `remote_cluster_mission`
- จุดไกลหลายจุดควรรวมเป็นเที่ยวเดียวเมื่อคุ้มค่า
- ระยะเข้าถึงต้องใช้ Network Distance ไม่ใช่เส้นตรง
- Route ทางบกและทางเรือต้องประเมินแยก
- Route จากภาพเป็น Candidate ต้องให้ทีมพื้นที่ยืนยัน

# Token-efficient load sets

| งาน | อ่านเต็ม | อ่านเฉพาะ Output Contract |
|---|---|---|
| อ่านภาพ/แบ่ง Tile | 01 | ไม่มี |
| จำแนก Surface/Hydrology | 16 | 01 |
| แยก Palm/Coconut | 17 | 01, 02 หรือ 05 ตาม Input |
| นับต้นปลูกเต็ม | 01, 11 | ไม่มี |
| หา Grid | 12 | 11 |
| วิเคราะห์อัตรารอด | 13 | 11, 12 และ 16 เมื่อใช้ Surface Context |
| วงขอบเขตปลูกเต็ม | 14 | 11, 12, 13 และ 16/17 เมื่อเรียกใช้ |
| QA ปลูกเต็ม | 15 | 11–14 และ Optional Outputs |
| จำแนกสิ่งปกคลุมปลูกเสริม | 01, 02 | ไม่มี |
| หาโกงกางในปลูกเสริม | 03 | 01, 02 |
| แยกวัชพืช | 04 | 01, 02 |
| แยกต้นจาก | 05 | 01, 02 |
| แยกต้นจาก vs ปาล์ม/มะพร้าว | 05, 17 | 01, 02 |
| วิเคราะห์ Gap ปลูกเสริม | 07 | 02–06 และ 16/17 เมื่อมี |
| วงขอบเขตปลูกเสริม | 08 | 07 |
| QA ปลูกเสริม | 09 | 02–08 และ Optional Outputs |
| เลือกจุดเข้าตรวจ | 18 | Analysis, Boundary และ QA Outputs |
| วิเคราะห์ทางบก/ทางเรือ | 19 | 18 และ Route Layers |
| จัด Mission ภาคสนาม | 20 | 18–19 |

## Trigger ของ Optional Skill

### Skill 16

เรียกเมื่อพบ:

- พื้นขาว–เทา ส้ม–น้ำตาล หรือเลนเปียกต่างกันชัด
- แอ่งน้ำ จุดน้ำขัง หรือร่องน้ำจำนวนมาก
- ต้องการเปรียบเทียบ Surface Zone กับอัตรารอด

ห้ามเรียกเพื่อยืนยันความเค็มหรือชนิดดินจากภาพ

### Skill 17

เรียกเมื่อพบ:

- ต้นเดี่ยวทรงดาวหรือรัศมี
- Crown Center ชัด
- อาจเห็นลำต้นหรือเงาลำต้น

ต้นจากที่เป็นกอ/ผืนและศูนย์กลางไม่ชัดให้ใช้ Skill 05

### Skill 18

เรียกเมื่อผู้ใช้ต้องการตอบว่า:

- จุดไหนควรเข้าตรวจ
- จุดไหนเป็นต้นหาย/อัตรารอดต่ำ/ขอบเขตไม่มั่นใจ
- จุดไหนต้องเก็บ Ground Truth หรือ QA Control

### Skill 19

เรียกเมื่อมี Candidate Inspection Points และต้องพิจารณา:

- ถนน ทางเดิน คันดิน หรือเส้นทางบก
- คลอง เส้นทางเรือ ท่าเรือ หรือจุดขึ้นฝั่ง
- เวลา ระยะทาง Tide, Permission และ Safety

### Skill 20

เรียกเมื่อมีผล Priority และ Access แล้ว และต้องการ:

- เลือกจุดจริงตามกำลังทีม
- รวมจุดไกลเป็น Mission
- วางแผนรายวันและ Field Checklist

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
- วางจุดตรวจอย่างเดียว: Skill 18
- วาง Route อย่างเดียว: Skill 19
- จัด Mission: Skill 20
- Skill 16–20 เป็น Optional Shared Skill เรียกเฉพาะเมื่อมี Trigger
- ห้ามอ่าน Skill ปลูกเต็มเมื่องานเป็นปลูกเสริม หรือกลับกัน โดยไม่มีเหตุผล
- ภาพ Crop ใช้สอนการตีความ ไม่ใช่พิกัดจริง
- งานจริงต้องรักษา CRS และ Affine Transform
- AI สร้างได้เฉพาะ Candidate Result และห้ามตั้งสถานะ `approved`
