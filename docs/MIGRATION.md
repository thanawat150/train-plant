# Migration Notes

## Count Validation Skill

เดิม:

```text
skills/21-spacing-guided-tree-count-validator/SKILL.md
```

ใหม่:

```text
skills/12b-spacing-guided-tree-count-validator/SKILL.md
```

เหตุผล: ขั้นตรวจจำนวนอยู่ระหว่าง Preliminary Pattern (12) และ Survival (13)

Prompt หรือ Code ที่อ้าง Path เดิมต้องแก้เป็น 12b

## Skill 21–22 ใหม่

```text
21 = field-feedback-ground-truth-ingestor
22 = model-calibration-evaluator
```

ใช้หลังลงตรวจภาคสนามเพื่อปิด Feedback Loop

## Active Config and Schemas

ใช้:

```text
config/defaults.yaml
schemas/analysis_manifest.schema.json
schemas/field_observation.schema.json
```

ไฟล์ใน:

```text
planting-evidence-boundary/config/
planting-evidence-boundary/schemas/
```

เป็น Legacy Reference

## Mandatory Production Pipeline

```text
01 → 01b → 11 → 12 → 12b → 13 → 14 → 15
```

ห้ามใช้ผล Skill 11 เป็นจำนวนต้นสุดท้าย
