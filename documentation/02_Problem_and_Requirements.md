# 02 Problem & Requirements

## Problems Addressed

### Existing Challenges (Manual Process)

**1. Manual Enrollment Process**
- Students submitting handwritten forms or unstructured digital files
- Administrative staff manually entering data into spreadsheets
- High risk of data entry errors and duplicate records
- No centralized repository for enrollment documents
- Inconsistent data formats across documents

**2. Document Management Issues**
- Physical documents stored in filing cabinets (security risk, space constraints)
- No way to verify document authenticity without manual inspection
- Lost or damaged documents requiring re-submission
- Difficulty retrieving student documents for verification
- No audit trail of document submissions

**3. Identity Verification Challenges**
- Manual verification of birth certificates and government IDs is time-consuming
- No way to confirm student identity without in-person visits
- Risk of fraudulent submissions
- No standardized verification process
- Difficult to prevent duplicate account creation

**4. Billing & Payment Confusion**
- Manual calculation of tuition fees prone to errors
- No integrated payment processing system
- Students unsure about billing status
- Manual receipt generation and record-keeping
- Difficulty tracking payment history

**5. Limited Visibility & Reporting**
- No real-time enrollment status tracking
- Difficult to generate compliance reports for regulatory bodies
- No analytics on enrollment trends or bottlenecks
- Administrative staff cannot quickly answer student inquiries about status
- No audit trail for regulatory compliance

**6. Communication Gaps**
- Students unsure about enrollment status or next steps
- Administrative notifications sent manually via email or phone
- No centralized notification system
- Students cannot access their records after hours
- Inconsistent communication across departments

**7. Security & Data Protection**
- Student data stored in unsecured formats
- No access controls or role-based permissions
- Sensitive documents not encrypted
- No tracking of who accesses what data
- Risk of unauthorized data access

## System Requirements

### Functional Requirements

#### Authentication & Authorization
- **FR-1**: Users must register with email verification via OTP
- **FR-2**: System must support three distinct roles: Student, Administrator, Super Administrator
- **FR-3**: Each role has specific access permissions to modules and data
- **FR-4**: Users must be able to reset passwords using security questions and OTP
- **FR-5**: Session management with automatic logout after inactivity
- **FR-6**: Multi-factor authentication through OTP during sensitive operations

#### Student Enrollment Workflow
- **FR-7**: Students must complete multi-step identity verification using government-issued ID
- **FR-8**: Birth certificate upload and automated validation using AI analysis
- **FR-9**: Medical certificate submission and verification
- **FR-10**: Students must complete registration form with TESDA compliance
- **FR-11**: Students must submit application form with work experience and training history
- **FR-12**: Transcript of Records (TOR) upload and validation
- **FR-13**: Selfie verification for biometric confirmation
- **FR-14**: Students can select courses from available options
- **FR-15**: Enrollment status displayed with approval/rejection notifications
- **FR-16**: Students cannot enroll if prerequisites not met or prior enrollments not completed

#### Billing & Payment
- **FR-17**: System must calculate tuition fees based on course selection and batch
- **FR-18**: Support multiple payment methods (cash, card, transfer)
- **FR-19**: Process payments and generate receipts
- **FR-20**: Track payment history and generate billing ledger
- **FR-21**: Prevent enrollment until billing requirements satisfied
- **FR-22**: Generate payment reports for financial reconciliation

#### Document Management
- **FR-23**: Upload and store documents in secure cloud storage (Google Cloud Storage)
- **FR-24**: Automated document analysis using AI (Google Gemini API)
- **FR-25**: Validate document authenticity (birth certificate ID verification)
- **FR-26**: Support multiple document types with format validation
- **FR-27**: Generate QR codes for document linking and tracking
- **FR-28**: Allow manual document submission by administrators

#### Admin Operations
- **FR-29**: View all student enrollments with filtering and search
- **FR-30**: Process enrollment applications (approve/reject/request info)
- **FR-31**: Add manual payments to student accounts
- **FR-32**: Enter grades for enrolled students
- **FR-33**: Send notifications to individual students or groups
- **FR-34**: Access student information and enrollment history
- **FR-35**: View and export analytics data

#### Super Admin Operations
- **FR-36**: Manage course catalog (add, edit, delete, activate/deactivate courses)
- **FR-37**: Manage batch groups and enrollment periods
- **FR-38**: Oversee all administrative operations with higher-level access
- **FR-39**: Manage administrative user accounts
- **FR-40**: Access comprehensive system analytics
- **FR-41**: Export institutional-level reports
- **FR-42**: Configure system parameters and settings

#### Notification System
- **FR-43**: Send email notifications to students on status updates
- **FR-44**: Display in-system notifications visible in user dashboard
- **FR-45**: Allow users to view notification history
- **FR-46**: Support notification templates for common events
- **FR-47**: Bulk notification capability for administrators

#### Academic Management
- **FR-48**: Maintain course enrollment records
- **FR-49**: Store and retrieve grades for enrolled students
- **FR-50**: Generate transcript of records (TOR) documents
- **FR-51**: Track student course history and progression
- **FR-52**: Support course prerequisites and validation

#### Reporting & Analytics
- **FR-53**: Generate enrollment statistics by course, batch, status
- **FR-54**: Create payment analytics (revenue, outstanding balances)
- **FR-55**: Export data to CSV/Excel formats
- **FR-56**: Generate compliance reports for regulatory bodies
- **FR-57**: Activity logging and audit trails
- **FR-58**: Student demographic analytics

### Non-Functional Requirements

#### Security
- **NFR-1**: All passwords must be hashed using bcrypt with salt
- **NFR-2**: All data transmission must use HTTPS/TLS encryption
- **NFR-3**: Implement Role-Based Access Control (RBAC) for all operations
- **NFR-4**: Sensitive data (emails, IDs) must be encrypted at rest
- **NFR-5**: API endpoints must validate input and use prepared statements
- **NFR-6**: CAPTCHA protection on public-facing forms (Cloudflare Turnstile)
- **NFR-7**: Rate limiting on API endpoints to prevent brute force attacks
- **NFR-8**: Audit logging of all data access and modifications
- **NFR-9**: Session tokens must expire after 30 minutes of inactivity
- **NFR-10**: Implement CSRF token protection on all state-changing operations

#### Performance
- **NFR-11**: Page load time must be under 2 seconds for typical views
- **NFR-12**: Database queries must be optimized with proper indexing
- **NFR-13**: Support concurrent users (minimum 500 simultaneous connections)
- **NFR-14**: File uploads must complete within reasonable timeframe (< 60 seconds for documents)
- **NFR-15**: Report generation must handle large datasets efficiently

#### Reliability & Availability
- **NFR-16**: System uptime target of 99.5% during operating hours
- **NFR-17**: Database backups must occur at least daily
- **NFR-18**: Graceful error handling with user-friendly error messages
- **NFR-19**: Recovery from temporary API failures (retry logic)
- **NFR-20**: Redundancy for external API calls where possible

#### Usability
- **NFR-21**: Interface must be intuitive and require minimal training
- **NFR-22**: Mobile-responsive design supporting tablets and smartphones
- **NFR-23**: Accessibility compliance (WCAG 2.1 Level AA standards)
- **NFR-24**: Support for common browsers (Chrome, Firefox, Safari, Edge)
- **NFR-25**: Clear navigation and help documentation available

#### Scalability
- **NFR-26**: Architecture must support future growth to 10,000+ enrolled students
- **NFR-27**: Database schema designed for efficient data retrieval at scale
- **NFR-28**: File storage system capable of handling thousands of documents
- **NFR-29**: Ability to add new courses and batch groups without system redesign

#### Compliance
- **NFR-30**: TESDA (Technical Education and Skills Development Authority) compliance
- **NFR-31**: Data Privacy Act (RA 10173) compliance for student data
- **NFR-32**: Audit trails for regulatory inspection
- **NFR-33**: Document retention policies and archival support
- **NFR-34**: Report generation in required formats for government submission

#### Maintainability
- **NFR-35**: Code follows consistent style and conventions
- **NFR-36**: Comprehensive inline documentation and comments
- **NFR-37**: Clear separation of concerns (MVC architecture)
- **NFR-38**: Version control and change tracking
- **NFR-39**: Test coverage for critical functions

## Major System Requirements

1. **Identity Verification System**: Automated verification of government-issued documents with AI support
2. **Multi-Role Access Control**: Three distinct user roles with granular permissions
3. **Secure Document Storage**: Cloud-based storage with encryption and version control
4. **Payment Processing**: Integration with payment systems and reconciliation
5. **Notification Engine**: Email and in-system notification delivery
6. **Audit & Compliance**: Complete logging of all system activities for regulatory review
7. **Analytics Dashboard**: Real-time insights into enrollment metrics and system health
8. **Batch & Course Management**: Support for multiple enrollment batches and course offerings
9. **Report Generation**: Export capabilities for administrative and compliance reporting
10. **External API Integration**: Google Gemini, Google Cloud Storage, Cloudflare Turnstile, SMTP

---

These requirements form the foundation of the system design and implementation, ensuring that all development decisions align with institutional needs and user expectations.
