# train-plant

ชุด Skill สำหรับวิเคราะห์ Orthomosaic งานปลูกป่าชายเลน ตั้งแต่ตรวจต้น หาแนวปลูก แยกเรือนยอดชิด ตรวจจำนวน วิเคราะห์อัตรารอด วงขอบเขต วางแผนลงพื้นที่ และนำผลภาคสนามกลับมาวัดความแม่นยำ

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
→ 12c แยก Crown Center ในเรือนยอดชิด
→ 12b ตรวจจำนวนสุดท้ายด้วย Canopy + Spacing
→ 13 Survival/Mortality
→ 14 Boundary
→ 15 QA
```

เริ่มที่:

```text
skills/10-full-area-planting-orchestrator/SKILL.md
```

กฎสำคัญ:

- จุดจาก Detector ยังไม่ใช่จำนวนต้นสุดท้าย
- ห้ามข้าม Skill 12c เมื่อมีเรือนยอดชิดหรือรวมเป็นผืน
- ห้ามข้าม Skill 12b
- Grid ห้ามสร้างต้นที่ไม่มีเรือนยอด
- `grid_created_tree_count` และ `center_without_canopy_count` ต้องเป็น 0
- น้ำไม่ใช่ Exclusion ทั้งหมด แต่จุดบนผิวน้ำที่ไม่มีพุ่มต้องตัดออก
- เรือนยอดชิดให้นับจาก Crown Center ที่มีหลักฐานภาพ ร่วมกับระยะและแนวปลูก
- ภาพละเอียดไม่พอให้รายงาน Unresolved ห้ามบังคับแยกต้น
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

## Prompt: วิเคราะห์แปลงปลูกเต็ม

```text
อ่าน AGENTS.md
ใช้ skills/10-full-area-planting-orchestrator/SKILL.md

Plot code: <PLOT_CODE>
Orthomosaic: <ORTHOMOSAIC_PATH>
Output: <OUTPUT_PATH>/<PLOT_CODE>
Config: config/defaults.yaml

ทำ Production Preflight ด้วย Skill 01 และ 01b
จากนั้นรัน 11 → 12 → 12c → 12b → 13 → 14 → 15

ห้ามใช้ Raw Detection หรือ Grid เป็นจำนวนต้นสุดท้าย
หยุดให้ตรวจทุก Checkpoint
ห้ามตั้งสถานะ approved
```

## Prompt: วางแผนเข้าตรวจ

```text
อ่าน AGENTS.md
ใช้ Skill 18 → 19 → 20

Plot code: <PLOT_CODE>
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