# train-plant

ชุด Skill สำหรับวิเคราะห์ภาพโดรนและ Orthomosaic ของงานปลูกป่าชายเลน ตั้งแต่ตรวจต้น หาแถว/กริด วิเคราะห์อัตรารอด วงขอบเขต ไปจนถึงเลือกจุดเข้าตรวจและวางแผนเส้นทางทางบก–ทางเรือ

Workflow หลักมี 2 แบบ:

1. **ปลูกเสริม** — ปลูกแทรกในช่องว่างของป่าเดิม
2. **ปลูกเต็มพื้นที่** — ปลูกครอบคลุมพื้นที่กว้างและมีแถวหรือกริดต่อเนื่อง

> ผลทั้งหมดเป็น Candidate Evidence, Candidate Route และ Candidate Mission ต้องผ่าน Human Review ไม่ใช่ขอบเขตตามกฎหมาย ไม่ใช่ผลตรวจดินทางห้องปฏิบัติการ และไม่ใช่การรับรองความปลอดภัยของเส้นทาง

# เลือก Workflow ก่อน

| งาน | Skill เริ่มต้น |
|---|---|
| วิเคราะห์ปลูกเสริมครบระบบ | `skills/00-enrichment-analysis-orchestrator/SKILL.md` |
| วิเคราะห์ปลูกเต็มครบระบบ | `skills/10-full-area-planting-orchestrator/SKILL.md` |
| อ่าน Orthomosaic และแบ่ง Tile | `skills/01-large-orthomosaic-reader/SKILL.md` |
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
└─ 20-field-inspection-mission-planner/
```

# กฎประหยัด Token

Codex ต้องอ่านเฉพาะ:

1. `AGENTS.md`
2. `skills/README.md`
3. Skill ที่กำลังใช้
4. Skill ก่อนหน้าเฉพาะ Output Contract เมื่อจำเป็น

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

ตัวอย่าง Load Set:

| งาน | อ่านเต็ม |
|---|---|
| นับต้นปลูกเต็ม | 01, 11 |
| หา Grid | 12 และ Output Contract ของ 11 |
| วิเคราะห์อัตรารอด | 13 และ Output Contract ของ 11–12 |
| วงขอบเขตปลูกเต็ม | 14 และ Output Contract ของ 11–13 |
| เลือกจุดเข้าตรวจ | 18 และ Analysis/QA Outputs |
| วิเคราะห์เส้นทาง | 19 และ Output ของ 18 |
| จัด Mission | 20 และ Output ของ 18–19 |

# Workflow ปลูกเต็มพื้นที่

```text
Orthomosaic
→ Skill 01 อ่านภาพและแบ่ง Tile
→ Skill 11 ตรวจต้นปลูก
→ Skill 12 หาแถว กริด และตำแหน่งต้นหาย
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

## Prompt ปลูกเต็มครบระบบ

```text
อ่าน AGENTS.md และ skills/README.md
ใช้ skills/10-full-area-planting-orchestrator/SKILL.md

Plot code: 17-STC
Input Orthomosaic:
<ORTHOMOSAIC_PATH>

Output:
<OUTPUT_PATH>\17-STC

ทำตามลำดับ Skill 01, 11, 12, 13, 14 และ 15
โหลด Skill ทีละตัว

วัตถุประสงค์:
- นับต้นปลูก
- หาแถวและกริด
- หาตำแหน่งต้นหาย
- วิเคราะห์อัตรารอด/ตาย
- สร้าง Candidate Boundary
- สร้าง QA Package

เรียก Skill 16 เมื่อ Surface/Hydrology ต่างกันชัด
เรียก Skill 17 เมื่อพบต้นเดี่ยวทรงรัศมี
หยุดให้ตรวจทุก Checkpoint
ห้ามตั้งสถานะ approved
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

Optional:

```text
Skill 16 Surface/Hydrology
Skill 17 Palm/Coconut
Skill 18 → 19 → 20 Field Inspection Plan
```

# แนวคิดการวางแผนเข้าตรวจ

ระบบต้องแยก 2 คะแนน:

```text
evidence_priority_score = จำเป็นต้องตรวจมากเพียงใด
access_burden_score     = เข้าถึงยากเพียงใด
```

ห้ามลดความสำคัญของจุดเพียงเพราะไกล

| ความจำเป็น | การเข้าถึง | การจัดการ |
|---|---|---|
| สูง | ง่าย/ปานกลาง | `inspect_now` |
| สูง | ยาก/ไม่ทราบ | `special_mission` หรือ `route_scout_first` |
| ปานกลาง | ง่าย | `bundle_opportunistically` |
| ปานกลาง | ยาก | `remote_cluster_mission` เมื่อรวมหลายจุดคุ้มค่า |
| ต่ำ | ง่าย | QA Control หรือแวะตรวจร่วม |
| ต่ำ | ยาก | เลื่อนหรือ Remote Monitor เว้นแต่ขาดตัวแทนพื้นที่ |

## จุดที่ควรเข้าตรวจ

ตัวอย่าง Trigger:

- กลุ่มต้นหายต่อเนื่อง
- อัตรารอดต่ำ
- `probable_missing`, `uncertain`, `not_observable` จำนวนมาก
- Grid กับต้นที่ตรวจพบไม่ตรงกัน
- ขอบเขต Confidence ต่ำ
- โกงกางสับสนกับวัชพืช ต้นจาก หรือปาล์ม/มะพร้าว
- น้ำขัง ร่องน้ำ หรือ Surface Zone สัมพันธ์กับการรอดต่ำ
- ผลต่างจากภาพครั้งก่อนผิดปกติ
- พื้นที่ห่างไกลที่ยังไม่มีตัวแทนตรวจ
- High-confidence QA Control สำหรับตรวจความแม่นของระบบ

# ข้อมูลเส้นทางที่ควรเตรียม

```text
road_access_lines
trail_access_lines
dike_or_embankment_lines
boat_route_lines
canal_or_tidal_channel_lines
parking_points
pier_points
boat_landing_points
plot_entry_points
mode_transfer_points
restricted_or_no_go_areas
hazard_zones
```

เส้นทางจากภาพต้องระบุ Source และใช้เป็น Candidate จนกว่าทีมพื้นที่จะยืนยัน

# Prompt เลือกจุดตรวจและวาง Route

```text
อ่าน AGENTS.md และ skills/README.md

ใช้ตามลำดับ:
1. skills/18-field-inspection-priority-analyzer/SKILL.md
2. skills/19-multimodal-access-route-planner/SKILL.md
3. skills/20-field-inspection-mission-planner/SKILL.md

Plot code: 17-STC
Analysis Output:
<ANALYSIS_OUTPUT_PATH>

Access Layers:
<LAND_ROUTE_PATH>
<BOAT_ROUTE_PATH>
<ACCESS_POINT_PATH>
<BARRIER_PATH>

Output:
<FIELD_PLAN_OUTPUT_PATH>

เงื่อนไข:
- เลือกจุดจากผลวิเคราะห์จริง ไม่สุ่มจากพื้นที่อย่างเดียว
- แสดงเหตุผลของทุกจุด
- แยก Evidence Priority กับ Access Burden
- วิเคราะห์ทางบกและทางเรือเป็นทางเลือก
- ใช้ Network Distance และเวลาไป–กลับ
- คำนึงถึงการเดินช่วงสุดท้าย จุดขึ้นฝั่ง Tide, Permission และ Safety
- จุดสำคัญแต่ไกลห้ามตัดทิ้ง ให้สร้าง Special Mission หรือรวมหลายจุด
- มีทั้งจุดปัญหา จุดตัวแทน และ High-confidence QA Control
- หยุดให้ตรวจหลัง Candidate Points, Route Options และ Mission Grouping
- ห้ามตั้งสถานะ approved
```

# ผลลัพธ์ Field Planning

```text
inspection_priority_zones.gpkg
inspection_candidate_points.gpkg
inspection_priority_summary.csv
inspection_route_options.gpkg
inspection_access_assessment.csv
selected_inspection_points.gpkg
field_missions.gpkg
mission_routes.gpkg
mission_day_plan.csv
inspection_backlog.csv
field_checklist.csv
field_mission_preview.png
route_cards/
```

# ข้อควรระวัง

- ระยะเส้นตรงไม่ใช่ระยะเข้าถึงจริง
- คลองทุกเส้นไม่ได้แปลว่าเรือผ่านได้
- คันดินทุกเส้นไม่ได้แปลว่าเดินหรือขับรถได้
- จุดสำคัญและไกลต้องเพิ่มการวางแผน ไม่ใช่ถูกตัดออก
- Route Candidate ต้องให้ทีมพื้นที่ยืนยัน
- ต้องคิดเวลาไป–กลับ Buffer น้ำขึ้นน้ำลง การเปลี่ยนพาหนะ และความปลอดภัย
- ภาพช่วงเวลาเดียวไม่ยืนยันว่าต้นเกิดจากการปลูก
- สีภาพไม่ยืนยันความเค็ม pH หรือชนิดดิน

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

Repository นี้เป็นชุด Skill และข้อกำหนดสำหรับให้ Codex พัฒนาและควบคุม Workflow หากยังไม่มี Python Pipeline ที่ทำงานครบ ต้องสร้างและทดสอบ Code ก่อนรัน Orthomosaic และวาง Route จริง
