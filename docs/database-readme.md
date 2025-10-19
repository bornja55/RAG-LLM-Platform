# 🗄️ Database Schema Documentation

## 📋 Table of Contents

- [Overview](#overview)
- [Database Architecture](#database-architecture)
- [Universal Platform Tables](#universal-platform-tables)
- [English Mania Tables](#english-mania-tables)
- [Shared Tables](#shared-tables)
- [Indexes & Performance](#indexes--performance)
- [Migrations](#migrations)
- [Data Flow](#data-flow)

---

## 🎯 Overview

ระบบใช้ **PostgreSQL 16** เป็น main database แบ่งออกเป็น 3 กลุ่มตาราง:

1. **Universal Platform Tables** - สำหรับ multi-tenant RAG platform
2. **English Mania Tables** - สำหรับระบบโรงเรียน
3. **Shared Tables** - ตารางร่วมกัน (courses, schedules, teachers, etc.)

### Database Statistics

```
Total Tables: 28
Total Indexes: 45+
Storage Engine: PostgreSQL 16
Encoding: UTF8
Collation: en_US.UTF-8
Extensions: uuid-ossp, pgcrypto
```

---

## 🏗️ Database Architecture

### ER Diagram (High-Level)

```
┌─────────────────────────────────────────────────────────────┐
│                  Universal Platform                         │
├─────────────────────────────────────────────────────────────┤
│                                                             │
│  organizations (1) ──< workspaces (N)                       │
│                          │                                  │
│                          ├──< database_connections          │
│                          ├──< documents                     │
│                          └──< chat_threads ──< messages     │
│                                                             │
│  users ──< workspace_members                                │
│                                                             │
└─────────────────────────────────────────────────────────────┘

┌─────────────────────────────────────────────────────────────┐
│                    English Mania                            │
├─────────────────────────────────────────────────────────────┤
│                                                             │
│  em_users (LINE users)                                      │
│      │                                                      │
│      ├──< em_students (1:N parents can have N students)     │
│      │        │                                             │
│      │        ├──< em_enrollments ──< em_payments           │
│      │        ├──< em_attendance                            │
│      │        └──< em_homework                              │
│      │                                                      │
│      └──< em_teaching_logs (teachers)                       │
│                                                             │
│  em_expenses, em_broadcasts (admin)                         │
│                                                             │
└─────────────────────────────────────────────────────────────┘

┌─────────────────────────────────────────────────────────────┐
│                      Shared Tables                          │
├─────────────────────────────────────────────────────────────┤
│                                                             │
│  courses, schedules, teachers, pricing, faq, policies       │
│                                                             │
└─────────────────────────────────────────────────────────────┘
```

---

## 🏢 Universal Platform Tables

### 1. organizations

องค์กรหลักในระบบ (Multi-tenant isolation)

```sql
CREATE TABLE organizations (
    id UUID PRIMARY KEY DEFAULT uuid_generate_v4(),
    name VARCHAR(255) NOT NULL,
    slug VARCHAR(100) UNIQUE NOT NULL,
    domain VARCHAR(255),
    logo_url TEXT,
    settings JSONB DEFAULT '{}',
    status VARCHAR(50) DEFAULT 'active',
    created_at TIMESTAMP DEFAULT NOW(),
    updated_at TIMESTAMP DEFAULT NOW()
);

CREATE INDEX idx_organizations_slug ON organizations(slug);
CREATE INDEX idx_organizations_status ON organizations(status);
```

**Example:**
```json
{
  "id": "123e4567-e89b-12d3-a456-426614174000",
  "name": "Acme Corporation",
  "slug": "acme-corp",
  "domain": "acme.com",
  "logo_url": "https://cdn.example.com/logos/acme.png",
  "settings": {
    "max_workspaces": 10,
    "max_users": 100,
    "features": ["sql_agent", "document_rag", "line_bot"]
  },
  "status": "active"
}
```

---

### 2. workspaces

Workspace ภายในองค์กร (แผนก/ทีม/โปรเจค)

```sql
CREATE TABLE workspaces (
    id UUID PRIMARY KEY DEFAULT uuid_generate_v4(),
    organization_id UUID NOT NULL REFERENCES organizations(id) ON DELETE CASCADE,
    name VARCHAR(255) NOT NULL,
    slug VARCHAR(100) NOT NULL,
    description TEXT,
    settings JSONB DEFAULT '{}',
    status VARCHAR(50) DEFAULT 'active',
    created_at TIMESTAMP DEFAULT NOW(),
    updated_at TIMESTAMP DEFAULT NOW(),
    UNIQUE(organization_id, slug)
);

CREATE INDEX idx_workspaces_org ON workspaces(organization_id);
CREATE INDEX idx_workspaces_status ON workspaces(status);
```

**Example:**
```json
{
  "id": "223e4567-e89b-12d3-a456-426614174001",
  "organization_id": "123e4567-e89b-12d3-a456-426614174000",
  "name": "Sales Department",
  "slug": "sales",
  "description": "Sales team workspace",
  "settings": {
    "llm_provider": "gemini",
    "llm_model": "gemini-1.5-pro",
    "max_documents": 1000
  }
}
```

---

### 3. users

ผู้ใช้งานระบบ

```sql
CREATE TABLE users (
    id UUID PRIMARY KEY DEFAULT uuid_generate_v4(),
    email VARCHAR(255) UNIQUE NOT NULL,
    password_hash VARCHAR(255),
    line_user_id VARCHAR(100) UNIQUE,
    full_name VARCHAR(255),
    display_name VARCHAR(100),
    phone_number VARCHAR(20),
    avatar_url TEXT,
    email_verified BOOLEAN DEFAULT FALSE,
    is_active BOOLEAN DEFAULT TRUE,
    is_superuser BOOLEAN DEFAULT FALSE,
    last_login_at TIMESTAMP,
    created_at TIMESTAMP DEFAULT NOW(),
    updated_at TIMESTAMP DEFAULT NOW()
);

CREATE INDEX idx_users_email ON users(email);
CREATE INDEX idx_users_line_id ON users(line_user_id);
CREATE INDEX idx_users_active ON users(is_active);
```

---

### 4. workspace_members

สมาชิกใน workspace (Many-to-Many)

```sql
CREATE TABLE workspace_members (
    id UUID PRIMARY KEY DEFAULT uuid_generate_v4(),
    workspace_id UUID NOT NULL REFERENCES workspaces(id) ON DELETE CASCADE,
    user_id UUID NOT NULL REFERENCES users(id) ON DELETE CASCADE,
    role VARCHAR(50) NOT NULL, -- admin, manager, member, viewer
    permissions JSONB DEFAULT '{}',
    joined_at TIMESTAMP DEFAULT NOW(),
    UNIQUE(workspace_id, user_id)
);

CREATE INDEX idx_workspace_members_workspace ON workspace_members(workspace_id);
CREATE INDEX idx_workspace_members_user ON workspace_members(user_id);
CREATE INDEX idx_workspace_members_role ON workspace_members(role);
```

**Roles:**
- `admin` - Full access, manage members
- `manager` - Manage data, no member management
- `member` - Read/write access
- `viewer` - Read-only access

---

### 5. database_connections

การเชื่อมต่อ database ลูกค้า

```sql
CREATE TABLE database_connections (
    id UUID PRIMARY KEY DEFAULT uuid_generate_v4(),
    workspace_id UUID NOT NULL REFERENCES workspaces(id) ON DELETE CASCADE,
    name VARCHAR(255) NOT NULL,
    db_type VARCHAR(50) NOT NULL, -- mssql, mysql, postgresql
    host VARCHAR(255) NOT NULL,
    port INTEGER NOT NULL,
    database_name VARCHAR(255) NOT NULL,
    username VARCHAR(255) NOT NULL,
    password_encrypted TEXT NOT NULL, -- AES-256 encrypted
    ssl_enabled BOOLEAN DEFAULT FALSE,
    is_read_only BOOLEAN DEFAULT TRUE,
    status VARCHAR(50) DEFAULT 'active',
    last_connected_at TIMESTAMP,
    metadata JSONB DEFAULT '{}',
    created_by UUID REFERENCES users(id),
    created_at TIMESTAMP DEFAULT NOW(),
    updated_at TIMESTAMP DEFAULT NOW()
);

CREATE INDEX idx_db_conn_workspace ON database_connections(workspace_id);
CREATE INDEX idx_db_conn_status ON database_connections(status);
```

**Security:**
- Password encrypted using AES-256
- Encryption key stored in environment variable
- Read-only mode enforced by default

---

### 6. documents

เอกสารที่ upload

```sql
CREATE TABLE documents (
    id UUID PRIMARY KEY DEFAULT uuid_generate_v4(),
    workspace_id UUID NOT NULL REFERENCES workspaces(id) ON DELETE CASCADE,
    filename VARCHAR(255) NOT NULL,
    original_filename VARCHAR(255) NOT NULL,
    file_type VARCHAR(50) NOT NULL, -- docx, pdf, xlsx, txt, csv, md
    file_size BIGINT NOT NULL, -- bytes
    mime_type VARCHAR(100),
    storage_path TEXT NOT NULL, -- MinIO path
    storage_bucket VARCHAR(100) DEFAULT 'rag-documents',
    processed BOOLEAN DEFAULT FALSE,
    processing_status VARCHAR(50) DEFAULT 'pending', -- pending, processing, completed, failed
    chunk_count INTEGER DEFAULT 0,
    metadata JSONB DEFAULT '{}',
    uploaded_by UUID REFERENCES users(id),
    created_at TIMESTAMP DEFAULT NOW(),
    updated_at TIMESTAMP DEFAULT NOW()
);

CREATE INDEX idx_documents_workspace ON documents(workspace_id);
CREATE INDEX idx_documents_processed ON documents(processed);
CREATE INDEX idx_documents_status ON documents(processing_status);
CREATE INDEX idx_documents_type ON documents(file_type);
```

---

### 7. document_chunks

ชิ้นข้อมูลที่แบ่งจากเอกสาร

```sql
CREATE TABLE document_chunks (
    id UUID PRIMARY KEY DEFAULT uuid_generate_v4(),
    document_id UUID NOT NULL REFERENCES documents(id) ON DELETE CASCADE,
    chunk_index INTEGER NOT NULL,
    content TEXT NOT NULL,
    token_count INTEGER,
    embedding_id VARCHAR(100), -- Qdrant point ID
    metadata JSONB DEFAULT '{}',
    created_at TIMESTAMP DEFAULT NOW(),
    UNIQUE(document_id, chunk_index)
);

CREATE INDEX idx_chunks_document ON document_chunks(document_id);
CREATE INDEX idx_chunks_embedding ON document_chunks(embedding_id);
```

---

### 8. chat_threads

บทสนทนา

```sql
CREATE TABLE chat_threads (
    id UUID PRIMARY KEY DEFAULT uuid_generate_v4(),
    workspace_id UUID NOT NULL REFERENCES workspaces(id) ON DELETE CASCADE,
    user_id UUID NOT NULL REFERENCES users(id) ON DELETE CASCADE,
    title VARCHAR(255),
    channel VARCHAR(50) DEFAULT 'web', -- web, line, api
    llm_provider VARCHAR(50) DEFAULT 'gemini',
    llm_model VARCHAR(100) DEFAULT 'gemini-1.5-pro',
    metadata JSONB DEFAULT '{}',
    created_at TIMESTAMP DEFAULT NOW(),
    updated_at TIMESTAMP DEFAULT NOW()
);

CREATE INDEX idx_threads_workspace ON chat_threads(workspace_id);
CREATE INDEX idx_threads_user ON chat_threads(user_id);
CREATE INDEX idx_threads_channel ON chat_threads(channel);
CREATE INDEX idx_threads_updated ON chat_threads(updated_at DESC);
```

---

### 9. messages

ข้อความในบทสนทนา

```sql
CREATE TABLE messages (
    id UUID PRIMARY KEY DEFAULT uuid_generate_v4(),
    thread_id UUID NOT NULL REFERENCES chat_threads(id) ON DELETE CASCADE,
    role VARCHAR(50) NOT NULL, -- user, assistant, system
    content TEXT NOT NULL,
    tokens_used INTEGER,
    context_sources JSONB DEFAULT '[]', -- Array of source references
    metadata JSONB DEFAULT '{}',
    created_at TIMESTAMP DEFAULT NOW()
);

CREATE INDEX idx_messages_thread ON messages(thread_id);
CREATE INDEX idx_messages_created ON messages(created_at);
```

**Context Sources Example:**
```json
{
  "context_sources": [
    {
      "type": "document",
      "document_id": "abc-123",
      "filename": "sales_policy.pdf",
      "chunk_index": 3,
      "relevance_score": 0.92
    },
    {
      "type": "sql",
      "query": "SELECT SUM(amount) FROM sales WHERE date = '2025-10-19'",
      "result_count": 1
    }
  ]
}
```

---

## 🎓 English Mania Tables

### 1. em_users

ผู้ใช้งาน LINE OA (role-based)

```sql
CREATE TABLE em_users (
    line_user_id VARCHAR(100) PRIMARY KEY,
    role VARCHAR(50) NOT NULL DEFAULT 'customer', -- customer, parent, student, teacher, admin
    status VARCHAR(50) DEFAULT 'active',
    display_name VARCHAR(255),
    phone_number VARCHAR(20),
    email VARCHAR(255),
    registered_at TIMESTAMP DEFAULT NOW(),
    last_active_at TIMESTAMP,
    linked_student_id VARCHAR(50), -- for parents & students
    linked_teacher_id VARCHAR(50), -- for teachers
    notify_preference JSONB DEFAULT '{"line": true, "email": false}',
    language VARCHAR(10) DEFAULT 'th',
    metadata JSONB DEFAULT '{}'
);

CREATE INDEX idx_em_users_role ON em_users(role);
CREATE INDEX idx_em_users_status ON em_users(status);
CREATE INDEX idx_em_users_student ON em_users(linked_student_id);
CREATE INDEX idx_em_users_teacher ON em_users(linked_teacher_id);
```

**Roles:**
- `customer` - ผู้สนใจทั่วไป
- `parent` - ผู้ปกครอง
- `student` - นักเรียน
- `teacher` - ครู
- `admin` - ผู้ดูแลระบบ

---

### 2. em_students

ข้อมูลนักเรียน

```sql
CREATE TABLE em_students (
    student_id VARCHAR(50) PRIMARY KEY,
    student_name VARCHAR(255) NOT NULL,
    nickname VARCHAR(100),
    date_of_birth DATE,
    parent_line_id VARCHAR(100) REFERENCES em_users(line_user_id),
    parent_name VARCHAR(255),
    parent_phone VARCHAR(20),
    parent_email VARCHAR(255),
    level VARCHAR(50), -- beginner, elementary, intermediate, advanced
    school VARCHAR(255),
    grade VARCHAR(20),
    interests TEXT,
    learning_style VARCHAR(50),
    notes TEXT,
    status VARCHAR(50) DEFAULT 'active',
    registered_at TIMESTAMP DEFAULT NOW(),
    updated_at TIMESTAMP DEFAULT NOW()
);

CREATE INDEX idx_em_students_parent ON em_students(parent_line_id);
CREATE INDEX idx_em_students_status ON em_students(status);
CREATE INDEX idx_em_students_level ON em_students(level);
```

---

### 3. em_enrollments

การลงทะเบียนเรียน

```sql
CREATE TABLE em_enrollments (
    enrollment_id VARCHAR(50) PRIMARY KEY,
    student_id VARCHAR(50) NOT NULL REFERENCES em_students(student_id),
    course_id VARCHAR(50) NOT NULL REFERENCES courses(course_id),
    enrollment_type VARCHAR(50) NOT NULL, -- course_based, hour_based
    start_date DATE NOT NULL,
    end_date DATE,
    
    -- For course-based
    total_sessions INTEGER,
    completed_sessions INTEGER DEFAULT 0,
    remaining_sessions INTEGER GENERATED ALWAYS AS (total_sessions - completed_sessions) STORED,
    
    -- For hour-based
    total_hours DECIMAL(10,2),
    used_hours DECIMAL(10,2) DEFAULT 0,
    remaining_hours DECIMAL(10,2) GENERATED ALWAYS AS (total_hours - used_hours) STORED,
    
    -- Payment
    total_amount DECIMAL(10,2) NOT NULL,
    amount_paid DECIMAL(10,2) DEFAULT 0,
    amount_due DECIMAL(10,2) GENERATED ALWAYS AS (total_amount - amount_paid) STORED,
    payment_status VARCHAR(50) DEFAULT 'pending', -- pending, partial, paid
    
    status VARCHAR(50) DEFAULT 'active', -- active, completed, cancelled, suspended
    notes TEXT,
    created_at TIMESTAMP DEFAULT NOW(),
    updated_at TIMESTAMP DEFAULT NOW()
);

CREATE INDEX idx_em_enrollments_student ON em_enrollments(student_id);
CREATE INDEX idx_em_enrollments_course ON em_enrollments(course_id);
CREATE INDEX idx_em_enrollments_status ON em_enrollments(status);
CREATE INDEX idx_em_enrollments_payment ON em_enrollments(payment_status);
```

---

### 4. em_attendance

การเข้าเรียน

```sql
CREATE TABLE em_attendance (
    attendance_id VARCHAR(50) PRIMARY KEY,
    schedule_id VARCHAR(50) NOT NULL REFERENCES schedules(schedule_id),
    student_id VARCHAR(50) NOT NULL REFERENCES em_students(student_id),
    class_date DATE NOT NULL,
    status VARCHAR(50) NOT NULL, -- present, absent, late, excused
    hours_attended DECIMAL(4,2), -- for hour-based courses
    checked_by VARCHAR(100) REFERENCES em_users(line_user_id),
    checked_at TIMESTAMP,
    notes TEXT,
    created_at TIMESTAMP DEFAULT NOW(),
    UNIQUE(schedule_id, student_id, class_date)
);

CREATE INDEX idx_em_attendance_schedule ON em_attendance(schedule_id);
CREATE INDEX idx_em_attendance_student ON em_attendance(student_id);
CREATE INDEX idx_em_attendance_date ON em_attendance(class_date DESC);
CREATE INDEX idx_em_attendance_status ON em_attendance(status);
```

---

### 5. em_teaching_logs

บันทึกการสอนของครู

```sql
CREATE TABLE em_teaching_logs (
    log_id VARCHAR(50) PRIMARY KEY,
    schedule_id VARCHAR(50) NOT NULL REFERENCES schedules(schedule_id),
    teacher_id VARCHAR(50) NOT NULL REFERENCES teachers(teacher_id),
    class_date DATE NOT NULL,
    topic VARCHAR(500) NOT NULL,
    activities TEXT,
    homework_assigned TEXT,
    preparation_next TEXT,
    student_notes TEXT, -- notes about individual students
    overall_notes TEXT,
    created_by VARCHAR(100) REFERENCES em_users(line_user_id),
    created_at TIMESTAMP DEFAULT NOW(),
    updated_at TIMESTAMP DEFAULT NOW(),
    UNIQUE(schedule_id, class_date)
);

CREATE INDEX idx_em_logs_schedule ON em_teaching_logs(schedule_id);
CREATE INDEX idx_em_logs_teacher ON em_teaching_logs(teacher_id);
CREATE INDEX idx_em_logs_date ON em_teaching_logs(class_date DESC);
```

---

### 6. em_homework

การบ้าน

```sql
CREATE TABLE em_homework (
    homework_id VARCHAR(50) PRIMARY KEY,
    log_id VARCHAR(50) REFERENCES em_teaching_logs(log_id) ON DELETE SET NULL,
    student_id VARCHAR(50) NOT NULL REFERENCES em_students(student_id),
    course_id VARCHAR(50) REFERENCES courses(course_id),
    title VARCHAR(500) NOT NULL,
    description TEXT,
    assigned_date DATE NOT NULL,
    due_date DATE NOT NULL,
    
    -- Submission
    submission_url TEXT,
    submission_type VARCHAR(50), -- photo, file, text
    submitted_at TIMESTAMP,
    
    -- Grading
    grade VARCHAR(10), -- A, B+, B, C+, C, D+, D, F
    score DECIMAL(5,2), -- 0-10
    max_score DECIMAL(5,2) DEFAULT 10,
    feedback TEXT,
    graded_by VARCHAR(100) REFERENCES em_users(line_user_id),
    graded_at TIMESTAMP,
    
    status VARCHAR(50) DEFAULT 'pending', -- pending, submitted, graded, late, excused
    created_at TIMESTAMP DEFAULT NOW(),
    updated_at TIMESTAMP DEFAULT NOW()
);

CREATE INDEX idx_em_homework_student ON em_homework(student_id);
CREATE INDEX idx_em_homework_course ON em_homework(course_id);
CREATE INDEX idx_em_homework_status ON em_homework(status);
CREATE INDEX idx_em_homework_due ON em_homework(due_date);
CREATE INDEX idx_em_homework_log ON em_homework(log_id);
```

---

### 7. em_payments

การชำระเงิน

```sql
CREATE TABLE em_payments (
    payment_id VARCHAR(50) PRIMARY KEY,
    student_id VARCHAR(50) NOT NULL REFERENCES em_students(student_id),
    enrollment_id VARCHAR(50) REFERENCES em_enrollments(enrollment_id),
    payment_type VARCHAR(50) NOT NULL, -- enrollment, add_hours, outstanding
    amount DECIMAL(10,2) NOT NULL,
    payment_date DATE NOT NULL,
    payment_method VARCHAR(50) DEFAULT 'bank_transfer',
    
    -- Slip upload
    slip_url TEXT,
    slip_uploaded_at TIMESTAMP,
    ocr_data JSONB, -- OCR extracted data
    
    -- Receipt
    receipt_number VARCHAR(50) UNIQUE,
    receipt_url TEXT,
    receipt_generated_at TIMESTAMP,
    
    -- Approval
    status VARCHAR(50) DEFAULT 'pending', -- pending, approved, rejected
    approved_by VARCHAR(100) REFERENCES em_users(line_user_id),
    approved_at TIMESTAMP,
    rejection_reason TEXT,
    
    notes TEXT,
    created_at TIMESTAMP DEFAULT NOW(),
    updated_at TIMESTAMP DEFAULT NOW()
);

CREATE INDEX idx_em_payments_student ON em_payments(student_id);
CREATE INDEX idx_em_payments_enrollment ON em_payments(enrollment_id);
CREATE INDEX idx_em_payments_status ON em_payments(status);
CREATE INDEX idx_em_payments_date ON em_payments(payment_date DESC);
CREATE INDEX idx_em_payments_receipt ON em_payments(receipt_number);
```

---

### 8. em_expenses

ค่าใช้จ่ายของโรงเรียน

```sql
CREATE TABLE em_expenses (
    expense_id VARCHAR(50) PRIMARY KEY,
    expense_date DATE NOT NULL,
    category VARCHAR(100) NOT NULL, -- salary, rent, utilities, materials, marketing, transportation, other
    subcategory VARCHAR(100),
    description TEXT NOT NULL,
    amount DECIMAL(10,2) NOT NULL,
    payment_method VARCHAR(50),
    
    -- Receipt
    receipt_url TEXT,
    receipt_number VARCHAR(50),
    
    -- Approval
    approved_by VARCHAR(100) REFERENCES em_users(line_user_id),
    approved_at TIMESTAMP,
    
    tags TEXT[], -- Array of tags for grouping
    notes TEXT,
    created_by VARCHAR(100) REFERENCES em_users(line_user_id),
    created_at TIMESTAMP DEFAULT NOW(),
    updated_at TIMESTAMP DEFAULT NOW()
);

CREATE INDEX idx_em_expenses_date ON em_expenses(expense_date DESC);
CREATE INDEX idx_em_expenses_category ON em_expenses(category);
CREATE INDEX idx_em_expenses_tags ON em_expenses USING GIN(tags);
```

---

### 9. em_broadcasts

ส่งข้อความ broadcast

```sql
CREATE TABLE em_broadcasts (
    broadcast_id VARCHAR(50) PRIMARY KEY,
    title VARCHAR(255) NOT NULL,
    message TEXT NOT NULL,
    message_type VARCHAR(50) DEFAULT 'text', -- text, flex, image, video
    flex_message JSONB, -- Flex Message template
    
    -- Targeting
    target_role VARCHAR(50), -- all, parent, student, teacher
    target_course_id VARCHAR(50),
    target_custom JSONB, -- Custom targeting rules
    
    -- Scheduling
    scheduled_at TIMESTAMP,
    sent_at TIMESTAMP,
    
    -- Stats
    recipient_count INTEGER DEFAULT 0,
    sent_count INTEGER DEFAULT 0,
    delivered_count INTEGER DEFAULT 0,
    failed_count INTEGER DEFAULT 0,
    
    status VARCHAR(50) DEFAULT 'draft', -- draft, scheduled, sending, sent, failed
    sent_by VARCHAR(100) REFERENCES em_users(line_user_id),
    created_at TIMESTAMP DEFAULT NOW(),
    updated_at TIMESTAMP DEFAULT NOW()
);

CREATE INDEX idx_em_broadcasts_status ON em_broadcasts(status);
CREATE INDEX idx_em_broadcasts_scheduled ON em_broadcasts(scheduled_at);
CREATE INDEX idx_em_broadcasts_sent ON em_broadcasts(sent_at DESC);
```

---

## 📚 Shared Tables

### 1. courses

คอร์สเรียน

```sql
CREATE TABLE courses (
    course_id VARCHAR(50) PRIMARY KEY,
    course_code VARCHAR(20) UNIQUE NOT NULL,
    course_name VARCHAR(255) NOT NULL,
    course_name_en VARCHAR(255),
    description TEXT,
    category VARCHAR(100), -- english, stem, coding, art
    level VARCHAR(50), -- beginner, elementary, intermediate, advanced
    age_group VARCHAR(50), -- 3-5, 6-8, 9-12, 13-15, 16+
    duration_type VARCHAR(50), -- course_based, hour_based
    total_sessions INTEGER,
    hours_per_session DECIMAL(4,2),
    max_students INTEGER DEFAULT 8,
    min_students INTEGER DEFAULT 3,
    is_active BOOLEAN DEFAULT TRUE,
    created_at TIMESTAMP DEFAULT NOW(),
    updated_at TIMESTAMP DEFAULT NOW()
);

CREATE INDEX idx_courses_code ON courses(course_code);
CREATE INDEX idx_courses_category ON courses(category);
CREATE INDEX idx_courses_level ON courses(level);
CREATE INDEX idx_courses_active ON courses(is_active);
```

---

### 2. schedules

ตารางเรียน

```sql
CREATE TABLE schedules (
    schedule_id VARCHAR(50) PRIMARY KEY,
    course_id VARCHAR(50) NOT NULL REFERENCES courses(course_id),
    teacher_id VARCHAR(50) REFERENCES teachers(teacher_id),
    day_of_week VARCHAR(20) NOT NULL, -- monday, tuesday, wednesday, ...
    start_time TIME NOT NULL,
    end_time TIME NOT NULL,
    room VARCHAR(50),
    location VARCHAR(100),
    max_students INTEGER DEFAULT 8,
    current_students INTEGER DEFAULT 0,
    is_active BOOLEAN DEFAULT TRUE,
    start_date DATE,
    end_date DATE,
    notes TEXT,
    created_at TIMESTAMP DEFAULT NOW(),
    updated_at TIMESTAMP DEFAULT NOW()
);

CREATE INDEX idx_schedules_course ON schedules(course_id);
CREATE INDEX idx_schedules_teacher ON schedules(teacher_id);
CREATE INDEX idx_schedules_day ON schedules(day_of_week);
CREATE INDEX idx_schedules_active ON schedules(is_active);
```

---

### 3. teachers

ข้อมูลครู

```sql
CREATE TABLE teachers (
    teacher_id VARCHAR(50) PRIMARY KEY,
    teacher_code VARCHAR(20) UNIQUE,
    full_name VARCHAR(255) NOT NULL,
    nickname VARCHAR(100),
    email VARCHAR(255),
    phone VARCHAR(20),
    line_user_id VARCHAR(100) REFERENCES em_users(line_user_id),
    specialization TEXT[], -- Array of specializations
    education TEXT,
    experience_years INTEGER,
    bio TEXT,
    photo_url TEXT,
    is_active BOOLEAN DEFAULT TRUE,
    joined_date DATE,
    created_at TIMESTAMP DEFAULT NOW(),
    updated_at TIMESTAMP DEFAULT NOW()
);

CREATE INDEX idx_teachers_code ON teachers(teacher_code);
CREATE INDEX idx_teachers_line ON teachers(line_user_id);
CREATE INDEX idx_teachers_active ON teachers(is_active);
```

---

### 4. pricing

ราคาคอร์ส

```sql
CREATE TABLE pricing (
    pricing_id VARCHAR(50) PRIMARY KEY,
    course_id VARCHAR(50) REFERENCES courses(course_id),
    pricing_type VARCHAR(50) NOT NULL, -- course, hourly, package
    name VARCHAR(255) NOT NULL,
    description TEXT,
    
    -- Pricing
    base_price DECIMAL(10,2) NOT NULL,
    discount_percent DECIMAL(5,2) DEFAULT 0,
    final_price DECIMAL(10,2) GENERATED ALWAYS AS (base_price * (1 - discount_percent/100)) STORED,
    
    -- For packages
    hours_included DECIMAL(6,2),
    sessions_included INTEGER,
    
    valid_from DATE,
    valid_to DATE,
    is_active BOOLEAN DEFAULT TRUE,
    created_at TIMESTAMP DEFAULT NOW(),
    updated_at TIMESTAMP DEFAULT NOW()
);

CREATE INDEX idx_pricing_course ON pricing(course_id);
CREATE INDEX idx_pricing_type ON pricing(pricing_type);
CREATE INDEX idx_pricing_active ON pricing(is_active);
```

---

## 🔍 Indexes & Performance

### Primary Indexes (Automatically Created)

```sql
-- All PRIMARY KEY constraints automatically create indexes
-- All UNIQUE constraints automatically create indexes
```

### Foreign Key Indexes

```sql
-- Already created in table definitions above
-- Example: idx_workspaces_org, idx_documents_workspace, etc.
```

### Performance Indexes

```sql
-- Composite indexes for common queries
CREATE INDEX idx_messages_thread_created ON messages(thread_id, created_at DESC);
CREATE INDEX idx_documents_workspace_status ON documents(workspace_id, processing_status);
CREATE INDEX idx_em_homework_student_status ON em_homework(student_id, status);
CREATE INDEX idx_em_attendance_student_date ON em_attendance(student_id, class_date DESC);

-- Full-text search indexes
CREATE INDEX idx_documents_filename_fts ON documents USING GIN(to_tsvector('english', filename));
CREATE INDEX idx_courses_name_fts ON courses USING GIN(to_tsvector('english', course_name));

-- JSONB indexes
CREATE INDEX idx_organizations_settings ON organizations USING GIN(settings);
CREATE INDEX idx_messages_context ON messages USING GIN(context_sources);
```

---

## 🔄 Migrations

### Alembic Migration Setup

```bash
# Location
backend/alembic/

# Files
├── alembic.ini
├── env.py
└── versions/
    ├── 001_initial_schema.py
    ├── 002_add_english_mania.py
    └── ...
```

### Running Migrations

```bash
# Upgrade to latest
alembic upgrade head

# Downgrade one version
alembic downgrade -1

# Show current version
alembic current

# Show migration history
alembic history
```

### Migration from Google Sheets

See: [migrations.md](migrations.md) for detailed migration guide

---

## 📊 Data Flow Examples

### Document Upload Flow

```
User uploads → MinIO storage → documents table
    → Celery job → Process & chunk → document_chunks
    → Generate embeddings → Qdrant (vectors)
    → Update processed=true
```

### Query Processing Flow

```
User query → Classify type
    → If SQL: database_connections → Execute query
    → If Doc: Qdrant search → Retrieve chunks
    → Build context → LLM → Response
    → Save to messages table
```

### Payment Approval Flow

```
Parent uploads slip → em_payments (pending)
    → OCR extract data → Update ocr_data
    → Notify admin → Admin reviews
    → Approve → Update enrollment amount_paid
    → Generate receipt PDF → Send LINE + Email
```

---

## 🔐 Security & Best Practices

### Row-Level Security (RLS)

```sql
-- Enable RLS on sensitive tables
ALTER TABLE documents ENABLE ROW LEVEL SECURITY;

-- Policy: Users can only see documents in their workspaces
CREATE POLICY documents_workspace_policy ON documents
    FOR SELECT
    USING (
        workspace_id IN (
            SELECT workspace_id FROM workspace_members
            WHERE user_id = current_user_id()
        )
    );
```

### Data Encryption

- Database passwords: AES-256 encrypted
- API keys: Stored in environment variables
- User passwords: bcrypt hashed
- At rest: PostgreSQL encryption (optional)

### Backup Strategy

```bash
# Daily full backup
pg_dump -h localhost -U raguser ragdb > backup_$(date +%Y%m%d).sql

# Continuous archiving (WAL)
# See: DEPLOYMENT.md for full backup strategy
```

---

## 📚 Additional Resources

- [Migration Guide](migrations.md) - Google Sheets → PostgreSQL
- [Query Examples](queries.md) - Common SQL queries
- [Performance Tuning](performance.md) - Optimization tips
- [Backup & Restore](backup.md) - Backup procedures

---

**Last Updated:** 2025-10-19  
**Version:** 1.0.0