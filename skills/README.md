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
| `11-full-area-tree-detector` | สร้าง Raw Crown/Tree Candidates จากภาพจริง ยังไม่ใช่จำนวนสุดท้าย |
| `12-planting-grid-inference` | หา Reference Spacing, แนวแถว, Grid และ Planting Pattern Block เบื้องต้น |
| `21-spacing-guided-tree-count-validator` | กรองจุดฟุ้ง ตัดต้นใหญ่/จุดบนดินน้ำ และนับเรือนยอดชิดด้วย Canopy+Spacing |
| `13-full-area-mortality-analyzer` | วิเคราะห์อัตรารอดจากผล Skill 21 เท่านั้น |
| `14-full-area-boundary-delineator` | สร้าง Core, Evidence Boundary และ Uncertain Edge |
| `15-full-area-qa-human-review` | QA และชุด Human Review สำหรับปลูกเต็ม |

## Full-area core pipeline

```text
01 อ่านภาพ/แบ่ง Tile
→ 11 Raw Crown Detection
→ 12 Reference Spacing และ Pattern Blocks
→ 21 Validate Count ด้วย Canopy + Spacing + จุดปลูก
→ 13 Survival/Mortality
→ 14 Boundary
→ 15 QA
```

ห้ามข้าม Skill 21 แล้วนำ Raw Detection จาก Skill 11 ไปคำนวณอัตรารอด

## ความเข้าใจสำหรับ Skill 21

- จุดแดงเป็น Raw Candidate ไม่ใช่จำนวนต้นสุดท้าย
- ต้นปลูกต้องมีเรือนยอดรองรับและสอดคล้องกับระยะ/แนว/จุดปลูก
- จุดฟุ้งบนดิน น้ำ ถนน วัชพืช หรือเงาต้องถูก Flag เป็น False Positive
- น้ำไม่ใช่ Exclusion ทั้งหมด เพราะต้นป่าชายเลนสามารถมีเรือนยอดอยู่เหนือผิวน้ำ
- เรือนยอดชิดกันนับจาก Center Spacing และ Pattern ไม่ใช้ `1 blob = 1 tree`
- ต้นเดิมขนาดใหญ่ไม่นับ ไม่ใช้ Fit Grid และจุดใต้พุ่มใหญ่เป็น Not Observable
- แปลงนากุ้งรองรับ `shrimp_pond_planting_block` ที่ระยะสม่ำเสมอเฉพาะภายใน Block
- Grid ห้ามสร้างต้นรอดขึ้นมาเองเมื่อไม่มี Canopy Evidence

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
- ระยะเข้าถึงต้องใช้ Network Distance ไม่ใช่เส้นตรง
- Route จากภาพเป็น Candidate ต้องให้ทีมพื้นที่ยืนยัน

# Token-efficient load sets

| งาน | อ่านเต็ม | อ่านเฉพาะ Output Contract |
|---|---|---|
| อ่านภาพ/แบ่ง Tile | 01 | ไม่มี |
| Raw Crown Detection | 11 | 01 และ 16/17 เมื่อใช้ |
| หา Reference Spacing/Grid | 12 | 11 |
| ตรวจนับต้นจริง/เรือนยอดชิด | 21 | 11, 12 และ 16 เมื่อใช้ |
| วิเคราะห์อัตรารอด | 13 | 21, 12 และ 16 เมื่อใช้ Surface Context |
| วงขอบเขตปลูกเต็ม | 14 | 12, 13, 21 และ Optional Context |
| QA ปลูกเต็ม | 15 | 11–14, 21 และ Optional Outputs |
| จำแนก Surface/Hydrology | 16 | 01 |
| แยก Palm/Coconut | 17 | 01, 02 หรือ 05 ตาม Input |
| จำแนกสิ่งปกคลุมปลูกเสริม | 01, 02 | ไม่มี |
| หาโกงกางในปลูกเสริม | 03 | 01, 02 |
| แยกวัชพืช | 04 | 01, 02 |
| แยกต้นจาก | 05 | 01, 02 |
| วิเคราะห์ Gap ปลูกเสริม | 07 | 02–06 และ 16/17 เมื่อมี |
| วงขอบเขตปลูกเสริม | 08 | 07 |
| QA ปลูกเสริม | 09 | 02–08 และ Optional Outputs |
| เลือกจุดเข้าตรวจ | 18 | Analysis, Boundary และ QA Outputs |
| วิเคราะห์ทางบก/ทางเรือ | 19 | 18 และ Route Layers |
| จัด Mission ภาคสนาม | 20 | 18–19 |

## Trigger ของ Skill 21

เรียกเป็น Core Step ของปลูกเต็มทุกครั้ง โดยเฉพาะเมื่อพบ:

- จุดแดงกระจายฟุ้งและไม่ตรงพุ่ม
- จุดบนดินโล่ง ผิวน้ำ ถนน หรือคันดิน
- เรือนยอดชิดกันแต่ระยะต้นใกล้เคียงกัน
- ต้นเดิมขนาดใหญ่ปะปน
- มีจุดปลูกเดิมหรือ Grid ที่ต้องจับคู่กับต้นจริง
- แปลงนากุ้งมีหลาย Planting Block

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
- งานครบปลูกเต็ม: Skill 10 และต้องผ่าน Skill 21
- ตรวจจำนวนต้นปลูกเต็มจากผลที่มีอยู่: Skill 21
- วางจุดตรวจอย่างเดียว: Skill 18
- วาง Route อย่างเดียว: Skill 19
- จัด Mission: Skill 20
- Skill 16–20 เป็น Optional Shared Skill เรียกเฉพาะเมื่อมี Trigger
- Skill 21 เป็น Core Validation ของปลูกเต็ม ไม่ใช่ Optional
- ภาพ Crop ใช้สอนการตีความ ไม่ใช่พิกัดจริง
- งานจริงต้องรักษา CRS และ Affine Transform
- AI สร้างได้เฉพาะ Candidate Result และห้ามตั้งสถานะ `approved`
