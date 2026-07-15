# Roadmap and Remaining Gaps

Skill architecture จัดครบสำหรับ Analysis → Field Plan → Feedback → Evaluation แล้ว แต่ Repository ยังไม่ใช่ Production Pipeline จนกว่าจะมีรายการต่อไปนี้

## P0 — ต้องมีก่อนใช้ผลอย่างเป็นทางการ

1. **Python Implementation**
   - Raster I/O, Tiling, Detection, Pattern Fitting, Validation, Survival, Boundary และ Export
2. **Automated Contract Tests**
   - ตรวจชื่อไฟล์, Required Fields, CRS, Geometry และ Schema ทุก Stage
3. **Labeled Example Dataset**
   - Positive/Negative ของต้นจริง จุดฟุ้งบนน้ำ/ดิน ต้นใหญ่ เรือนยอดชิด และต้นตกหล่น
4. **Independent Test Set**
   - แยกแปลงหรือวันที่ออกจากชุด Calibration
5. **Project-specific Config**
   - ค่าใน `config/defaults.yaml` ที่เป็น `null` ต้อง Calibration ก่อน
6. **Reproducible Environment**
   - Python version, dependencies, lock file และคำสั่งรัน

## P1 — ควรเพิ่มหลัง MVP

### Temporal Change Monitor

เปรียบเทียบหลายวันที่เพื่อแยก:

- ต้นเกิดใหม่/ปลูกซ่อม
- ต้นหายใหม่
- Crown Growth/Decline
- Surface/Hydrology Change
- Survival Curve

### Active Learning Sample Selector

เลือกตัวอย่างที่ควร Label เพิ่มจาก:

- False Positive สูง
- Merged Canopy Unresolved
- Random Scatter
- Context ที่ Ground Truth น้อย
- Model Disagreement

### Standard Report Exporter

สร้างรายงาน:

- จำนวน Confirmed/Probable/Estimated
- อัตรารอดเป็นช่วง
- จุดปลูกซ่อม
- จุดเข้าตรวจและ Route
- QA/Limitations

## P2 — Integration

- Mobile Field Form พร้อม Offline GPS/Photo
- Dashboard ติดตามหลายแปลงและหลายช่วงเวลา
- Route/Permission/Tide Data Integration
- Model Registry และ Config Approval Workflow
- Automated CI ตรวจ Schema และ Skill Registry

## สิ่งที่ไม่ควรเพิ่มเป็น Skill แยกทันที

- Classifier เล็ก ๆ สำหรับทุกสีหรือพืชชนิดย่อย
- Threshold Skill แยกหลายตัว
- Boundary Skill ซ้ำตามประเภทแปลงย่อย

ควรเพิ่มเมื่อมี Ground Truth และพบว่ากฎเดิมไม่สามารถรองรับจริง เพื่อป้องกัน Skill Fragmentation และ Token Waste
