---
name: full-area-mortality-analyzer
description: วิเคราะห์อัตรารอดและช่องว่างต้นตายในแปลงปลูกเต็มพื้นที่จากต้นที่ตรวจพบและกริดที่อนุมานแล้ว
---

# Full-area Survival and Mortality Analyzer

## Scope

ทำเฉพาะการจัดสถานะตำแหน่งปลูกและสร้างโซนอัตรารอด ห้ามวงขอบเขตแปลงสุดท้าย

## Inputs

```text
planted_tree_candidates.gpkg
planting_grid.gpkg
expected_missing_positions.gpkg
grid_blocks.gpkg
```

## Position classes

```text
surviving_tree
probable_surviving_tree
probable_missing_tree
uncertain_position
not_observable
```

## Rules

- `probable_missing_tree` ต้องอยู่ใน Grid ที่เชื่อถือได้
- เงา ภาพเบลอ น้ำขุ่น หรือ NoData ให้เป็น `not_observable` ไม่ใช่ต้นตาย
- ห้ามสรุปอัตรารอดในบริเวณที่ตรวจไม่ได้
- คำนวณแยกตาม Grid Block และ Zone ไม่รวมทุกพื้นที่เป็นค่าเดียวอย่างเดียว
- ต้นเดิมขนาดใหญ่ไม่รวมเป็นต้นปลูกที่รอด

## Zone classes

```text
high_survival_zone
medium_survival_zone
low_survival_zone
mortality_gap_zone
uncertain_observation_zone
```

Threshold ต้องตั้งใน Config และรายงานค่าที่ใช้ ห้ามฝังค่าคงที่โดยไม่เปิดเผย

## Metrics

```text
observed_surviving_count
probable_surviving_count
probable_missing_count
observable_expected_count
not_observable_count
survival_rate_observed
mortality_rate_observed
uncertain_ratio
```

สูตรอัตรารอดต้องไม่รวม `not_observable` ในตัวหาร เว้นแต่ผู้ใช้ระบุวิธีอื่น

## Outputs

```text
planting_position_status.gpkg
survival_mortality_zones.gpkg
survival_mortality_metrics.json
survival_mortality_preview.png
```

## QA

- แจ้งเตือนเมื่อพื้นที่ตรวจไม่ได้มีสัดส่วนสูง
- แยก Missing Tree ออกจาก Unobservable
- แสดงตัวหารและสูตรทุกครั้ง
- ห้ามเรียก `probable_missing_tree` ว่าต้นตายยืนยันแล้วโดยไม่มี Ground Truth
