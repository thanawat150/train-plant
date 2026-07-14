---
name: enrichment-analysis-orchestrator
description: วางแผนและควบคุม Workflow วิเคราะห์แปลงปลูกเสริมจาก Orthomosaic โดยเรียกใช้ Skill ย่อยตามลำดับ ใช้เมื่อผู้ใช้ต้องการวิเคราะห์ครบตั้งแต่ภาพขนาดใหญ่ จำแนกพืช ตรวจโกงกาง ไปจนถึงสร้าง Candidate Boundary และ QA
---

# Enrichment Analysis Orchestrator

## หน้าที่

Skill นี้เป็นตัวควบคุมงาน ไม่ทำ Detection หรือสร้าง Polygon แทน Skill เฉพาะทาง

## Required Pipeline

1. `large-orthomosaic-reader`
2. `mangrove-land-cover-classifier`
3. `rhizophora-crown-detector`
4. `weed-groundcover-classifier`
5. `nypa-palm-detector`
6. `existing-canopy-occlusion`
7. `enrichment-gap-analyzer`
8. `enrichment-boundary-delineator`
9. `enrichment-qa-human-review`

## Inputs

- Orthomosaic: GeoTIFF, COG หรือ VRT
- `project_id` และ `plot_code`
- Output directory
- AOI ถ้ามี
- ภาพตัวอย่างหรือ Ground Truth ถ้ามี
- จุดปลูก ภาพก่อน–หลัง หรือข้อมูลภาคสนามถ้ามี

## Planning Rules

ก่อนรันต้องรายงาน:

- Input ที่พบและข้อมูลที่ขาด
- CRS, Pixel Size, Bounds และขนาดไฟล์
- Skill ที่ต้องใช้
- Skill ที่ข้ามได้พร้อมเหตุผล
- Intermediate Outputs ที่จะสร้าง
- Stop Conditions

## Modes

### Full Pipeline

รัน Skill 01–09 ทั้งหมด

### Classification Only

รัน 01–06 และหยุดก่อนวิเคราะห์ช่องว่าง

### Target Detection Only

รัน 01, 02, 03 และ Class ที่ใช้ตัดสิ่งรบกวนตามความจำเป็น

### Boundary Only

ใช้ได้เมื่อมี Detection/Class Layers พร้อมแล้ว รัน 07–09

### QA Only

ใช้ตรวจ Candidate Output ที่มีอยู่แล้วด้วย Skill 09

## Checkpoints

หยุดให้ตรวจอย่างน้อย 3 จุด:

1. หลัง Land-cover Classification
2. หลัง Target Detection และ Exclusion Classes
3. หลัง Candidate Boundary ก่อน Export ขั้นสุดท้าย

## Failure Handling

- ห้ามข้ามขั้นตอนที่ล้มเหลวโดยไม่รายงาน
- ห้ามใช้ผลเก่าโดยไม่ตรวจ Source Raster และ CRS
- หาก Class สำคัญแยกไม่ได้ ให้ลด Confidence และสร้าง Review Zone
- หากไฟล์ใหญ่เกินทรัพยากร ให้ลด Overview Resolution หรือปรับ Tile Size ไม่ให้โหลดทั้งไฟล์

## Final Deliverables

- Classification layers
- Target and exclusion layers
- Enrichment gap candidates
- `enrichment_planting_core`
- `enrichment_evidence_boundary`
- `uncertain_enrichment_boundary`
- QA report, warnings และ previews
- Manifest ที่ระบุ Skill, Version, Input และ Parameters

AI ห้ามตั้งสถานะ `approved`
