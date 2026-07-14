---
name: planting-evidence-boundary
description: Compatibility router สำหรับงานเดิมที่เรียก Planting Evidence Boundary โดยส่งต่องานไปยังชุด Skill แยกใน skills/ ใช้เพื่อรองรับ Prompt เก่าเท่านั้น งานใหม่ให้เรียก Skill เฉพาะทางโดยตรง
---

# Planting Evidence Boundary — Compatibility Router

ไฟล์นี้ไม่ใช่ Skill วิเคราะห์แบบรวมทั้งหมดอีกต่อไป

โครงสร้างหลักถูกแยกตามหน้าที่เพื่อป้องกัน Codex ปนการจำแนกชนิดพืช การตรวจต้น และการวงขอบเขตเข้าด้วยกัน

## งานเต็มระบบ

เรียก:

`skills/00-enrichment-analysis-orchestrator/SKILL.md`

## Routing Table

| คำขอของผู้ใช้ | Skill ที่ต้องใช้ |
|---|---|
| อ่าน Orthomosaic ขนาดใหญ่หรือแบ่ง Tile | `skills/01-large-orthomosaic-reader/SKILL.md` |
| จำแนกสิ่งปกคลุม | `skills/02-mangrove-land-cover-classifier/SKILL.md` |
| หา/นับพุ่มโกงกาง | `skills/03-rhizophora-crown-detector/SKILL.md` |
| จำแนกวัชพืช | `skills/04-weed-groundcover-classifier/SKILL.md` |
| ตรวจต้นจาก | `skills/05-nypa-palm-detector/SKILL.md` |
| แยกป่าเดิม เงา และเรือนยอดปิด | `skills/06-existing-canopy-occlusion/SKILL.md` |
| วิเคราะห์ช่องว่างปลูกเสริม | `skills/07-enrichment-gap-analyzer/SKILL.md` |
| วง Candidate Boundary | `skills/08-enrichment-boundary-delineator/SKILL.md` |
| ตรวจ QA และเตรียม Human Review | `skills/09-enrichment-qa-human-review/SKILL.md` |

## Legacy References

ไฟล์อื่นภายใน `planting-evidence-boundary/` เช่น:

- `LARGE_ORTHOMOSAIC_WORKFLOW.md`
- `SKILL_ADDENDUM_ADVANCED.md`
- `QA_DEBUG_OUTPUTS.md`
- `examples/`

ยังใช้เป็น Reference และชุดตัวอย่างได้ แต่กฎปฏิบัติงานหลักให้ยึด `AGENTS.md` และ Skill ภายใน `skills/`

## Mandatory Rules

- ห้ามใช้สีเพียงอย่างเดียว
- ห้ามใช้ภาพ Crop เป็นพิกัดจริง
- ห้ามสรุปว่าโกงกาง Candidate เกิดจากการปลูกโดยไม่มีหลักฐานเสริม
- ห้ามใช้ `closed_canopy_unknown` เป็น `not_planted`
- ห้ามสร้าง Boundary ก่อนมี Classification, Detection และ Gap Evidence
- AI ห้ามตั้งสถานะ `approved`
