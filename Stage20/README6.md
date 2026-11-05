# Báo Cáo An Toàn Mã Nguồn & Quy Trình Chuẩn Bị Push GitHub Public

## 1. Pre-push Checklist: Quét API Keys & Secrets Hardcode

Trước khi thực hiện `git push` lên repository công khai (Public Repository), toàn bộ lập trình viên phải thực hiện quy trình 4 bước kiểm tra sau:

```
[1. Quét tự động bằng Gitleaks/Trufflehog] ➔ [2. Kiểm tra Git Staging] ➔ [3. Kiểm tra Git History] ➔ [4. Cấu hình Git Hooks]
```

### 1.1. Công cụ quét tự động (Automated Scanning Tools)

* **Sử dụng Gitleaks (Khuyên dùng):**
  ```bash
  # Cài đặt gitleaks
  brew install gitleaks # macOS
  # Hoặc sử dụng Docker / binary trên Windows/Linux

  # Quét toàn bộ repository trước khi push
  gitleaks detect --source . -v
  ```

* **Sử dụng TruffleHog:**
  ```bash
  trufflehog git file://. --since-commit HEAD~5
  ```

### 1.2. Kiểm tra danh sách file trước khi commit
```bash
# Kiểm tra các file đang ở trạng thái Staged
git status

# Kiểm tra nội dung chi tiết sắp commit xem có chứa chuỗi nhạy cảm (API_KEY, SECRET, PASSWORD...)
git diff --staged | grep -E "(API_KEY|SECRET|PASSWORD|PRIVATE_KEY|DATABASE_URL)"
```

### 1.3. Cấu hình Pre-push Git Hook tự động

Tạo tệp `.git/hooks/pre-push` (hoặc cấu hình qua Husky trong dự án Node.js) để chặn thao tác push nếu phát hiện bí mật:

```bash
#!/bin/sh
# .git/hooks/pre-push
echo "Đang quét API Keys và Secrets trước khi push..."

if command -v gitleaks >/dev/null 2>&1; then
    gitleaks protect --verbose --staged
    if [ $? -ne 0 ]; then
        echo "❌ ERROR: Phát hiện API Key hoặc Secret bị hardcode! Lệnh push bị hủy."
        exit 1
    fi
else
    echo "⚠️ Warning: Chưa cài đặt Gitleaks. Vui lòng cài đặt để đảm bảo an toàn."
fi
```

---

## 2. Cấu Trúc Tệp `.env.example` Chuyên Nghiệp

Tệp `.env.example` được commit lên GitHub để làm khuôn mẫu cho các môi trường, tuyệt đối **KHÔNG chứa giá trị thực (Sensitive Values)**.

### Tệp `.env.example` Mẫu
```ini
# ==============================================================================
# ENVIRONMENT CONFIGURATION TEMPLATE
# Do NOT place actual passwords or secret keys in this file.
# Copy this file to .env and fill in the actual values.
# ==============================================================================

# ------------------------------------------------------------------------------
# APP CONFIGURATION
# ------------------------------------------------------------------------------
APP_ENV=development # Options: development, testing, staging, production
APP_NAME="E-Commerce API Service"
APP_PORT=8080
APP_URL=http://localhost:8080

# ------------------------------------------------------------------------------
# DATABASE CREDENTIALS
# ------------------------------------------------------------------------------
DB_HOST=127.0.0.1
DB_PORT=5432
DB_NAME=ecommerce_db
DB_USER=your_db_user_here
DB_PASSWORD=your_db_password_here

# ------------------------------------------------------------------------------
# SECURITY & AUTHENTICATION (JWT / TOKENS)
# ------------------------------------------------------------------------------
# Generate a strong 256-bit key using: openssl rand -hex 32
JWT_SECRET=your_jwt_secret_key_placeholder
JWT_EXPIRATION_HOURS=24
REFRESH_TOKEN_SECRET=your_refresh_token_secret_placeholder

# ------------------------------------------------------------------------------
# PAYMENT GATEWAY INTEGRATION (STRIPE / PAYPAL / VNPAY)
# ------------------------------------------------------------------------------
STRIPE_PUBLIC_KEY=pk_test_placeholder_key
STRIPE_SECRET_KEY=sk_test_placeholder_key
STRIPE_WEBHOOK_SECRET=whsec_placeholder_key

# ------------------------------------------------------------------------------
# MAIL SERVICE (SMTP / SENDGRID)
# ------------------------------------------------------------------------------
SMTP_HOST=smtp.mailtrap.io
SMTP_PORT=2525
SMTP_USER=your_smtp_username
SMTP_PASS=your_smtp_password
MAIL_FROM_ADDRESS=noreply@example.com

# ------------------------------------------------------------------------------
# REDIS CACHE CONFIGURATION
# ------------------------------------------------------------------------------
REDIS_HOST=127.0.0.1
REDIS_PORT=6379
REDIS_PASSWORD=your_redis_password_here
```

---

## 3. Hướng Dẫn Cấu Hình Bắt Buộc 2FA Cho Repository GitHub

Để đảm bảo mọi tài khoản cộng tác viên (Collaborators/Team members) tham gia vào dự án đều phải bảo vệ bằng xác thực 2 yếu tố:

### Các bước thiết lập trên GitHub (Dành cho Organization / Repo Owner):

1. **Đối với GitHub Organization:**
   * Truy cập **Organization Settings** ➔ **Org security**.
   * Tại mục **Two-factor authentication**, tích chọn **Require two-factor authentication for everyone in your organization**.
   * Nhấn **Save**. Những thành viên chưa bật 2FA sẽ có thời gian gia hạn (grace period) trước khi bị gỡ quyền truy cập.

2. **Đối với Repository cá nhân/Dự án riêng:**
   * Thêm chính sách quy định trong tệp `CONTRIBUTING.md`.
   * Truy cập **Repository Settings** ➔ **Branches** ➔ Thêm **Branch protection rule** cho branch `main`/`master`.
   * Bật **Require a pull request before merging** và **Require status checks to pass before merging** (kết hợp với GitHub Actions quét 2FA / Secret Scanning).

---

## 4. Cấu Trúc Repository Mẫu & Báo Cáo Đánh Giá Rủi Ro Khi Thiếu `.gitignore`

### 4.1. Cấu trúc Repository An Toàn
```text
ecommerce-project/
├── .github/
│   ├── workflows/
│   │   └── secret-scan.yml      # CI/CD tự động quét secret bằng Gitleaks
│   └── CODEOWNERS
├── src/
│   └── ...                      # Mã nguồn ứng dụng
├── .env.example                 # File mẫu cấu hình (đã xoá thông tin nhạy cảm)
├── .gitignore                   # Chặn commit các file nhạy cảm và build artifacts
├── CONTRIBUTING.md              # Hướng dẫn đóng góp & yêu cầu an toàn 2FA
├── README.md                    # Tài liệu dự án
└── package.json
```

### 4.2. Báo Cáo Đánh Giá Rủi Ro Khi Thiếu Hoặc Cấu Hình Sai Tệp `.gitignore`

| Hậu Quả / Rủi Ro | Phân Tích Chi Tiết | Mức Độ Nguy Hiểm |
| ----- | ----- | ----- |
| **Rò rỉ API Keys & Database Credentials** | File `.env` chứa mật khẩu DB, Stripe Secret Key, AWS Access Keys bị commit lên public repo. Hacker sử dụng bot quét tự động (GitHub Bots) sẽ phát hiện ra chỉ trong vài giây. | 🚨 **CRITICAL (Thảm họa)** |
| **Tấn công chiếm quyền hạ tầng (Cloud Takeover)** | Đã có nhiều trường hợp lập trình viên lộ AWS Key trên GitHub Public, bị bot chiếm quyền tạo hàng loạt instance để đào tiền điện tử (Cryptomining), gây thiệt hại tài chính lớn. | 🚨 **CRITICAL (Thảm họa)** |
| **Lộ dữ liệu cá nhân & Mã khóa JWT (Data Breach)** | Lộ `JWT_SECRET` cho phép hacker tự tạo Token giả danh bất kỳ người dùng/Admin nào trong hệ thống mà không cần mật khẩu. | 🔴 **HIGH (Rất cao)** |
| **Phát tán thư mục `node_modules` / Build Artifacts** | Đẩy hàng chục ngàn file rác lên repo gây phình to dung lượng git, xung đột phiên bản giữa các môi trường OS khác nhau. | 🟡 **MEDIUM (Trung bình)** |

### Cấu hình tệp `.gitignore` chuẩn mẫu cho dự án:
```gitignore
# Environment variables (TỆP QUAN TRỌNG NHẤT)
.env
.env.local
.env.development.local
.env.test.local
.env.production.local

# Dependency directories
node_modules/
jspm_packages/

# Build and production output
dist/
build/
*.logs

# IDE & OS Files
.vscode/
.idea/
.DS_Store
Thumbs.db

# Keys & Certificates
*.pem
*.key
*.p12
```