# 05 System Workflow

## System Architecture Flow

```
┌─────────────────────────────────────────────────────────────┐
│                    User Entry Points                         │
├──────────────────┬──────────────────┬──────────────────┐    │
│ Student Portal   │ Admin Portal     │ Super Admin      │    │
│ (Public)         │ (Admin Auth)     │ (Super Auth)     │    │
└────────┬─────────┴────────┬─────────┴────────┬────────┘    │
         │                  │                  │              │
         └──────────────────┴──────────────────┘              │
                        │                                    │
                        ▼                                    │
         ┌──────────────────────────────┐                  │
         │   CodeIgniter Framework      │                  │
         │  Routes → Controller → Model │                  │
         └──────────────────┬───────────┘                  │
                        │                                    │
         ┌──────────────▼──────────────┐                  │
         │  RBAC Permission Check       │                  │
         │  (Middleware)                │                  │
         └──────────────┬───────────────┘                  │
                        │                                    │
         ┌──────────────▼──────────────────────┐           │
         │  Business Logic & Data Processing   │           │
         │  - Document Processing              │           │
         │  - AI Analysis (Gemini API)         │           │
         │  - Payment Processing               │           │
         │  - Notification System              │           │
         └──────────────┬──────────────────────┘           │
                        │                                    │
         ┌──────────────▼──────────────────────┐           │
         │      Database Layer (MySQL)         │           │
         │  - Users, Enrollments, Payments    │           │
         │  - Documents, Grades, Courses      │           │
         └──────────────┬──────────────────────┘           │
                        │                                    │
         ┌──────────────▼──────────────────────┐           │
         │   External Services                │           │
         │  - Google Cloud Storage            │           │
         │  - Google Gemini API               │           │
         │  - SMTP (Email)                    │           │
         │  - Cloudflare Turnstile            │           │
         └─────────────────────────────────────┘           │
└─────────────────────────────────────────────────────────────┘
```

---

## Overall System Workflow

```
START
  │
  ├─► User Visits Portal
  │
  ├─► Is User Logged In?
  │   ├─ NO → Redirect to Login
  │   │       ├─► Existing User? Login with email/password
  │   │       └─► New User? Register with OTP verification
  │   │
  │   └─ YES → Check Role
  │           ├─► STUDENT → Student Dashboard
  │           ├─► ADMIN → Admin Dashboard
  │           └─► SUPERADMIN → Super Admin Dashboard
  │
  ├─► Execute User Action
  │
  └─► END
```

---

## Detailed Student Workflow (Enrollment Journey)

### Phase 1: Account Setup & Verification

```
┌─────────────────────────────────────────────────────────────────┐
│              STUDENT ACCOUNT CREATION                           │
└─────────────────────────────────────────────────────────────────┘

STEP 1: Registration
  ├─ Student enters email address
  ├─ System checks if email already registered
  ├─ If unique → Generate OTP and send via email
  └─ Student receives OTP

STEP 2: Email Verification (OTP)
  ├─ Student enters OTP received in email
  ├─ System validates OTP (5-10 minute expiry)
  ├─ If valid → Proceed to password creation
  └─ If invalid → Request new OTP

STEP 3: Password Creation
  ├─ Student sets secure password (8+ characters, mixed case, numbers)
  ├─ System hashes password with bcrypt + salt
  ├─ Account created with status "UNVALIDATED"
  ├─ Redirect to account validation
  └─ Email confirmation sent

STEP 4: Account Status: UNVALIDATED
  └─ Student cannot enroll until identity verified
```

### Phase 2: Identity Verification (Document Submission)

```
┌─────────────────────────────────────────────────────────────────┐
│          IDENTITY VERIFICATION PHASE                            │
│          (Status: UNVALIDATED → VALIDATED)                      │
└─────────────────────────────────────────────────────────────────┘

STEP 1: Birth Certificate Submission
  ├─ Student uploads birth certificate image/PDF
  ├─ System validates file format and size
  ├─ File uploaded to Google Cloud Storage
  ├─ AI Analysis via Google Gemini API:
  │  ├─ Extract birth certificate data
  │  ├─ Verify document authenticity
  │  ├─ Extract name, DOB, gender, ID
  │  └─ Cross-reference with government database
  │
  ├─ AI Results stored in database
  ├─ Admin notified of document submission
  ├─ Admin reviews AI analysis results
  ├─ Admin confirms or rejects verification
  ├─ If approved → Data populated to student profile
  └─ Student notified of verification result

STEP 2: Government ID Verification
  ├─ Student uploads ID front and back
  ├─ System validates image quality and readability
  ├─ Files uploaded to Google Cloud Storage
  ├─ AI Analysis via Google Gemini API:
  │  ├─ Extract ID information
  │  ├─ Verify ID number matches birth certificate
  │  └─ Check for forgery indicators
  │
  ├─ Admin reviews and confirms
  └─ ID information linked to student account

STEP 3: Medical Certificate Submission
  ├─ Student uploads medical clearance certificate
  ├─ System validates PDF/image format
  ├─ File uploaded to Google Cloud Storage
  ├─ Manual review by admin:
  │  ├─ Verify certificate is current (not expired)
  │  ├─ Confirm medical clearance status
  │  └─ Note any medical accommodations if needed
  │
  ├─ Admin approves or requests resubmission
  └─ Status updated in student record

STEP 4: Selfie Verification (Biometric)
  ├─ Student captures selfie using device camera
  ├─ System captures timestamp and device info
  ├─ Image uploaded to Google Cloud Storage
  ├─ Optional: AI facial recognition (liveness detection)
  ├─ Manual review by admin:
  │  ├─ Compare selfie with ID photo
  │  ├─ Verify identity match
  │  └─ Confirm liveness
  │
  ├─ Admin confirms verification
  └─ Biometric verification complete

STEP 5: Last Verification Review
  ├─ System displays all verification results:
  │  ├─ Birth Certificate ✓/✗
  │  ├─ Government ID ✓/✗
  │  ├─ Medical Certificate ✓/✗
  │  ├─ Selfie Verification ✓/✗
  │  └─ All Required Documents ✓/✗
  │
  ├─ If ALL verified → Account status = "VALIDATED"
  ├─ Student notified of validation completion
  ├─ Student eligible to proceed with enrollment
  └─ Proceed to Phase 3

STEP 6: Account Status: VALIDATED
  └─ Student can now enroll in courses
```

### Phase 3: Application & Registration Forms

```
┌─────────────────────────────────────────────────────────────────┐
│              APPLICATION & REGISTRATION FORMS                   │
└─────────────────────────────────────────────────────────────────┘

STEP 1: Complete Registration Form (TESDA Compliance)
  ├─ Student opens registration form
  ├─ Fill out sections:
  │  ├─ Personal Information (pre-filled from verification)
  │  ├─ Educational Background
  │  ├─ TESDA ULI Number (if applicable)
  │  ├─ Batch Selection
  │  └─ Additional required fields
  │
  ├─ Form validation (client & server-side)
  ├─ Option to save as draft
  ├─ Submit when complete
  ├─ Status = "SUBMITTED"
  └─ Email confirmation sent

STEP 2: Complete Application Form (Advanced Information)
  ├─ Student opens application form
  ├─ Add work experience entries (multiple):
  │  ├─ Company, position, duration, responsibilities
  │  └─ Add/remove entries as needed
  │
  ├─ Add training programs (multiple):
  │  ├─ Program name, provider, duration, certificate status
  │  └─ Add/remove entries as needed
  │
  ├─ Add licensure/certifications (if any):
  │  ├─ License type, issue date, expiry
  │  └─ Reference number
  │
  ├─ Form validation
  ├─ Save as draft or submit
  ├─ If submitted → Email confirmation
  └─ Application stored as JSON for flexibility

STEP 3: Document Uploads (Transcript of Records)
  ├─ Student uploads TOR document
  ├─ File format validation (PDF, JPG, PNG)
  ├─ File uploaded to cloud storage
  ├─ AI analysis extracts grade information
  ├─ Admin reviews for academic standing
  ├─ Admin approves or requests resubmission
  └─ Grade history recorded in system
```

### Phase 4: Enrollment & Course Selection

```
┌─────────────────────────────────────────────────────────────────┐
│                 COURSE ENROLLMENT PHASE                         │
└─────────────────────────────────────────────────────────────────┘

STEP 1: View Available Courses
  ├─ Student navigates to enrollment section
  ├─ System displays active courses for current batch:
  │  ├─ Course code, title, description
  │  ├─ Credits, tuition amount
  │  ├─ Prerequisites (if any)
  │  └─ Available seats
  │
  ├─ Filter by course type, schedule
  └─ Student selects courses

STEP 2: Validate Enrollment Eligibility
  ├─ System checks prerequisites for each selected course
  ├─ Verify no concurrent enrollments in same course
  ├─ Check if prior enrollments completed
  ├─ Verify course capacity not exceeded
  ├─ Confirm enrollment period still active
  ├─ If all validations pass → Allow enrollment
  └─ If validation fails → Display reason and prevent enrollment

STEP 3: Submit Enrollment Application
  ├─ Student reviews selected courses:
  │  ├─ Course details
  │  ├─ Total tuition amount
  │  ├─ Schedule information
  │  └─ Prerequisites summary
  │
  ├─ Student submits enrollment application
  ├─ System records submission timestamp
  ├─ Enrollment status = "PENDING REVIEW"
  ├─ Email confirmation sent to student
  ├─ Admin notification of new enrollment
  └─ Proceed to Phase 5 (Admin Review)

STEP 4: Student Views Enrollment Status
  ├─ Student dashboard shows enrollment status:
  │  ├─ PENDING REVIEW (awaiting admin decision)
  │  ├─ APPROVED (admin approved, awaiting payment)
  │  ├─ REJECTED (admin rejected with reason)
  │  ├─ ADDITIONAL INFO NEEDED (admin requesting documentation)
  │  ├─ ENROLLED (payment received, enrollment confirmed)
  │  └─ COMPLETED (course finished, grades recorded)
  │
  ├─ Student receives notifications on status changes
  └─ Student can view admin feedback/notes
```

---

## Administrator Workflow (Enrollment Processing)

```
┌─────────────────────────────────────────────────────────────────┐
│         ADMIN ENROLLMENT PROCESSING WORKFLOW                    │
└─────────────────────────────────────────────────────────────────┘

STEP 1: Review Pending Enrollments
  ├─ Admin logs into admin dashboard
  ├─ Views count of pending enrollments
  ├─ Navigates to enrollment processing section
  ├─ System displays list of pending applications:
  │  ├─ Student name, ID
  │  ├─ Selected courses
  │  ├─ Submission date
  │  ├─ Verification status
  │  └─ Payment status
  │
  └─ Admin selects enrollment to review

STEP 2: Review Enrollment Details
  ├─ Admin views complete enrollment information:
  │  ├─ Student profile (verified identity)
  │  ├─ Selected courses
  │  ├─ Registration form data
  │  ├─ Application form data
  │  ├─ Submitted documents status
  │  ├─ Verification confirmations
  │  └─ Any notes/flags from previous review
  │
  ├─ Admin may request AI re-analysis of documents
  └─ Admin may add internal notes

STEP 3: Make Enrollment Decision
  ├─ Admin options:
  │  ├─ APPROVE (all requirements met)
  │  ├─ REJECT (doesn't meet requirements, with reason)
  │  └─ REQUEST MORE INFO (needs additional documentation)
  │
  ├─ If APPROVED:
  │  ├─ Enrollment status = "APPROVED"
  │  ├─ System records approval timestamp
  │  ├─ Admin name recorded
  │  ├─ Enrollment moves to billing phase
  │  └─ Student receives approval notification (email & in-system)
  │
  ├─ If REJECTED:
  │  ├─ Enrollment status = "REJECTED"
  │  ├─ Admin provides rejection reason
  │  ├─ Student notified with explanation
  │  └─ Student may resubmit if they address issues
  │
  └─ If MORE INFO REQUESTED:
     ├─ Enrollment status = "ADDITIONAL INFO NEEDED"
     ├─ Admin specifies what's needed
     ├─ Student notified to submit requested documents
     ├─ 72-hour response window
     └─ If no response → Status auto-expires to REJECTED

STEP 4: Process Payment (If Approved)
  ├─ Payment due amount calculated:
  │  ├─ Sum of tuition for all selected courses
  │  ├─ Any applicable fees
  │  └─ Minus any advance payments
  │
  ├─ Admin enters payment when received:
  │  ├─ Payment method (cash, check, transfer, card)
  │  ├─ Payment amount
  │  ├─ Payment date
  │  ├─ Reference number (receipt, transaction ID)
  │  └─ Notes
  │
  ├─ System generates receipt:
  │  ├─ Receipt number (unique identifier)
  │  ├─ Student name & ID
  │  ├─ Payment details
  │  ├─ Amount paid
  │  ├─ Amount remaining (if any)
  │  └─ Date processed
  │
  ├─ Receipt emailed to student
  ├─ Enrollment status updated = "ENROLLED"
  ├─ Course enrollment records created
  └─ Student ready to begin coursework

STEP 5: Grade Entry (After Course Completion)
  ├─ Admin navigates to grade entry section
  ├─ Select course and batch
  ├─ System displays enrolled students for that course:
  │  ├─ Student name, ID
  │  ├─ Enrollment date
  │  └─ Current grade (if previously entered)
  │
  ├─ Admin enters grades:
  │  ├─ Student name identifies record
  │  ├─ Grade entered (0-100 or per institution scale)
  │  ├─ Grade validated (must be in valid range)
  │  └─ Comment added if needed (optional)
  │
  ├─ Batch entry allowed:
  │  ├─ Copy grade down (fill all with same grade)
  │  ├─ Import grades from file
  │  └─ Manual entry for each student
  │
  ├─ System calculates GPA (if applicable)
  ├─ Grades saved to database
  ├─ Audit trail recorded (who entered, when)
  ├─ Student notified of grade entry via dashboard
  └─ Student can view grades (if configured)

STEP 6: Payment Management
  ├─ Admin can process payments:
  │  ├─ Receive cash at counter → Record in system
  │  ├─ Receive bank transfer → Verify receipt → Record
  │  ├─ Receive check → Verify clearing → Record
  │  └─ Process online payment → Reconcile with processor
  │
  ├─ For each payment:
  │  ├─ Record payment amount
  │  ├─ Record date received
  │  ├─ Record payment method
  │  ├─ Generate receipt
  │  └─ Email receipt to student
  │
  ├─ Can view payment history:
  │  ├─ Filter by date range
  │  ├─ Filter by payment method
  │  ├─ Filter by student
  │  └─ Total payments by method
  │
  └─ Can reconcile payments:
     ├─ Compare recorded payments with bank deposits
     ├─ Identify discrepancies
     ├─ Generate reconciliation report
     └─ Archive reconciled payments

STEP 7: Communication & Notifications
  ├─ Admin can send messages to students:
  │  ├─ Select notification template
  │  ├─ Customize message content
  │  ├─ Select recipients (individual or group)
  │  ├─ Specify subject and message
  │  └─ Send immediately or schedule
  │
  ├─ Notifications sent via:
  │  ├─ Email
  │  ├─ In-system notification (appears in dashboard)
  │  └─ Both (configurable per template)
  │
  ├─ Admin can track:
  │  ├─ Messages sent (date, recipient, status)
  │  ├─ Delivery status (sent, delivered, read)
  │  ├─ Send bulk notifications
  │  └─ View notification history
  │
  └─ Templates available for:
     ├─ Enrollment approval
     ├─ Payment received confirmation
     ├─ Grades posted
     ├─ Course starting
     ├─ Payment reminder
     └─ Custom messages
```

---

## Super Admin System Configuration Workflow

```
┌─────────────────────────────────────────────────────────────────┐
│         SUPER ADMIN CONFIGURATION WORKFLOW                      │
└─────────────────────────────────────────────────────────────────┘

STEP 1: Course Management
  ├─ Super Admin navigates to Course Management
  ├─ Add new course:
  │  ├─ Enter course code (unique identifier)
  │  ├─ Enter course title
  │  ├─ Enter description
  │  ├─ Set credit hours
  │  ├─ Set tuition amount
  │  ├─ Add prerequisites (if any)
  │  ├─ Set course capacity (max enrollment)
  │  ├─ Mark as active or inactive
  │  └─ Save course
  │
  ├─ Edit existing course:
  │  ├─ Select course from list
  │  ├─ Update course information
  │  ├─ Change active/inactive status
  │  └─ Save changes
  │
  ├─ Delete course:
  │  ├─ System checks if course has enrollments
  │  ├─ If no enrollments → Allow deletion
  │  ├─ If has enrollments → Prevent deletion (archive instead)
  │  └─ Deleted courses archived for audit
  │
  └─ View course statistics:
     ├─ Total enrollments per course
     ├─ Revenue per course
     └─ Enrollment trends

STEP 2: Batch Management
  ├─ Super Admin navigates to Batch Management
  ├─ Create new batch:
  │  ├─ Enter batch identifier (e.g., "2026-Spring")
  │  ├─ Set batch start date
  │  ├─ Set batch end date
  │  ├─ Set enrollment start date (when applications open)
  │  ├─ Set enrollment end date (when applications close)
  │  ├─ Select courses available for batch
  │  ├─ Set tuition rates for batch
  │  ├─ Mark as active or inactive
  │  └─ Save batch
  │
  ├─ Edit batch:
  │  ├─ Update batch dates
  │  ├─ Update courses in batch
  │  ├─ Update tuition rates
  │  └─ Save changes
  │
  ├─ Activate/Deactivate batch:
  │  ├─ Active batch allows new enrollments
  │  ├─ Inactive batch prevents new enrollments
  │  └─ Can reactivate batch if needed
  │
  ├─ Close batch:
  │  ├─ Enrollment period ends
  │  ├─ Batch archived for records
  │  ├─ Can view historical batch data
  │  └─ Cannot create new enrollments
  │
  └─ View batch statistics:
     ├─ Total enrollments per batch
     ├─ Total revenue per batch
     └─ Enrollment by course

STEP 3: Administrative User Management
  ├─ Super Admin navigates to User Management
  ├─ Create new admin account:
  │  ├─ Enter admin email address
  │  ├─ Generate temporary password
  │  ├─ Assign role (ADMIN or SUPERADMIN)
  │  ├─ Enter admin name
  │  ├─ Enter contact information
  │  ├─ Save account (marked as "new")
  │  └─ Email temporary password to admin
  │
  ├─ Edit admin account:
  │  ├─ Update admin information
  │  ├─ Change role assignment
  │  ├─ Enable/disable account
  │  └─ Save changes
  │
  ├─ Deactivate admin:
  │  ├─ Disable admin account
  │  ├─ Admin cannot login
  │  ├─ Account archived for audit
  │  └─ Cannot reactivate (create new instead)
  │
  ├─ View admin activity:
  │  ├─ Last login date
  │  ├─ Actions performed
  │  ├─ Documents processed
  │  ├─ Enrollments processed
  │  └─ Payments processed
  │
  └─ Manage admin permissions:
     ├─ View role assignments
     ├─ Change role level
     └─ Track role-based access

STEP 4: System Configuration
  ├─ Super Admin accesses Settings
  ├─ Email/SMTP Configuration:
  │  ├─ SMTP server address
  │  ├─ SMTP port
  │  ├─ SMTP authentication (username/password)
  │  ├─ From email address
  │  ├─ Test email delivery
  │  └─ Save configuration
  │
  ├─ API Configuration:
  │  ├─ Google Gemini API key (securely stored)
  │  ├─ Google Cloud Storage credentials
  │  ├─ Cloudflare Turnstile site key
  │  ├─ Test API connectivity
  │  └─ Save configuration
  │
  ├─ Business Rules:
  │  ├─ Enrollment deadline rules
  │  ├─ Payment requirements
  │  ├─ Document requirements per batch
  │  ├─ Validation requirements
  │  └─ Save rules
  │
  ├─ UI Configuration:
  │  ├─ System name/branding
  │  ├─ Logo upload
  │  ├─ Color scheme
  │  ├─ Feature flags (enable/disable features)
  │  └─ Save configuration
  │
  └─ Security Settings:
     ├─ Session timeout duration
     ├─ Password requirements
     ├─ Rate limiting
     └─ Save settings

STEP 5: Analytics & Reporting
  ├─ Super Admin accesses Analytics Dashboard
  ├─ Institutional Overview:
  │  ├─ Total students
  │  ├─ Total enrollments
  │  ├─ Total revenue
  │  ├─ Enrollment trends (chart)
  │  ├─ Revenue trends (chart)
  │  └─ System uptime
  │
  ├─ Enrollment Analytics:
  │  ├─ Enrollments by course
  │  ├─ Enrollments by batch
  │  ├─ Enrollment by status
  │  ├─ Demographics (age, gender distribution)
  │  └─ Geographic distribution
  │
  ├─ Financial Analytics:
  │  ├─ Revenue by course
  │  ├─ Revenue by payment method
  │  ├─ Outstanding balances
  │  ├─ Collections rate
  │  └─ Financial projections
  │
  ├─ Generate Reports:
  │  ├─ Select report type
  │  ├─ Select date range
  │  ├─ Select filters (course, batch, status)
  │  ├─ Generate report
  │  ├─ View or download (PDF, Excel)
  │  └─ Schedule recurring reports
  │
  ├─ Export Data:
  │  ├─ Select data type to export
  │  ├─ Select date range
  │  ├─ Choose file format (CSV, Excel)
  │  ├─ Include filters
  │  └─ Download file
  │
  └─ Compliance Reports:
     ├─ TESDA compliance report
     ├─ Student demographics report
     ├─ Audit trail report
     └─ Export for government submission

STEP 6: Audit Logging & Monitoring
  ├─ Super Admin accesses System Logs
  ├─ View activity logs:
  │  ├─ Filter by user (admin, super admin, student)
  │  ├─ Filter by action type (login, enrollment, payment, etc.)
  │  ├─ Filter by date range
  │  ├─ Filter by resource (student ID, course ID)
  │  ├─ View log entry details (timestamp, user, action, result)
  │  └─ Search logs by keyword
  │
  ├─ View error logs:
  │  ├─ System errors and exceptions
  │  ├─ Filter by severity (error, warning, info)
  │  ├─ Filter by date and time
  │  ├─ View error stack trace
  │  └─ Download error logs
  │
  ├─ Monitor system health:
  │  ├─ Database status
  │  ├─ File storage usage
  │  ├─ API status
  │  ├─ Email queue status
  │  ├─ Active sessions count
  │  └─ System performance metrics
  │
  └─ Export audit logs:
     ├─ Generate audit report
     ├─ Specify date range
     ├─ Include all or filtered logs
     ├─ Download as PDF or CSV
     └─ For compliance submission
```

---

## Error & Exception Flows

### Enrollment Validation Failure

```
Student submits enrollment
         │
         ▼
Validation checks:
├─ Is student validated? (ID verified)
├─ Has active enrollment for same course?
├─ Prerequisites met?
├─ Course capacity available?
└─ Enrollment period active?
         │
    ▼────┴────▼
    PASS     FAIL
    │        │
    │        ├─► Display error message to student
    │        ├─► Enrollment NOT saved
    │        ├─► Suggest resolution (e.g., "Complete identity verification")
    │        └─► Student returns to fix issue
    │
    ▼
Enrollment saved as PENDING REVIEW
    │
    ▼
Admin review (see Administrator Workflow above)
```

### Document Verification Failure

```
Student uploads document
    │
    ▼
File validation:
├─ Correct file format?
├─ Reasonable file size?
└─ File readable?
    │
    ▼ PASS
    │
File uploaded to cloud storage
    │
    ▼
AI Analysis via Gemini API
    │
    ├─► Extract data
    ├─► Verify authenticity
    └─► Return confidence score
    │
    ▼
Admin review:
├─ Review AI results
├─ Verify matches student info
└─ Confirm or request resubmission
    │
    ├─► APPROVED: Data accepted, marked as verified
    │   └─► Student notified
    │
    └─► REJECTED: Request resubmission
        ├─► Admin specifies reason
        ├─► Student notified with instructions
        └─► Student uploads new document (repeat process)
```

### Payment Processing Flow

```
Enrollment APPROVED (awaiting payment)
    │
    ▼
Calculate amount due:
├─ Tuition for all selected courses
├─ Minus any advance payments
└─ Total amount due = X
    │
    ▼
Student pays (multiple methods):
├─ Cash at counter → Admin enters payment
├─ Bank transfer → Admin verifies receipt, enters payment
├─ Card online → Auto-recorded if integrated
└─ Check → Admin verifies clearing, enters payment
    │
    ▼
Payment recorded in system:
├─ Generate receipt number (unique)
├─ Record payment timestamp
├─ Record payment method
├─ Verify amount matches enrollment balance
└─ Calculate remaining balance
    │
    ▼
    ├─► Full payment received?
    │   ├─ YES: Enrollment status = ENROLLED (ready to start)
    │   │       Email receipt to student
    │   │       Notify admin of completion
    │   │       Course enrollment records created
    │   │       Student can begin coursework
    │   │
    │   └─ NO: Partial payment
    │       Balance due = X - Y
    │       Status = AWAITING FINAL PAYMENT
    │       Email receipt with remaining balance
    │
    └─► If payment fails/rejected:
        ├─ Notify admin of issue
        ├─ Notify student to resubmit
        ├─ Do not mark as enrolled
        └─ Keep in APPROVED status pending payment
```

---

These workflows cover the major processes in the enrollment system and provide a comprehensive understanding of how different components interact to support the institution's enrollment operations.
