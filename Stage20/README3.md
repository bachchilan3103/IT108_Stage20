# Báo Cáo Phân Tích Quy Trình Khôi Phục Quyền Truy Cập (IPO Model) & Quản Lý Backup Codes

## 1. Tổng Quan Bối Cảnh

* **Vấn đề:** Lead Developer bị mất điện thoại chứa ứng dụng Google Authenticator (thiết bị duy nhất lưu trữ mã OTP 2FA).
* **Mục tiêu:** Sử dụng **Backup Codes** (Mã khôi phục dự phòng) để đăng nhập vào tài khoản AWS khôi phục quyền quản trị, đồng thời thu hồi/khóa thiết bị bị mất nhằm đảm bảo an toàn tuyệt đối cho hệ thống.

---

## 2. Báo Cáo Phân Tích Quy Trình Khôi Phục Theo Mô Hình IPO

```
 ┌─────────────────────────────────────────────────────────┐
 │                      INPUT (Đầu vào)                    │
 │ 1. Tên đăng nhập / Root Email + Password                │
 │ 2. Một mã Backup Code hợp lệ (chưa qua sử dụng)        │
 └────────────────────────────┬────────────────────────────┘
                              │
                              ▼
 ┌─────────────────────────────────────────────────────────┐
 │                     PROCESS (Xử lý)                     │
 │ Bước 1: Xác thực đăng nhập qua Backup Code              │
 │ Bước 2: Hủy bỏ thiết bị 2FA cũ (Vô hiệu hóa Google Auth) │
 │ Bước 3: Đổi Password tài khoản                           │
 │ Bước 4: Khởi tạo thiết bị 2FA mới (Re-enrollment)       │
 │ Bước 5: Cấp lại bộ Backup Codes mới & Hủy mã cũ          │
 └────────────────────────────┬────────────────────────────┘
                              │
                              ▼
 ┌─────────────────────────────────────────────────────────┐
 │                     OUTPUT (Đầu ra)                     │
 │ 1. Đăng nhập thành công, khôi phục quyền bảo trì AWS.   │
 │ 2. Thiết bị cũ bị hủy quyền (Kẻ nhặt điện thoại bó tay).│
 │ 3. Hệ thống AWS được cập nhật bộ 2FA & Backup Code mới. │
 └─────────────────────────────────────────────────────────┘
```

### Phân Tích Chi Tiết Các Bước Trọng Tâm Trong Phần Process

1. **Bước 1: Xác thực khôi phục (Emergency Login)**
   * Đăng nhập bằng tài khoản AWS với `Username`/`Password`.
   * Khi màn hình hỏi mã OTP 2FA xuất hiện, chọn phương thức *Troubleshoot MFA* / *Use Backup Code*.
   * Nhập 1 mã Backup Code ngẫu nhiên trong danh sách dự phòng đã lưu trữ từ trước (ở nơi an toàn ngoài điện thoại). Mã này hoạt động như chìa khóa đơn (Single-use token).

2. **Bước 2: Thu hồi quyền truy cập của thiết bị mất (Revocation)**
   * Ngay sau khi vào được console AWS IAM / Root Security Credentials, lập tức chọn **Deactivate MFA Device** đối với thiết bị Google Authenticator trên điện thoại đã mất.
   * Thao tác này ngay lập tức vô hiệu hóa khả năng tạo mã OTP của kẻ nhặt được điện thoại.

3. **Bước 3: Đổi mật khẩu tài khoản (Password Reset)**
   * Thực hiện đổi ngay mật khẩu đăng nhập AWS để ngăn ngừa rủi ro điện thoại bị mất có lưu sẵn session hoặc ghi chú mật khẩu.

4. **Bước 4: Cấu hình thiết bị 2FA mới (Re-enrollment)**
   * Cấu hình thiết bị 2FA mới (điện thoại mới hoặc YubiKey/Hardware Token) bằng cách quét mã QR mới do AWS cấp.

5. **Bước 5: Tạo mới và Lưu trữ bộ Backup Codes (Regenerate Backup Codes)**
   * Chọn **Generate New Backup Codes** trên hệ thống AWS.
   * Hành động này sẽ tự động **hủy toàn bộ các mã Backup Codes cũ** (bao gồm cả mã chưa dùng).
   * Lưu trữ bộ mã mới vào môi trường bảo mật đạt chuẩn.

---

## 3. Rủi Ro Khi Lưu Backup Codes Dưới Dạng Ảnh Trong Thư viện Điện Thoại

Việc chụp ảnh màn hình (screenshot) hoặc lưu hình ảnh bộ Backup Codes trong bộ sưu tập ảnh (Camera Roll/Gallery) của điện thoại là một **lỗi bảo mật nghiêm trọng** vì những lý do sau:

### 1. Rủi ro rò rỉ khi mất thiết bị (Physical Access)
* Khi điện thoại rơi vào tay kẻ khác, nếu họ mở khóa được màn hình (hoặc cắm thẻ nhớ/kết nối máy tính qua giao thức MTP/ADB), họ có thể dễ dàng duyệt thư viện ảnh và tìm thấy tấm ảnh chứa mã dự phòng.
* Kết hợp với mật khẩu hoặc các thông tin cá nhân khác rò rỉ trên điện thoại, kẻ nhặt được có đủ thông tin để đăng nhập và chiếm quyền kiểm soát AWS.

### 2. Tự động đồng bộ lên Cloud (Cloud Auto-Sync Exposure)
* Hầu hết điện thoại thông minh đều tự động bật tính năng đồng bộ ảnh lên các dịch vụ đám mây (Google Photos, iCloud, OneDrive).
* Nếu tài khoản Cloud bị hacker xâm nhập hoặc bị lộ mật khẩu, toàn bộ thư viện ảnh chứa Backup Codes sẽ bị quét và thu thập tự động.

### 3. Nguy cơ từ phần mềm độc hại (Malware & App Permissions)
* Rất nhiều ứng dụng di động (đặc biệt là ứng dụng chỉnh sửa ảnh, game, công cụ miễn phí) yêu cầu quyền truy cập thư viện ảnh (`READ_EXTERNAL_STORAGE` hoặc Photo Library Access).
* Đã có nhiều dòng mã độc (Malware/Spyware) chạy ngầm trên điện thoại thực hiện quét toàn bộ ảnh trong thư viện để tìm kiếm các hình ảnh chứa chữ dạng số, mã QR, hoặc từ khóa như "Backup Code", "Passcode", "Secret Key" bằng công nghệ OCR (Optical Character Recognition).

---

## 4. Khuyến Nghị Phương Án Lưu Trữ Backup Codes An Toàn

* **Trình quản lý mật khẩu (Password Manager):** Lưu mã Backup Codes dưới dạng ghi chú bảo mật (Secure Note) trong các ứng dụng như Bitwarden, 1Password (được bảo vệ bằng chuẩn mã hóa AES-256).
* **Lưu trữ ngoại tuyến (Offline Physical Vault):** In bộ mã ra giấy và cất giữ trong két sắt bảo mật hoặc tủ khóa cá nhân.
* **Mã hóa file độc lập:** Lưu file text chứa mã trong một ổ đĩa được mã hóa (Veracrypt) hoặc USB bảo mật bằng phần cứng.