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

เหตุผล: ขั้นตรวจจำนวนอยู่ระหว่าง Preliminary Pattern และ Survival

Prompt หรือ Code ที่อ้าง Path เดิมต้องแก้เป็น 12b

## Touching Crown Stage

เพิ่ม:

```text
skills/12c-touching-crown-tree-counter/SKILL.md
```

ลำดับใหม่:

```text
11 Raw Detection
→ 12 Preliminary Pattern
→ 12c Image-supported Touching Crown Centers
→ 12b Final Count Validation
```

กฎ Migration:

- Prompt เดิมที่ใช้ `11 → 12 → 12b` ให้แก้เป็น `11 → 12 → 12c → 12b`
- 12b ต้องรับ `touching_crown_centers.gpkg` และ `touching_crown_metrics.json`
- เมื่อไม่มี Touching Crown ให้ 12c ส่ง Empty Valid Layers และ Metrics จำนวน 0
- ห้ามใช้พื้นที่ Canopy Blob หรือ Grid สร้างจำนวนต้น

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
schemas/tree_count_validation.schema.json
schemas/touching_crown_metrics.schema.json
```

ไฟล์ใน:

```text
planting-evidence-boundary/config/
planting-evidence-boundary/schemas/
```

เป็น Legacy Reference

## Mandatory Production Pipeline

```text
01 → 01b → 11 → 12 → 12c → 12b → 13 → 14 → 15
```

ห้ามใช้ผล Skill 11 หรือ 12c เป็นจำนวนต้นสุดท้ายโดยข้าม 12b