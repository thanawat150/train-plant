# Negative Examples

โฟลเดอร์นี้ใช้เก็บตัวอย่างที่ “ห้ามทำ” เพื่อให้ Codex แยกความผิดพลาดแต่ละชนิดได้

แต่ละตัวอย่างควรมี:

```text
input.jpg
wrong_overlay.jpg
correct_overlay.jpg
notes.md
manifest.json
```

## ชุดตัวอย่างที่ควรเตรียม

### 01 — Over Boundary

วงกว้างเกินไป เช่น:

- ครอบน้ำเปิด
- ครอบพืชธรรมชาติ
- ตามคันดินทั้งแนวทั้งที่ต้นปลูกสิ้นสุดก่อน
- รวมพื้นที่ว่างขนาดใหญ่

Label:

```text
error_type = over_boundary
```

### 02 — Under Boundary

วงแคบเกินไป เช่น:

- วงเฉพาะต้นที่ยังรอด
- ตัด Mortality Gap ออก
- ตัดแนวแถวที่มีต้นขาดบางช่วง

Label:

```text
error_type = under_boundary
```

### 03 — Cross Canal

รวมพื้นที่คนละฝั่งคลองหรือร่องน้ำเป็น Polygon เดียว

```text
error_type = crossed_barrier
```

### 04 — Merge Neighbor Plot

รวมแปลงข้างเคียงเพราะมีรูปแบบการปลูกคล้ายกัน

```text
error_type = neighbor_plot_merged
```

### 05 — Convex Hull Error

ใช้ Convex Hull ปิดรอยเว้า คลอง หรือพื้นที่ไม่มีหลักฐาน

```text
error_type = invalid_convex_hull
```

### 06 — Natural Vegetation Error

ตีความพืชธรรมชาติที่ขึ้นไม่เป็นระเบียบว่าเป็นพื้นที่ปลูก

```text
error_type = natural_vegetation_included
```

### 07 — Tile Edge Artifact

สร้างเส้นตรงตามขอบ Tile เพราะขาด Context

```text
error_type = tile_edge_artifact
```

### 08 — Barrier Following Error

ลากตามถนนหรือคันดินอัตโนมัติโดยไม่ตรวจแนวต้นปลูก

```text
error_type = barrier_following_error
```

## Notes Template

```markdown
# Negative Example

## สิ่งที่ผิด

## เหตุผลที่ผิด

## หลักฐานที่ควรใช้

## แนวขอบที่ถูกต้องควรเป็นอย่างไร

## Warning ที่ระบบควรแจ้ง

## QA Metric ที่ตรวจพบความผิดได้
```

Negative Example ใช้สอนและทดสอบเท่านั้น ห้ามนำ Wrong Overlay ไปใช้เป็น Ground Truth
