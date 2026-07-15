---
name: project-input-preflight
description: ตรวจความพร้อมและความสอดคล้องของ Orthomosaic, AOI, จุดปลูก จุดภาคสนาม และเส้นทางก่อนเริ่ม Workflow จริง โดยรักษา CRS วันที่ Provenance และระดับความน่าเชื่อถือ
---

# Project Input Preflight

## หน้าที่

สร้าง Input Contract กลาง ป้องกันไฟล์คนละแปลง คนละวัน คนละ CRS หรือจุดอนุมานจากกริดปะปนกับจุดปลูกที่สำรวจจริง

JPG/PNG ไม่มีพิกัดใช้ได้เฉพาะ `image_demo_only` และห้ามรายงานระยะ พื้นที่ หรืออัตรารอดจริง

## Required Inputs

```text
project_id
plot_code
source_raster หรือ raster_metadata.json
output_directory
```

Optional:

```text
plot_aoi
planned_planting_points
surveyed_planting_points
contract_planting_plan
field_observations
DSM / DTM / CHM
surface or hydrology layers
land route layers
boat route layers
access points
barriers / restricted areas
acquisition_date
planting_date
```

## Source Classes

```text
surveyed_field
contract_or_approved_plan
existing_gis
human_digitized_from_orthomosaic
model_inferred
unknown
```

Planting Point Classes:

```text
surveyed_planting_point
approved_plan_point
digitized_candidate_point
inferred_grid_position
```

ห้ามเรียก `inferred_grid_position` ว่าจุดปลูกจริง

## Reliability

```text
R1_verified_field_or_approved
R2_trusted_existing_gis
R3_human_digitized_candidate
R4_model_inferred
R5_unknown
```

## Validation Rules

1. ตรวจ `project_id`, `plot_code`, วันที่ และชื่อไฟล์
2. ตรวจ CRS และกำหนด Working CRS แบบ Projected สำหรับระยะ/พื้นที่
3. บันทึก Source CRS, Target CRS และ Reprojection Method
4. ตรวจ Bounds ซ้อนกับ Raster/AOI
5. ตรวจ Duplicate, Geometry Invalid, NoData และ Missing Attribute
6. ห้ามใช้ Degree คำนวณเมตร
7. ตรวจวันที่ภาพเทียบวันที่ปลูก/สำรวจ
8. Planned, Surveyed และ Inferred Point ต้องเก็บ Provenance แยก
9. Route จาก Orthomosaic เป็น Candidate ไม่ใช่ Verified Route
10. Input ไม่มีแหล่งที่มาหรือวันที่ต้องลด Reliability

## Preflight Status

```text
ready
ready_with_warnings
blocked
image_demo_only
```

- `ready`: Required Input ครบและไม่มี Warning สำคัญ
- `ready_with_warnings`: รันต่อได้ แต่ต้องส่ง Warning ไปทุก Stage
- `blocked`: ห้ามเริ่ม Analysis Production
- `image_demo_only`: วิเคราะห์เชิงภาพได้ แต่ไม่มีผลพิกัดจริง

Critical Warning ต้องทำให้เป็น `blocked` ไม่ใช่ `ready_with_warnings`

## Outputs

```text
project_manifest.json
input_inventory.csv
input_warnings.json
normalized_input_references.json
preflight_summary.md
```

Manifest ต้องมี:

```text
schema_version
project_id
plot_code
workflow_requested
source_raster
acquisition_date
working_crs
pixel_size
input_layers
input_source_class
input_reliability
config_path
output_directory
warnings
preflight_status
```

## Stop Conditions

- Raster ไม่มี CRS/Transform แต่ต้องการผลพิกัดจริง
- AOI/จุดปลูกอยู่นอก Raster ผิดปกติ
- CRS ไม่ทราบ
- Plot Code ขัดกัน
- วันที่ภาพเก่ากว่าวันปลูกแต่ขอ Survival หลังปลูก
- จุดปลูกไม่มี Source Class แต่จะใช้เป็นตัวหาร
- Output เสี่ยงทับผลเดิมหรือเขียนไม่ได้

## Downstream Contract

Skill ถัดไปอ่าน `project_manifest.json` และ Input ที่เกี่ยวข้อง ไม่ต้องอ่านคำอธิบายโครงการซ้ำ

## Restrictions

- ห้ามแก้ Source File
- ห้ามรวม Planned Point กับ Inferred Grid โดยไม่เก็บ Provenance
- ห้ามถือ AOI เป็นขอบเขตผลอัตโนมัติ
- ห้ามสร้างข้อมูลที่ขาดเพื่อให้ Workflow ผ่าน
