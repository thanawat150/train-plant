# train-plant

ชุด Skill สำหรับวิเคราะห์ภาพโดรนและ Orthomosaic ของงานปลูกป่าชายเลน ตั้งแต่ตรวจต้น หาแถว/กริด ตรวจความถูกต้องของจำนวนต้น วิเคราะห์อัตรารอด วงขอบเขต ไปจนถึงเลือกจุดเข้าตรวจและวางเส้นทางทางบก–ทางเรือ

Workflow หลักมี 2 แบบ:

1. **ปลูกเสริม** — ปลูกแทรกในช่องว่างของป่าเดิม
2. **ปลูกเต็มพื้นที่** — ปลูกครอบคลุมพื้นที่กว้าง มีแถว กริด หรือ Planting Block ที่ระยะใกล้เคียงกัน

> ผลทั้งหมดเป็น Candidate Evidence, Candidate Count, Candidate Route และ Candidate Mission ต้องผ่าน Human Review

# เลือก Skill ตามงาน

| งาน | Skill เริ่มต้น |
|---|---|
| วิเคราะห์ปลูกเสริมครบระบบ | `skills/00-enrichment-analysis-orchestrator/SKILL.md` |
| วิเคราะห์ปลูกเต็มครบระบบ | `skills/10-full-area-planting-orchestrator/SKILL.md` |
| อ่าน Orthomosaic และแบ่ง Tile | `skills/01-large-orthomosaic-reader/SKILL.md` |
| Raw Crown Detection | `skills/11-full-area-tree-detector/SKILL.md` |
| หาแนวและระยะปลูก | `skills/12-planting-grid-inference/SKILL.md` |
| ตรวจจำนวนต้นด้วย Canopy + Spacing | `skills/21-spacing-guided-tree-count-validator/SKILL.md` |
| วิเคราะห์อัตรารอด | `skills/13-full-area-mortality-analyzer/SKILL.md` |
| จำแนกดิน–เลน–น้ำที่มองเห็น | `skills/16-surface-hydrology-condition-classifier/SKILL.md` |
| แยกปาล์ม/มะพร้าวออกจากต้นจาก | `skills/17-palm-coconut-detector/SKILL.md` |
| เลือกจุดที่ควรเข้าตรวจ | `skills/18-field-inspection-priority-analyzer/SKILL.md` |
| วางเส้นทางทางบก/ทางเรือ | `skills/19-multimodal-access-route-planner/SKILL.md` |
| จัดกลุ่มภารกิจ วัน และทีม | `skills/20-field-inspection-mission-planner/SKILL.md` |

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
├─ 17-palm-coconut-detector/
├─ 18-field-inspection-priority-analyzer/
├─ 19-multimodal-access-route-planner/
├─ 20-field-inspection-mission-planner/
└─ 21-spacing-guided-tree-count-validator/
```

# กฎประหยัด Token

Codex ต้องอ่านเฉพาะ:

1. `AGENTS.md`
2. `skills/README.md`
3. Skill ที่กำลังใช้
4. Output Contract ของ Skill ก่อนหน้าเท่าที่จำเป็น

ห้ามอ่าน Skill ทั้ง Repository พร้อมกัน

เมื่อส่งงานต่อขั้นถัดไป ให้ส่งเพียง:

```text
source_file_paths
schema/version
crs
config_used
summary_metrics
warnings
```

# Workflow ปลูกเต็มพื้นที่ฉบับปรับใหม่

```text
Orthomosaic
→ Skill 01 อ่านภาพและแบ่ง Tile
→ Skill 11 สร้าง Raw Crown/Tree Candidates
→ Skill 12 หา Reference Spacing, แนวแถว และ Pattern Blocks
→ Skill 21 ตรวจว่าอะไรนับเป็นต้นได้จริง
→ Skill 13 วิเคราะห์อัตรารอด/ตาย
→ Skill 14 วงขอบเขตปลูกเต็ม
→ Skill 15 ตรวจ QA
```

Optional:

```text
Skill 16 วิเคราะห์ Surface/Hydrology
Skill 17 แยก Palm/Coconut
Skill 18 → 19 → 20 วางแผนเข้าตรวจภาคสนาม
```

## ความเข้าใจใหม่สำหรับการนับต้น

### จุดแดงไม่ใช่จำนวนต้น

จุดจาก Detector เป็น Raw Candidate ต้องตรวจต่อว่า:

- มีเรือนยอดจริงรองรับหรือไม่
- อยู่บนศูนย์กลางพุ่มหรือไม่
- สอดคล้องกับระยะปลูก แนวแถว กริด หรือจุดปลูกหรือไม่
- เป็นจุดซ้ำในพุ่มเดียวหรือไม่
- ตกบนดิน น้ำเปล่า ถนน วัชพืช หรือเงาหรือไม่

### น้ำไม่ใช่พื้นที่ห้ามนับทั้งหมด

```text
จุดบนผิวน้ำ + ไม่มีพุ่ม = false_positive_on_water
ต้นอยู่ในน้ำตื้น + เห็นเรือนยอดจริง = valid_tree_in_water_context
```

### เรือนยอดชิดกัน

ห้ามใช้:

```text
1 canopy blob = 1 tree
```

ให้ใช้:

```text
Reference Spacing
+ Row/Grid/Planting Block
+ Planned Point เมื่อมี
+ Canopy Support
```

แม้ขอบเรือนยอดจะชนกัน แต่ถ้าศูนย์กลางต้นยังห่างใกล้เคียงกัน สามารถวาง Center ตาม Pattern ได้

### ต้นใหญ่เดิม

- ไม่นับเป็นต้นปลูก
- ไม่ใช้ Fit Grid
- ไม่แบ่งยอดย่อยภายในต้นใหญ่แล้วนับหลายต้น
- จุดปลูกใต้พุ่มใหญ่เป็น `not_observable_under_existing_tree`
- ห้ามเรียกเป็นต้นตายโดยอัตโนมัติ

### จุดฟุ้ง

จุดที่กระจายบนพื้นที่โล่งโดยไม่มี:

- Canopy Support
- แนวหรือ Pattern
- Local Spacing Consistency

ให้เป็น `unreliable_random_scatter_zone` และห้ามใช้คำนวณอัตรารอด

# Prompt สำหรับแปลง 17-STC

```text
อ่าน AGENTS.md และ skills/README.md
ใช้ skills/10-full-area-planting-orchestrator/SKILL.md

Plot code: 17-STC
Input Orthomosaic:
<ORTHOMOSAIC_PATH>

Optional planned planting points:
<PLANNED_PLANTING_POINTS_PATH>

Output:
<OUTPUT_PATH>\17-STC

ทำตามลำดับ:
1. Skill 01 อ่านและแบ่ง Tile
2. Skill 11 สร้าง Raw Crown/Tree Candidates จากเรือนยอดจริง
3. Skill 12 หา Reference Spacing, แนวแถว และ Planting Pattern Blocks
4. Skill 21 ตรวจจำนวนต้นด้วย Canopy + Spacing + จุดปลูก
5. Skill 13 วิเคราะห์อัตรารอด/ตาย
6. Skill 14 วง Candidate Boundary
7. Skill 15 ตรวจ QA

กฎสำคัญ:
- จุดแดงจาก Skill 11 ยังไม่ใช่จำนวนต้นสุดท้าย
- ตัดจุดบนดินโล่ง น้ำเปล่า ถนน คันดิน วัชพืช และ Shadow-only
- ต้นในน้ำตื้นนับได้เมื่อมีเรือนยอดจริง
- เรือนยอดชิดกันให้นับจากระยะปลูกและแนว ไม่ใช้หนึ่งก้อนเท่ากับหนึ่งต้น
- ต้นเดิมขนาดใหญ่ไม่ต้องนับและห้ามใช้ Fit Grid
- จุดปลูกใต้ต้นใหญ่ให้เป็น not_observable
- Grid ห้ามสร้างต้นรอดเมื่อไม่มี Canopy Evidence
- แบ่งหลาย Pattern Block หากแนวหรือระยะเปลี่ยน
- จุดฟุ้งที่ไม่มี Canopy/Pattern ให้ Flag เป็น Validation Failed
- รายงาน Confirmed, Spacing-supported, Probable, Missing และ Not-observable แยกกัน
- ห้ามตั้งสถานะ approved

หยุดให้ตรวจหลัง:
1. Raw Detection Preview
2. Reference Spacing และ Pattern Blocks
3. False-positive / Existing Large Tree Preview
4. Merged-canopy Center Preview
5. Final Count Status
6. Survival/Mortality Preview
```

# ผลลัพธ์หลักของ Skill 21

```text
validated_planted_tree_points.gpkg
planting_point_status.gpkg
false_positive_detections.gpkg
existing_large_trees_validated.gpkg
merged_canopy_clusters.gpkg
planting_pattern_blocks.gpkg
spacing_model.json
tree_count_validation_metrics.json
tree_count_validation_preview.png
false_positive_preview.png
merged_canopy_preview.png
```

# Workflow ปลูกเสริม

```text
Orthomosaic
→ Skill 01 อ่านภาพและแบ่ง Tile
→ Skill 02 จำแนกสิ่งปกคลุม
→ Skill 03 ตรวจโกงกาง Candidate
→ Skill 04 แยกวัชพืช
→ Skill 05 ตรวจต้นจาก
→ Skill 06 ป่าเดิม/เรือนยอดปิด
→ Skill 07 วิเคราะห์ Gap
→ Skill 08 วง Candidate Boundary
→ Skill 09 ตรวจ QA
```

ปลูกเสริมไม่ใช้ Grid ของปลูกเต็มไปบังคับช่องว่างป่าเดิม

# Field Inspection Planning

```text
Analysis/QA Outputs
→ Skill 18 เลือกจุดที่ต้องตรวจ
→ Skill 19 ประเมินทางบก/ทางเรือ
→ Skill 20 จัด Mission, วัน, ทีม และ Checklist
```

ระบบต้องแยก:

```text
evidence_priority_score = จำเป็นต้องตรวจมากเพียงใด
access_burden_score     = เข้าถึงยากเพียงใด
```

ห้ามลดความสำคัญของจุดเพียงเพราะไกล จุดสำคัญแต่ไกลให้เป็น `special_mission`, `remote_cluster_mission` หรือ `route_scout_first`

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
