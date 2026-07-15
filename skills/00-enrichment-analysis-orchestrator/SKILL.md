---
name: enrichment-analysis-orchestrator
description: ควบคุม Workflow ปลูกเสริมแบบ Gap-by-gap ตั้งแต่ Preflight, Classification, Target/Exclusion, Boundary, QA ไปจนถึง Field Plan และ Feedback Loop
---

# Enrichment Analysis Orchestrator

## หน้าที่

ควบคุมลำดับงานเท่านั้น ไม่ทำ Detection, Boundary หรือ Routing แทน Skill เฉพาะทาง

## Production Pipeline

```text
01  inspect/tile raster
→ 01b validate project inputs and provenance
→ 02 land-cover context
→ 03 target Rhizophora candidates
→ 04 weed/groundcover
→ 05 Nypa patches
→ 06 existing canopy/occlusion
→ 07 enrichment gap analysis
→ 08 candidate boundary
→ 09 QA/human review package
```

Optional:

```text
16 Surface/Hydrology เมื่อมี Trigger
17 Palm/Coconut เมื่อมี Trigger
```

## Core Rules

- วิเคราะห์เป็น Gap-by-gap ไม่วงป่าทั้งผืน
- ไม่บังคับ Grid แบบปลูกเต็ม
- ภาพช่วงเวลาเดียวไม่ยืนยันว่าต้นเกิดจากการปลูก
- `closed_canopy_unknown` ไม่ใช่ `not_planted`
- แยก Target จากวัชพืช ต้นจาก ปาล์ม/มะพร้าว ป่าเดิม น้ำ เลน และเงา
- AOI และ PDD เป็นข้อมูลช่วยค้นหา ไม่ใช่คำตอบอัตโนมัติ
- Surface Class ไม่ใช่ผลตรวจดิน

## Token Rules

อ่าน `AGENTS.md`, Skill นี้, Skill ขั้นที่กำลังรัน และ Output Contract ของขั้นก่อนหน้าเท่านั้น

## Optional Field Plan

```text
หลัง 09
→ 18 inspection priority
→ 19 land/boat access
→ 20 missions/day plan
```

## Optional Feedback Loop

```text
หลังภาคสนาม
→ 21 ingest field feedback/ground truth
→ 22 evaluate accuracy and recommend calibration
```

## Inputs

```text
Orthomosaic
project_id
plot_code
output_directory
```

Optional: AOI, จุดปลูก, ภาพก่อน–หลัง, DSM/DTM/CHM, Route Layers, Access Points, Tide Window และข้อมูลภาคสนาม

## Checkpoints

1. Preflight Summary
2. Land-cover/Surface Preview
3. Target vs Exclusion Preview
4. Gap Classification
5. Candidate Boundary
6. QA
7. Field Plan/Feedback เมื่อเรียกใช้

## Stop Conditions

- Preflight ไม่ผ่าน
- CRS/Transform ไม่พร้อมสำหรับงานพิกัดจริง
- Class สำคัญแยกไม่ได้และไม่มี Unknown/Review Zone
- Water กับ Shadow หรือ Nypa กับ Palm แยกไม่ได้อย่างน่าเชื่อถือ
- Boundary ไม่มี Evidence/Confidence รองรับ
- Access Network ขาดแต่ถูกขอ Route จริง

## Final Deliverables

```text
project_manifest.json
classification layers
target and exclusion layers
enrichment_gap_candidates.gpkg
enrichment_planting_core.gpkg
enrichment_evidence_boundary.gpkg
uncertain_enrichment_boundary.gpkg
qa_report.json
review_package/
```

Optional: Inspection Points, Route Options, Missions, Ground Truth และ Evaluation Report

## Restrictions

- ห้ามสร้างต้นหรือขอบเขตให้ตรงค่าที่คาด
- ห้ามซ่อนพื้นที่ Unknown
- ห้ามรับรอง Route จากภาพว่าปลอดภัย
- ห้ามตั้งสถานะ `approved`
