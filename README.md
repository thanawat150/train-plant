# train-plant

ชุด Skill สำหรับวิเคราะห์ภาพโดรนและ Orthomosaic ของงานปลูกป่าชายเลน แยกเป็น 2 Workflow ชัดเจน:

1. **ปลูกเสริม** — ปลูกแทรกในช่องว่างของป่าเดิม
2. **ปลูกเต็มพื้นที่** — ปลูกครอบคลุมพื้นที่กว้างและมีแถวหรือกริดต่อเนื่อง

> ผลทั้งหมดเป็น Candidate Evidence จากภาพ ต้องผ่าน Human Review ไม่ใช่ขอบเขตตามกฎหมาย และภาพช่วงเวลาเดียวไม่ยืนยันว่าต้นเกิดจากการปลูก

# เลือก Workflow ก่อน

| ลักษณะภาพ | Workflow | Skill เริ่มต้น |
|---|---|---|
| ป่าเดิมหนาแน่น มีโกงกางปลูกแทรก วัชพืช ต้นจาก และช่องว่างเป็นหย่อม | ปลูกเสริม | `skills/00-enrichment-analysis-orchestrator/SKILL.md` |
| ต้นปลูกกระจายทั่วพื้นที่ เห็นแนวแถว กริด หรือระยะปลูกซ้ำ | ปลูกเต็มพื้นที่ | `skills/10-full-area-planting-orchestrator/SKILL.md` |
| ต้องการอ่านภาพใหญ่และแบ่ง Tile เท่านั้น | ใช้ร่วมกัน | `skills/01-large-orthomosaic-reader/SKILL.md` |

ห้ามใช้ Workflow ปลูกเสริมกับแปลงปลูกเต็ม และห้ามใช้ Grid ของปลูกเต็มไปบังคับพื้นที่ปลูกเสริม

# กฎประหยัด Token

Codex ต้องอ่านเฉพาะ:

1. `AGENTS.md`
2. `skills/README.md`
3. Skill ที่ผู้ใช้เรียก
4. Skill ก่อนหน้าที่เป็น Input โดยอ่านเฉพาะ Output Contract เมื่อจำเป็น

ห้ามอ่าน Skill ทั้ง Repository พร้อมกัน

ตัวอย่าง:

- ต้องการนับต้นปลูกเต็ม: อ่าน Skill 01 และ 11 เท่านั้น
- ต้องการหา Grid: อ่าน Skill 11 และ 12 เท่านั้น
- ต้องการวงขอบเขตปลูกเต็ม: อ่าน Output Contract ของ 11–13 และอ่าน Skill 14 เต็ม
- ต้องการแยกต้นจากในปลูกเสริม: อ่าน Skill 01 และ 05 เท่านั้น
- งานเต็มระบบ: ใช้ Orchestrator 00 หรือ 10 ให้เรียก Skill ทีละตัว

# โครงสร้าง Skill

```text
skills/
├─ 00-enrichment-analysis-orchestrator/
├─ 01-large-orthomosaic-reader/
├─ 02-mangrove-land-cover-classifier/
├─ 03-rhizophora-crown-detector/
├─ 04-weed-groundcover-classifier/
├─ 05-nypa-palm-detector/
├─ 06-existing-canopy-occlusion/
├─ 07-enrichment-gap-analyzer/
├─ 08-enrichment-boundary-delineator/
├─ 09-enrichment-qa-human-review/
├─ 10-full-area-planting-orchestrator/
├─ 11-full-area-tree-detector/
├─ 12-planting-grid-inference/
├─ 13-full-area-mortality-analyzer/
├─ 14-full-area-boundary-delineator/
└─ 15-full-area-qa-human-review/
```

# คำสั่งปลูกเต็มพื้นที่

## รันครบทั้งระบบ

```text
อ่าน AGENTS.md และ skills/README.md
ใช้ skills/10-full-area-planting-orchestrator/SKILL.md

Input Orthomosaic:
D:\Drone\Full_Area\Input\plot_001_orthomosaic.tif

Output:
D:\Drone\Full_Area\Outputs\plot_001

ให้ทำตามลำดับ Skill 01, 11, 12, 13, 14 และ 15
โหลดและอ่าน Skill ทีละตัวเท่านั้น
หยุดให้ตรวจหลัง Tree Detection, Grid Inference,
Survival/Mortality และ Candidate Boundary
ห้ามตั้งสถานะ approved
```

## ตรวจและนับต้นเท่านั้น

```text
อ่านเฉพาะ AGENTS.md, skills/README.md,
skills/01-large-orthomosaic-reader/SKILL.md และ
skills/11-full-area-tree-detector/SKILL.md

ตรวจต้นปลูกเป็นรายพุ่ม แยกเงาและต้นเดิมขนาดใหญ่
ป้องกันการนับซ้ำระหว่าง Tile
ห้ามสร้าง Grid หรือ Boundary

Input: <ORTHOMOSAIC_PATH>
Output: <OUTPUT_PATH>
```

## หาแถว กริด และตำแหน่งต้นหาย

```text
อ่านเฉพาะ skills/12-planting-grid-inference/SKILL.md
ใช้ planted_tree_candidates.gpkg ที่มีอยู่แล้ว

สร้าง Planting Rows, Grid Blocks, ระยะต้น ระยะแถว
และ expected_missing_positions
ห้ามตรวจต้นใหม่และห้ามสร้าง Boundary

Input: <TREE_DETECTION_OUTPUT>
Output: <OUTPUT_PATH>
```

## วิเคราะห์อัตรารอด

```text
อ่านเฉพาะ skills/13-full-area-mortality-analyzer/SKILL.md
ใช้ผล Tree Detection และ Grid ที่มีอยู่แล้ว

แยก surviving, probable_missing, uncertain และ not_observable
แสดงสูตร ตัวหาร และอัตรารอดแยกตาม Grid Block
ห้ามถือ not_observable เป็นต้นตาย

Input: <TREE_AND_GRID_OUTPUTS>
Output: <OUTPUT_PATH>
```

## วงขอบเขตปลูกเต็ม

```text
อ่านเฉพาะ skills/14-full-area-boundary-delineator/SKILL.md
ใช้ผลต้น กริด และ Survival/Mortality ที่มีอยู่แล้ว

สร้าง full_area_planting_core,
full_area_planting_evidence_boundary และ
full_area_uncertain_edge

รวม Mortality Gap เมื่อ Grid ยังต่อเนื่อง
ห้ามใช้ Convex Hull เป็นค่าเริ่มต้น
ห้ามตั้งสถานะ approved

Input: <TREE_GRID_MORTALITY_OUTPUTS>
Output: <OUTPUT_PATH>
```

# คำสั่งปลูกเสริม

```text
อ่าน AGENTS.md และ skills/README.md
ใช้ skills/00-enrichment-analysis-orchestrator/SKILL.md

Input Orthomosaic:
D:\Drone\Enrichment\Input\plot_001_orthomosaic.tif

Output:
D:\Drone\Enrichment\Outputs\plot_001

แยกโกงกาง Candidate, วัชพืช, ต้นจาก, ป่าเดิม,
เรือนยอดปิด น้ำ เลน เงา และช่องว่างปลูกเสริม
วิเคราะห์แบบ Gap-by-gap
หยุดให้ตรวจทุก Checkpoint
ห้ามตั้งสถานะ approved
```

# ความแตกต่างหลัก

## ปลูกเต็มพื้นที่

- ใช้แนวแถวและกริดเป็นหลักฐานหลัก
- ตำแหน่งที่ต้นหายอาจเป็น Mortality Gap
- ขอบเขตสามารถรวมพื้นที่ต้นรอดต่ำได้ หาก Grid ยังต่อเนื่อง
- ต้นเดิมขนาดใหญ่และเงาต้องถูกแยกออกก่อน Fit Grid

## ปลูกเสริม

- วิเคราะห์ทีละช่องว่างของป่าเดิม
- ไม่บังคับให้มีกริดสม่ำเสมอ
- ต้องแยกวัชพืช ต้นจาก ป่าเดิม และเรือนยอดปิด
- พบโกงกางจากภาพยังไม่ยืนยันว่าเกิดจากการปลูก

# Input ที่แนะนำ

- GeoTIFF, COG หรือ VRT ที่มี CRS และ Affine Transform
- Optional AOI, PDD หรือขอบเขตแผนงาน
- จุดปลูกหรือข้อมูลภาคสนาม
- ภาพก่อน–หลังปลูก
- DSM/DTM/CHM หากมี

อย่านำ Orthomosaic หลาย GB เข้า Git โดยตรง ให้เก็บไว้ในเครื่องหรือ Object Storage

# สถานะผลลัพธ์

AI ใช้ได้สูงสุด:

```text
draft
needs_human_review
rework
```

มนุษย์เท่านั้นที่ตั้ง:

```text
reviewed
rejected
approved
```

# หมายเหตุ

Repository นี้เป็นชุด Skill และข้อกำหนดสำหรับให้ Codex พัฒนาและควบคุม Workflow หากยังไม่มี Python Pipeline ที่ทำงานครบ ต้องสร้างและทดสอบ Code ก่อนรัน Orthomosaic จริง
