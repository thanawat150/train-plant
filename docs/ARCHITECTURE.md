# Architecture

Repository นี้แบ่งงานตาม Stage ไม่ใช้หมายเลขโฟลเดอร์เป็นลำดับเพียงอย่างเดียว

## Stage 1 — Input

```text
01  Large Orthomosaic Reader
01b Project Input Preflight
```

งาน Production ต้องผ่าน 01b เพื่อยืนยัน Plot, CRS, วันที่ และ Provenance ของจุดปลูก/เส้นทาง

## Stage 2A — Enrichment Planting

```text
00 Orchestrator
01 → 01b → 02 → 03/04/05/06 → 07 → 08 → 09
                  ↘ 16/17 เมื่อมี Trigger
```

- 02 สร้าง Land-cover Context
- 03 ตรวจ Target Candidate
- 04–06 แยกสิ่งรบกวนและพื้นที่บดบัง
- 07 วิเคราะห์ Gap-by-gap
- 08 สร้าง Candidate Boundary
- 09 QA

## Stage 2B — Full-area Planting

```text
10 Orchestrator
01 → 01b → 11 → 12 → 12c → 12b → 13 → 14 → 15
                              ↖ Refit 12 และ rerun 12c ได้สูงสุด 2 รอบ
```

- 11 สร้าง Raw Crown Candidates และ Touching/Merged Clusters
- 12 หา Preliminary Spacing/Pattern จากต้นเดี่ยว High-confidence
- 12c แยก Image-supported Crown Centers ในเรือนยอดชิดด้วย Local Spacing และ 2D Row Support
- 12b รวมผล 11/12/12c และตัดสิน Final Count
- 13 คำนวณ Survival/Mortality
- 14 สร้าง Candidate Boundary
- 15 QA

กฎ Separation:

```text
11 raw detection
12 preliminary pattern
12c touching crown center evidence
12b validated final count
```

ห้ามนำผล Skill 11 หรือ 12c ไปคำนวณอัตรารอดหรือสร้างขอบเขตโดยข้าม 12b

## Stage 3 — Optional Context

```text
16 Surface/Hydrology
17 Palm/Coconut
```

เรียกเมื่อมี Trigger เท่านั้น

## Stage 4 — Field Planning

```text
18 Inspection Priority
→ 19 Land/Boat Access
→ 20 Field Mission
```

แยก `evidence_priority_score` ออกจาก `access_burden_score`

## Stage 5 — Feedback and Evaluation

```text
21 Field Feedback / Ground Truth
→ 22 Model Calibration and Evaluation
```

Ground Truth ต้องมีผู้ตรวจ วันที่ และ GPS Accuracy ห้ามทับผล AI เดิม

## Data Confidence Hierarchy

```text
surveyed_field / approved_plan
> trusted_existing_gis
> human_digitized_candidate
> model_inferred
> unknown
```

## Mandatory Output Separation

```text
raw_detection
preliminary_pattern
touching_crown_evidence
validated_count
survival_status
boundary
field_plan
human_feedback
model_evaluation
```

ห้ามใช้ชื่อหรือ Layer เดียวแทนหลาย Stage

## Touching Crown Contract

Skill 12c ต้องส่ง:

```text
touching_crown_centers.gpkg
touching_crown_clusters_validated.gpkg
touching_crown_unresolved.gpkg
touching_crown_metrics.json
```

และต้องรักษา:

```text
grid_created_tree_count = 0
center_without_canopy_count = 0
cluster_processed_ratio = 1.0
```

Skill 12b เป็นผู้ตัดสิน Final Class เสมอ

## Token-efficient Loading

1. อ่าน `AGENTS.md`
2. หากยังไม่รู้เส้นทาง อ่าน `skills/registry.yaml` หรือ `skills/README.md`
3. อ่าน Skill ที่กำลังรันเต็มไฟล์เพียงตัวเดียว
4. อ่าน Skill ก่อนหน้าเฉพาะ Output Contract
5. ส่งต่อเฉพาะ Path, Schema, CRS, Config, Metrics และ Warnings

Root `README.md` เป็นคู่มือผู้ใช้ ไม่ใช่ไฟล์บังคับสำหรับทุก Run