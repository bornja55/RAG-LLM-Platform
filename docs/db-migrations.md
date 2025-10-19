# 🔄 Database Migration Strategy

## Google Sheets → PostgreSQL Migration Plan

---

## 📋 Overview

### Current State

```
Google Sheets: 16 tabs
Total Rows: ~500-1,000
Format: Spreadsheet
Access: Manual + Google Apps Script
Issues: 
  - ไม่มี transaction
  - ไม่มี referential integrity
  - ช้าเมื่อข้อมูลเยอะ
  - ยาก query ซับซ้อน
```

### Target State

```
PostgreSQL: 28 tables
Expected Rows: ~1,000-5,000
Format: Relational Database
Access: SQL + ORM (SQLAlchemy)
Benefits:
  - ACID compliance
  - Foreign keys
  - Indexes (fast query)
  - Backup & recovery
  - Scalable
```

---

## 📅 Migration Timeline

### Week 1: Preparation

```
Day 1-2: Data Audit
- ตรวจสอบข้อมูลใน Google Sheets
- หา duplicate, missing data
- Document data issues
- จัดลำดับความสำคัญของแต่ละ sheet

Day 3-4: Schema Design
- ออกแบบ PostgreSQL schema
- กำหนด primary keys
- กำหนด foreign keys
- กำหนด indexes
- Document relationships

Day 5-7: Backup & Testing
- Export ข้อมูลทั้งหมดเป็น CSV
- สำรองข้อมูล (3 copies)
- Setup test database
- ทดสอบ import script
```

### Week 2: Migration Execution

```
Day 1: Core Tables (Shared Data)
- courses
- teachers
- schedules
- pricing

Day 2: English Mania Tables
- em_users (สำคัญที่สุด!)
- em_students
- em_enrollments

Day 3: Transaction Tables
- em_attendance
- em_teaching_logs
- em_homework
- em_payments

Day 4: Support Tables
- em_expenses
- em_broadcasts
- faq, policies, etc.

Day 5: Validation
- ตรวจสอบ row counts
- ตรวจสอบ data integrity
- ทดสอบ queries
- แก้ issues ที่เจอ

Day 6-7: Testing & Rollback Prep
- ทดสอบทุก flow
- เตรียม rollback plan
- Document lessons learned
```

---

## 🎯 Migration Approach

### Option 1: Big Bang (NOT Recommended)

```
ข้อดี:
+ เร็ว (1 วัน)
+ ไม่ต้อง sync 2 systems

ข้อเสีย:
- Risk สูง
- ถ้าผิดพลาดต้องย้อนกลับทั้งหมด
- Downtime นาน
```

### Option 2: Phased Migration (Recommended)

```
ข้อดี:
+ Risk ต่ำ
+ ทดสอบทีละส่วน
+ ถ้าผิดพลาด rollback แค่บางส่วน
+ เรียนรู้ไปด้วย

ข้อเสีย:
- ใช้เวลานาน (1-2 สัปดาห์)
- ต้อง sync 2 systems ชั่วคราว

Phases:
1. อ่านอย่างเดียว (read-only from PostgreSQL)
2. เขียนทั้ง 2 ที่ (write to both)
3. อ่านจาก PostgreSQL, backup Google Sheets
4. ปิด Google Sheets
```

### Option 3: Dual-Write (Best for Production)

```
Step 1: Setup PostgreSQL
- สร้าง tables ทั้งหมด
- Import ข้อมูลเก่า

Step 2: Dual-Write Mode (1 สัปดาห์)
- เขียนข้อมูลใหม่ทั้ง Google Sheets + PostgreSQL
- อ่านข้อมูลจาก Google Sheets (เดิม)

Step 3: Switch Read (3 วัน)
- อ่านข้อมูลจาก PostgreSQL
- ยังเขียนทั้ง 2 ที่

Step 4: PostgreSQL Only
- หยุดเขียน Google Sheets
- ใช้ PostgreSQL 100%
- เก็บ Google Sheets ไว้ backup 1 เดือน

Step 5: Decommission
- ปิด Google Sheets
```

---

## 📊 Data Mapping

### Priority 1: Critical Tables (ต้อง migrate ก่อน)

```
Google Sheets → PostgreSQL

courses → courses
  - เปลี่ยน ID format
  - เพิ่ม timestamps
  
teachers → teachers
  - Link กับ LINE user_id
  - เพิ่ม status field

schedules → schedules
  - เพิ่ม foreign keys
  - Validate day_of_week
```

### Priority 2: User Data

```
[สร้างใหม่] → em_users
  - ดึงจาก LINE user data
  - กำหนด roles
  - Link กับ students/teachers

[สร้างใหม่] → em_students
  - นำข้อมูลจาก enrollment records
  - Link กับ parents

[สร้างใหม่] → em_enrollments
  - สร้างจาก current students
  - คำนวณ remaining sessions/hours
```

### Priority 3: Transactional Data

```
[สร้างใหม่] → em_attendance
  - ว่างไว้ (เริ่มบันทึกใหม่)

[สร้างใหม่] → em_teaching_logs
  - ว่างไว้ (เริ่มบันทึกใหม่)

[สร้างใหม่] → em_homework
  - ว่างไว้ (เริ่มบันทึกใหม่)

[สร้างใหม่] → em_payments
  - Import payment history ถ้ามี
```

---

## ✅ Pre-Migration Checklist

### Week Before Migration

```
Technical:
☐ Export ข้อมูลทั้งหมด (CSV)
☐ Backup 3 copies (local, cloud, external drive)
☐ Setup PostgreSQL database
☐ สร้าง tables ทั้งหมด (Alembic migration)
☐ ทดสอบ import script
☐ Setup monitoring (row counts, errors)

Data Quality:
☐ Clean duplicate data
☐ Fix missing values
☐ Standardize formats (dates, phone numbers)
☐ Validate foreign key references
☐ Document data issues

Team:
☐ แจ้ง stakeholders
☐ กำหนด maintenance window
☐ Assign roles (who does what)
☐ Prepare rollback plan
☐ Setup communication channel
```

---

## 🔄 Migration Process

### Day of Migration

```
T-0 (09:00): Start
  - Announce maintenance mode
  - Stop accepting new data in Google Sheets

T+1 (09:00-10:00): Export
  - Export all sheets to CSV
  - Verify file integrity
  - Backup files

T+2 (10:00-12:00): Transform
  - Clean data
  - Convert formats
  - Generate SQL

T+3 (12:00-14:00): Import
  - Import to PostgreSQL
  - Run in order: Shared → Users → Transactions

T+4 (14:00-15:00): Validation
  - Check row counts
  - Verify foreign keys
  - Test queries
  - Sample data check

T+5 (15:00-16:00): Testing
  - Test all flows
  - Test all roles
  - Performance check

T+6 (16:00-17:00): Go Live
  - Switch app to PostgreSQL
  - Monitor errors
  - Fix issues

T+7 (17:00): Complete
  - Announce completion
  - Keep Google Sheets as backup
```

---

## ✅ Post-Migration Validation

### Data Validation

```
Row Counts:
☐ Compare total rows (Sheets vs PostgreSQL)
☐ Check each table individually
☐ Document any discrepancies

Data Integrity:
☐ All primary keys unique
☐ No NULL in required fields
☐ Foreign keys valid
☐ Dates in correct range
☐ Enums match allowed values

Business Logic:
☐ Active courses correct
☐ Student enrollment status correct
☐ Teacher assignments correct
☐ Schedule conflicts checked
```

### Functional Testing

```
Teacher Flow:
☐ เช็คชื่อได้
☐ บันทึกการสอนได้
☐ ให้คะแนนได้

Parent Flow:
☐ ดูข้อมูลลูกได้
☐ ตรวจเวลาคงเหลือได้

Student Flow:
☐ ดูตารางได้
☐ ดูคะแนนได้

Admin Flow:
☐ Dashboard แสดงผลถูก
☐ รายงานถูกต้อง
```

---

## 🔙 Rollback Plan

### When to Rollback

```
Critical Issues:
- Data loss > 5%
- Foreign key violations > 10 errors
- Application errors > 50% requests
- Performance degradation > 5x slower

Minor Issues:
- Fix forward (don't rollback)
- Document and schedule fix
```

### Rollback Procedure

```
Step 1: Announce (15 min)
  - แจ้ง stakeholders
  - Switch app back to Google Sheets

Step 2: Investigate (30 min)
  - Find root cause
  - Document issues
  - Assess if fixable

Step 3: Decision (15 min)
  - Fix forward OR
  - Postpone migration

Step 4: Recovery (30 min)
  - Restore normal operations
  - Schedule new migration date
  - Update stakeholders
```

---

## 📝 Data Transformation Rules

### Data Cleaning

```
Dates:
Google Sheets: Various formats
PostgreSQL: YYYY-MM-DD
Action: Standardize all dates

Phone Numbers:
Google Sheets: 081-234-5678, 0812345678
PostgreSQL: 0812345678 (no dash)
Action: Remove all non-numeric except leading 0

Boolean:
Google Sheets: YES/NO, TRUE/FALSE, 1/0
PostgreSQL: true/false
Action: Normalize to boolean

IDs:
Google Sheets: Numbers (1, 2, 3)
PostgreSQL: Prefixed (STU-001, T-01, CRS-001)
Action: Generate new IDs with prefix
```

### New Fields

```
Add to all tables:
- created_at (TIMESTAMP)
- updated_at (TIMESTAMP)

Add to specific tables:
- status (active/inactive)
- is_deleted (soft delete)
- metadata (JSONB for flexible data)
```

---

## 📊 Success Metrics

### Must Have (Go/No-Go)

```
✓ 100% row count match
✓ 0 foreign key violations
✓ All critical flows working
✓ < 5 min downtime
✓ < 1% error rate
```

### Nice to Have

```
○ < 2x response time vs Google Sheets
○ All minor issues documented
○ Team trained on new system
○ Monitoring dashboards ready
```

---

## 🎓 Lessons Learned (Post-Migration)

### What Went Well

```
(Fill after migration)
```

### What Went Wrong

```
(Fill after migration)
```

### What to Improve

```
(Fill after migration)
```

---

**Last Updated:** 2025-10-19  
**Version:** 1.0.0