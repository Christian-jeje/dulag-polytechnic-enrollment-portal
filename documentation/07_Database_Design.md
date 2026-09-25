# 07 Database Design

## Database Overview

The Dulag Polytechnic Enrollment System uses MySQL/MariaDB as its relational database management system. The database name is `polytechnicportal` and contains 14 primary tables plus supporting tables for configuration and logging.

**Database Characteristics**:
- **Type**: Relational Database (MySQL 5.7+)
- **Collation**: UTF-8 General Collation (supports Filipino characters)
- **Normalization**: 3rd Normal Form
- **Total Tables**: 14+ core tables
- **Relationships**: Foreign key constraints maintain referential integrity

---

## Core Database Tables (ERD)

```
                            ┌─────────────────┐
                            │     users       │
                            ├─────────────────┤
                            │ id (PK)         │
                            │ name            │
                            │ email (UNIQUE)  │
                            │ password        │
                            │ role            │ ◄────┐
                            │ is_validated    │      │
                            │ batch_year      │      │
                            │ gender          │      │
                            │ last_known_ip   │      │
                            │ created_at      │      │
                            │ updated_at      │      │
                            └────────┬────────┘      │
                                     │               │
                    ┌────────────────┼───────┬───────┼─────────────┐
                    │                │       │       │             │
                    ▼                ▼       ▼       ▼             ▼
        ┌─────────────────────┐  ┌─────────────────┐  ┌────────────────────┐
        │ enrollments         │  │ application_    │  │ birth_certificates │
        ├─────────────────────┤  │ forms           │  ├────────────────────┤
        │ id (PK)             │  ├─────────────────┤  │ id (PK)            │
        │ user_id (FK)        │  │ id (PK)         │  │ user_id (FK)       │
        │ course_id (FK)  ────┼──┤ user_id (FK)    │  │ document_url       │
        │ batch_group_id (FK) │  │ work_experience │  │ extracted_data     │
        │ status              │  │ (JSON)          │  │ ai_analysis_result │
        │ submitted_at        │  │ trainings(JSON) │  │ verification_status│
        │ approved_at         │  │ licensure_exams │  │ verified_by        │
        │ admin_notes         │  │ (JSON)          │  │ verified_at        │
        │ reference_id        │  │ competency_     │  │ created_at         │
        │ uli                 │  │ assessments     │  │ updated_at         │
        │ created_at          │  │ (JSON)          │  └────────────────────┘
        │ updated_at          │  │ status          │
        └─────────┬───────────┘  │ submitted_at    │   ┌────────────────────┐
                  │               │ created_at      │   │ medical_certificates
                  │               │ updated_at      │   ├────────────────────┤
                  ▼               └─────────────────┘   │ id (PK)            │
        ┌─────────────────────┐                       │ user_id (FK)       │
        │     courses         │                       │ document_url       │
        ├─────────────────────┤                       │ extracted_data     │
        │ id (PK)             │                       │ verification_status│
        │ course_code         │                       │ verified_by        │
        │ course_title        │                       │ verified_at        │
        │ description         │                       │ created_at         │
        │ credits             │                       │ updated_at         │
        │ tuition_amount      │                       └────────────────────┘
        │ is_active           │
        │ capacity            │   ┌────────────────────────────┐
        │ created_at          │   │ transcripts_of_records     │
        │ updated_at          │   ├────────────────────────────┤
        └─────────────────────┘   │ id (PK)                    │
                                  │ user_id (FK)              │
        ┌─────────────────────┐   │ document_url               │
        │   batch_groups      │   │ extracted_grades (JSON)    │
        ├─────────────────────┤   │ verification_status        │
        │ id (PK)             │   │ verified_by                │
        │ batch_name          │   │ verified_at                │
        │ start_date          │   │ created_at                 │
        │ end_date            │   │ enrollment_start           │
        │ enrollment_start    │   │ enrollment_end             │
        │ enrollment_end      │   │ created_at                 │
        │ is_active           │   │ updated_at                 │
        │ created_at          │   └────────────────────────────┘
        │ updated_at          │
        └─────────────────────┘   ┌────────────────────────────┐
                                  │      payments              │
        ┌─────────────────────┐   ├────────────────────────────┤
        │ grades              │   │ id (PK)                    │
        ├─────────────────────┤   │ enrollment_id (FK)         │
        │ id (PK)             │   │ user_id (FK)               │
        │ enrollment_id (FK)  │   │ amount_paid                │
        │ user_id (FK)        │   │ payment_method             │
        │ course_id (FK)      │   │ payment_date               │
        │ grade               │   │ reference_number           │
        │ entered_by          │   │ receipt_number             │
        │ created_at          │   │ processed_by               │
        │ updated_at          │   │ notes                      │
        └─────────────────────┘   │ created_at                 │
                                  │ updated_at                 │
        ┌─────────────────────┐   └────────────────────────────┘
        │ notifications       │
        ├─────────────────────┤
        │ id (PK)             │
        │ user_id (FK)        │
        │ title               │
        │ message             │
        │ type                │
        │ is_read             │
        │ email_sent          │
        │ created_at          │
        └─────────────────────┘

        ┌─────────────────────┐
        │ activity_log        │
        ├─────────────────────┤
        │ id (PK)             │
        │ user_id (FK)        │
        │ action              │
        │ resource            │
        │ resource_id         │
        │ details             │
        │ ip_address          │
        │ user_agent          │
        │ created_at          │
        └─────────────────────┘

        ┌─────────────────────┐
        │ otp                 │
        ├─────────────────────┤
        │ id (PK)             │
        │ email               │
        │ code                │
        │ expires_at          │
        │ used                │
        │ created_at          │
        └─────────────────────┘
```

---

## Table Definitions

### 1. users

**Purpose**: Stores all user accounts (students, admins, super admins)

**Columns**:
| Column | Type | Constraints | Description |
|--------|------|-----------|-------------|
| id | INT | PK, AUTO_INCREMENT | Unique user identifier |
| name | VARCHAR(255) | NULL | Full name (populated after validation) |
| email | VARCHAR(255) | UNIQUE, NOT NULL | Email address (registration identifier) |
| password | VARCHAR(255) | NOT NULL | Hashed password (bcrypt) |
| role | ENUM | NOT NULL, DEFAULT='STUDENT' | User role: STUDENT, ADMIN, SUPERADMIN |
| gender | ENUM('M','F','Other') | NULL | Gender (from birth certificate) |
| batch_year | INT | NULL | Enrollment year/batch |
| is_validated | TINYINT | NOT NULL, DEFAULT=0 | 1 if identity verified, 0 if not |
| last_known_ip | VARCHAR(45) | NULL | IP address of last login (IPv4 & IPv6) |
| created_at | TIMESTAMP | NOT NULL, DEFAULT=NOW() | Account creation timestamp |
| updated_at | TIMESTAMP | NOT NULL, DEFAULT=NOW() | Last update timestamp |

**Indices**: email (UNIQUE), role, is_validated, created_at

**Relationships**: 
- One-to-Many: enrollments, application_forms, payments, notifications, activity_log

---

### 2. enrollments

**Purpose**: Tracks student course enrollments and their status through the approval process

**Columns**:
| Column | Type | Constraints | Description |
|--------|------|-----------|-------------|
| id | INT | PK, AUTO_INCREMENT | Unique enrollment identifier |
| user_id | INT | FK→users.id, NOT NULL | Student ID |
| course_id | INT | FK→courses.id, NOT NULL | Course being enrolled in |
| batch_group_id | INT | FK→batch_groups.id, NOT NULL | Enrollment batch/period |
| status | ENUM | NOT NULL, DEFAULT='pending_review' | pending_review, approved, rejected, additional_info_needed, enrolled, completed |
| submitted_at | TIMESTAMP | NOT NULL | When student submitted application |
| approved_at | TIMESTAMP | NULL | When admin approved (if approved) |
| admin_notes | TEXT | NULL | Admin notes/feedback during processing |
| reference_id | VARCHAR(50) | UNIQUE, NULL | System-generated reference for student |
| uli | VARCHAR(50) | NULL | Unique Learner Identifier (TESDA) |
| created_at | TIMESTAMP | NOT NULL, DEFAULT=NOW() | Record creation time |
| updated_at | TIMESTAMP | NOT NULL, DEFAULT=NOW() | Last update time |

**Indices**: user_id, course_id, batch_group_id, status, created_at, reference_id (UNIQUE)

**Relationships**:
- Many-to-One: users, courses, batch_groups
- One-to-Many: payments, grades

---

### 3. courses

**Purpose**: Stores course catalog and availability

**Columns**:
| Column | Type | Constraints | Description |
|--------|------|-----------|-------------|
| id | INT | PK, AUTO_INCREMENT | Unique course identifier |
| course_code | VARCHAR(50) | UNIQUE, NOT NULL | Internal course code (e.g., "CS101") |
| course_title | VARCHAR(255) | NOT NULL | Course name |
| description | TEXT | NULL | Course description and objectives |
| credits | DECIMAL(4,2) | NULL | Credit hours for the course |
| tuition_amount | DECIMAL(10,2) | NOT NULL | Tuition cost |
| is_active | TINYINT | NOT NULL, DEFAULT=1 | 1 = available, 0 = archived |
| capacity | INT | NULL | Maximum enrollment (NULL = unlimited) |
| prerequisites | JSON | NULL | Array of prerequisite course IDs |
| created_at | TIMESTAMP | NOT NULL, DEFAULT=NOW() | Creation time |
| updated_at | TIMESTAMP | NOT NULL, DEFAULT=NOW() | Last update time |

**Indices**: course_code (UNIQUE), is_active, created_at

**Relationships**:
- One-to-Many: enrollments, grades

---

### 4. batch_groups

**Purpose**: Defines enrollment periods/batches (e.g., "Spring 2026", "Fall 2026")

**Columns**:
| Column | Type | Constraints | Description |
|--------|------|-----------|-------------|
| id | INT | PK, AUTO_INCREMENT | Unique batch identifier |
| batch_name | VARCHAR(100) | NOT NULL | Display name (e.g., "Batch 2026-Spring") |
| start_date | DATE | NOT NULL | When enrollment/courses begin |
| end_date | DATE | NOT NULL | When enrollment/courses end |
| enrollment_start | DATETIME | NOT NULL | When enrollment applications open |
| enrollment_end | DATETIME | NOT NULL | When enrollment applications close |
| is_active | TINYINT | NOT NULL, DEFAULT=1 | 1 = open, 0 = closed |
| created_at | TIMESTAMP | NOT NULL, DEFAULT=NOW() | Creation time |
| updated_at | TIMESTAMP | NOT NULL, DEFAULT=NOW() | Last update time |

**Indices**: batch_name (UNIQUE), is_active, enrollment_start, enrollment_end

**Relationships**:
- One-to-Many: enrollments

---

### 5. application_forms

**Purpose**: Stores TESDA application form data including work experience and training history

**Columns**:
| Column | Type | Constraints | Description |
|--------|------|-----------|-------------|
| id | INT | PK, AUTO_INCREMENT | Unique form identifier |
| user_id | INT | FK→users.id, UNIQUE, NOT NULL | Student ID (one form per student) |
| work_experience | JSON | NULL | Array of work experience entries {company, position, duration, etc.} |
| trainings | JSON | NULL | Array of training programs {program_name, provider, duration, etc.} |
| licensure_exams | JSON | NULL | Array of licensure exams {exam_type, issue_date, expiry, etc.} |
| competency_assessments | JSON | NULL | Array of competency assessments {assessment_type, score, date, etc.} |
| status | ENUM | NOT NULL, DEFAULT='draft' | draft, submitted, under_review, approved, rejected |
| submitted_at | TIMESTAMP | NULL | When form was officially submitted |
| created_at | TIMESTAMP | NOT NULL, DEFAULT=NOW() | Creation time |
| updated_at | TIMESTAMP | NOT NULL, DEFAULT=NOW() | Last update time |

**Indices**: user_id (UNIQUE, FK), status, submitted_at

**Relationships**:
- Many-to-One: users (foreign key relationship)

---

### 6. birth_certificates

**Purpose**: Stores birth certificate document uploads and verification results

**Columns**:
| Column | Type | Constraints | Description |
|--------|------|-----------|-------------|
| id | INT | PK, AUTO_INCREMENT | Unique certificate record |
| user_id | INT | FK→users.id, NOT NULL | Student who submitted |
| document_url | VARCHAR(500) | NOT NULL | URL to file in Cloud Storage |
| extracted_data | JSON | NULL | AI-extracted data {name, dob, gender, id_number, etc.} |
| ai_analysis_result | JSON | NULL | AI analysis results {confidence, authenticity_score, warnings, etc.} |
| verification_status | ENUM | NOT NULL, DEFAULT='pending' | pending, approved, rejected, manual_review |
| verified_by | INT | FK→users.id, NULL | Admin user ID who verified |
| verified_at | TIMESTAMP | NULL | Verification completion timestamp |
| created_at | TIMESTAMP | NOT NULL, DEFAULT=NOW() | Upload time |
| updated_at | TIMESTAMP | NOT NULL, DEFAULT=NOW() | Last update |

**Indices**: user_id, verification_status, verified_at, created_at

**Relationships**:
- Many-to-One: users (for user_id and verified_by)

---

### 7. medical_certificates

**Purpose**: Stores medical clearance document uploads and verification

**Columns**:
| Column | Type | Constraints | Description |
|--------|------|-----------|-------------|
| id | INT | PK, AUTO_INCREMENT | Unique record |
| user_id | INT | FK→users.id, NOT NULL | Student who submitted |
| document_url | VARCHAR(500) | NOT NULL | File location in Cloud Storage |
| extracted_data | JSON | NULL | AI-extracted {clearance_status, date, expiry, notes, etc.} |
| verification_status | ENUM | NOT NULL, DEFAULT='pending' | pending, approved, rejected |
| verified_by | INT | FK→users.id, NULL | Admin who verified |
| verified_at | TIMESTAMP | NULL | Verification time |
| created_at | TIMESTAMP | NOT NULL, DEFAULT=NOW() | Upload time |
| updated_at | TIMESTAMP | NOT NULL, DEFAULT=NOW() | Last update |

**Indices**: user_id, verification_status, verified_at

**Relationships**:
- Many-to-One: users

---

### 8. transcripts_of_records

**Purpose**: Stores transcript documents and extracted grade information

**Columns**:
| Column | Type | Constraints | Description |
|--------|------|-----------|-------------|
| id | INT | PK, AUTO_INCREMENT | Unique record |
| user_id | INT | FK→users.id, NOT NULL | Student |
| document_url | VARCHAR(500) | NOT NULL | Transcript file in storage |
| extracted_grades | JSON | NULL | Courses and grades {course_name, grade, date, etc.} |
| verification_status | ENUM | NOT NULL, DEFAULT='pending' | pending, verified, rejected |
| verified_by | INT | FK→users.id, NULL | Admin verification |
| verified_at | TIMESTAMP | NULL | Verification timestamp |
| enrollment_start | DATE | NULL | Student's academic period start |
| enrollment_end | DATE | NULL | Student's academic period end |
| created_at | TIMESTAMP | NOT NULL | Upload time |
| updated_at | TIMESTAMP | NOT NULL | Last update |

**Indices**: user_id, verification_status

**Relationships**:
- Many-to-One: users

---

### 9. payments

**Purpose**: Records all payment transactions from students

**Columns**:
| Column | Type | Constraints | Description |
|--------|------|-----------|-------------|
| id | INT | PK, AUTO_INCREMENT | Unique payment record |
| enrollment_id | INT | FK→enrollments.id, NULL | Associated enrollment (if applicable) |
| user_id | INT | FK→users.id, NOT NULL | Student who made payment |
| amount_paid | DECIMAL(10,2) | NOT NULL | Amount received |
| payment_method | ENUM | NOT NULL | cash, check, bank_transfer, credit_card, debit_card |
| payment_date | DATE | NOT NULL | When payment was received |
| reference_number | VARCHAR(100) | NULL | External reference (transaction ID, check #, etc.) |
| receipt_number | VARCHAR(50) | UNIQUE, NOT NULL | System-generated receipt number |
| processed_by | INT | FK→users.id, NOT NULL | Admin who processed payment |
| notes | TEXT | NULL | Additional payment notes |
| created_at | TIMESTAMP | NOT NULL, DEFAULT=NOW() | When payment recorded |
| updated_at | TIMESTAMP | NOT NULL, DEFAULT=NOW() | Last update |

**Indices**: user_id, enrollment_id, payment_date, receipt_number (UNIQUE), created_at

**Relationships**:
- Many-to-One: users, enrollments, users (processed_by)

---

### 10. grades

**Purpose**: Stores student grades for completed courses

**Columns**:
| Column | Type | Constraints | Description |
|--------|------|-----------|-------------|
| id | INT | PK, AUTO_INCREMENT | Unique grade record |
| enrollment_id | INT | FK→enrollments.id, NOT NULL | Associated enrollment |
| user_id | INT | FK→users.id, NOT NULL | Student |
| course_id | INT | FK→courses.id, NOT NULL | Course grade is for |
| grade | DECIMAL(5,2) | NOT NULL | Numeric grade (0-100 or per scale) |
| entered_by | INT | FK→users.id, NOT NULL | Admin who entered grade |
| comment | TEXT | NULL | Optional notes on grade |
| created_at | TIMESTAMP | NOT NULL, DEFAULT=NOW() | When grade was entered |
| updated_at | TIMESTAMP | NOT NULL, DEFAULT=NOW() | When grade was last updated |

**Indices**: user_id, course_id, enrollment_id, created_at

**Relationships**:
- Many-to-One: enrollments, users, courses

---

### 11. notifications

**Purpose**: Stores system messages and notifications for users

**Columns**:
| Column | Type | Constraints | Description |
|--------|------|-----------|-------------|
| id | INT | PK, AUTO_INCREMENT | Unique notification |
| user_id | INT | FK→users.id, NOT NULL | Recipient user |
| title | VARCHAR(255) | NOT NULL | Notification subject |
| message | TEXT | NOT NULL | Notification body |
| type | ENUM | NOT NULL | enrollment, payment, document, system, other |
| is_read | TINYINT | NOT NULL, DEFAULT=0 | 1 = read, 0 = unread |
| email_sent | TINYINT | NOT NULL, DEFAULT=0 | 1 = emailed, 0 = not emailed |
| created_at | TIMESTAMP | NOT NULL, DEFAULT=NOW() | Creation time |

**Indices**: user_id, is_read, created_at

**Relationships**:
- Many-to-One: users

---

### 12. activity_log

**Purpose**: Complete audit trail of all system activities for compliance

**Columns**:
| Column | Type | Constraints | Description |
|--------|------|-----------|-------------|
| id | INT | PK, AUTO_INCREMENT | Unique log entry |
| user_id | INT | FK→users.id, NULL | User performing action |
| action | VARCHAR(100) | NOT NULL | Action performed (login, enrollment_created, payment_received, etc.) |
| resource | VARCHAR(100) | NOT NULL | Resource type (user, enrollment, payment, etc.) |
| resource_id | INT | NULL | ID of resource affected |
| details | JSON | NULL | Additional details {old_value, new_value, reason, etc.} |
| ip_address | VARCHAR(45) | NOT NULL | User's IP address |
| user_agent | VARCHAR(500) | NULL | Browser/client information |
| created_at | TIMESTAMP | NOT NULL, DEFAULT=NOW() | Timestamp of action |

**Indices**: user_id, resource, resource_id, action, created_at

**Relationships**:
- Many-to-One: users

---

### 13. otp (One-Time Passwords)

**Purpose**: Temporary OTP codes for email verification and password reset

**Columns**:
| Column | Type | Constraints | Description |
|--------|------|-----------|-------------|
| id | INT | PK, AUTO_INCREMENT | Unique OTP record |
| email | VARCHAR(255) | NOT NULL | Email address OTP sent to |
| code | VARCHAR(6) | NOT NULL | OTP code (usually 6 digits) |
| expires_at | TIMESTAMP | NOT NULL | When OTP expires (typically 10 minutes) |
| used | TINYINT | NOT NULL, DEFAULT=0 | 1 = already used, 0 = not yet used |
| created_at | TIMESTAMP | NOT NULL, DEFAULT=NOW() | When OTP was generated |

**Indices**: email, expires_at, used, created_at

**Relationships**: None (temporary data)

---

### 14. settings

**Purpose**: System configuration values

**Columns**:
| Column | Type | Constraints | Description |
|--------|------|-----------|-------------|
| id | INT | PK, AUTO_INCREMENT | Unique setting |
| config_key | VARCHAR(100) | UNIQUE, NOT NULL | Configuration key name |
| config_value | TEXT | NULL | Configuration value |
| data_type | ENUM | NOT NULL | string, integer, boolean, json |
| created_at | TIMESTAMP | NOT NULL | Creation time |
| updated_at | TIMESTAMP | NOT NULL | Last update |

**Indices**: config_key (UNIQUE)

**Relationships**: None

---

## Key Database Design Decisions

### 1. JSON Columns for Multi-Entry Data
**Why**: Application forms have multiple entries (work experience, trainings) without fixed structure
**Solution**: Store as JSON arrays for flexibility without creating excessive normalized tables
**Trade-off**: Slightly more complex queries but much simpler application logic

**Examples**:
```json
{
  "work_experience": [
    {"company": "ABC Corp", "position": "Analyst", "duration": "2 years"},
    {"company": "XYZ Ltd", "position": "Manager", "duration": "1.5 years"}
  ]
}
```

### 2. Soft Deletes Not Implemented (Instead: Status Flags)
**Why**: Regulatory compliance requires audit trails; better to keep and mark status
**Solution**: status column indicates draft/submitted/approved/rejected; no actual deletion
**Result**: Complete history maintained for auditing

### 3. Audit Trail Table (activity_log)
**Why**: Must track all system activities for compliance and troubleshooting
**Solution**: Every significant action logged with user, timestamp, IP, and details
**Coverage**: User logins, form submissions, approvals, payments, grade entries, admin changes

### 4. Separate Document Tables
**Why**: Different document types have different workflows and requirements
**Solution**: Separate tables for birth_certificates, medical_certificates, transcripts
**Benefit**: Flexible verification workflows, easy filtering, clear audit trail per document type

### 5. Payments and Enrollments Linked
**Why**: Payment must be traceable to specific enrollment
**Solution**: payment.enrollment_id foreign key to enrollments table
**Benefit**: Easy to determine payment status per enrollment

### 6. User Role Stored in Users Table
**Why**: Simple role-based access control
**Solution**: ENUM role column (STUDENT, ADMIN, SUPERADMIN) in users table
**Scalability**: If complex permissions needed, can migrate to role_permissions table

---

## Database Relationships

### One-to-Many Relationships
- `users` → `enrollments`: One student has many enrollments
- `users` → `payments`: One student makes many payments
- `users` → `application_forms`: One student submits one application (unique constraint)
- `users` → `notifications`: One user receives many notifications
- `users` → `activity_log`: One user performs many actions
- `courses` → `enrollments`: One course has many enrollments
- `batch_groups` → `enrollments`: One batch has many enrollments
- `enrollments` → `payments`: One enrollment may have many payments (partial/full)
- `enrollments` → `grades`: One enrollment links to one grade (for that course)

### Many-to-Many Relationships (Implicit through Enrollments)
- `users` ↔ `courses`: Through `enrollments` table

---

## Indexing Strategy

**Primary Keys** (All tables): Fast unique lookups

**Foreign Keys**: Enable efficient joins
- user_id (users)
- course_id (courses)
- batch_group_id (batch_groups)
- enrollment_id (enrollments)

**Search Columns**:
- email (users) - UNIQUE for login
- course_code (courses) - UNIQUE for course lookups
- reference_number (payments) - UNIQUE for payment tracking
- receipt_number (payments) - UNIQUE for receipt lookups

**Status Columns** (High-cardinality, frequently filtered):
- status (enrollments, application_forms, birth_certificates, medical_certificates)
- verification_status (birth_certificates, medical_certificates, transcripts)
- is_active (courses, batch_groups)

**Temporal Columns** (For date range queries):
- created_at (all tables) - Latest records, reports
- submitted_at (enrollments, application_forms)
- payment_date (payments)
- enrollment_start, enrollment_end (batch_groups)

---

## Backup & Recovery

- **Backup Frequency**: Daily automated backups
- **Backup Location**: Separate secure storage location
- **Retention**: 30-day backup rotation
- **Recovery**: Point-in-time recovery capability
- **Testing**: Regular backup restore testing

---

## Data Privacy & Security

- **Encryption**: Passwords hashed with bcrypt; sensitive fields encrypted at rest
- **Access Control**: Database user accounts with minimal required privileges
- **Audit Trail**: activity_log table tracks all data access
- **Compliance**: GDPR/Data Privacy Act compliance with data retention policies
- **Anonymization**: Ability to anonymize student data if needed for testing

---

This comprehensive database design provides a solid foundation for reliable, scalable, and compliant enrollment system operations.
