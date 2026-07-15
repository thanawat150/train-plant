# AGENTS.md

## Purpose

ควบคุม Workflow วิเคราะห์ Orthomosaic งานปลูกป่าชายเลน โดยแยกปลูกเสริม ปลูกเต็ม การวางแผนภาคสนาม และ Feedback/Evaluation

ผลทั้งหมดเป็น Candidate จนกว่าจะผ่าน Human Review

## Mandatory Token Routing

อ่านตามลำดับ:

1. `AGENTS.md`
2. `skills/registry.yaml` หรือ `skills/README.md` เฉพาะเมื่อยังไม่ทราบ Skill
3. Skill ที่กำลังรันเต็มไฟล์เพียงตัวเดียว
4. Skill ก่อนหน้าเฉพาะ Output Contract

ห้ามอ่าน Root `README.md`, เอกสารทั้งหมด หรือ Skill ทั้ง Repository ในทุก Run

ส่งต่อระหว่างขั้นเฉพาะ:

```text
source_file_paths
schema_version
project_id / plot_code
crs
config_path
summary_metrics
warnings
review_status
```

## Production Preflight

งานที่รายงานพิกัด ระยะ พื้นที่ จำนวน หรืออัตรารอดจริง ต้องผ่าน:

```text
01-large-orthomosaic-reader
→ 01b-project-input-preflight
```

JPG/PNG ไม่มีพิกัดใช้ได้เฉพาะ `image_demo_only`

## Workflow Routing

### Enrichment Planting

```text
00 orchestrator
01 → 01b → 02 → 03/04/05/06 → 07 → 08 → 09
```

Optional: 16 Surface/Hydrology, 17 Palm/Coconut

หลักสำคัญ:

- วิเคราะห์ Gap-by-gap
- ไม่บังคับ Grid
- ภาพครั้งเดียวไม่ยืนยันว่าต้นเกิดจากการปลูก
- `closed_canopy_unknown` ไม่ใช่ `not_planted`

### Full-area Planting

```text
10 orchestrator
01 → 01b → 11 → 12 → 12b → 13 → 14 → 15
```

หลักสำคัญ:

- Skill 11 เป็น Raw Detection ไม่ใช่จำนวนสุดท้าย
- Skill 12 สร้าง Preliminary Spacing/Pattern
- Skill 12b ตรวจ Canopy + Spacing + จุดปลูก
- ห้ามข้าม 12b ไปคำนวณอัตรารอดหรือวง Boundary
- Grid ห้ามสร้างต้นที่ไม่มี Canopy Evidence
- น้ำไม่ใช่ Exclusion ทั้งหมด แต่จุดบนผิวน้ำที่ไม่มีพุ่มเป็น False Positive
- เรือนยอดชิดกันห้ามใช้ `1 blob = 1 tree`
- ต้นเดิมขนาดใหญ่ไม่นับ ไม่ใช้ Fit Grid และจุดใต้พุ่มเป็น Not Observable
- รองรับหลาย Pattern Block โดยเฉพาะแปลงนากุ้ง

### Field Planning

```text
18 Inspection Priority
→ 19 Land/Boat Access
→ 20 Field Mission
```

- แยก `evidence_priority_score` จาก `access_burden_score`
- ห้ามตัดจุดสำคัญเพียงเพราะไกล
- ใช้ Network Distance ไม่ใช้เส้นตรงเป็น Route จริง
- Candidate Route ต้องให้ทีมพื้นที่ยืนยัน

### Feedback and Evaluation

```text
21 Field Feedback / Ground Truth
→ 22 Model Calibration / Evaluation
```

- Ground Truth ต้องมีผู้ตรวจ วันที่ และ GPS Accuracy
- ห้ามทับ AI Output เดิม
- Threshold Recommendation ต้องผ่าน Human Approval
- ห้ามอ้าง Production Accuracy โดยไม่มี Independent Test Set

## Optional Skill Triggers

### Skill 16

เรียกเมื่อมี Surface/Hydrology ต่างกันชัด ห้ามใช้สีภาพยืนยันความเค็ม pH หรือชนิดดิน

### Skill 17

เรียกเมื่อต้องแยกต้นจากที่เป็นกอ/ผืน ออกจากปาล์ม/มะพร้าวต้นเดี่ยวทรงรัศมี

Optional Skill ที่ไม่มี Trigger ต้องไม่ถูกอ่าน

## Shared Raster Rules

- รักษา CRS และ Affine Transform
- ใช้ Overview และ Tile/Window พร้อม Overlap 10–20%
- ห้ามโหลด Orthomosaic ใหญ่ทั้งหมดเข้า RAM
- รวมผลใน CRS ต้นฉบับ ไม่ใช้ Local Pixel ข้าม Tile
- ห้ามใช้ Tile Edge เป็นขอบเขตจริง

## Provenance Rules

เก็บแยก:

```text
surveyed_field
contract_or_approved_plan
existing_gis
human_digitized_from_orthomosaic
model_inferred
unknown
```

ห้ามเรียก `inferred_grid_position` ว่าจุดปลูกจริง

## Status Control

AI ใช้ได้สูงสุด:

```text
draft
needs_human_review
rework
```

มนุษย์เท่านั้น:

```text
reviewed
rejected
approved
```

## Mandatory Checkpoints

Full-area:

1. Raw Detection
2. Preliminary Spacing/Pattern
3. 12b False-positive/Large-tree/Merged-canopy Validation
4. Final Count Status
5. Survival/Mortality
6. Boundary
7. QA

Enrichment:

1. Classification
2. Target vs Exclusion
3. Gap Classification
4. Boundary
5. QA

Field Plan:

1. Inspection Points
2. Land/Boat Route Options
3. Mission/Day Plan

## Failure Rule

เมื่อ Input, Pattern, Validation, Route หรือ Ground Truth ไม่เพียงพอ ให้หยุดหรือส่ง `needs_human_review` ห้ามสร้างข้อมูลขึ้นมาเพื่อให้ Workflow ผ่าน
