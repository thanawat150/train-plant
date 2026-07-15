# train-plant

ชุด Skill สำหรับวิเคราะห์ Orthomosaic งานปลูกป่าชายเลน ตั้งแต่ตรวจต้น หาแนวปลูก ตรวจจำนวน วิเคราะห์อัตรารอด วงขอบเขต วางแผนลงพื้นที่ และนำผลภาคสนามกลับมาวัดความแม่นยำ

> ผลทั้งหมดเป็น Candidate จนกว่าจะผ่าน Human Review

## Workflow หลัก

### ปลูกเสริม

```text
01 อ่าน Raster
→ 01b ตรวจ Input
→ 02–06 จำแนกพืช/สิ่งรบกวน
→ 07 วิเคราะห์ Gap
→ 08 วง Candidate Boundary
→ 09 QA
```

เริ่มที่:

```text
skills/00-enrichment-analysis-orchestrator/SKILL.md
```

### ปลูกเต็มพื้นที่

```text
01 อ่าน Raster
→ 01b ตรวจ Input
→ 11 Raw Detection
→ 12 Preliminary Spacing/Pattern
→ 12b ตรวจจำนวนจริงด้วย Canopy + Spacing
→ 13 Survival/Mortality
→ 14 Boundary
→ 15 QA
```

เริ่มที่:

```text
skills/10-full-area-planting-orchestrator/SKILL.md
```

กฎสำคัญ:

- จุดแดงจาก Detector ยังไม่ใช่จำนวนต้นสุดท้าย
- ห้ามข้าม Skill 12b
- Grid ห้ามสร้างต้นที่ไม่มีเรือนยอด
- น้ำไม่ใช่ Exclusion ทั้งหมด แต่จุดบนผิวน้ำที่ไม่มีพุ่มต้องตัดออก
- เรือนยอดชิดกันให้นับจากศูนย์กลาง ระยะปลูก และแนวปลูก
- ต้นเดิมขนาดใหญ่ไม่นับ และจุดใต้พุ่มใหญ่เป็น Not Observable
- แปลงนากุ้งแบ่งเป็นหลาย Planting Pattern Block ได้

## วางแผนเข้าตรวจ

```text
18 เลือกจุดตรวจ
→ 19 ประเมินทางบก/ทางเรือ
→ 20 จัด Mission/วัน/ทีม
```

จุดสำคัญแต่ไกลต้องเป็น Special Mission หรือรวมหลายจุดในเที่ยวเดียว ไม่ตัดทิ้งเพราะระยะทาง

## นำผลภาคสนามกลับมาใช้

```text
21 รับ Field Feedback / Ground Truth
→ 22 ประเมิน Precision, Recall, Count Error และ Threshold
```

Ground Truth ต้องมีผู้ตรวจ วันที่ และ GPS Accuracy

## ไฟล์นำทาง

- `AGENTS.md` — กฎบังคับสำหรับ Codex
- `skills/README.md` — เลือก Skill ตามงาน
- `skills/registry.yaml` — Registry สำหรับระบบ
- `docs/ARCHITECTURE.md` — โครงสร้างและ Data Flow
- `config/defaults.yaml` — ค่าเริ่มต้นที่ต้องปรับตามโครงการ
- `schemas/` — Data Contracts

## Prompt: วิเคราะห์แปลงปลูกเต็ม 17-STC

```text
อ่าน AGENTS.md
ใช้ skills/10-full-area-planting-orchestrator/SKILL.md

Plot code: 17-STC
Orthomosaic: <ORTHOMOSAIC_PATH>
Output: <OUTPUT_PATH>/17-STC
Config: config/defaults.yaml

ทำ Production Preflight ด้วย Skill 01 และ 01b
จากนั้นรัน 11 → 12 → 12b → 13 → 14 → 15

ห้ามใช้ Raw Detection เป็นจำนวนต้นสุดท้าย
หยุดให้ตรวจทุก Checkpoint
ห้ามตั้งสถานะ approved
```

## Prompt: วางแผนเข้าตรวจ

```text
อ่าน AGENTS.md
ใช้ Skill 18 → 19 → 20

Plot code: 17-STC
Analysis outputs: <ANALYSIS_OUTPUT_PATH>
Land routes: <LAND_ROUTE_PATH>
Boat routes: <BOAT_ROUTE_PATH>
Access points: <ACCESS_POINT_PATH>
Output: <FIELD_PLAN_OUTPUT_PATH>

แยก Evidence Priority จาก Access Burden
ประเมินทางบกและทางเรือ
จุดสำคัญแต่ไกลห้ามตัดทิ้ง
```

## สถานะโครงการ

Repository นี้เป็นข้อกำหนด Skill และ Data Contract สำหรับให้ Codex พัฒนา/ควบคุม Pipeline หากยังไม่มี Python Implementation ที่ผ่านการทดสอบ ต้องสร้าง Code, Ground Truth และ Evaluation ก่อนใช้ผลอย่างเป็นทางการ

โฟลเดอร์ `planting-evidence-boundary/` เป็น Legacy Reference ไม่ใช่ Config/Schema หลักของระบบใหม่
