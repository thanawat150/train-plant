---
name: enrichment-analysis-orchestrator
description: วางแผนและควบคุม Workflow วิเคราะห์แปลงปลูกเสริมจาก Orthomosaic โดยเรียกใช้ Skill ย่อยตามลำดับและโหลดเฉพาะส่วนที่จำเป็น
---

# Enrichment Analysis Orchestrator

## หน้าที่

Skill นี้เป็นตัวควบคุมงาน ไม่ทำ Detection หรือสร้าง Polygon แทน Skill เฉพาะทาง

## Core Pipeline

1. `01-large-orthomosaic-reader`
2. `02-mangrove-land-cover-classifier`
3. `03-rhizophora-crown-detector`
4. `04-weed-groundcover-classifier`
5. `05-nypa-palm-detector`
6. `06-existing-canopy-occlusion`
7. `07-enrichment-gap-analyzer`
8. `08-enrichment-boundary-delineator`
9. `09-enrichment-qa-human-review`

## Optional Routed Skills

เรียกเฉพาะเมื่อภาพมีหลักฐานที่เกี่ยวข้อง:

- `16-surface-hydrology-condition-classifier` เมื่อมีดิน เลน น้ำขัง แอ่งน้ำ ร่องน้ำ หรือพื้นผิวต่างกันชัด
- `17-palm-coconut-detector` เมื่อมีต้นเดี่ยวทรงดาวหรือรัศมีที่อาจเป็นปาล์ม/มะพร้าว ไม่ใช่ต้นจาก

ตำแหน่งแนะนำ:

```text
01 → 02
   → 16 เมื่อจำเป็น
   → 03 → 04 → 05
   → 17 เมื่อพบ radial single crowns
   → 06 → 07 → 08 → 09
```

## Inputs

- Orthomosaic: GeoTIFF, COG หรือ VRT
- `project_id` และ `plot_code`
- Output directory
- AOI ถ้ามี
- ภาพตัวอย่างหรือ Ground Truth ถ้ามี
- จุดปลูก ภาพก่อน–หลัง หรือข้อมูลภาคสนามถ้ามี
- DSM/DTM/DEM และข้อมูลดิน/น้ำ เมื่อมี

## Planning Rules

ก่อนรันต้องรายงาน:

- Input ที่พบและข้อมูลที่ขาด
- CRS, Pixel Size, Bounds และขนาดไฟล์
- Skill ที่ต้องใช้
- Optional Skill 16/17 ที่ต้องใช้หรือข้าม พร้อมเหตุผล
- Intermediate Outputs ที่จะสร้าง
- Stop Conditions

## Token Rules

- โหลด Skill ทีละตัว
- Optional Skill ไม่เกี่ยวข้องต้องไม่ถูกอ่าน
- ขั้นถัดไปอ่านเฉพาะ Output Contract และไฟล์ผลลัพธ์
- ห้ามส่งคำอธิบายจากขั้นก่อนหน้าซ้ำทั้งหมด

## Modes

### Full Pipeline

รัน Core Pipeline และ Optional Skill เฉพาะที่ Trigger

### Classification Only

รัน 01–06 พร้อม 16/17 ตามความจำเป็น แล้วหยุดก่อนวิเคราะห์ช่องว่าง

### Target Detection Only

รัน 01, 02, 03 และ Exclusion Skill ที่จำเป็น เช่น 04, 05, 17

### Surface/Hydrology Only

รัน 01 และ 16 เท่านั้น

### Palm Separation Only

รัน 01, 02, 05 และ 17 เท่านั้น

### Boundary Only

ใช้ได้เมื่อมี Detection/Class Layers พร้อมแล้ว รัน 07–09

### QA Only

ใช้ตรวจ Candidate Output ที่มีอยู่แล้วด้วย Skill 09

## Checkpoints

หยุดให้ตรวจอย่างน้อย:

1. หลัง Land-cover และ Surface Classification
2. หลัง Target Detection และ Exclusion Classes
3. หลัง Gap Classification
4. หลัง Candidate Boundary ก่อน Export ขั้นสุดท้าย

## Failure Handling

- ห้ามข้ามขั้นตอนที่ล้มเหลวโดยไม่รายงาน
- ห้ามใช้ผลเก่าโดยไม่ตรวจ Source Raster และ CRS
- หาก Class สำคัญแยกไม่ได้ ให้ลด Confidence และสร้าง Review Zone
- หาก Nypa กับปาล์ม/มะพร้าวแยกไม่ได้ ให้ใช้ Unknown Class ไม่บังคับคำตอบ
- หาก Water กับ Shadow แยกไม่ได้ ให้ส่ง `needs_human_review`
- หากไฟล์ใหญ่เกินทรัพยากร ให้ปรับ Overview/Tile โดยไม่โหลดทั้งไฟล์

## Final Deliverables

- Classification layers
- Target and exclusion layers
- Optional surface/hydrology and palm layers
- Enrichment gap candidates
- `enrichment_planting_core`
- `enrichment_evidence_boundary`
- `uncertain_enrichment_boundary`
- QA report, warnings และ previews
- Manifest ที่ระบุ Skill, Version, Input และ Parameters

AI ห้ามตั้งสถานะ `approved`
