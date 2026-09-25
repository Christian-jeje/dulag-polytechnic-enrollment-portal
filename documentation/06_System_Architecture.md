# 06 System Architecture

## Architectural Overview

The Dulag Polytechnic Enrollment System uses a **three-tier MVC (Model-View-Controller) architecture** built on the CodeIgniter 3 PHP framework. This traditional architecture provides clear separation of concerns, maintainability, and scalability.

```
┌─────────────────────────────────────────────────────────────────────┐
│                         PRESENTATION LAYER                          │
│                      (Web Browser / Client)                         │
│  - Student Portal (Public)                                          │
│  - Admin Portal (Authenticated)                                     │
│  - Super Admin Portal (Super Admin Only)                            │
└────────────────────────┬────────────────────────────────────────────┘
                         │
                    HTTPS/SSL
                         │
                         ▼
┌─────────────────────────────────────────────────────────────────────┐
│                      APPLICATION LAYER                              │
│                   (CodeIgniter Framework)                           │
│                                                                     │
│  ┌────────────────────────────────────────────────────────────┐   │
│  │ ROUTING LAYER (Routes.php)                                 │   │
│  │ - URL pattern matching                                     │   │
│  │ - Route parameters extraction                              │   │
│  │ - Request delegation to controllers                        │   │
│  └────────────────────────────────────────────────────────────┘   │
│                         │                                           │
│  ┌──────────────────────▼───────────────────────────────────────┐  │
│  │ MIDDLEWARE LAYER (Permission Checks)                        │  │
│  │ - RBAC enforcement                                          │  │
│  │ - Role validation                                           │  │
│  │ - Session verification                                      │  │
│  │ - CSRF token validation                                     │  │
│  └────────────────────────────────────────────────────────────┘   │
│                         │                                           │
│  ┌──────────────────────▼───────────────────────────────────────┐  │
│  │ CONTROLLER LAYER                                             │  │
│  │ - Welcome (public, auth)                                    │  │
│  │ - Student (student functions)                               │  │
│  │ - Admin (admin functions)                                   │  │
│  │ - Superadmin (super admin functions)                        │  │
│  │ - Request handling and validation                           │  │
│  │ - Business logic orchestration                              │  │
│  └────────────────────────────────────────────────────────────┘   │
│                         │                                           │
│  ┌──────────────────────▼───────────────────────────────────────┐  │
│  │ BUSINESS LOGIC LAYER (Models + Libraries)                  │  │
│  │ - Document processing & validation                          │  │
│  │ - Payment calculations & processing                         │  │
│  │ - Enrollment validation & management                        │  │
│  │ - User authentication & profile                             │  │
│  │ - Report generation                                         │  │
│  │ - Notification composition                                  │  │
│  │ - QR code generation                                        │  │
│  └────────────────────────────────────────────────────────────┘   │
│                         │                                           │
│  ┌──────────────────────▼───────────────────────────────────────┐  │
│  │ DATA LAYER (Models)                                          │  │
│  │ - User_model (authentication, user management)              │  │
│  │ - Enrollment_model (enrollment CRUD & logic)                │  │
│  │ - ApplicationForm_model (multi-section forms)               │  │
│  │ - Payment_model (billing & payment records)                 │  │
│  │ - Course_model (course management)                          │  │
│  │ - BirthCertificate_model (identity documents)               │  │
│  │ - MedicalCertificate_model (health documents)               │  │
│  │ - TranscriptOfRecords_model (academic records)              │  │
│  │ - Notification_model (messaging)                            │  │
│  │ - ActivityLog_model (audit trail)                           │  │
│  │ - And others...                                             │  │
│  └────────────────────────────────────────────────────────────┘   │
└────────────────────────┬────────────────────────────────────────────┘
                         │
        ┌────────────────┼────────────────┐
        │                │                │
        ▼                ▼                ▼
┌─────────────┐  ┌────────────────┐  ┌──────────────┐
│   DATABASE  │  │  FILE STORAGE  │  │ EXTERNAL API │
│   (MySQL)   │  │ (Google Cloud  │  │  SERVICES    │
│             │  │   Storage)     │  │              │
└─────────────┘  └────────────────┘  └──────────────┘
        │
        ▼
    ┌───────────────┐
    │ Eloquent      │
    │ Query Builder │
    │ (CodeIgniter) │
    └───────────────┘
```

---

## Component Breakdown

### 1. Frontend / Presentation Layer

#### Student Portal (`/student/*` routes)
- **Dashboard**: Overview of enrollment status, courses, payments
- **Profile Management**: View and edit student information
- **Document Submission**: Upload identification, medical, transcript
- **Enrollment Forms**: TESDA registration and application forms
- **Course Selection**: Browse and enroll in available courses
- **Billing Dashboard**: View tuition, payment history, receipts
- **Notifications**: View system and email notifications
- **Transcript Download**: Generate and download academic records

**Technology Stack**:
- HTML5 + Bootstrap 4 for responsive design
- jQuery for interactivity and AJAX
- Custom CSS for theming
- Font Awesome for icons
- Browser-native file upload API
- Camera API for selfie capture (Web camera access)

#### Admin Portal (`/admin/*` routes)
- **Dashboard**: Pending enrollments, payments, key metrics
- **Enrollment Management**: Review and process applications
- **Student Information**: Comprehensive student database
- **Payment Processing**: Record and reconcile payments
- **Grade Entry**: Bulk grade management
- **Notifications**: Send messages to students
- **Reports**: Analytics and reporting dashboard

**Technology Stack**: Same as student portal, with additional charting libraries

#### Super Admin Portal (`/superadmin/*` routes)
- **System Configuration**: Settings and API configuration
- **Course Management**: CRUD operations on courses
- **Batch Management**: Create and manage enrollment batches
- **Admin Management**: Create and manage admin accounts
- **Analytics**: Comprehensive institutional analytics
- **Audit Logs**: System activity monitoring
- **Backup Management**: Database backup controls

### 2. Application / Business Logic Layer

#### Controllers

**Welcome Controller** (`/application/controllers/Welcome.php`)
- Public landing page
- User login form and processing
- User registration flow with OTP
- Password reset workflows
- Forgot password recovery

**Student Controller** (`/application/controllers/Student.php`)
- Student account management
- Document submission and verification
- Application form handling
- Enrollment management
- Payment viewing and receipt generation
- Notification management
- Academic records access

**Admin Controller** (`/application/controllers/Admin.php`)
- Enrollment review and processing
- Student information access
- Payment processing and reconciliation
- Grade entry and management
- Notification sending
- Report generation
- Activity logging

**Superadmin Controller** (`/application/controllers/Superadmin.php`)
- Course management (CRUD)
- Batch group management
- Admin account management
- System configuration
- Advanced analytics and reporting
- Audit log viewing
- System maintenance functions

#### Models (Data Access Layer)

**User_model**: User authentication, profile management, role assignment

**Enrollment_model**: Enrollment CRUD, status management, prerequisite validation

**ApplicationForm_model**: Multi-section TESDA form storage (with JSON columns)

**Payment_model**: Payment records, billing, receipt generation

**Course_model**: Course information, availability, prerequisites

**BirthCertificate_model**: Birth certificate upload, AI analysis results, verification

**MedicalCertificate_model**: Medical document submission and verification

**TranscriptOfRecords_model**: Academic transcript storage and GPA calculation

**Notification_model**: Message storage, delivery tracking

**ActivityLog_model**: Audit trail, user actions logging

**BatchGroup_model**: Enrollment batch management

**Otp_model**: One-time password generation and verification

#### Libraries

**RegFormGenerator**: TESDA registration form generation and validation

**ULI_Generator**: TESDA ULI (Unique Learner Identifier) generation

**LocationCodes**: Philippine location code handling

**Custom Utilities**: QR code generation, document processing, PDF generation (FPDF)

### 3. Data Layer

#### Database Architecture (MySQL/MariaDB)

**Core User Tables**:
- `users`: Student and admin accounts, roles, validation status
- `activity_log`: Complete audit trail of all system activities

**Document Tables**:
- `birth_certificates`: Birth certificate uploads and AI analysis results
- `medical_certificates`: Medical clearance submissions
- `transcripts_of_records`: Academic transcript documents

**Enrollment Tables**:
- `enrollments`: Student course enrollment records, status tracking
- `courses`: Course catalog, prerequisites, capacity
- `batch_groups`: Enrollment batch definitions and periods
- `application_forms`: TESDA application form data (JSON storage for multi-entry fields)

**Academic Tables**:
- `grades`: Student grades for courses
- `enrollments`: Links students to courses with status

**Financial Tables**:
- `payments`: Payment records, methods, amounts
- `billing_records`: Tuition charges and billing history

**Communication Tables**:
- `notifications`: System messages and notifications
- `email_queue`: Outgoing email tracking

**Configuration Tables**:
- `settings`: System configuration values
- `system_parameters`: Business rules and constants

#### Database Design Decisions

1. **Normalized Schema**: 3rd Normal Form for data consistency
2. **JSON Columns**: TESDA form multi-entry sections stored as JSON for flexibility
3. **Soft Deletes**: Records marked as deleted, not permanently removed (audit compliance)
4. **Audit Columns**: `created_at`, `updated_at`, `deleted_at` on important tables
5. **Indexing**: Primary keys, foreign keys, and search indices optimized
6. **Relationships**: Foreign key constraints maintain referential integrity

### 4. External Services Integration

#### Google Gemini API
- **Purpose**: AI-powered document analysis
- **Usage**:
  - Birth certificate verification (data extraction, authenticity check)
  - Government ID verification (information validation)
  - Document quality assessment
- **Integration**: HTTP REST API calls from server-side
- **Error Handling**: Fallback to manual admin review if API fails

#### Google Cloud Storage
- **Purpose**: Secure document storage
- **Usage**:
  - Store uploaded documents (birth certificates, IDs, medical certificates, transcripts)
  - Backup and archival
  - Secure retrieval for admin review
- **Security**: Documents encrypted at rest, access controlled via IAM
- **Integration**: Google Cloud SDK (PHP client library)

#### Email (SMTP)
- **Purpose**: Notification delivery
- **Usage**:
  - Account verification emails (OTP)
  - Enrollment status updates
  - Payment confirmations
  - Password reset links
  - Notification delivery
- **Configuration**: SMTP server details in `config/email.php`
- **Queue System**: Email queue table for reliability (retry failed sends)

#### Cloudflare Turnstile (CAPTCHA)
- **Purpose**: Bot protection on public forms
- **Usage**:
  - Registration form protection
  - Login form protection (optional, configurable)
  - Payment form protection
- **Integration**: JavaScript on frontend, server-side verification
- **Security**: Prevents automated abuse and credential stuffing

### 5. Security Architecture

#### Authentication
- **Method**: Session-based with email/password credentials
- **Password Security**: bcrypt hashing (cost factor 12) + salt
- **Session Management**: Secure session tokens, 30-minute timeout
- **Multi-Factor**: OTP for email verification, security questions for password reset

#### Authorization (RBAC)
- **Roles**: STUDENT, ADMIN, SUPERADMIN
- **Enforcement**: Middleware checks on every route
- **Data Filtering**: Queries filtered by role to prevent unauthorized access
- **Audit Trail**: All access attempts logged

#### Data Protection
- **Transit**: HTTPS/TLS encryption for all data in transit
- **At Rest**: Sensitive fields (emails, IDs) encrypted in database
- **File Storage**: Cloud storage with encryption and access controls
- **Backup**: Database backups encrypted and stored securely

#### Input Validation
- **Client-Side**: Form validation via JavaScript
- **Server-Side**: PHP validation on all inputs
- **Prepared Statements**: All SQL queries use parameterized statements (prevents SQL injection)
- **Output Encoding**: XSS prevention through output encoding

#### Additional Security
- **CSRF Tokens**: All state-changing operations protected
- **Rate Limiting**: API endpoints rate-limited to prevent abuse
- **IP Tracking**: Unusual IP changes flagged and logged
- **Activity Logging**: Complete audit trail of all system activities

### 6. Scalability & Performance

#### Caching Strategy
- **Query Caching**: CodeIgniter query cache for frequently accessed data
- **Session Caching**: Efficient session storage
- **File Caching**: Template caching for views

#### Database Optimization
- **Indexing**: Strategic indices on commonly searched fields
- **Query Optimization**: Efficient queries using Query Builder
- **Connection Pooling**: Multiple database connections for concurrent requests
- **Archive Tables**: Old data moved to archive for faster queries

#### File Storage Optimization
- **Cloud Storage**: Offload documents from web server to cloud
- **CDN Ready**: Files can be served via CDN for faster delivery
- **Compression**: Documents compressed where possible

#### Concurrency Support
- **Sessions**: Multiple simultaneous users supported
- **Database Locks**: Pessimistic/optimistic locking for data consistency
- **Queue System**: Email queue processes messages asynchronously

---

## Deployment Architecture

### Local Development (Current)
```
XAMPP Server (Apache + MySQL + PHP)
    └─ Project Files: C:\xampp\htdocs\DulagPolytechnicPortal_github\
    └─ Database: polytechnicportal (MySQL)
    └─ Base URL: http://localhost/DulagPolytechnicPortal_github/
```

### Production Deployment (Ready)
```
Web Server (Apache/Nginx)
    ├─ Load Balancer
    ├─ Web Server Instance 1
    ├─ Web Server Instance 2
    └─ Web Server Instance N
         │
         ├─► Database Server (MySQL Replication)
         │   ├─ Primary Database
         │   └─ Replica Database (backup)
         │
         ├─► File Storage (Google Cloud Storage)
         │   └─ Documents
         │
         ├─► Email Service (SMTP)
         │
         └─► API Services (Gemini, Turnstile)
```

### Key Architectural Decisions

1. **Framework Choice (CodeIgniter 3)**:
   - Lightweight and fast
   - Easy to learn and maintain
   - Built-in security features (CSRF protection, input validation)
   - Excellent documentation
   - Good balance of features vs. simplicity

2. **Database (MySQL/MariaDB)**:
   - Reliable and widely supported
   - Excellent for relational data
   - Strong consistency guarantees
   - Good performance for this scale

3. **Cloud Storage (Google Cloud Storage)**:
   - Secure document storage
   - Reduces web server disk usage
   - Easy to integrate with backup/archival
   - Meets compliance requirements

4. **AI Document Analysis (Gemini API)**:
   - Advanced computer vision capabilities
   - Fast processing
   - Fallback to manual review for edge cases
   - Cost-effective

5. **Session-Based Authentication**:
   - Traditional and reliable
   - Good for institutional use cases
   - Simpler than token-based for administrative interfaces
   - Can migrate to JWT if needed for mobile apps

6. **Monolithic Architecture**:
   - Suitable for current scale
   - Simpler to deploy and maintain
   - Can be refactored to microservices if needed in future

---

## System Integration Points

1. **User Registration** → Email OTP → SMTP Service
2. **Document Upload** → AI Analysis → Gemini API & Cloud Storage
3. **Payment Processing** → Receipt Generation → PDF Library & Email
4. **Form Submission** → Validation → Database Storage
5. **Notifications** → Email Queue → SMTP Service
6. **Reports** → Data Extraction → PDF/Excel Export

---

This architecture provides a solid foundation for the enrollment system while maintaining security, performance, and maintainability. It's designed to be scalable as the institution grows and can accommodate future enhancements such as mobile applications or microservices migration if needed.
