# Study Vault — Web & Native Android Mobile System

**Digital Thesis and Research Repository for Isabela State University (Ilagan Campus)**

This repository contains:
1. **`study-vault/` (`study-vault.zip`)** — The PHP + MySQL web application and RESTful JSON API backend (XAMPP / Apache + MariaDB).
2. **`StudyVault/` (`StudyVault.zip`)** — The native Android mobile application written strictly in **Java** with **Android XML layouts** and **Gradle Kotlin DSL (`.gradle.kts`)**.

---

## 1. System Architecture & Key Alignment Updates

### Web Backend (`study-vault/`)
- **Revocable Mobile Bearer Token Authentication (`includes/auth.php`)**:
  - Automatically creates and manages the `mobile_tokens` table (`token_hash` SHA-256, `student_id`, `device_name`, `expires_at`, `last_used_at`).
  - `api_require_student($pdo)` supports both PHP session cookies (`STUDYVAULTSESSID`) and `Authorization: Bearer <token>` / `X-Auth-Token` headers, verifying on every request that the student account is `approved`.
- **Complete Mobile JSON API (`study-vault/api/`)**:
  - `POST /api/login.php`: Authenticates by `student_id` and `password`, enforces brute-force throttling (`login_attempts`), blocks `pending` and `rejected` accounts with descriptive status codes (`pending_approval`, `account_rejected`, `throttled`), and returns the student profile, active subscription, and Bearer token.
  - `POST /api/logout.php`: Revokes the mobile Bearer token and destroys the PHP session.
  - `POST /api/verify.php`: Implements the 3-step Student ID email verification workflow (`send_code`, `resend`, `verify_code`, `create_account`, `restart`) against `school_student_records` and `verification_codes` with 60-second resend cooldown, 10-minute code TTL, and signed `verification_token` fallback.
  - `POST /api/register.php`: Delegates to `verify.php` or returns HTTP `410 Gone` directing clients to Student ID verification.
  - `GET /api/home.php`: Returns authenticated student summary, active/expired subscription state, repository counters, dynamic subject categories (`categories` with approved counts), publication years, and recently added approved research.
  - `GET /api/research-list.php`: Browses and searches `approved` research by title, abstract, author, or keywords (`q`), filters by `category` and `year`, sorts (`recent`, `oldest`, `title`), paginates (`page`, `per_page`), and returns dynamic filter options (`filters.categories`, `filters.years`).
  - `GET /api/research-detail.php`: Returns full metadata, abstract, parsed `keyword_list`, related studies, and server-verified manuscript access status (`can_view_manuscript`, `access_code`, `access_reason`).
  - `GET /api/categories.php`: Returns all subject categories with approved research counts.
  - `GET /api/suggestions.php`: Autocomplete title suggestions for approved research.
  - `GET /api/plans.php`: Returns active access plans (**Free Access ₱0**, **Research Access ₱99 / 30 days**, **Research Access Plus ₱249 / 90 days**), current/expired subscription details, and prototype checkout notice.
  - `POST /api/checkout.php`: Prototype/demonstration plan activation (`confirm_demo=1`) that records a subscription row with reference code (`SV-YYYYMMDD-XXXXXXXX`) and stacked expiration date.
  - `GET /api/profile.php`: Returns full student profile, verification status, active subscription, and subscription history.
  - `GET /api/manuscript.php` & `secure-document.php`: Server-authorized manuscript metadata (`mode=meta` with personalized watermark lines) and protected PDF binary streaming (`mode=stream`) for users with an active paid subscription.

### Native Android App (`StudyVault/`)
- **100% Native Java + XML Layouts**:
  - `SplashActivity`: Institutional branding, local session check, and server profile validation before routing.
  - `LoginActivity`: Student ID + password sign-in, show/hide password toggle, server error banners (`pending_approval`, `account_rejected`, `throttled`), link to Student ID verification, and Server Connection Settings dialog.
  - `VerifyAccountActivity`: Native 3-step Student ID verification (`send_code` → 6-digit email `verify_code` with 60s resend timer → `create_account` password setup).
  - `MainActivity` + 4 Bottom Navigation Fragments:
    - `HomeFragment`: Personalized greeting, role badge, active access plan indicator, repository statistics, hero search bar, dynamic subject area tiles, and recently added approved research.
    - `SearchFragment`: Keyword search with live suggestions, dynamic Subject Area & Publication Year spinners, sort selector, clear filters button, and pagination.
    - `PlansFragment` + `CheckoutActivity`: Live access plans from backend, current subscription banner, and clearly labeled prototype/demo checkout workflow.
    - `ProfileFragment`: Student identity, email, Student ID, user type (`Enrolled IT Student` / `IT Alumni`), program, verification & approval badges, active plan summary, subscription history, server URL settings, and logout confirmation dialog.
  - `ResearchDetailActivity`: Full title, authors, program, publication date, abstract (open to all authenticated users), clickable keyword chips, related studies, and subscription-gated manuscript access box.
  - `ManuscriptViewerActivity`: Protected native PDF reader using Android `PdfRenderer` + `FLAG_SECURE` (blocks screenshots/screen recording), diagonal `WatermarkOverlayView` + bitmap-baked watermarks (`STUDY VAULT · Authorized User: email · Student ID · CONFIDENTIAL`), zoom controls (`75%`–`200%`), page navigation, and automatic cache cleanup on exit.

---

## 2. Setup Instructions (XAMPP & Android Studio)

### Step 1: Configure XAMPP Web Backend
1. Copy the `study-vault/` folder into your XAMPP `htdocs` directory:
   - Windows: `C:\xampp\htdocs\study-vault\`
   - macOS / Linux: `/Applications/XAMPP/htdocs/study-vault/` or `/opt/lampp/htdocs/study-vault/`
2. Start **Apache** and **MySQL** in the XAMPP Control Panel.
3. Open **phpMyAdmin** (`http://localhost/phpmyadmin`) and import:
   - `study-vault/database/study_vault.sql` (creates the `study_vault` database and all tables including `students`, `school_student_records`, `verification_codes`, `mobile_tokens`, `research`, `categories`, `access_plans`, and `subscriptions`).
   - *(If upgrading an existing database, import `study-vault/database/upgrade.sql` and `study-vault/database/school-db.sql`.)*
4. Verify database credentials in `study-vault/config/database.php` (default: host `127.0.0.1`, database `study_vault`, user `root`, password `""`).
5. *(Optional — Email Verification SMTP)*: Configure your Gmail App Password in `study-vault/config/mail.local.php` so 6-digit verification codes are delivered via SMTP.

### Step 2: Open & Run in Android Studio
1. Open **Android Studio** and select **Open** → choose the `StudyVault/` folder.
2. Wait for Gradle sync to complete (`Gradle Kotlin DSL` with Android Gradle Plugin).
3. **Connecting from Android Emulator vs. Physical Android Device**:
   - **Android Emulator**: The app automatically detects the emulator and uses:
     ```
     http://10.0.2.2/study-vault/api/
     ```
   - **Physical Android Phone (USB / Wi-Fi on same LAN)**:
     1. Find your computer's local Wi-Fi IPv4 address (`ipconfig` on Windows or `ifconfig` / `ip a` on macOS/Linux, e.g., `192.168.100.197`).
     2. Ensure your phone and computer are connected to the same Wi-Fi network and Apache port `80` is allowed through Windows Firewall.
     3. On the Study Vault **Login Screen** (tap **⚙ Server Connection Settings**) or **Profile Screen** (tap **Server Connection Settings**), enter:
        ```
        http://<YOUR-PC-LAN-IP>/study-vault/api/
        ```
4. Click **Run ('app')** in Android Studio.
