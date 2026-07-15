---
name: planting-evidence-boundary
description: Compatibility router สำหรับ Prompt เก่า งานใหม่ให้เรียก Skill ใน skills/ ตาม registry โดยตรง
---

# Planting Evidence Boundary — Legacy Router

โฟลเดอร์นี้เป็น Legacy Reference ไม่ใช่ Config, Schema หรือ Workflow หลัก

## งานปลูกเสริม

```text
skills/00-enrichment-analysis-orchestrator/SKILL.md
```

## งานปลูกเต็มพื้นที่

```text
skills/10-full-area-planting-orchestrator/SKILL.md
```

## Routing ใหม่

| งาน | Skill |
|---|---|
| อ่าน Raster | `skills/01-large-orthomosaic-reader/SKILL.md` |
| ตรวจ Input/จุดปลูก/CRS | `skills/01b-project-input-preflight/SKILL.md` |
| จำแนกปลูกเสริม | `skills/02-mangrove-land-cover-classifier/SKILL.md` |
| Raw Detection ปลูกเต็ม | `skills/11-full-area-tree-detector/SKILL.md` |
| หา Spacing/Pattern | `skills/12-planting-grid-inference/SKILL.md` |
| ตรวจจำนวนต้นจริง | `skills/12b-spacing-guided-tree-count-validator/SKILL.md` |
| Survival/Mortality | `skills/13-full-area-mortality-analyzer/SKILL.md` |
| เลือกจุดตรวจ | `skills/18-field-inspection-priority-analyzer/SKILL.md` |
| วาง Route บก/เรือ | `skills/19-multimodal-access-route-planner/SKILL.md` |
| จัด Field Mission | `skills/20-field-inspection-mission-planner/SKILL.md` |
| รับ Ground Truth | `skills/21-field-feedback-ground-truth-ingestor/SKILL.md` |
| ประเมินโมเดล | `skills/22-model-calibration-evaluator/SKILL.md` |

Registry หลัก:

```text
skills/registry.yaml
```

## Legacy References

ไฟล์อื่นในโฟลเดอร์นี้ยังใช้เป็นตัวอย่างเก่าได้ แต่กฎล่าสุดให้ยึด:

```text
AGENTS.md
skills/registry.yaml
skills/README.md
config/defaults.yaml
schemas/
```

## Mandatory Rules

- Raw Detection ไม่ใช่จำนวนต้นสุดท้าย
- ห้ามข้าม Skill 12b ในงานปลูกเต็ม
- ห้ามใช้ภาพ Crop เป็นพิกัดจริง
- ห้ามสรุปการปลูกหรือสาเหตุการตายเกินหลักฐาน
- AI ห้ามตั้งสถานะ `approved`
