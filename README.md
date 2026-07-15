# train-plant

ชุด Skill สำหรับวิเคราะห์ภาพโดรนและ Orthomosaic ของงานปลูกป่าชายเลน แยกเป็น 2 Workflow:

1. **ปลูกเสริม** — ปลูกแทรกในช่องว่างของป่าเดิม
2. **ปลูกเต็มพื้นที่** — ปลูกครอบคลุมพื้นที่กว้างและมีแถวหรือกริดต่อเนื่อง

มี Shared Skill เพิ่มสำหรับอ่าน Raster ขนาดใหญ่ จำแนกสภาพดิน–เลน–น้ำที่มองเห็น และแยกปาล์ม/มะพร้าวออกจากต้นจาก

> ผลทั้งหมดเป็น Candidate Evidence จากภาพ ต้องผ่าน Human Review ไม่ใช่ขอบเขตตามกฎหมาย ภาพช่วงเวลาเดียวไม่ยืนยันว่าต้นเกิดจากการปลูก และสีภาพไม่สามารถยืนยันคุณสมบัติดินทางห้องปฏิบัติการ

# เลือก Workflow ก่อน

| ลักษณะภาพ | Workflow | Skill เริ่มต้น |
|---|---|---|
| ป่าเดิมหนาแน่น มีโกงกางปลูกแทรก วัชพืช ต้นจาก และช่องว่างเป็นหย่อม | ปลูกเสริม | `skills/00-enrichment-analysis-orchestrator/SKILL.md` |
| ต้นปลูกกระจายทั่วพื้นที่ เห็นแนวแถว กริด หรือระยะปลูกซ้ำ | ปลูกเต็มพื้นที่ | `skills/10-full-area-planting-orchestrator/SKILL.md` |
| อ่านภาพใหญ่และแบ่ง Tile เท่านั้น | Shared | `skills/01-large-orthomosaic-reader/SKILL.md` |
| จำแนกดิน เลน น้ำขัง แอ่งน้ำ และร่องน้ำที่มองเห็น | Shared | `skills/16-surface-hydrology-condition-classifier/SKILL.md` |
| แยกต้นเดี่ยวทรงดาวที่อาจเป็นปาล์ม/มะพร้าว | Shared | `skills/17-palm-coconut-detector/SKILL.md` |

ห้ามใช้ Workflow ปลูกเสริมกับแปลงปลูกเต็ม และห้ามใช้ Grid ของปลูกเต็มไปบังคับพื้นที่ปลูกเสริม

# ความเข้าใจจากภาพตัวอย่าง

## ต้นจาก

- ขึ้นเป็นกอหรือผืนหนาแน่นต่อเนื่อง
- ใบยาวหลายกอซ้อนกัน ศูนย์กลางไม่ชัด
- ไม่เห็นลำต้นตั้งเด่น
- มักสัมพันธ์กับเลน คลอง และพื้นที่ชุ่มน้ำ
- ใช้ Skill 05

## ปาล์ม/มะพร้าว Candidate

- มักเป็นต้นเดี่ยว
- มีศูนย์กลางเรือนยอดชัด
- ใบแผ่จากยอดเดียวเป็นดาวหรือรัศมี
- อาจเห็นลำต้นหรือเงาลำต้น
- ใช้ Skill 17 และห้ามเรียกเป็นต้นจากโดยอัตโนมัติ

## ผิวดิน–เลน–น้ำ

Class จากภาพเป็นเพียงสภาพผิวที่มองเห็น เช่น `dry_pale_mud`, `wet_mud`, `waterlogged_depression`, `disturbed_fill_soil` และ `possible_salt_crust`

`possible_salt_crust` ไม่ใช่ผลยืนยันดินเค็ม ต้องตรวจ EC, pH, ระดับพื้นที่ และข้อมูลภาคสนาม

# กฎประหยัด Token

Codex ต้องอ่านเฉพาะ:

1. `AGENTS.md`
2. `skills/README.md`
3. Skill ที่ผู้ใช้เรียก
4. Skill ก่อนหน้าที่เป็น Input โดยอ่านเฉพาะ Output Contract เมื่อจำเป็น

ห้ามอ่าน Skill ทั้ง Repository พร้อมกัน

ตัวอย่าง:

- นับต้นปลูกเต็ม: อ่าน Skill 01 และ 11
- หา Grid: อ่าน Skill 12 และอ่าน Output Contract ของ 11
- วงขอบเขตปลูกเต็ม: อ่าน Skill 14 และ Output Contract ของ 11–13
- แยกต้นจาก: อ่าน Skill 05 และ Context ที่จำเป็น
- แยกปาล์ม/มะพร้าว: อ่าน Skill 17 เท่านั้น พร้อม Input Class/Tile
- วิเคราะห์พื้นผิวและน้ำ: อ่าน Skill 16 เท่านั้น พร้อม Skill 01 เมื่อยังไม่มี Tile
- งานเต็มระบบ: ใช้ Orchestrator 00 หรือ 10 ให้เรียกทีละ Skill

หลังจบแต่ละขั้น ส่งต่อเพียง:

```text
source_file_paths
schema/version
crs
config_used
summary_metrics
warnings
```

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
├─ 15-full-area-qa-human-review/
├─ 16-surface-hydrology-condition-classifier/
└─ 17-palm-coconut-detector/
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

ทำตามลำดับ Skill 01, 11, 12, 13, 14 และ 15
โหลด Skill ทีละตัว

เรียก Skill 16 เฉพาะเมื่อมีพื้นดิน เลน น้ำขัง หรือ Surface Zone ต่างกันชัด
เรียก Skill 17 เฉพาะเมื่อพบต้นเดี่ยวทรงรัศมีที่อาจเป็นปาล์ม/มะพร้าว

หยุดให้ตรวจหลัง Tree Detection, Grid Inference,
Survival/Mortality, Optional Context และ Candidate Boundary
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
```

# คำสั่งปลูกเสริม

```text
อ่าน AGENTS.md และ skills/README.md
ใช้ skills/00-enrichment-analysis-orchestrator/SKILL.md

Input Orthomosaic:
D:\Drone\Enrichment\Input\plot_001_orthomosaic.tif

Output:
D:\Drone\Enrichment\Outputs\plot_001

แยกโกงกาง Candidate, วัชพืช, ต้นจาก, ปาล์ม/มะพร้าว,
ป่าเดิม เรือนยอดปิด น้ำ เลน เงา และช่องว่างปลูกเสริม
วิเคราะห์แบบ Gap-by-gap

เรียก Skill 16 เมื่อภาพมี Surface/Hydrology ต่างกันชัด
เรียก Skill 17 เมื่อพบต้นเดี่ยวทรงรัศมี
หยุดให้ตรวจทุก Checkpoint
ห้ามตั้งสถานะ approved
```

# คำสั่งจำแนกดิน–เลน–น้ำที่มองเห็น

```text
อ่าน AGENTS.md, skills/README.md และ
skills/16-surface-hydrology-condition-classifier/SKILL.md

Input: <ORTHOMOSAIC_OR_TILE_MANIFEST>
Output: <OUTPUT_PATH>

แยก shallow water, waterlogged depression, tidal microchannel,
wet mud, dry pale mud, dry oxidized soil, disturbed fill soil,
transitional mud และ possible salt crust

ห้ามยืนยันความเค็ม pH หรือชนิดดินจากภาพ
สร้าง Surface Preview, Confidence และจุดที่ควรตรวจภาคสนาม
```

# คำสั่งแยกต้นจากกับปาล์ม/มะพร้าว

```text
อ่านเฉพาะ skills/05-nypa-palm-detector/SKILL.md และ
skills/17-palm-coconut-detector/SKILL.md

แยก:
- Nypa: เป็นกอ/ผืน ใบซ้อน ศูนย์กลางไม่ชัด ไม่มีลำต้นเด่น
- Palm/Coconut Candidate: ต้นเดี่ยว ศูนย์กลางชัด ทรงดาว อาจเห็นลำต้นหรือเงา

Input: <ORTHOMOSAIC_OR_TILE_MANIFEST>
Output: <OUTPUT_PATH>
ห้ามยืนยันชนิดพฤกษศาสตร์เมื่อหลักฐานไม่พอ
```

# Input ที่แนะนำ

- GeoTIFF, COG หรือ VRT ที่มี CRS และ Affine Transform
- Optional AOI, PDD หรือขอบเขตแผนงาน
- จุดปลูกหรือข้อมูลภาคสนาม
- ภาพก่อน–หลังปลูก
- DSM/DTM/CHM หากมี
- จุดตรวจ pH, EC, ความเค็ม เนื้อดิน และระดับพื้นที่เมื่อวิเคราะห์ Surface

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
