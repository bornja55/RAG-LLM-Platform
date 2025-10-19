# 👥 User Flows Documentation

## 📋 Table of Contents

- [Overview](#overview)
- [Teacher Flows](#-teacher-flows)
- [Parent Flows](#-parent-flows)
- [Student Flows](#-student-flows)
- [Admin Flows](#-admin-flows)
- [General Platform Flows](#-general-platform-flows)
- [LINE Rich Menu Design](#-line-rich-menu-design)

---

## 🎯 Overview

เอกสารนี้อธิบาย User Flows ทั้งหมดในระบบ แบ่งเป็น 4 กลุ่มผู้ใช้หลัก:

1. 👨‍🏫 **Teacher** - ครูผู้สอน
2. 👨‍👩‍👧 **Parent** - ผู้ปกครอง
3. 👶 **Student** - นักเรียน
4. 🏢 **Admin** - ผู้ดูแลระบบ

---

## 👨‍🏫 Teacher Flows

### Entry Point

```
ครูเปิด LINE OA
    ↓
System ตรวจสอบ line_user_id
    ↓
Query: SELECT role FROM em_users WHERE line_user_id = 'Uxxxxx'
    ↓
role = 'teacher'
    ↓
แสดง Rich Menu สำหรับครู
```

### Rich Menu Layout (Teacher)

```
┌─────────────────────────────────────┐
│         [English Mania 🎓]          │
├─────────────────┬───────────────────┤
│  📋 เช็คชื่อ      │  📝 บันทึกการสอน   │
├─────────────────┼───────────────────┤
│  📚 การบ้าน      │  👥 นักเรียน       │
├─────────────────┼───────────────────┤
│  📅 ตารางสอน     │  💬 แชท           │
└─────────────────┴───────────────────┘
```

---

### Flow 1: เช็คชื่อ (Attendance)

```
User: กด "📋 เช็คชื่อ"
    ↓
Bot: ดึงคลาสที่สอนวันนี้
    Query: SELECT s.* FROM schedules s
           JOIN teachers t ON s.teacher_id = t.teacher_id
           WHERE t.line_user_id = 'Uxxxxx'
           AND s.day_of_week = 'saturday'
           AND s.is_active = true
    ↓
Bot: "คุณครูสอนคลาสวันนี้:
     1. ENG-PRIM-A1 (เสาร์ 10:00-12:00) - 8 คน
     2. ENG-PRIM-A2 (เสาร์ 14:00-16:00) - 6 คน
     
     พิมพ์หมายเลขคลาสที่จะเช็คชื่อ:"
    ↓
User: "1"
    ↓
Bot: ดึงรายชื่อนักเรียน
    Query: SELECT s.student_id, s.student_name, s.nickname
           FROM em_students s
           JOIN em_enrollments e ON s.student_id = e.student_id
           WHERE e.course_id = 'ENG-PRIM-A1'
           AND e.status = 'active'
    ↓
Bot: แสดงรายชื่อ (Quick Reply แบบ interactive)
    
    "รายชื่อนักเรียน ENG-PRIM-A1 (8 คน)
     
     ✅ 1. น้องมิ้น (STU-001)
     ✅ 2. น้องปอนด์ (STU-002)
     ✅ 3. น้องเฟิร์น (STU-003)
     ✅ 4. น้องพีช (STU-004)
     ✅ 5. น้องมะม่วง (STU-005)
     ✅ 6. น้องส้ม (STU-006)
     ✅ 7. น้องกล้วย (STU-007)
     ✅ 8. น้องแอปเปิ้ล (STU-008)
     
     กดหมายเลขคนที่ขาด (คั่นด้วยเว้นวรรค)
     หรือกด 'เช็คเสร็จ' ถ้าครบทุกคน"
    
    [Quick Reply: "เช็คเสร็จ"]
    ↓
User: "2 5" (น้องปอนด์และน้องมะม่วงขาด)
    ↓
Bot: บันทึกการเข้าเรียน
    
    For each student:
    INSERT INTO em_attendance (
        attendance_id, schedule_id, student_id,
        class_date, status, checked_by, checked_at
    ) VALUES (
        'ATT-20251019-001', 'SCH-001', 'STU-001',
        '2025-10-19', 'present', 'Uxxxxx', NOW()
    );
    
    For absent students (2, 5):
    INSERT INTO em_attendance (...) VALUES (..., 'absent', ...);
    ↓
Bot: ส่งการแจ้งเตือนไปยังผู้ปกครอง
    
    For student_id IN ('STU-002', 'STU-005'):
    SELECT parent_line_id FROM em_students
    WHERE student_id IN ('STU-002', 'STU-005');
    
    Send LINE message to parents:
    "สวัสดีค่ะคุณ [parent_name]
     น้อง[nickname] ขาดเรียนวันนี้ (19 ต.ค. 68)
     คลาส: ENG-PRIM-A1 (10:00-12:00)
     
     หากมีเหตุจำเป็น กรุณาแจ้งครูล่วงหน้าค่ะ"
    ↓
Bot: สรุปการเช็คชื่อ
    
    "✅ บันทึกเช็คชื่อสำเร็จ!
     
     📊 สรุป ENG-PRIM-A1 (19 ต.ค. 68)
     • มา: 6 คน (75%)
     • ขาด: 2 คน (25%)
       - น้องปอนด์ (แจ้งผู้ปกครองแล้ว ✓)
       - น้องมะม่วง (แจ้งผู้ปกครองแล้ว ✓)
     
     มีคลาสอื่นที่ต้องเช็คชื่อไหมคะ?"
    
    [Quick Reply: "เช็คคลาสอื่น" | "เสร็จแล้ว"]
```

---

### Flow 2: บันทึกการสอน (Teaching Log)

```
User: กด "📝 บันทึกการสอน"
    ↓
Bot: "เลือกคลาสที่สอนวันนี้:
     1. ENG-PRIM-A1 (เสาร์ 10:00-12:00)
     2. ENG-PRIM-A2 (เสาร์ 14:00-16:00)"
    ↓
User: "1"
    ↓
Bot: เริ่มการบันทึกแบบ Step-by-Step
    
    "📝 บันทึกการสอน ENG-PRIM-A1
     
     1️⃣ หัวข้อที่สอนวันนี้:"
    ↓
User: "Phonics - Long vowels (ai, ay, ee, ea)"
    ↓
Bot: "2️⃣ กิจกรรมที่ทำในคลาส:"
    ↓
User: "- เล่นเกม matching sounds
      - อ่านหนังสือสั้น 'Rain in May'
      - ฝึกเขียนคำ"
    ↓
Bot: "3️⃣ การบ้านที่มอบหมาย:"
    ↓
User: "แบบฝึกหัด หน้า 15-17 ครบทุกข้อ
      กำหนดส่ง: พรุ่งนี้ (20 ต.ค.)"
    ↓
Bot: "4️⃣ สิ่งที่ต้องเตรียมครับครั้งหน้า:"
    ↓
User: "Flash cards สำหรับ blending practice
      Worksheets เพิ่ม"
    ↓
Bot: "5️⃣ หมายเหตุเกี่ยวกับนักเรียน (ถ้ามี):"
    ↓
User: "น้องปอนด์ขาดเรียน ต้อง catch up
      น้องมิ้นเก่งมาก อ่านคล่อง"
    ↓
Bot: บันทึกข้อมูล
    
    INSERT INTO em_teaching_logs (
        log_id, schedule_id, teacher_id, class_date,
        topic, activities, homework_assigned,
        preparation_next, student_notes, created_by
    ) VALUES (
        'LOG-20251019-001', 'SCH-001', 'T-01', '2025-10-19',
        'Phonics - Long vowels (ai, ay, ee, ea)',
        '- เล่นเกม matching sounds...',
        'แบบฝึกหัด หน้า 15-17...',
        'Flash cards สำหรับ blending practice...',
        'น้องปอนด์ขาดเรียน...',
        'Uxxxxx'
    );
    ↓
Bot: "✅ บันทึกสำเร็จ!
     
     📋 สรุปการสอน ENG-PRIM-A1 (19 ต.ค. 68)
     • หัวข้อ: Phonics - Long vowels
     • การบ้าน: แบบฝึกหัด หน้า 15-17
     • กำหนดส่ง: 20 ต.ค. 68
     
     ต้องการส่งการบ้านให้นักเรียนไหมคะ?"
    
    [Quick Reply: "ส่งเลย" | "ไว้ทีหลัง"]
    ↓
User: "ส่งเลย"
    ↓
Bot: สร้างการบ้านสำหรับนักเรียนทุกคน
    
    For each student in class:
    INSERT INTO em_homework (
        homework_id, log_id, student_id, course_id,
        title, description, assigned_date, due_date,
        status
    ) VALUES (
        'HW-20251019-001', 'LOG-20251019-001', 'STU-001', 'ENG-PRIM-A1',
        'Phonics - Long vowels',
        'แบบฝึกหัด หน้า 15-17 ครบทุกข้อ',
        '2025-10-19', '2025-10-20',
        'pending'
    );
    ↓
Bot: ส่ง LINE message ถึงนักเรียนทุกคน
    
    "📚 การบ้านใหม่!
     
     วิชา: ENG-PRIM-A1
     หัวข้อ: Phonics - Long vowels
     
     📝 งาน: แบบฝึกหัด หน้า 15-17 ครบทุกข้อ
     📅 กำหนดส่ง: พรุ่งนี้ (20 ต.ค.)
     
     กดปุ่ม 'ส่งการบ้าน' เมื่อทำเสร็จนะคะ"
    
    [Quick Reply: "ส่งการบ้าน"]
    ↓
Bot: "✅ ส่งการบ้านให้นักเรียน 8 คนแล้ว
     (ยกเว้นน้องปอนด์ที่ขาดเรียน)
     
     บันทึกการสอนเรียบร้อย! 🎉"
```

---

### Flow 3: ให้คะแนนการบ้าน

```
User: กด "📚 การบ้าน"
    ↓
Bot: "📚 การบ้านที่ต้องตรวจ:
     
     🔴 รอตรวจ (5 งาน)
     1. น้องมิ้น - Phonics Long vowels (ส่งแล้ว)
     2. น้องเฟิร์น - Phonics Long vowels (ส่งแล้ว)
     3. น้องพีช - Phonics Long vowels (ส่งแล้ว)
     4. น้องส้ม - Phonics Long vowels (ส่งแล้ว)
     5. น้องแอปเปิ้ล - Phonics Long vowels (ส่งแล้ว)
     
     🟡 ยังไม่ส่ง (2 งาน)
     6. น้องกล้วย - Phonics Long vowels (เกินกำหนด 1 วัน)
     7. น้องปอนด์ - ขาดเรียน
     
     ✅ ตรวจแล้ว (1 งาน)
     
     พิมพ์หมายเลขที่จะตรวจ:"
    ↓
User: "1"
    ↓
Bot: ดึงการบ้าน + ไฟล์ที่ส่ง
    
    SELECT h.*, s.student_name, s.nickname
    FROM em_homework h
    JOIN em_students s ON h.student_id = s.student_id
    WHERE h.homework_id = 'HW-20251019-001';
    ↓
Bot: แสดงการบ้านพร้อมรูป
    
    "📝 การบ้าน - น้องมิ้น
     
     วิชา: ENG-PRIM-A1
     หัวข้อ: Phonics - Long vowels
     งาน: แบบฝึกหัด หน้า 15-17
     
     ส่งเมื่อ: 20 ต.ค. 10:30 (ตรงเวลา ✓)
     
     [รูปที่ส่ง - คลิกดูใหญ่]
     
     ให้คะแนนเท่าไหร่? (0-10):"
    ↓
User: "9"
    ↓
Bot: "เกรด: (A, B+, B, C+, C, D+, D, F):"
    ↓
User: "A"
    ↓
Bot: "💬 Feedback ให้น้องมิ้น:"
    ↓
User: "เก่งมากค่ะ! ทำครบทุกข้อ เข้าใจหลักการแล้ว 
      ต่อไปฝึกเขียนให้สวยขึ้นอีกนิดนะคะ"
    ↓
Bot: บันทึกคะแนน
    
    UPDATE em_homework
    SET score = 9,
        grade = 'A',
        feedback = 'เก่งมากค่ะ!...',
        graded_by = 'Uxxxxx',
        graded_at = NOW(),
        status = 'graded'
    WHERE homework_id = 'HW-20251019-001';
    ↓
Bot: ส่งผลไปยังนักเรียน
    
    LINE message to student:
    "🎉 คะแนนการบ้านออกแล้ว!
     
     📝 Phonics - Long vowels
     📊 คะแนน: 9/10 (A)
     
     💬 ความเห็นจากครู:
     เก่งมากค่ะ! ทำครบทุกข้อ เข้าใจหลักการแล้ว 
     ต่อไปฝึกเขียนให้สวยขึ้นอีกนิดนะคะ
     
     เยี่ยมมาก! 👏"
    ↓
Bot: "✅ ให้คะแนนเรียบร้อย!
     
     คะแนนเฉลี่ยของน้องมิ้น: 8.7/10
     
     เหลืออีก 4 งานรอตรวจ
     ต้องการตรวจต่อไหมคะ?"
    
    [Quick Reply: "ตรวจต่อ" | "เสร็จแล้ว"]
```

---

## 👨‍👩‍👧 Parent Flows

### Rich Menu Layout (Parent)

```
┌─────────────────────────────────────┐
│         [English Mania 🎓]          │
├─────────────────┬───────────────────┤
│  👶 ข้อมูลลูก     │  💰 ชำระเงิน       │
├─────────────────┼───────────────────┤
│  📊 ผลการเรียน    │  📅 ตารางเรียน     │
├─────────────────┼───────────────────┤
│  ⏰ เวลาคงเหลือ   │  💬 ติดต่อครู      │
└─────────────────┴───────────────────┘
```

---

### Flow 1: ตรวจเวลาคงเหลือ

```
User: กด "⏰ เวลาคงเหลือ"
    ↓
Bot: ดึงข้อมูลลูก
    
    SELECT s.student_id, s.student_name, s.nickname
    FROM em_students s
    WHERE s.parent_line_id = 'Uxxxxx'
    AND s.status = 'active';
    ↓
Bot: (ถ้ามีลูกหลายคน)
    "เลือกลูก:
     1. น้องมิ้น (STU-001)
     2. น้องแมว (STU-010)"
    ↓
User: "1"
    ↓
Bot: ดึงข้อมูลการลงทะเบียน
    
    SELECT e.*, c.course_name
    FROM em_enrollments e
    JOIN courses c ON e.course_id = c.course_id
    WHERE e.student_id = 'STU-001'
    AND e.status = 'active'
    ORDER BY e.created_at DESC;
    ↓
Bot: แสดงเวลาคงเหลือ (Flex Message)
    
    "👶 น้องมิ้น (STU-001)
     
     📚 คอร์ส ENG-PRIM-A1 (Phonics & Reading)
     ━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
     • ประเภท: รายคอร์ส (ไม่หักชั่วโมง)
     • เรียนไปแล้ว: 3/12 คาบ
     • คาบที่เหลือ: 9 คาบ
     • คาบถัดไป: เสาร์ 26 ต.ค. 10:00-12:00
     • สถานะการชำระ: ✅ ชำระเต็มจำนวนแล้ว
     
     📚 คอร์ส STEM-SUN-EX
     ━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
     • ประเภท: หักชั่วโมง
     • ซื้อไว้: 20 ชั่วโมง
     • ใช้ไป: 6 ชั่วโมง
     • เหลือ: 14 ชั่วโมง ⚠️
     • คาบถัดไป: อาทิตย์ 27 ต.ค. 13:00-15:00
     • สถานะ: ✅ พร้อมเรียน
     
     💡 หากต้องการซื้อเวลาเพิ่ม กด '💰 ชำระเงิน'"
    
    [Quick Reply: "💰 ชำระเงิน" | "📊 ดูผลเรียน"]
```

---

### Flow 2: ชำระเงิน (Payment with OCR)

```
User: กด "💰 ชำระเงิน"
    ↓
Bot: "💰 ชำระเงิน
     
     เลือกรายการ:
     1. ชำระค่าคอร์ส ENG-PRIM-A2 (เทอม 2) - 8,500 บาท
     2. ซื้อเวลาเพิ่ม STEM-SUN-EX
     3. ชำระค่าเรียนค้างชำระ - 0 บาท
     4. อื่นๆ"
    ↓
User: "2" (ซื้อเวลาเพิ่ม)
    ↓
Bot: ดึงแพ็คเกจราคา
    
    SELECT * FROM pricing
    WHERE pricing_type = 'hourly'
    AND is_active = true
    ORDER BY hours_included ASC;
    ↓
Bot: "💰 ซื้อเวลาเพิ่ม STEM-SUN-EX
     
     เลือกแพ็คเกจ:
     1. 10 ชั่วโมง - 3,500 บาท
     2. 20 ชั่วโมง - 6,500 บาท 🔥 (ประหยัด 500 บาท)
     3. 30 ชั่วโมง - 9,000 บาท 🔥 (ประหยัด 1,500 บาท)"
    ↓
User: "2"
    ↓
Bot: สรุปรายการ (Flex Message)
    
    "📋 สรุปรายการชำระเงิน
     ━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
     รายการ: STEM-SUN-EX (20 ชม.)
     จำนวนเงิน: 7,000 บาท
     ส่วนลด: -500 บาท
     ━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
     ยอดชำระ: 6,500 บาท
     
     📱 โอนเงินมาที่:
     🏦 ธนาคาร: ไทยพาณิชย์
     💳 เลขบัญชี: 123-4-56789-0
     👤 ชื่อบัญชี: English Mania Co., Ltd.
     
     หลังโอนแล้ว กรุณาถ่ายรูปสลิปและส่งมาให้"
    ↓
User: [ส่งรูปสลิป]
    ↓
Bot: บันทึกข้อมูล + OCR
    
    // Save slip image to MinIO
    slip_url = upload_to_minio(image)
    
    // OCR extraction
    ocr_result = extract_slip_data(image)
    {
        "amount": 6500,
        "date": "2025-10-19",
        "time": "14:35",
        "from_account": "xxx-x-xxxxx-x",
        "to_account": "123-4-56789-0"
    }
    
    // Create payment record
    INSERT INTO em_payments (
        payment_id, student_id, payment_type, amount,
        payment_date, payment_method, slip_url, ocr_data,
        status, created_at
    ) VALUES (
        'PAY-20251019-001', 'STU-001', 'add_hours', 6500,
        '2025-10-19', 'bank_transfer', 'https://minio.../slip.jpg',
        '{"amount": 6500, "date": "2025-10-19", ...}',
        'pending', NOW()
    );
    ↓
Bot: "✅ ได้รับสลิปแล้ว!
     
     🔍 กำลังตรวจสอบ...
     
     📋 รายละเอียดที่ตรวจพบ:
     • วันที่โอน: 19 ต.ค. 68
     • เวลา: 14:35 น.
     • จำนวนเงิน: 6,500 บาท ✅
     
     สถานะ: ⏳ รอตรวจสอบ
     
     ⏰ ใช้เวลาตรวจสอบประมาณ 10-30 นาที
     จะแจ้งเตือนเมื่อเสร็จสิ้นค่ะ"
    
    // Notify admin
    Send LINE to admin:
    "💰 มีสลิปใหม่รอตรวจสอบ
     
     ลูกค้า: น้องมิ้น (STU-001)
     ผู้ปกครอง: คุณสมหญิง
     จำนวน: 6,500 บาท
     
     [ดูสลิป] [อนุมัติ] [ปฏิเสธ]"
    
    ↓
[10 นาทีต่อมา - Admin อนุมัติ]
    
    UPDATE em_payments
    SET status = 'approved',
        approved_by = 'Uadmin',
        approved_at = NOW()
    WHERE payment_id = 'PAY-20251019-001';
    
    // Update enrollment
    UPDATE em_enrollments
    SET total_hours = total_hours + 20,
        amount_paid = amount_paid + 6500
    WHERE student_id = 'STU-001'
    AND course_id = 'STEM-SUN-EX';
    
    // Generate receipt PDF
    receipt_url = generate_receipt_pdf(payment)
    receipt_number = 'REC-2025-00123'
    
    UPDATE em_payments
    SET receipt_url = receipt_url,
        receipt_number = receipt_number,
        receipt_generated_at = NOW()
    WHERE payment_id = 'PAY-20251019-001';
    ↓
Bot: ส่งข้อความยืนยัน
    
    "🎉 ชำระเงินสำเร็จ!
     
     💰 เพิ่มเวลาเรียน STEM-SUN-EX
     • เวลาเดิม: 14 ชั่วโมง
     • เพิ่ม: +20 ชั่วโมง
     • รวมเป็น: 34 ชั่วโมง
     
     📄 ใบเสร็จเลขที่: REC-2025-00123
     [ดาวน์โหลด PDF]
     
     📧 ส่งอีเมล: parent@email.com ✅
     
     ขอบคุณที่ไว้วางใจ English Mania ค่ะ 💚"
    
    // Also send email with receipt
    send_email(
        to: "parent@email.com",
        subject: "ใบเสร็จชำระเงิน - English Mania",
        attachment: receipt_pdf
    );
```

---

### Flow 3: ดูผลการเรียน

```
User: กด "📊 ผลเรียน"
    ↓
Bot: แสดงผลการเรียน (Flex Message สวยงาม)
    
    "📊 ผลการเรียน - น้องมิ้น
     
     📚 ENG-PRIM-A1 (Phonics & Reading)
     ━