# Báo Cáo Phân Tích Lỗ Hổng URL & Quy Trình Kiểm Tra An Toàn

## 1. Phân Tích Ký Tự Lạ và Cơ Chế Tấn Công (IDN Homograph Attack)

### Ký tự bất thường trong URL
- **Ký tự lạ:** Ký tự **`ú`** (`U+00FA` - Latin Small Letter U with Acute) trong tên miền `https://githúb.com/login`.
- **Tên miền chuẩn:** `https://github.com` (Sử dụng 100% ký tự ASCII tiêu chuẩn).

### Lý do đánh lừa người dùng
1. **Trực quan thị giác (Visual Deception):** Chữ `ú` có hình dáng tương tự chữ `u` chuẩn. Khi nhìn lướt nhanh trên thanh địa chỉ của trình duyệt hoặc trên màn hình thiết bị di động, người dùng rất dễ bỏ qua chi tiết dấu sắc nhỏ này.
2. **Cơ chế mã hóa Punycode:**
   - Hệ thống DNS gốc chỉ làm việc với bộ ký tự ASCII. Để hỗ trợ các tên miền quốc tế chứa ký tự Unicode, hệ thống sử dụng chuẩn mã hóa **Punycode**.
   - Tên miền `githúb.com` thực tế được hệ thống chuyển đổi thành tên miền Punycode dạng: `xn--githb-1ua.com`.
   - Kẻ tấn công lợi dụng việc một số trình duyệt hiển thị dạng Unicode thân thiện thay vì hiển thị mã Punycode gốc (`xn--...`) để che đậy đường dẫn lừa đảo (Phishing).

---

## 2. Quy Trình Kiểm Tra URL Trước Khi Nhập Master Password

Để phòng tránh triệt để tấn công Homograph và các hình thức lừa đảo qua URL, quy trình kiểm tra chuẩn gồm 3 bước sau:

```
[Bước 1: Kiểm tra mắt thường] ➔ [Bước 2: Sử dụng Password Manager] ➔ [Bước 3: Công cụ kỹ thuật]
```

### Bước 1: Kiểm tra nhanh bằng mắt thường (Visual Inspection)
* **Soi kỹ phần Tên miền chính (Domain):** Kiểm tra từng ký tự xem có dấu phụ (`ú`, `à`, `ê`), ký tự Cyrillic/Greek trông giống ký tự Latinh hay không.
* **Nhận biết lỗi Typosquatting:** Kiểm tra việc thay thế chữ cái bằng số (như `g1thub.com`, `g00gle.com`) hoặc lặp/thiếu ký tự (`githuub.com`).
* **Đường dẫn chuẩn SSL/TLS:** Kiểm tra giao thức `https://` và biểu tượng khóa bảo mật, tuy nhiên **không phụ thuộc hoàn toàn** vào khóa SSL vì trang web giả mạo vẫn có thể đăng ký SSL hợp lệ.

### Bước 2: Kiểm tra tự động bằng Trình quản lý mật khẩu (Password Manager)
* **Cơ chế hoạt động:** Các công cụ quản lý mật khẩu uy tín (như Bitwarden, 1Password, KeePass, hoặc trình duyệt) liên kết tài khoản lưu trữ với **Domain chính xác từng ký tự ASCII**.
* **Dấu hiệu cảnh báo:** 
  - Khi truy cập đúng trang chủ, trình quản lý mật khẩu sẽ **tự động đề xuất / điền (Auto-fill)** thông tin.
  - Nếu truy cập trang giả mạo (`githúb.com`), trình quản lý mật khẩu sẽ **không nhận diện được** và **không tự động điền**. Đây là dấu hiệu cảnh báo đỏ nguy hiểm, cần dừng ngay thao tác nhập Master Password.

### Bước 3: Xác minh bằng công cụ hỗ trợ & Trình duyệt (Technical Verification)
1. **Kiểm tra mã Punycode:**
   - Copy URL từ thanh địa chỉ và dán vào một trình chỉnh sửa văn bản đơn giản (Notepad/TextEdit) hoặc thanh tìm kiếm để xem URL có bị biến thành dạng `xn--...` hay không.
2. **Kiểm tra tính năng chống Homograph của trình duyệt:**
   - Đảm bảo trình duyệt luôn được cập nhật phiên bản mới nhất (Chrome, Firefox, Edge đều có tính năng tự động chuyển tên miền Unicode nghi vấn thành dạng Punycode `xn--`).
3. **Quét URL qua công cụ Security Online:**
   - Sử dụng các dịch vụ kiểm tra uy tín trước khi nhập thông tin nhạy cảm:
     - [VirusTotal](https://www.virustotal.com/)
     - [Google Safe Browsing Transparency Report](https://transparencyreport.google.com/safe-browsing/search)