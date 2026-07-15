---
name: project-input-preflight
description: ตรวจความพร้อมและความสอดคล้องของ Orthomosaic, AOI, จุดปลูก, จุดภาคสนาม และเส้นทางก่อนเริ่ม Workflow จริง โดยรักษาแหล่งที่มา CRS วันที่ และระดับความน่าเชื่อถือของแต่ละข้อมูล
---

# Project Input Preflight

## หน้าที่

สร้างสัญญา Input กลางของโครงการก่อนวิเคราะห์จริง เพื่อป้องกันการนำไฟล์คนละแปลง คนละวัน คนละ CRS หรือจุดที่อนุมานจากกริดไปปะปนกับจุดปลูกที่สำรวจจริง

งานทดลองจาก JPG/PNG สามารถข้ามได้ แต่ผลต้องเป็น `image_demo_only` และห้ามรายงานพื้นที่หรือระยะจริง

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

## Input Source Classes

ทุก Layer ต้องมี Source Class:

```text
surveyed_field
contract_or_approved_plan
existing_gis
human_digitized_from_orthomosaic
model_inferred
unknown
```

จุดปลูกต้องแยกสถานะ:

```text
surveyed_planting_point
approved_plan_point
digitized_candidate_point
inferred_grid_position
```

ห้ามเรียก `inferred_grid_position` ว่าจุดปลูกจริง

## Validation Rules

1. ตรวจ `project_id`, `plot_code`, วันที่ และชื่อไฟล์ว่าไม่ขัดกัน
2. ตรวจ CRS ของทุก Spatial Layer และกำหนด Working CRS แบบ Projected สำหรับระยะ/พื้นที่
3. ห้าม Reproject หรือแก้ Geometry โดยไม่บันทึก Source CRS, Target CRS และ Method
4. ตรวจ Bounds ว่า Layer ซ้อนกับ Raster/AOI อย่างสมเหตุสมผล
5. ตรวจ Duplicate Point, Geometry Invalid, NoData และ Missing Attribute
6. ตรวจหน่วยระยะและพื้นที่ ห้ามใช้ Degree คำนวณเมตร
7. ตรวจวันที่ภาพเทียบกับวันที่ปลูกและวันที่สำรวจ
8. Planned Point, Surveyed Point และ Inferred Grid ต้องเก็บแยก Layer หรือแยก `point_source`
9. Route จาก Orthomosaic ต้องเป็น Candidate ไม่ใช่ Verified Route
10. ไฟล์ที่ไม่มีแหล่งที่มาหรือวันที่ให้ลด Reliability และสร้าง Warning

## Reliability Classes

```text
R1_verified_field_or_approved
R2_trusted_existing_gis
R3_human_digitized_candidate
R4_model_inferred
R5_unknown
```

## Required Outputs

```text
project_manifest.json
input_inventory.csv
input_warnings.json
normalized_input_references.json
preflight_summary.md
```

`project_manifest.json` ต้องระบุ:

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

หยุดและขอข้อมูลเพิ่มเมื่อ:

- Raster ไม่มี CRS/Transform แต่ผู้ใช้ต้องการพิกัดจริง
- AOI หรือจุดปลูกอยู่นอก Raster อย่างผิดปกติ
- CRS ไม่ทราบและไม่สามารถกำหนดอย่างมีหลักฐาน
- `plot_code` ของ Input ขัดกัน
- วันที่ภาพเก่ากว่าวันปลูก แต่ถูกขอให้คำนวณอัตรารอดหลังปลูก
- จุดปลูกไม่มี Source Class และจะถูกใช้เป็นตัวหารอัตรารอด
- Output Directory เขียนไม่ได้หรือเสี่ยงทับผลเดิม

## Downstream Contract

ทุก Skill ถัดไปอ่าน `project_manifest.json` และ Input ที่เกี่ยวข้องเท่านั้น ไม่ต้องอ่านคำอธิบายโครงการซ้ำทั้งหมด

## Restrictions

- ห้ามแก้ Source File
- ห้ามรวม Planned Point กับ Inferred Grid โดยไม่เก็บ Provenance
- ห้ามถือ AOI เป็นขอบเขตผลวิเคราะห์โดยอัตโนมัติ
- ห้ามสร้างข้อมูลที่ขาดขึ้นมาเพื่อให้ Workflow รันต่อ
- สถานะสูงสุดของ Preflight คือ `ready_with_warnings`; ผู้ใช้ต้องตัดสิน Warning สำคัญ