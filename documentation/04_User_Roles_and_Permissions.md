# 04 User Roles & Permissions

## Overview

The Dulag Polytechnic Enrollment System implements a three-tier Role-Based Access Control (RBAC) system. Each role has specific permissions, data access levels, and operational capabilities. This structure ensures data security, operational efficiency, and clear delineation of responsibilities.

---

## Role Definitions

### 1. Student Role

#### Purpose
Students are the primary users of the system. They create accounts, submit enrollment applications, upload required documents, and manage their academic records and billing information.

#### Key Characteristics
- **Access Level**: Limited to own data and public information
- **Account Status**: Can be "unvalidated" (pending identity verification) or "validated" (verified and eligible to enroll)
- **Data Visibility**: Cannot view other students' information
- **Module Access**: Restricted to student-specific modules

#### Permissions & Access Rights

**Account & Profile Management**
- ✓ Create personal account via registration
- ✓ View own profile information (name, email, contact)
- ✓ Update own profile details
- ✓ Change own password
- ✓ Reset password using recovery flow
- ✓ Update security questions
- ✓ View account activity log
- ✗ Access other students' profiles
- ✗ Create/manage other accounts

**Document Submission & Verification**
- ✓ Upload birth certificate
- ✓ Upload government-issued ID (front and back)
- ✓ Submit medical certificate
- ✓ Upload transcript of records (TOR)
- ✓ Take selfie for biometric verification
- ✓ View verification status
- ✓ Re-submit documents if requested
- ✗ Verify other students' documents
- ✗ Edit or approve document verification

**Application & Enrollment**
- ✓ Complete registration form (TESDA)
- ✓ Save draft registration form
- ✓ Submit registration form
- ✓ Complete application form with work/training history
- ✓ Save draft application form
- ✓ Submit application form
- ✓ View enrollment status
- ✓ Select courses for enrollment
- ✓ Submit enrollment applications
- ✓ View pending enrollments and status
- ✓ Receive notifications on enrollment status
- ✗ Approve or reject own enrollments
- ✗ Edit other students' enrollments
- ✗ Process enrollments administratively

**Billing & Payment**
- ✓ View billing/tuition amount for own enrollment
- ✓ View payment due dates
- ✓ View payment history
- ✓ View outstanding balance
- ✓ Submit proof of external payment (bank transfer)
- ✓ Download payment receipts
- ✓ Receive billing notifications
- ✓ View ledger of charges and payments
- ✗ Modify billing amounts
- ✗ Delete payment records
- ✗ View other students' billing

**Academic & Records**
- ✓ View own grades (once entered by admin)
- ✓ View own course enrollment history
- ✓ View own course completion status
- ✓ Download transcript of records (TOR)
- ✓ View academic standing
- ✗ Edit grades
- ✗ Access other students' academic records

**Notifications**
- ✓ View in-system notifications
- ✓ Mark notifications as read
- ✓ View notification history
- ✓ Set notification preferences
- ✓ Receive email notifications (if enabled)
- ✗ Send notifications to others
- ✗ Edit notification templates

**System Access**
- ✓ Access student dashboard
- ✓ Access student portal pages
- ✓ Submit forms and applications
- ✓ Download documents (own only)
- ✗ Access admin panel
- ✗ Access super admin functions
- ✗ View system logs or audit trails

---

### 2. Administrator Role (ADMIN)

#### Purpose
Administrators manage day-to-day enrollment operations, process applications, manage payments, and provide support to students. They work within policies set by Super Administrators.

#### Key Characteristics
- **Access Level**: Full access to all student data and enrollment operations
- **Authority**: Can approve/reject enrollments and process payments
- **Scope**: Limited to operational functions; cannot modify system configuration
- **Data Visibility**: Can view all students' information, enrollment status, and financial records

#### Permissions & Access Rights

**Student Information Access**
- ✓ View all student profiles and information
- ✓ Search for students by name, email, ID
- ✓ View student enrollment history
- ✓ View student payment history
- ✓ View student document submission status
- ✓ View student verification status
- ✓ Access comprehensive student detail page
- ✗ Edit student personal information (without authorization)
- ✗ Delete student records
- ✗ Create student accounts directly (students must self-register)

**Enrollment Management**
- ✓ View all student enrollments
- ✓ Filter enrollments by status (pending, approved, rejected)
- ✓ View enrollment details and submitted documents
- ✓ Approve student enrollments
- ✓ Reject enrollments with reasons
- ✓ Request additional information from students
- ✓ Process multiple enrollments in batch
- ✓ Add notes to enrollment records
- ✓ Archive completed enrollments
- ✓ View enrollment history
- ✗ Delete enrollment records
- ✗ Modify enrollment dates
- ✗ Override validation rules

**Payment Processing**
- ✓ View all student payments
- ✓ View outstanding balances
- ✓ Manually enter payment records
- ✓ Verify bank transfer receipts
- ✓ Process cash payments at counter
- ✓ Generate payment receipts
- ✓ Email receipts to students
- ✓ View payment methods and reconcile
- ✓ Archive payment records
- ✗ Modify payment amounts after entry
- ✗ Delete payment records
- ✗ Write off balances without authorization

**Grade Entry & Academic Management**
- ✓ Enter grades for enrolled students
- ✓ View student grade history
- ✓ Correct grades (with audit trail)
- ✓ View transcript of records
- ✓ Generate transcripts for students
- ✗ Modify course requirements
- ✗ Change course prerequisites

**Document & Verification Review**
- ✓ View submitted documents
- ✓ View AI analysis results for documents
- ✓ Confirm document verification
- ✓ Request document resubmission
- ✓ View verification status for all students
- ✗ Modify AI analysis results
- ✗ Override verification decisions without proper documentation

**Communication & Notifications**
- ✓ Send individual notifications to students
- ✓ Send group notifications to selected students
- ✓ Use pre-built notification templates
- ✓ View notification history
- ✓ Track notification delivery
- ✓ Access student contact information
- ✗ Send notifications outside approved templates
- ✗ Delete notifications after sending

**Reporting & Analytics**
- ✓ View enrollment statistics and dashboards
- ✓ View payment reports
- ✓ Filter reports by date, course, batch, status
- ✓ Export reports to CSV/Excel
- ✓ View student demographic data
- ✓ Generate custom reports
- ✗ Access system performance metrics (superadmin only)
- ✗ View complete audit logs (superadmin only)

**Administrative Dashboard**
- ✓ Access admin dashboard
- ✓ View pending enrollments count
- ✓ View outstanding payments summary
- ✓ View key metrics and alerts
- ✓ Quick access to important functions
- ✗ Modify dashboard configuration
- ✗ Access system settings

**System Access**
- ✓ Access admin portal
- ✓ Access admin dashboard
- ✓ Access student detail pages
- ✓ Access enrollment management
- ✓ Access payment processing
- ✓ Access grade entry
- ✓ Access reporting
- ✗ Access super admin panel
- ✗ Manage system configuration
- ✗ Create admin accounts
- ✗ Access system logs

**Limitations**
- Cannot override business rules (e.g., cannot enroll student in course if prerequisites not met)
- Cannot modify system settings or configurations
- Cannot create or manage administrator accounts
- Cannot delete any records permanently
- Cannot access super admin functions
- Actions are logged for audit purposes

---

### 3. Super Administrator Role (SUPERADMIN)

#### Purpose
Super Administrators manage system configuration, course management, batch administration, oversee all operational activities, and handle high-level institutional decisions. They have complete system access and can override operational restrictions when necessary.

#### Key Characteristics
- **Access Level**: Unrestricted access to all system functions and data
- **Authority**: Highest level of system access; can perform all operations and configuration
- **Scope**: System-wide configuration and oversight
- **Responsibility**: System health, data integrity, and institutional policy enforcement

#### Permissions & Access Rights

**All Administrator Permissions Plus:**

**System Configuration**
- ✓ Access system settings and configuration panel
- ✓ Configure business rules and parameters
- ✓ Set document requirements per batch
- ✓ Configure payment methods and processing
- ✓ Configure email/SMTP settings
- ✓ Manage external API integrations
- ✓ Set backup frequency and retention
- ✓ Configure system features and workflows
- ✓ Modify system constants and values

**Course Management**
- ✓ Add new courses with all details
- ✓ Edit course information
- ✓ Set course prerequisites
- ✓ Set course capacity limits
- ✓ Configure course tuition
- ✓ Activate/deactivate courses
- ✓ Delete courses (if not enrolled students)
- ✓ View all course enrollments
- ✓ Generate course enrollment reports
- ✗ Delete courses with active enrollments (system prevents)

**Batch Management**
- ✓ Create new enrollment batches
- ✓ Configure batch details (name, dates, courses)
- ✓ Assign courses to batches
- ✓ Set batch tuition rates
- ✓ Activate/deactivate batches
- ✓ Close/archive batches
- ✓ View all batch enrollments
- ✓ Generate batch reports

**Administrative User Management**
- ✓ Create admin accounts
- ✓ Assign ADMIN or SUPERADMIN roles
- ✓ Edit admin account details
- ✓ Deactivate admin accounts
- ✓ Reset admin passwords
- ✓ View admin activity logs
- ✓ Manage admin permissions
- ✗ Cannot delete other superadmin accounts (multi-admin safety)

**Comprehensive Audit & Logging**
- ✓ View complete activity logs for all users
- ✓ View system error logs
- ✓ View database logs
- ✓ View admin action audit trail
- ✓ Search logs by date, user, action, resource
- ✓ Export logs for external auditing
- ✓ Manage log retention policies
- ✓ Generate compliance reports

**Advanced Analytics & Reporting**
- ✓ Access institutional dashboard with all metrics
- ✓ View enrollment trends and forecasts
- ✓ View revenue analytics and projections
- ✓ View student demographics analysis
- ✓ View system health metrics
- ✓ View performance metrics (response times, errors)
- ✓ Generate comprehensive institutional reports
- ✓ Export all data to multiple formats
- ✓ Schedule automated reports

**Data Management**
- ✓ View all data in system
- ✓ Export student data
- ✓ Export payment data
- ✓ Export enrollment data
- ✓ Perform bulk operations on data
- ✓ Archive old data
- ✓ Configure data retention policies
- ✗ Permanently delete production data (with safeguards)

**Document Management**
- ✓ All permissions of Admin, plus:
- ✓ Configure document requirements per batch
- ✓ View document processing statistics
- ✓ Archive documents
- ✓ Configure document retention

**Advanced Student Management**
- ✓ All Admin permissions, plus:
- ✓ Manually create student accounts (for staff)
- ✓ Override validation requirements (with audit log)
- ✓ Delete student records (with backup)
- ✓ Merge duplicate student records
- ✓ Perform bulk student operations

**System Administration**
- ✓ Access super admin dashboard
- ✓ View system health status
- ✓ Monitor active users and sessions
- ✓ View real-time system alerts
- ✓ Manage system maintenance windows
- ✓ View database statistics
- ✓ Trigger manual backups
- ✓ Configure system features

**Financial Management**
- ✓ All Admin payment permissions, plus:
- ✓ Override payment validation rules
- ✓ Write off student balances
- ✓ Configure refund policies
- ✓ Perform bulk payment processing
- ✓ View complete financial reports
- ✓ Reconcile accounts

**Compliance & Legal**
- ✓ Generate TESDA compliance reports
- ✓ Generate regulatory compliance documentation
- ✓ Export data for government submission
- ✓ View audit trails for regulatory inspection
- ✓ Manage data privacy compliance
- ✓ Handle data subject access requests

**System Access**
- ✓ Access super admin panel
- ✓ Access all admin functions
- ✓ Access student portal (as super admin)
- ✓ Access system configuration
- ✓ Access all dashboards and reports
- ✓ Access system logs and monitoring
- ✓ Access database directly (if needed)

**Emergency & Recovery**
- ✓ Override any system restriction
- ✓ Handle emergency situations
- ✓ Perform emergency data modifications
- ✓ Execute emergency procedures
- ✓ Restore from backups (with safeguards)

---

## Permissions Matrix

| Function | Student | Admin | Super Admin |
|----------|---------|-------|------------|
| **Profile Management** | | | |
| View own profile | ✓ | ✓ | ✓ |
| View other profiles | - | ✓ | ✓ |
| Edit own profile | ✓ | ✓ | ✓ |
| Edit other profiles | - | - | ✓ |
| | | | |
| **Authentication** | | | |
| Register account | ✓ | - | - |
| Login | ✓ | ✓ | ✓ |
| Change password | ✓ | ✓ | ✓ |
| Reset password | ✓ | ✓ | ✓ |
| Create other accounts | - | - | ✓ |
| | | | |
| **Document Submission** | | | |
| Upload documents | ✓ | - | - |
| Verify documents | - | ✓ | ✓ |
| View documents | ✓* | ✓ | ✓ |
| | | | |
| **Enrollment** | | | |
| Submit enrollment | ✓ | - | - |
| View own enrollment | ✓ | ✓ | ✓ |
| View all enrollments | - | ✓ | ✓ |
| Approve enrollment | - | ✓ | ✓ |
| Reject enrollment | - | ✓ | ✓ |
| Override enrollment rules | - | - | ✓ |
| | | | |
| **Billing & Payment** | | | |
| View own billing | ✓ | ✓ | ✓ |
| View all billing | - | ✓ | ✓ |
| Enter payments | - | ✓ | ✓ |
| Process payments | - | ✓ | ✓ |
| Write off balance | - | - | ✓ |
| | | | |
| **Grades** | | | |
| View own grades | ✓ | ✓ | ✓ |
| View all grades | - | ✓ | ✓ |
| Enter grades | - | ✓ | ✓ |
| Modify grades | - | ✓ | ✓ |
| | | | |
| **Notifications** | | | |
| View own notifications | ✓ | ✓ | ✓ |
| Send notifications | - | ✓ | ✓ |
| Send bulk notifications | - | ✓ | ✓ |
| | | | |
| **Course Management** | | | |
| View courses | ✓ | ✓ | ✓ |
| Add courses | - | - | ✓ |
| Edit courses | - | - | ✓ |
| Delete courses | - | - | ✓ |
| | | | |
| **Batch Management** | | | |
| View batches | ✓ | ✓ | ✓ |
| Create batches | - | - | ✓ |
| Configure batches | - | - | ✓ |
| Close batches | - | - | ✓ |
| | | | |
| **Admin Management** | | | |
| Create admin accounts | - | - | ✓ |
| Edit admin accounts | - | - | ✓ |
| Deactivate admins | - | - | ✓ |
| View admin logs | - | - | ✓ |
| | | | |
| **Reports & Analytics** | | | |
| View own reports | ✓ | ✓ | ✓ |
| View admin reports | - | ✓ | ✓ |
| View system reports | - | - | ✓ |
| Export data | - | ✓ | ✓ |
| | | | |
| **System Logs** | | | |
| View own activity | ✓ | ✓ | ✓ |
| View user activity logs | - | - | ✓ |
| View system logs | - | - | ✓ |
| Export audit logs | - | - | ✓ |
| | | | |
| **System Configuration** | | | |
| Access settings | - | - | ✓ |
| Modify settings | - | - | ✓ |
| Manage integrations | - | - | ✓ |
| Configure workflows | - | - | ✓ |

*Students can only view their own documents

---

## Access Control Implementation

### Permission Checking Mechanism

All permission checks are implemented at multiple levels:

1. **Route-Level Protection**: Middleware checks role before allowing access to controller
2. **Method-Level Protection**: Private methods in controllers enforce specific permissions
3. **Data-Level Protection**: Queries filtered by role to prevent unauthorized data access
4. **UI-Level Hiding**: Menu items and buttons hidden for unauthorized users

### Session & Authentication

- Users identified by session variable `user_id` and `role`
- Every request validates that user still has appropriate permissions
- Session expires after 30 minutes of inactivity
- Re-authentication required for sensitive operations

### Audit Trail

- All permission checks logged for security audit
- Failed access attempts recorded and monitored
- Admin actions tracked with timestamp and user identification
- Reports available for compliance review

---

This comprehensive role-based permission structure ensures that the system maintains security while allowing each user type to perform their necessary functions effectively.
