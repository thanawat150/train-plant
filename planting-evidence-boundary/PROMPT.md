# Prompt สำหรับให้ Codex เรียนรู้ตัวอย่างก่อนทำงาน

```text
อ่าน planting-evidence-boundary/SKILL.md และไฟล์ทั้งหมดภายใน
planting-evidence-boundary/examples/example_01

ในขั้นตอนแรกยังห้ามเขียน Code และห้ามสร้าง Polygon ใหม่

ให้เปรียบเทียบ:

1. input.jpg
2. expected_overlay.jpg
3. expected_boundary_pixel.json
4. example_manifest.json
5. notes.md

จากนั้นจัดทำ Visual Interpretation Report โดยตอบ:

1. พื้นที่ใดมีร่องรอยการปลูก
2. หลักฐานที่ใช้พิจารณาคืออะไร
3. พื้นที่ใดต้องรวม
4. พื้นที่ใดต้องตัดออก
5. คลอง ถนน คันดิน และแปลงข้างเคียงอยู่บริเวณใด
6. แนวขอบด้านใดมองเห็นชัด
7. แนวขอบด้านใดไม่ชัด
8. ส่วนใดควรเป็น uncertain zone
9. เหตุใด expected boundary จึงมีรูปทรงดังกล่าว
10. กฎใดใน SKILL.md ถูกใช้กับแต่ละด้านของ Polygon

ให้สร้างตาราง:

| บริเวณ | หลักฐานที่พบ | รวม/ไม่รวม/ไม่มั่นใจ | เหตุผล |

หลังจากนั้นให้สรุป Boundary Decision ด้วยภาษาของตนเอง

หากยังไม่สามารถอธิบายเหตุผลของเส้นขอบแต่ละด้านได้ ห้ามเริ่มพัฒนาระบบ
```

# Prompt สำหรับภาพใหม่

```text
ใช้ planting-evidence-boundary skill

ภาพที่ต้องวิเคราะห์: <PATH_TO_IMAGE>
รหัสแปลง: <PLOT_CODE>
จังหวัด: <PROVINCE>

ก่อนสร้าง Polygon ให้เปรียบเทียบภาพใหม่กับตัวอย่างใน examples/example_01
และสร้าง Visual Interpretation Report ก่อนเสมอ

ลำดับการทำงาน:

1. ตรวจคุณภาพภาพ
2. ระบุร่องรอยการปลูก
3. ระบุคลอง ถนน คันดิน น้ำเปิด และแปลงข้างเคียง
4. แยกพื้นที่ include / exclude / uncertain
5. สร้าง candidate mask
6. สร้าง candidate polygon
7. ตรวจ geometry
8. สร้าง overlay preview
9. รอ Human Review
10. หลังได้รับการยืนยันจึง export GeoJSON, GPKG และ KML

ห้าม:

- ตั้งสถานะ approved
- ใช้ขอบภาพเป็นขอบเขต
- ใช้ PDD boundary เป็นคำตอบแทนหลักฐานในภาพ
- ครอบพื้นที่น้ำเปิดหรือพื้นที่ว่างขนาดใหญ่
- รวมแปลงข้างเคียงที่ถูกคลอง ถนน หรือคันดินแบ่งออก
- อ้างความมั่นใจสูงเมื่อแนวขอบไม่ชัด
```
