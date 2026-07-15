# Skill Index

ใช้ไฟล์นี้เมื่อยังไม่แน่ใจว่าต้องเรียก Skill ใด หากทราบ Path แล้วให้อ่าน Skill นั้นโดยตรง

Machine-readable registry: `skills/registry.yaml`

## Input

| ID | Skill | หน้าที่ |
|---|---|---|
| 01 | `large-orthomosaic-reader` | ตรวจ Raster, CRS, Overview และ Tile |
| 01b | `project-input-preflight` | ตรวจ Plot, วันที่, AOI, จุดปลูก, Route และ Provenance |

งาน Production ต้องผ่าน 01b ส่วน JPG/PNG ไม่มีพิกัดใช้ได้เฉพาะ Demo

## Enrichment Planting

```text
00 → 01 → 01b → 02 → 03/04/05/06 → 07 → 08 → 09
```

| ID | Skill | หน้าที่ |
|---|---|---|
| 00 | `enrichment-analysis-orchestrator` | ควบคุม Workflow ปลูกเสริม |
| 02 | `mangrove-land-cover-classifier` | สร้าง Land-cover Context |
| 03 | `rhizophora-crown-detector` | ตรวจ Target Rhizophora Candidate |
| 04 | `weed-groundcover-classifier` | แยกวัชพืช/การบดบัง |
| 05 | `nypa-palm-detector` | แยกต้นจากแบบกอ/ผืน |
| 06 | `existing-canopy-occlusion` | ป่าเดิม เงา และเรือนยอดปิด |
| 07 | `enrichment-gap-analyzer` | วิเคราะห์ Gap-by-gap |
| 08 | `enrichment-boundary-delineator` | สร้าง Candidate Boundary |
| 09 | `enrichment-qa-human-review` | QA และ Human Review Package |

## Full-area Planting

```text
10 → 01 → 01b → 11 → 12 → 12c → 12b → 13 → 14 → 15
```

| ID | Skill | หน้าที่ |
|---|---|---|
| 10 | `full-area-planting-orchestrator` | ควบคุม Workflow ปลูกเต็ม |
| 11 | `full-area-tree-detector` | สร้าง Raw Crown Candidates ยังไม่ใช่จำนวนสุดท้าย |
| 12 | `planting-grid-inference` | หา Preliminary Spacing, Rows และ Pattern Blocks |
| 12c | `touching-crown-tree-counter` | แยก Crown Center ในเรือนยอดชิดจากภาพ + Local Spacing + 2D Row Support |
| 12b | `spacing-guided-tree-count-validator` | รวมผล 11/12/12c กรองจุดฟุ้ง ต้นใหญ่ และตัดสิน Final Count |
| 13 | `full-area-mortality-analyzer` | วิเคราะห์ Survival/Mortality จากผล 12b |
| 14 | `full-area-boundary-delineator` | สร้าง Candidate Boundary จากผล Validated |
| 15 | `full-area-qa-human-review` | QA ตั้งแต่ Raw ถึง Boundary |

ห้ามข้าม 12c เมื่อมีเรือนยอดชิด/รวมเป็นผืน และห้ามข้าม 12b แล้วใช้ Raw Detection คำนวณอัตรารอดหรือวงขอบเขต

## Optional Context

| ID | Skill | Trigger |
|---|---|---|
| 16 | `surface-hydrology-condition-classifier` | ดิน เลน น้ำขัง ร่องน้ำ หรือ Surface ต่างกันชัด |
| 17 | `palm-coconut-detector` | ต้นเดี่ยวทรงรัศมีที่อาจเป็นปาล์ม/มะพร้าว |

ไม่มี Trigger ห้ามโหลด

## Field Planning

```text
18 → 19 → 20
```

| ID | Skill | หน้าที่ |
|---|---|---|
| 18 | `field-inspection-priority-analyzer` | เลือกจุดที่ควรตรวจและเหตุผล |
| 19 | `multimodal-access-route-planner` | ประเมินทางบก ทางเรือ และทางเดินช่วงสุดท้าย |
| 20 | `field-inspection-mission-planner` | จัด Mission, วัน, ทีม และ Checklist |

## Feedback and Evaluation

```text
21 → 22
```

| ID | Skill | หน้าที่ |
|---|---|---|
| 21 | `field-feedback-ground-truth-ingestor` | รับผลภาคสนามแบบมีผู้ตรวจและ Version |
| 22 | `model-calibration-evaluator` | วัด Precision/Recall, Count Error และเสนอ Threshold |

## Token-efficient Load Sets

| งาน | อ่านเต็ม | อ่านเฉพาะ Output Contract |
|---|---|---|
| Preflight | 01b | 01 |
| Land-cover ปลูกเสริม | 02 | 01, 01b |
| Gap ปลูกเสริม | 07 | 02–06 |
| Raw Detection ปลูกเต็ม | 11 | 01, 01b |
| Preliminary Pattern | 12 | 11 |
| แยกเรือนยอดชิด | 12c | 11, 12 และ 01b/16/17 เมื่อใช้ |
| ตรวจจำนวนต้นจริง | 12b | 11, 12, 12c และ 01b/16/17 เมื่อใช้ |
| Survival/Mortality | 13 | 12b, 12 |
| Boundary ปลูกเต็ม | 14 | 12, 12b, 13 |
| QA ปลูกเต็ม | 15 | 11–14 รวม 12c/12b |
| เลือกจุดตรวจ | 18 | 09 หรือ 15 |
| Route | 19 | 18 |
| Mission | 20 | 18–19 |
| รับผลภาคสนาม | 21 | 18/20 และผลวิเคราะห์ |
| ประเมินโมเดล | 22 | 21 และผล Prediction |

## Output Handoff

ส่งต่อเฉพาะ:

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

รายละเอียดสถาปัตยกรรม: `docs/ARCHITECTURE.md`