# 08 Technical Implementation

## Technology Stack

### Backend Technologies
- **Language**: PHP 7.2+ (CodeIgniter 3.1.11)
- **Framework**: CodeIgniter 3 (MVC architecture)
- **Database**: MySQL 5.7+ / MariaDB
- **Server**: Apache 2.4+ with mod_rewrite
- **Email**: SMTP (PHPMailer integration)

### Frontend Technologies
- **HTML5**: Semantic markup
- **CSS**: Bootstrap 4.5, Custom CSS
- **JavaScript**: jQuery 3.x, AJAX for asynchronous operations
- **Icons**: Font Awesome 5
- **Charts**: Chart.js for analytics visualizations
- **File Handling**: Dropzone.js for drag-drop uploads

### External Services & APIs
- **AI/ML**: Google Gemini API (document analysis)
- **File Storage**: Google Cloud Storage (document archival)
- **Email**: SMTP (Gmail or institutional email)
- **CAPTCHA**: Cloudflare Turnstile (bot protection)

### Development & Deployment Tools
- **Version Control**: Git (GitHub)
- **Database Admin**: phpMyAdmin / Adminer
- **API Testing**: Postman
- **Local Development**: XAMPP (Apache + MySQL + PHP)

---

## Authentication Implementation

### User Registration Flow

```php
// application/controllers/Welcome.php - register() method

1. Student enters email
   └─ Check if email already exists in users table
   
2. If unique:
   └─ Generate random 6-digit OTP
   └─ Store OTP with 10-minute expiry in otp table
   └─ Send OTP via SMTP
   └─ Redirect to OTP verification page

3. Student enters OTP
   └─ Validate OTP code
   └─ Check expiry timestamp
   └─ Verify not already used
   └─ If valid: Mark OTP as used, proceed to password creation
   └─ If invalid: Show error, request new OTP

4. Student creates password
   └─ Validate password strength:
      ├─ Minimum 8 characters
      ├─ At least one uppercase letter
      ├─ At least one lowercase letter
      ├─ At least one number
      └─ At least one special character
   
   └─ Hash password using bcrypt:
      password_hash($password, PASSWORD_BCRYPT, ['cost' => 12])
   
   └─ Create user record:
      INSERT INTO users (email, password, role, is_validated)
      VALUES ($email, $hashed_password, 'STUDENT', 0)
   
   └─ Create session:
      $_SESSION['user_id'] = $user_id
      $_SESSION['role'] = 'STUDENT'
   
   └─ Redirect to account validation page
```

### Login Implementation

```php
// application/controllers/Welcome.php - login() method

1. User enters email and password
   └─ Validate inputs (not empty)
   
2. Query database:
   SELECT * FROM users WHERE email = :email LIMIT 1
   
3. If user found:
   └─ Verify password using bcrypt:
      password_verify($input_password, $database_hash)
   
   └─ If password correct:
      ├─ Create session:
         $_SESSION['user_id'] = $user->id
         $_SESSION['role'] = $user->role
         $_SESSION['logged_in_at'] = time()
      
      ├─ Update last_known_ip:
         UPDATE users SET last_known_ip = :ip WHERE id = :user_id
      
      ├─ Log activity:
         INSERT INTO activity_log (user_id, action, ip_address, ...)
         VALUES ($user_id, 'LOGIN', $ip_address, ...)
      
      ├─ Check if IP changed:
         If different from last_known_ip: Alert user of unusual login
      
      ├─ Determine redirect based on role:
         - STUDENT → /student/index
         - ADMIN → /admin/index
         - SUPERADMIN → /superadmin/index
   
   └─ If password incorrect:
      ├─ Log failed attempt:
         INSERT INTO activity_log (...) VALUES (..., 'LOGIN_FAILED', ...)
      
      ├─ Implement rate limiting:
         - Track failed attempts per IP/email
         - Lock account after 5 failed attempts in 15 minutes
      
      └─ Display error: "Invalid email or password"
```

### Session Management

```php
// application/core/MY_Controller.php - BaseController constructor

class MY_Controller extends CI_Controller {
    public function __construct() {
        parent::__construct();
        
        // Check session
        if (!isset($_SESSION['user_id'])) {
            redirect('welcome/login');
        }
        
        // Check session timeout (30 minutes)
        $session_timeout = 30 * 60; // 1800 seconds
        if (time() - $_SESSION['logged_in_at'] > $session_timeout) {
            session_destroy();
            redirect('welcome/logout');
        }
        
        // Update last activity
        $_SESSION['last_activity'] = time();
    }
}
```

### CSRF Protection

```php
// CSRF tokens on all forms
// CodeIgniter automatically adds CSRF field to forms

1. Generate token:
   <?= form_hidden('csrf_token', $this->security->get_csrf_token_name()); ?>
   
2. In controller, validate before processing:
   $this->security->csrf_verify();
   
3. If CSRF check fails:
   └─ Throw exception
   └─ Log activity (possible attack)
   └─ Return 403 Forbidden
```

---

## Authorization Implementation (RBAC)

### Permission Checking Middleware

```php
// application/core/MY_Controller.php - Permission checks

private function _require_student() {
    if (!isset($_SESSION['role']) || $_SESSION['role'] !== 'STUDENT') {
        show_error('Unauthorized access', 403);
    }
}

private function _require_admin() {
    if (!isset($_SESSION['role']) || $_SESSION['role'] !== 'ADMIN') {
        show_error('Unauthorized access', 403);
    }
}

private function _require_superadmin() {
    if (!isset($_SESSION['role']) || $_SESSION['role'] !== 'SUPERADMIN') {
        show_error('Unauthorized access', 403);
    }
}

// Usage in controller methods:
public function admin_function() {
    $this->_require_admin(); // Permission check
    // Function code here
}
```

### Data-Level Access Control

```php
// application/models/Enrollment_model.php

public function get_for_student(int $user_id): array {
    // Students can only view their own enrollments
    return $this->db->get_where('enrollments', ['user_id' => $user_id])->result();
}

public function get_all(): array {
    // Admins can view all enrollments
    return $this->db->get('enrollments')->result();
}

// In controller:
public function view_enrollments() {
    $role = $_SESSION['role'];
    $user_id = $_SESSION['user_id'];
    
    if ($role === 'STUDENT') {
        $enrollments = $this->Enrollment_model->get_for_student($user_id);
    } elseif ($role === 'ADMIN' || $role === 'SUPERADMIN') {
        $enrollments = $this->Enrollment_model->get_all();
    }
}
```

---

## Input Validation & Security

### Server-Side Validation

```php
// In models and controllers

function validate_enrollment_data($data) {
    $errors = [];
    
    // Email validation
    if (empty($data['email'])) {
        $errors[] = "Email required";
    } elseif (!filter_var($data['email'], FILTER_VALIDATE_EMAIL)) {
        $errors[] = "Invalid email format";
    }
    
    // Password validation
    if (strlen($data['password']) < 8) {
        $errors[] = "Password must be at least 8 characters";
    } elseif (!preg_match('/[A-Z]/', $data['password'])) {
        $errors[] = "Password must contain uppercase letter";
    }
    
    // Course ID validation
    if (!is_numeric($data['course_id']) || $data['course_id'] <= 0) {
        $errors[] = "Invalid course selected";
    }
    
    return $errors;
}
```

### SQL Injection Prevention

```php
// Always use prepared statements with CodeIgniter Query Builder

// WRONG - Vulnerable to SQL injection:
$query = $this->db->query("SELECT * FROM users WHERE email = '".$email."'");

// CORRECT - Using Query Builder with parameterization:
$query = $this->db->where('email', $email)->get('users');

// Or with raw query:
$query = $this->db->query("SELECT * FROM users WHERE email = ?", array($email));
```

### XSS Prevention

```php
// Output encoding in views

// WRONG - Vulnerable to XSS:
<p><?php echo $user_input; ?></p>

// CORRECT - HTML entity encode output:
<p><?php echo htmlspecialchars($user_input, ENT_QUOTES, 'UTF-8'); ?></p>

// Or use CodeIgniter helper:
<p><?php echo html_escape($user_input); ?></p>

// In JSON responses:
echo json_encode($data, JSON_HEX_TAG | JSON_HEX_AMP | JSON_HEX_APOS | JSON_HEX_QUOT);
```

---

## Document Processing & AI Integration

### Document Upload & Validation

```php
// application/controllers/Student.php - upload_birth_certificate()

public function upload_birth_certificate() {
    $this->_require_student();
    $user_id = $_SESSION['user_id'];
    
    // File upload validation
    $config['upload_path'] = './uploads/temp/';
    $config['allowed_types'] = 'pdf|jpg|jpeg|png';
    $config['max_size'] = 5120; // 5MB
    $config['encrypt_name'] = TRUE;
    
    $this->load->library('upload', $config);
    
    if (!$this->upload->do_upload('document')) {
        $this->session->set_flashdata('error', $this->upload->display_errors());
        redirect('student/account_validation');
    }
    
    $upload_data = $this->upload->data();
    $file_path = $upload_data['full_path'];
    
    // Virus scanning (optional)
    if ($this->is_file_infected($file_path)) {
        unlink($file_path);
        $this->session->set_flashdata('error', 'File failed security scan');
        redirect('student/account_validation');
    }
    
    // Upload to Google Cloud Storage
    $cloud_path = $this->upload_to_gcs($file_path, 'birth-certificates');
    unlink($file_path); // Delete local temp file
    
    // Call AI analysis
    $ai_results = $this->analyze_with_gemini($cloud_path);
    
    // Store in database
    $this->BirthCertificate_model->save([
        'user_id' => $user_id,
        'document_url' => $cloud_path,
        'extracted_data' => json_encode($ai_results['data']),
        'ai_analysis_result' => json_encode($ai_results),
        'verification_status' => 'pending',
    ]);
    
    // Notify admin
    $this->notify_admin_of_submission('birth_certificate', $user_id);
}
```

### Google Gemini API Integration

```php
// application/libraries/GeminiAnalyzer.php

class GeminiAnalyzer {
    private $api_key;
    private $api_endpoint = 'https://generativelanguage.googleapis.com/v1beta/models';
    
    public function analyze_birth_certificate($image_url) {
        $prompt = "Analyze this birth certificate image and extract: full name, date of birth, gender, certificate ID number. Also assess document authenticity.";
        
        $request = [
            "contents" => [
                [
                    "parts" => [
                        ["text" => $prompt],
                        ["inline_data" => ["mime_type" => "image/jpeg", "data" => base64_encode(file_get_contents($image_url))]]
                    ]
                ]
            ]
        ];
        
        $curl = curl_init();
        curl_setopt($curl, CURLOPT_URL, "{$this->api_endpoint}/gemini-pro-vision:generateContent?key={$this->api_key}");
        curl_setopt($curl, CURLOPT_CUSTOMREQUEST, 'POST');
        curl_setopt($curl, CURLOPT_POSTFIELDS, json_encode($request));
        curl_setopt($curl, CURLOPT_HTTPHEADER, ['Content-Type: application/json']);
        curl_setopt($curl, CURLOPT_RETURNTRANSFER, true);
        
        $response = curl_exec($curl);
        $http_code = curl_getinfo($curl, CURLINFO_HTTP_CODE);
        curl_close($curl);
        
        if ($http_code === 200) {
            $parsed = json_decode($response, true);
            return [
                'status' => 'success',
                'data' => $this->parse_gemini_response($parsed),
                'confidence' => $this->calculate_confidence($parsed)
            ];
        } else {
            return ['status' => 'error', 'message' => 'API analysis failed'];
        }
    }
    
    private function parse_gemini_response($response) {
        // Extract structured data from Gemini's response
        $text = $response['candidates'][0]['content']['parts'][0]['text'] ?? '';
        
        // Parse text using regex or NLP
        preg_match('/Name:\s*(.+)/i', $text, $name_match);
        preg_match('/DOB:\s*(.+)/i', $text, $dob_match);
        
        return [
            'full_name' => $name_match[1] ?? '',
            'date_of_birth' => $dob_match[1] ?? '',
            'raw_response' => $text
        ];
    }
}
```

### Google Cloud Storage Upload

```php
// application/libraries/CloudStorageManager.php

use Google\Cloud\Storage\StorageClient;

class CloudStorageManager {
    private $storage;
    private $bucket;
    
    public function __construct() {
        $this->storage = new StorageClient([
            'projectId' => getenv('GOOGLE_PROJECT_ID'),
            'keyFilePath' => getenv('GOOGLE_KEY_FILE'),
        ]);
        $this->bucket = $this->storage->bucket(getenv('GOOGLE_BUCKET_NAME'));
    }
    
    public function upload_document($local_file_path, $folder_type, $user_id) {
        $timestamp = date('Y-m-d-H-i-s');
        $filename = "{$folder_type}/{$user_id}/{$timestamp}-" . basename($local_file_path);
        
        $file = fopen($local_file_path, 'r');
        $object = $this->bucket->upload($file, [
            'name' => $filename,
            'metadata' => [
                'cacheControl' => 'no-cache',
                'customMetadata' => [
                    'uploadedBy' => $user_id,
                    'uploadedAt' => date('Y-m-d H:i:s'),
                ]
            ]
        ]);
        fclose($file);
        
        return "gs://{$this->bucket->name()}/{$filename}";
    }
    
    public function delete_document($gs_path) {
        // gs://bucket-name/path/to/file
        preg_match('/gs:\/\/([^\/]+)\/(.+)/', $gs_path, $matches);
        $object_name = $matches[2];
        
        $this->bucket->object($object_name)->delete();
    }
}
```

---

## Enrollment Validation Logic

### Prerequisite Checking

```php
// application/models/Enrollment_model.php

public function can_enroll_in_course(int $user_id, int $course_id, int $current_batch_id): bool {
    // Check 1: Is student validated?
    $student = $this->db->get_where('users', ['id' => $user_id])->row();
    if (!$student || !$student->is_validated) {
        return false; // Not yet identity verified
    }
    
    // Check 2: Does course have prerequisites?
    $course = $this->db->get_where('courses', ['id' => $course_id])->row();
    if ($course && $course->prerequisites) {
        $prerequisites = json_decode($course->prerequisites, true);
        
        foreach ($prerequisites as $prereq_id) {
            // Check if student completed prerequisite
            if (!$this->has_completed_course($user_id, $prereq_id)) {
                return false; // Missing prerequisite
            }
        }
    }
    
    // Check 3: No concurrent enrollment in same course
    $existing = $this->db
        ->where('user_id', $user_id)
        ->where('course_id', $course_id)
        ->where_in('status', ['pending_review', 'approved', 'enrolled'])
        ->get('enrollments')
        ->row();
    
    if ($existing) {
        return false; // Already enrolled or pending
    }
    
    // Check 4: Course capacity
    $enrollment_count = $this->db->where('course_id', $course_id)
                                   ->where('status', 'enrolled')
                                   ->count_all_results('enrollments');
    
    if ($course->capacity && $enrollment_count >= $course->capacity) {
        return false; // Course full
    }
    
    // Check 5: Enrollment period active
    $batch = $this->db->get_where('batch_groups', ['id' => $current_batch_id])->row();
    $now = new DateTime();
    $enrollment_start = new DateTime($batch->enrollment_start);
    $enrollment_end = new DateTime($batch->enrollment_end);
    
    if ($now < $enrollment_start || $now > $enrollment_end) {
        return false; // Outside enrollment window
    }
    
    return true; // All checks passed
}
```

---

## Payment Processing

### Calculate Tuition Amount

```php
// application/models/Payment_model.php

public function calculate_enrollment_amount($enrollment_ids): float {
    // Enrollment IDs can be array of course enrollments
    if (!is_array($enrollment_ids)) {
        $enrollment_ids = [$enrollment_ids];
    }
    
    $total = 0;
    
    foreach ($enrollment_ids as $enrollment_id) {
        $enrollment = $this->db
            ->select('courses.tuition_amount')
            ->from('enrollments')
            ->join('courses', 'enrollments.course_id = courses.id')
            ->where('enrollments.id', $enrollment_id)
            ->get()
            ->row();
        
        if ($enrollment) {
            $total += $enrollment->tuition_amount;
        }
    }
    
    return $total;
}
```

### Process Payment

```php
// application/controllers/Admin.php

public function payment_process() {
    $this->_require_admin();
    
    $user_id = $this->input->post('user_id');
    $amount = $this->input->post('amount');
    $method = $this->input->post('payment_method');
    $reference = $this->input->post('reference_number');
    
    // Validate amount
    if (!is_numeric($amount) || $amount <= 0) {
        return json_response(['status' => 'error', 'message' => 'Invalid amount']);
    }
    
    // Generate receipt number
    $receipt_number = 'RCP-' . date('YmdHis') . '-' . rand(1000, 9999);
    
    // Insert payment record
    $this->db->insert('payments', [
        'user_id' => $user_id,
        'amount_paid' => $amount,
        'payment_method' => $method,
        'payment_date' => date('Y-m-d'),
        'reference_number' => $reference,
        'receipt_number' => $receipt_number,
        'processed_by' => $_SESSION['user_id'],
        'created_at' => date('Y-m-d H:i:s'),
    ]);
    
    // Log activity
    $this->Activity_log_model->log([
        'user_id' => $_SESSION['user_id'],
        'action' => 'PAYMENT_PROCESSED',
        'resource' => 'payment',
        'resource_id' => $this->db->insert_id(),
        'details' => json_encode(['amount' => $amount, 'method' => $method]),
    ]);
    
    // Update enrollment status if full payment received
    $this->update_enrollment_status($user_id);
    
    // Generate receipt PDF
    $receipt_pdf = $this->generate_receipt_pdf($receipt_number, $user_id, $amount);
    
    // Send email
    $this->send_payment_confirmation_email($user_id, $receipt_number);
    
    return json_response([
        'status' => 'success',
        'receipt_number' => $receipt_number,
        'pdf_url' => $receipt_pdf
    ]);
}
```

---

## Notification System

### Send Email Notification

```php
// application/libraries/NotificationManager.php

class NotificationManager {
    private $ci;
    
    public function __construct() {
        $this->ci =& get_instance();
        $this->ci->load->library('email');
    }
    
    public function send_enrollment_approved_email($user_id, $enrollment_id) {
        $user = $this->ci->db->get_where('users', ['id' => $user_id])->row();
        $enrollment = $this->ci->db
            ->select('courses.course_title, batch_groups.batch_name')
            ->from('enrollments')
            ->join('courses', 'enrollments.course_id = courses.id')
            ->join('batch_groups', 'enrollments.batch_group_id = batch_groups.id')
            ->where('enrollments.id', $enrollment_id)
            ->get()
            ->row();
        
        $subject = "Enrollment Approved - {$enrollment->course_title}";
        $message = "Dear {$user->name},<br/>";
        $message .= "Your enrollment for {$enrollment->course_title} in {$enrollment->batch_name} has been approved.<br/>";
        $message .= "Please proceed to payment at your earliest convenience.<br/>";
        $message .= "Login to view payment details.";
        
        return $this->send_email($user->email, $subject, $message);
    }
    
    private function send_email($to, $subject, $message) {
        $this->ci->email->from(getenv('SENDER_EMAIL'));
        $this->ci->email->to($to);
        $this->ci->email->subject($subject);
        $this->ci->email->message($message);
        
        if ($this->ci->email->send()) {
            return true;
        } else {
            // Queue for retry
            $this->queue_email($to, $subject, $message);
            return false;
        }
    }
    
    private function queue_email($to, $subject, $message) {
        $this->ci->db->insert('email_queue', [
            'recipient' => $to,
            'subject' => $subject,
            'message' => $message,
            'status' => 'pending',
            'attempts' => 0,
            'created_at' => date('Y-m-d H:i:s'),
        ]);
    }
}
```

### Send In-System Notification

```php
// application/models/Notification_model.php

public function create_notification($user_id, $title, $message, $type = 'system') {
    $this->db->insert('notifications', [
        'user_id' => $user_id,
        'title' => $title,
        'message' => $message,
        'type' => $type,
        'is_read' => 0,
        'created_at' => date('Y-m-d H:i:s'),
    ]);
    
    return $this->db->insert_id();
}
```

---

## Audit Logging

### Log Activities

```php
// application/models/ActivityLog_model.php

public function log_activity($action, $resource, $resource_id, $details = []) {
    $this->db->insert('activity_log', [
        'user_id' => $_SESSION['user_id'] ?? null,
        'action' => $action,
        'resource' => $resource,
        'resource_id' => $resource_id,
        'details' => json_encode($details),
        'ip_address' => $this->get_client_ip(),
        'user_agent' => $_SERVER['HTTP_USER_AGENT'],
        'created_at' => date('Y-m-d H:i:s'),
    ]);
}

private function get_client_ip() {
    if (!empty($_SERVER['HTTP_CLIENT_IP'])) {
        $ip = $_SERVER['HTTP_CLIENT_IP'];
    } elseif (!empty($_SERVER['HTTP_X_FORWARDED_FOR'])) {
        $ip = $_SERVER['HTTP_X_FORWARDED_FOR'];
    } else {
        $ip = $_SERVER['REMOTE_ADDR'];
    }
    return $ip;
}
```

---

## PDF Generation (Receipts, Transcripts)

### Generate Receipt PDF

```php
// application/libraries/ReceiptGenerator.php

require_once APPPATH . 'third_party/fpdf/fpdf.php';

class ReceiptGenerator extends FPDF {
    public function generate_payment_receipt($payment_data) {
        $this->AddPage();
        $this->SetFont('Arial', 'B', 14);
        $this->Cell(0, 10, 'PAYMENT RECEIPT', 0, 1, 'C');
        
        $this->SetFont('Arial', '', 10);
        $this->Cell(0, 5, 'Dulag Polytechnic', 0, 1);
        $this->Cell(0, 5, 'Enrollment System', 0, 1);
        
        $this->Ln(10);
        $this->SetFont('Arial', 'B', 10);
        $this->Cell(50, 5, 'Receipt No: ');
        $this->SetFont('Arial', '', 10);
        $this->Cell(0, 5, $payment_data['receipt_number'], 0, 1);
        
        $this->SetFont('Arial', 'B', 10);
        $this->Cell(50, 5, 'Date: ');
        $this->SetFont('Arial', '', 10);
        $this->Cell(0, 5, date('M d, Y', strtotime($payment_data['payment_date'])), 0, 1);
        
        $this->Ln(10);
        $this->SetFont('Arial', 'B', 10);
        $this->Cell(50, 5, 'Student Name: ');
        $this->SetFont('Arial', '', 10);
        $this->Cell(0, 5, $payment_data['student_name'], 0, 1);
        
        // More details...
        
        $this->Ln(10);
        $this->SetFont('Arial', 'B', 12);
        $this->Cell(0, 5, 'Amount: ' . number_format($payment_data['amount'], 2), 0, 1, 'R');
        
        $this->Output('D', $payment_data['receipt_number'] . '.pdf');
    }
}
```

---

This technical implementation provides a solid foundation for a secure, scalable, and maintainable enrollment system. All code follows security best practices, proper error handling, and audit trail requirements.
