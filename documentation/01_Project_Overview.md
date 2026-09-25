# 01 Project Overview

## System Description

The **Dulag Polytechnic Enrollment System** is a comprehensive, web-based student enrollment and management platform designed specifically for Dulag Polytechnic. This system automates and streamlines the entire student lifecycle—from initial registration through enrollment, billing, course management, and graduation tracking. It serves as the central hub for all enrollment-related operations across the institution.

The system is built with a focus on security, compliance, and user experience. It integrates modern authentication mechanisms, document verification technologies, and role-based access control to ensure data integrity and institutional standards.

## System Purpose

The primary purpose of this system is to:

1. **Digitize Enrollment Operations**: Replace manual, paper-based enrollment processes with an automated, digital workflow
2. **Ensure Student Validation**: Implement multi-step identity verification using government-issued documents, biometric data, and medical certificates
3. **Manage Academic Data**: Handle course enrollment, grades, transcripts, and academic records
4. **Process Billing & Payments**: Manage tuition billing, payment processing, and financial records
5. **Provide Role-Based Access**: Implement different portal experiences for students, administrators, and system administrators
6. **Generate Compliance Reports**: Create audit trails, analytics, and compliance documentation (TESDA standards)
7. **Ensure Data Security**: Protect sensitive student and institutional data through encryption, RBAC, and access controls

## Target Users

### Student Users
Students use the system to:
- Create and manage their enrollment accounts
- Submit registration and enrollment applications
- Upload required documents (birth certificates, medical certificates, transcripts)
- Verify their identity through document and biometric authentication
- View course enrollment status and billing information
- Pay tuition fees online
- Download certificates and transcripts

### Administrative Users (Admin Role)
Administrators manage day-to-day operations:
- Review and process student enrollments and applications
- Manage payment transactions and billing
- Enter grades and manage academic records
- Send notifications to students
- View student information and enrollment history
- Access system analytics and reports

### System Administrators (Super Admin Role)
System administrators perform high-level institutional management:
- Manage course catalogs and batch groups
- Oversee all enrollments and applications across the institution
- Configure system settings and parameters
- Manage administrative accounts and permissions
- Access comprehensive analytics and export reports
- Perform document validation and requirement review
- System maintenance and troubleshooting

## Main Purpose & Vision

This system was developed to transform Dulag Polytechnic's enrollment operations from a manual, time-consuming process into a streamlined, secure, and auditable digital system. The vision is to:

- **Reduce Administrative Burden**: Automate repetitive tasks and reduce manual data entry
- **Improve Student Experience**: Provide students with a modern, intuitive interface for managing their enrollment
- **Ensure Compliance**: Meet TESDA (Technical Education and Skills Development Authority) standards and government regulations
- **Enhance Data Quality**: Implement validation checks and document verification to maintain accurate records
- **Enable Analytics**: Provide institution leaders with insights into enrollment trends and system performance
- **Scale Efficiently**: Build a foundation that can grow with institutional needs

## Technical Foundation

- **Framework**: CodeIgniter 3 (PHP)
- **Database**: MySQL/MariaDB with relational schema
- **Frontend**: Bootstrap 4, jQuery, responsive design
- **Security**: RBAC (Role-Based Access Control), HTTPS, input validation, prepared statements
- **External Integrations**: Google Gemini API (document analysis), Google Cloud Storage (file storage), Cloudflare Turnstile (CAPTCHA)
- **Document Processing**: FPDF for PDF generation, image processing for document analysis
- **Deployment**: Apache/XAMPP local, ready for production hosting

## Key Features Summary

1. **Multi-Step Authentication**: OTP verification, password management, security questions
2. **Document Verification**: Automated document analysis using AI
3. **Identity Verification**: Birth certificate, medical certificate, ID verification, selfie verification
4. **Student Application Forms**: Multi-section TESDA-compliant registration forms
5. **Enrollment Management**: Course selection, batch management, enrollment processing
6. **Billing & Payments**: Tuition calculation, payment processing, receipt generation
7. **Academic Tracking**: Grade entry, transcript generation, course history
8. **Notification System**: Email and in-system notifications
9. **Audit Logging**: Comprehensive activity logging for compliance
10. **Analytics & Reporting**: Statistical dashboards and data export capabilities

---

*This documentation serves as a comprehensive technical and operational guide for developers, administrators, and stakeholders involved with the Dulag Polytechnic Enrollment System.*
