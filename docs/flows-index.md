# 👥 User Flows Overview

## 📋 Table of Contents

- [Introduction](#introduction)
- [LINE Rich Menu Design](#line-rich-menu-design)
- [Role Summary](#role-summary)
- [Detailed Flow Documents](#detailed-flow-documents)
- [Common Patterns](#common-patterns)
- [Interaction Guidelines](#interaction-guidelines)

---

## 🎯 Introduction

ระบบมี **4 กลุ่มผู้ใช้งานหลัก** ผ่าน LINE Official Account:

```
1. 👨‍🏫 Teacher    - ครูผู้สอน (8 คน)
2. 👨‍👩‍👧 Parent     - ผู้ปกครอง (120+ คน)
3. 👶 Student    - นักเรียน (156+ คน)
4. 🏢 Admin      - ผู้บริหาร (2-3 คน)
```

**Total Users:** 286+ active users

---

## 📱 LINE Rich Menu Design

### Design Specifications

```
Dimensions: 2500 x 1686 px
Format: PNG or JPEG
Max Size: 1 MB
Layout: 6 buttons (2x3 grid)
```

### Color Palette

```
Teacher Colors:
├── Primary: Blue #4A90E2
├── Secondary: Green #7ED321
├── Accent: Orange #F5A623
└── Neutral: Gray #4A4A4A

Parent Colors:
├── Primary: Pink #FF6B9D
├── Secondary: Gold #F8E71C
├── Accent: Blue #4A90E2
└── Neutral: Gray #4A4A4A

Student Colors:
├── Primary: Purple #9013FE
├── Secondary: Green #7ED321
├── Accent: Orange #F5A623
└── Fun: Gold #F8E71C

Admin Colors:
├── Primary: Dark Blue #1A237E
├── Secondary: Red #D32F2F
├── Accent: Green #388E3C
└── Neutral: Gray #424242
```

---

## 👨‍🏫 Teacher Summary

### Menu: เช็คชื่อ | บันทึกการสอน | การบ้าน | นักเรียน | ตารางสอน | แชท

**6 Core Functions:**

```
1. เช็คชื่อ (Attendance)
   - เลือกคลาส → แสดงรายชื่อ → เลือกคนขาด
   - บันทึกระบบ → แจ้งผู้ปกครองอัตโนมัติ
   - สรุปสถิติ

2. บันทึกการสอน (Teaching Log)
   - Step-by-step: หัวข้อ → กิจกรรม → การบ้าน → เตรียมครั้งหน้า
   - บันทึกระบบ → ส่งการบ้านให้นักเรียนอัตโนมัติ

3. ให้คะแนนการบ้าน (Grading)
   - แสดงงานรอตรวจ
   - ดูงาน → ให้คะแนน → Feedback
   - แจ้งนักเรียนอัตโนมัติ

4. ดูข้อมูลนักเรียน
   - เลือกคลาส → เลือกนักเรียน
   - ดูสถิติการเรียน, ความก้าวหน้า

5. ดูตารางสอน
   - แสดงตารางทั้งสัปดาห์
   - ดูรายละเอียดแต่ละคลาส

6. ติดต่อผู้ปกครอง
   - ส่งข้อความถึงผู้ปกครอง
```

**👉 [ดูรายละเอียด: flows-teacher.md](flows-teacher.md)**

---

## 👨‍👩‍👧 Parent Summary

### Menu: ข้อมูลลูก | ชำระเงิน | ผลการเรียน | ตารางเรียน | เวลาคงเหลือ | ติดต่อครู

**7 Core Functions:**

```
1. ดูข้อมูลลูก
   - ข้อมูลทั่วไป
   - คอร์สที่เรียน (2+ คอร์ส)
   - สรุปผลการเรียน

2. ชำระเงิน (พร้อม OCR)
   - เลือกรายการ → เลือกแพ็คเกจ
   - แสดงข้อมูลธนาคาร
   - Upload สลิป → OCR อัตโนมัติ
   - Admin approve → Generate receipt PDF
   - ส่ง LINE + Email

3. ดูผลการเรียน
   - เลือกวิชา → ดูคะแนนล่าสุด
   - ความเห็นครู
   - กราฟความก้าวหน้า

4. ดูตารางเรียน
   - แสดงตารางสัปดาห์นี้
   - ตั้งเตือน, ขอเลื่อนคาบ

5. ตรวจเวลาคงเหลือ
   - Course-based: คาบที่เหลือ
   - Hour-based: ชั่วโมงคงเหลือ
   - คาบถัดไป

6. ติดต่อครู
   - เลือกครู → ส่งข้อความ

7. ดาวน์โหลดใบเสร็จ
   - รายการใบเสร็จทั้งหมด
   - Download PDF, ส่งอีเมล
```

**👉 [ดูรายละเอียด: flows-parent.md](flows-parent.md)**

---

## 👶 Student Summary

### Menu: ตาราง | การบ้าน | คะแนน | เกม | ครู | รางวัล

**6 Core Functions:**

```
1. ดูตารางเรียน
   - แสดงตารางสัปดาห์นี้
   - รายละเอียดแต่ละคาบ

2. ส่งการบ้าน
   - ดูการบ้านค้าง
   - Upload รูปงาน → ส่ง
   - รอครูตรวจ → ได้คะแนน

3. ดูคะแนน
   - คะแนนล่าสุด
   - คะแนนเฉลี่ย
   - กราฟความก้าวหน้า
   - อันดับในห้อง

4. เล่นเกม (Gamification)
   - เกมการเรียนรู้
   - รับแต้ม
   - Leaderboard

5. ติดต่อครู
   - ถามคำถาม
   - ขอความช่วยเหลือ

6. รางวัล (Rewards)
   - ดูแต้มที่มี
   - แลกของรางวัล
   - Badges ที่ได้
```

**👉 [ดูรายละเอียด: flows-student.md](flows-student.md)**

---

## 🏢 Admin Summary

### Access: Web Dashboard + MCP Server (Claude Desktop)

**7 Core Functions:**

```
1. Dashboard Overview
   - ภาพรวมวันนี้
   - รายได้, ค่าใช้จ่าย, กำไร
   - สถิตินักเรียน

2. Payment Approval
   - ดูสลิปรอตรวจ
   - ตรวจสอบ OCR
   - อนุมัติ/ปฏิเสธ
   - Generate receipt

3. Analytics Dashboard
   - รายได้-รายจ่าย (กราฟ)
   - สถิตินักเรียน
   - ประสิทธิภาพครู

4. Expense Tracking
   - บันทึกค่าใช้จ่าย
   - จัดหมวดหมู่
   - รายงาน

5. Broadcast Management
   - ส่งประกาศ
   - เลือกกลุ่มเป้าหมาย
   - ตั้งเวลาส่ง

6. Generate Reports
   - รายงานการเงิน
   - รายงานนักเรียน
   - Export PDF/Excel

7. MCP Server (via Claude Desktop)
   - สรุปรายได้
   - ตรวจสอบค้างชำระ
   - อนุมัติผ่าน Claude
```

**👉 [ดูรายละเอียด: flows-admin.md](flows-admin.md)**

---

## 🔄 Common Patterns

### Role Detection

```
User เปิด LINE OA
    ↓
System: Query em_users WHERE line_user_id = 'Uxxxxx'
    ↓
Get role (teacher/parent/student/admin)
    ↓
Display appropriate Rich Menu
```

### Message Flow

```
User: Send message/Click button
    ↓
LINE Platform → Cloudflare Worker
    ↓
Verify signature → Forward to FastAPI
    ↓
Detect role → Route to handler
    ↓
Process request → Query database
    ↓
Generate response (Text/Flex Message)
    ↓
Send via LINE Messaging API
    ↓
User receives response
```

### Data Update Pattern

```
User performs action
    ↓
Update database (em_* tables)
    ↓
Trigger related actions:
    - Notify related users
    - Update statistics
    - Log activity
    ↓
Send confirmation to user
```

---

## 💬 Interaction Guidelines

### Conversational Style

```
✓ Natural Thai language
✓ Friendly and professional
✓ Clear instructions
✓ Use emojis appropriately (not too much)
✓ Step-by-step for complex tasks
✓ Always confirm before important actions
```

### Response Format

```
Simple Info:
→ Text message (concise)

Complex Data:
→ Flex Message (structured, visual)

Lists:
→ Numbered + Quick Reply buttons

Forms:
→ Step-by-step conversational
```

### Quick Reply Usage

```
Use Quick Reply for:
✓ Yes/No questions
✓ Multiple choice (3-10 options)
✓ Common actions
✓ Navigation shortcuts

Don't use for:
✗ More than 13 options
✗ Free-form input needed
✗ Complex choices
```

### Flex Message Usage

```
Use Flex Message for:
✓ Receipts (structured data)
✓ Timetables (calendar view)
✓ Progress reports (visual)
✓ Rich content (images + text)

Template Types:
- Bubble (single card)
- Carousel (multiple cards)
- Receipt (payment summary)
- Calendar (schedule view)
```

---

## 🎨 UX Best Practices

### Visual Hierarchy

```
1. Title (Bold, large)
2. Key info (Highlighted)
3. Details (Normal)
4. Actions (Buttons at bottom)
```

### Information Density

```
Optimal:
- 3-5 key points per message
- 2-4 actions per Flex Message
- Max 10 lines of text

Avoid:
- Walls of text
- Too many options
- Nested menus > 3 levels
```

### Loading States

```
Long operations (>3 seconds):
→ Send "กำลังประมวลผล..." message
→ Show progress if possible
→ Update when complete
```

### Error Handling

```
User-friendly messages:
✓ "ขออภัยค่ะ ไม่พบข้อมูล กรุณาลองใหม่"
✓ "เกิดข้อผิดพลาด กรุณาติดต่อแอดมิน"

NOT:
✗ "Error 500: Internal Server Error"
✗ "NullPointerException at line 123"
```

---

## 📊 Flow Statistics

### Complexity Analysis

```
Role        Flows    Avg Steps    DB Tables    External APIs
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
Teacher     6        5-8          6            1 (LINE)
Parent      7        4-10         8            3 (LINE, OCR, Email)
Student     6        3-5          4            1 (LINE)
Admin       7        3-8          12           2 (LINE, Email)
```

### Integration Points

```
All Roles:
- LINE Messaging API (send/receive)
- PostgreSQL (data storage)
- Redis (session/cache)

Parent Specific:
- OCR API (slip recognition)
- Email API (receipt delivery)
- PDF Generator (receipts)

Admin Specific:
- MCP Server (Claude Desktop)
- Analytics Engine
- Report Generator
```

---

## 🔗 Related Documentation

- [Database Schema](../database/README.md) - ดูโครงสร้างตาราง
- [API Documentation](../API.md) - ดู API endpoints
- [Architecture](../ARCHITECTURE.md) - ดูสถาปัตยกรรมระบบ

---

## 📝 Notes for Developers

### Implementation Priority

```
Phase 1 (Week 13-16): Teacher + Parent Basic
- เช็คชื่อ
- บันทึกการสอน
- ดูข้อมูลลูก
- ชำระเงิน (basic)

Phase 2 (Week 17-20): Student + Admin Basic
- ส่งการบ้าน
- ดูคะแนน
- Dashboard
- Payment approval

Phase 3 (Week 21-24): Advanced Features
- Gamification
- Advanced analytics
- Flex Messages
- OCR
```

### Testing Checklist

```
For each flow:
✓ Happy path works
✓ Error handling
✓ Edge cases (empty data, duplicates)
✓ Performance (response <2s)
✓ LINE message format correct
✓ Database transactions atomic
✓ Notifications sent properly
```

---

**Last Updated:** 2025-10-19  
**Version:** 1.0.0