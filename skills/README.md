# Skill Index

Repository นี้แยกความสามารถออกเป็น Skill ตามหน้าที่ เพื่อให้ Codex เลือกใช้เฉพาะส่วนที่เกี่ยวข้องและลดการตีความปนกัน

## ลำดับ Skill

| ลำดับ | Skill | หน้าที่ |
|---|---|---|
| 00 | `enrichment-analysis-orchestrator` | วางแผนและเรียก Skill ทั้งชุดตามลำดับ |
| 01 | `large-orthomosaic-reader` | อ่าน GeoTIFF/COG/VRT ขนาดใหญ่แบบ Overview และ Tile |
| 02 | `mangrove-land-cover-classifier` | จำแนกสิ่งปกคลุมหลักจากสี รูปทรง Texture และบริบท |
| 03 | `rhizophora-crown-detector` | ตรวจพุ่มโกงกางเป้าหมายเป็นรายต้นหรือรายพุ่ม |
| 04 | `weed-groundcover-classifier` | แยกวัชพืชและพืชคลุมดินออกจากต้นไม้เป้าหมาย |
| 05 | `nypa-palm-detector` | แยกต้นจากด้วยรูปทรงใบแบบรัศมีและ Texture |
| 06 | `existing-canopy-occlusion` | แยกป่าเดิมและพื้นที่เรือนยอดปิดที่มองไม่เห็นด้านล่าง |
| 07 | `enrichment-gap-analyzer` | วิเคราะห์ช่องว่างที่มีหลักฐานปลูกเสริมแบบ Gap-by-gap |
| 08 | `enrichment-boundary-delineator` | สร้าง Core, Evidence Boundary และ Uncertain Boundary |
| 09 | `enrichment-qa-human-review` | ตรวจ QA, Warning, Debug Layer และเตรียม Human Review |

## กฎการเลือกใช้

- งานเต็มกระบวนการ: เริ่มที่ Skill 00
- ต้องการอ่านภาพใหญ่หรือแบ่ง Tile เท่านั้น: Skill 01
- ต้องการแผนที่สิ่งปกคลุม: Skill 02
- ต้องการนับหรือหาโกงกาง: Skill 03
- ต้องการหาเฉพาะวัชพืช: Skill 04
- ต้องการหาเฉพาะต้นจาก: Skill 05
- ต้องการวิเคราะห์พื้นที่ที่ถูกเรือนยอดเดิมบดบัง: Skill 06
- ต้องการหาช่องว่างปลูกเสริมแต่ยังไม่วงขอบเขต: Skill 07
- ต้องการสร้าง Polygon: Skill 08
- ต้องการตรวจผลก่อนส่งมอบ: Skill 09

## ข้อบังคับร่วม

1. ห้ามใช้สีเพียงอย่างเดียวเป็นคำตอบสุดท้าย
2. ภาพ Crop ใช้เป็นตัวอย่างการตีความ ไม่ใช่พิกัดจริง
3. งานจริงต้องรักษา CRS และ Affine Transform ของ Orthomosaic
4. `target_rhizophora` หมายถึงวัตถุที่มีลักษณะคล้ายโกงกางเป้าหมายจากภาพ ไม่ได้ยืนยันว่าเกิดจากการปลูก
5. พื้นที่ใต้เรือนยอดปิดต้องเป็น `closed_canopy_unknown` ไม่ใช่ `not_planted`
6. AI สร้างได้เฉพาะ Candidate Result และห้ามตั้งสถานะ `approved`
