# 03 Features

## System Features Overview

The Dulag Polytechnic Enrollment System includes comprehensive features organized by functionality. Each feature is designed to serve specific user roles and business processes.

## Features by Category

### 1. Authentication & Account Management

#### User Registration & Account Creation
- **Email-based Registration**: Students register using institutional or personal email addresses
- **OTP Verification**: One-Time Password sent via email to verify account ownership
- **Password Creation**: Secure password setup with strength requirements (minimum 8 characters, mixed case, numbers)
- **Account Status Tracking**: New accounts start as "unvalidated" until identity verification complete

#### Login & Session Management
- **Email/Password Login**: Standard authentication with credentials
- **Session Management**: Secure session handling with automatic timeout after 30 minutes of inactivity
- **"Remember Me" Option**: Optional persistent login for convenience (7 days)
- **IP Address Tracking**: Logs user's IP for security monitoring

#### Password Management
- **Change Password**: Authenticated users can update password anytime
- **Forgot Password Flow**: Multi-step recovery using:
  - Email verification (OTP)
  - Security questions (user-defined during registration)
  - Password reset capability
  - Notification of password change for security awareness

#### Account Settings
- **Profile Information**: Update name, email, contact details
- **Security Settings**: Change password, update security questions
- **Notification Preferences**: Control email and in-system notifications
- **Account Activity Log**: View login history and recent account activities

---

### 2. Student Identity & Document Verification

#### Birth Certificate Verification
- **Document Upload**: Students upload digital copy of birth certificate
- **AI-Powered Analysis**: Google Gemini API analyzes certificate for:
  - Document authenticity verification
  - Data extraction (name, date of birth, gender)
  - ID number verification against government databases
- **Validation Confirmation**: Admin reviews AI analysis and confirms validation
- **Data Population**: Verified data automatically populates student profile (name, gender, age)

#### Government ID Verification
- **Front & Back Scan**: Capture or upload photos of government-issued ID (passport, driver's license, etc.)
- **AI Document Analysis**: Extract and validate ID information
- **ID Number Verification**: Cross-reference with birth certificate data
- **Photo Quality Check**: Ensure clear, readable document images
- **Manual Review**: Admin verification of automated analysis

#### Medical Certificate Submission
- **Document Upload**: Students submit medical clearance certificate
- **Format Validation**: Verify PDF/image format and file size
- **Expiry Checking**: Ensure certificate is current and not expired
- **Information Extraction**: Extract medical clearance status and relevant dates
- **Manual Verification**: Admin confirms medical eligibility

#### Selfie Verification (Biometric)
- **Live Capture**: Student takes selfie using device camera
- **Timestamp Recording**: Capture date/time and device information
- **Face Recognition**: Optional integration with facial recognition for liveness detection
- **Manual Comparison**: Admin compares selfie with ID photo
- **Verification Confirmation**: Records verification date and approving administrator

#### Last Verification Summary
- **Consolidation View**: Display all verification results in single view
- **Status Indicators**: Clear indication of which verifications are complete/pending
- **Missing Document Alert**: Highlight any outstanding verification requirements
- **Proceed to Enrollment**: Enable enrollment only after all verifications complete

---

### 3. Application & Registration Forms

#### Registration Form (TESDA Compliance)
- **Multi-Section Form**: Organized sections for different information categories
- **Personal Information**: Name, gender, date of birth, contact details
- **Educational Background**: Prior schooling and certifications
- **TESDA ULI Number**: Unique Learner Identifier field
- **Batch Assignment**: Selection of enrollment batch/cohort
- **Form Save**: Draft functionality to save incomplete forms
- **Validation**: Client-side and server-side validation of required fields
- **Submission**: Official submission marks form as "submitted" with timestamp

#### Application Form (Advanced)
- **Work Experience**: Multi-entry section for previous employment history
  - Company name, position, duration, responsibilities
  - Add/remove multiple work experience entries
- **Training Attended**: Multi-entry section for prior training programs
  - Training program name, provider, duration, certificate status
- **Licensure Examinations**: Track professional licenses and exam results
  - License type, issue date, expiry date, reference number
- **Competency Assessments**: Record of skills assessments and certifications
- **JSON Storage**: Complex data stored as JSON for flexibility
- **Field Validation**: Ensure data consistency and completeness

#### Transcript of Records (TOR) Upload
- **Document Upload**: Students submit academic transcript
- **Format Support**: PDF, JPG, PNG formats accepted
- **File Validation**: Verify file integrity and size limits
- **Grade History**: Information extracted from transcript
- **Manual Review**: Admin verifies against student's academic record
- **Academic Standing**: Validation of good academic standing

---

### 4. Enrollment Management

#### Course Selection & Enrollment
- **Available Courses Display**: Show all active courses for current batch
- **Course Information**: Course code, title, description, credits, tuition
- **Prerequisites Check**: Validate student meets course prerequisites
- **Concurrent Enrollment Limit**: Prevent over-enrollment in same semester
- **Enrollment Creation**: Register student in selected courses
- **Multiple Courses**: Support enrollment in multiple courses per batch
- **Enrollment Confirmation**: Receipt and confirmation information

#### Enrollment Status Tracking
- **Pending Review**: Initial status after student submission
- **Approved**: Admin has approved enrollment
- **Rejected**: Admin rejected with reason provided
- **Additional Info Needed**: Request from admin for missing documentation
- **Enrolled**: Student ready to begin coursework
- **Status Notifications**: Student notified of any status changes
- **Timeline Display**: Show dates of submission, review, approval

#### Batch Management
- **Batch Creation**: Super Admin creates enrollment batches (e.g., "Batch 2026-01")
- **Batch Configuration**: Set batch enrollment period, courses, tuition amount
- **Batch Activation**: Enable/disable batch for enrollment
- **Multiple Batches**: Support multiple concurrent batches
- **Batch History**: Archive completed batches for records
- **Batch Scheduling**: Define start/end dates for enrollment periods

#### Enrollment Rules & Validation
- **Prior Approval Requirement**: Student cannot re-enroll if previous enrollment not completed
- **Active Enrollment Check**: Prevent duplicate active enrollments for same course/batch
- **Course Capacity**: Enforce maximum enrollment per course if configured
- **Enrollment Deadline**: Prevent enrollment after batch deadline
- **Validation Messages**: Clear messages when enrollment cannot proceed

---

### 5. Billing & Payment Processing

#### Tuition Billing
- **Automatic Calculation**: Tuition computed based on courses selected
- **Payment Schedule**: Display payment schedule and due dates
- **Billing Ledger**: Track all charges and credits
- **Outstanding Balance**: Show current amount due
- **Billing History**: Previous billing records accessible

#### Payment Processing
- **Multiple Payment Methods**:
  - Cash payment (at counter)
  - Bank transfer (manual entry with receipt verification)
  - Credit/Debit card (if integrated)
- **Payment Entry**: Admin enters payment information
- **Receipt Generation**: Automatic PDF receipt creation with:
  - Transaction reference number
  - Payment date and amount
  - Account details
  - Payment method
- **Payment Confirmation**: Both system and email confirmation to student

#### Payment History & Ledger
- **Payment Records**: Complete history of all payments received
- **Timestamps**: Date and time of each transaction
- **Reference Numbers**: Unique identifier for each transaction
- **Payment Method Tracking**: Identify how payment was received
- **Ledger Report**: Full accounting of charges and payments
- **Outstanding Balance Calculation**: Real-time balance due

#### Receipt Management
- **Receipt Download**: Students can download receipt as PDF
- **Email Receipt**: Option to email receipt to student
- **Receipt Archival**: Receipts retained for audit purposes
- **Reference Number**: Unique identifier for payment tracking

---

### 6. Notification System

#### Email Notifications
- **Status Updates**: Enrollment approval/rejection messages
- **Billing Alerts**: Overdue payment reminders
- **Verification Updates**: Document verification completion
- **Password Changes**: Security alerts on account changes
- **New Messages**: Alerts for important system messages
- **Template-Based**: Customizable notification templates

#### In-System Notifications
- **Dashboard Display**: Notifications visible on user dashboard
- **Unread Indicator**: Show count of unread notifications
- **Notification Detail**: Click to view full notification content
- **Mark as Read**: Option to mark individual or all notifications as read
- **Notification History**: Archive of past notifications

#### Bulk Notifications
- **Admin Capability**: Send messages to student groups
- **Target Selection**: Filter students by role, batch, course, status
- **Message Composition**: Create custom messages
- **Scheduling**: Send immediately or schedule for later
- **Delivery Confirmation**: Track notification delivery status

#### Notification Preferences
- **User Control**: Students choose notification methods (email, in-system, both)
- **Notification Categories**: Enable/disable specific notification types
- **Quiet Hours**: Option to mute notifications during specific times
- **Communication Settings**: Update preferred contact information

---

### 7. Administrative Functions

#### Student Management
- **Student List**: View all students with filtering capabilities
- **Student Detail**: Comprehensive student information page including:
  - Personal information
  - Account status
  - Enrollment history
  - Payment history
  - Document verification status
- **Search Functionality**: Search by name, email, ID, reference number
- **Filtering**: Filter by batch, course, enrollment status, validation status

#### Enrollment Processing
- **Pending Enrollments**: View all awaiting review
- **Enrollment Detail**: View all submission details
- **Approval/Rejection**: Process with optional notes/feedback
- **Request Additional Info**: Ask student for missing documentation
- **Bulk Processing**: Process multiple enrollments in batch
- **Archive**: Move completed enrollments to archive

#### Payment Management
- **Manual Payment Entry**: Record payment received by student
- **Payment Verification**: Verify against supporting documentation
- **Payment Archive**: Move processed payments to archive
- **Payment Reports**: Generate payment summaries by date, method, student

#### Grade Entry
- **Grade Recording**: Enter student grades for completed courses
- **Grade Scale**: Support institutional grading system
- **Batch Entry**: Enter multiple student grades at once
- **Grade History**: Maintain complete record of all grades
- **Grade Validation**: Ensure grades within valid range

#### Notifications Management
- **Send Messages**: Compose and send notifications to students
- **Message Templates**: Pre-defined templates for common messages
- **Recipient Selection**: Choose individual or group of recipients
- **Message History**: View all messages sent
- **Delivery Tracking**: Confirm message delivery

#### Analytics & Reporting
- **Enrollment Dashboard**: Visual summary of enrollment statistics
  - Total enrollments, pending, approved, rejected
  - By course, by batch, by status
- **Payment Analytics**: Revenue summary, outstanding balances
- **Student Demographics**: Age, gender, batch distribution
- **Trends**: Historical comparison and growth metrics
- **Export Reports**: Download data as CSV/Excel

---

### 8. Super Admin Functions

#### Course Management
- **Add Course**: Create new courses with details:
  - Course code, title, description
  - Credit hours, tuition amount
  - Prerequisites (if any)
  - Course capacity limits
- **Edit Course**: Update course information
- **Activate/Deactivate**: Control course availability
- **Delete Course**: Remove unused courses from system
- **Course Listing**: View all courses with current enrollment

#### Batch Group Management
- **Create Batch**: Set up new enrollment batches with:
  - Batch identifier (e.g., "2026-Spring")
  - Start and end dates
  - Courses available in batch
  - Tuition configuration
- **Modify Batch**: Update batch settings
- **Activate/Complete**: Control batch enrollment periods
- **View Batch Enrollment**: See all enrollments for batch

#### Administrative User Management
- **Create Admin Accounts**: Add new administrator accounts
- **Assign Roles**: Assign ADMIN or SUPERADMIN role
- **Edit Admin Details**: Update admin information
- **Deactivate Accounts**: Disable access for departed staff
- **Admin Audit**: View admin activities and changes made

#### Comprehensive Analytics
- **Institutional Dashboard**: High-level overview of all system activity
- **Enrollment Analytics**: Detailed enrollment metrics and trends
- **Revenue Analytics**: Billing and payment trends
- **Compliance Metrics**: Track regulatory compliance metrics
- **System Health**: Database performance, error rates
- **Data Export**: Export all reports to Excel/CSV

#### System Configuration
- **System Settings**: Configure business rules and parameters
- **Email Configuration**: SMTP settings for notifications
- **API Configuration**: Manage external API keys and settings
- **Document Requirements**: Configure required documents per batch
- **Backup Settings**: Configure database backup frequency

#### Document Review & Validation
- **Document Queue**: View all submitted documents pending review
- **Document Detail**: Review uploaded documents and AI analysis results
- **Validation Decision**: Approve or request resubmission
- **Bulk Actions**: Process multiple documents at once
- **Document Archive**: Archive processed documents

#### Comprehensive Logging
- **Activity Logs**: View all system activities with full audit trail
- **User Actions**: Track all user interactions (login, data access, modifications)
- **Administrative Actions**: Monitor admin and superadmin actions
- **Error Logs**: System errors and exceptions for debugging
- **Log Export**: Download logs for external auditing

---

### 9. Academic Functions

#### Grade Management
- **Grade Entry**: Add student grades for courses
- **Grade History**: Maintain complete grade record
- **Transcript Generation**: Automatically generate student transcripts
- **GPA Calculation**: Compute student GPA if required
- **Academic Standing**: Determine if student in good academic standing

#### Transcript of Records (TOR) Generation
- **TOR Document**: Generate official transcript PDF with:
  - Student information
  - All courses and grades
  - GPA (if applicable)
  - Certification of academic standing
  - Signature/seal for authenticity
- **QR Code**: Include QR code for document verification
- **Date Generated**: Track document issuance date
- **Student Download**: Students can download official transcript

#### Course History
- **Enrollment History**: Show all courses student has enrolled in
- **Completion Status**: Indicate completed, in-progress, or pending courses
- **Grade Record**: Display grade received for each course
- **Prerequisites Tracking**: Track which prerequisites completed

---

### 10. Security & Access Control

#### Role-Based Access Control (RBAC)
- **Student Role**: Limited to own enrollment and profile
- **Admin Role**: Access to student enrollments, payments, grades
- **SuperAdmin Role**: Full system access and configuration
- **Permission Enforcement**: Middleware checks permissions on every action
- **Data Filtering**: Users see only data relevant to their role

#### Security Features
- **Password Hashing**: bcrypt with salt for secure password storage
- **HTTPS**: All data transmission encrypted
- **CSRF Protection**: Tokens on all state-changing operations
- **Input Validation**: Server-side validation of all user input
- **SQL Injection Prevention**: Prepared statements for all queries
- **XSS Prevention**: Output encoding and sanitization
- **CAPTCHA**: Cloudflare Turnstile on public forms
- **Rate Limiting**: Prevent brute force attacks
- **Session Security**: Secure session token management
- **IP Tracking**: Monitor for unusual access patterns

#### Audit & Compliance
- **Activity Logging**: Log all significant system activities
- **User Actions**: Track who accessed what data and when
- **Change History**: Record modifications to important data
- **Compliance Reports**: Generate audit trails for regulatory inspection
- **Data Retention**: Archive logs per retention policy

---

## Summary of Key Validations & Restrictions

| Feature | Validation/Restriction |
|---------|----------------------|
| User Registration | Email must be unique, password must meet strength requirements |
| Identity Verification | All three verification types must pass before enrollment allowed |
| Course Enrollment | Student must be validated, cannot enroll if has active/approved enrollment in same course |
| Payment | Must be received before enrollment confirmed (unless waived) |
| Grade Entry | Must be within valid range (0-100 or per scale), only for active enrollments |
| Admin Access | Must have ADMIN or SUPERADMIN role, permissions enforced on all operations |
| Document Upload | File size limits, format validation, virus scanning for security |
| Batch Enrollment | Can only occur during active batch period, must have available courses |
| Course Capacity | Cannot exceed maximum enrollment if limit set |
| Transcript Download | Only accessible to student owner or authenticated admin |

---

These features work together to create a comprehensive, secure, and compliant enrollment management system that serves the needs of students, administrators, and institutional leadership.
