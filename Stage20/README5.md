# Tài Liệu Thiết Kế Kiến Trúc Bảo Mật Đăng Nhập Crypto Wallet

## 1. Tổng Quan Bối Cảnh & Ràng Buộc Kiến Trúc

Đối với ứng dụng Ví tiền điện tử (Crypto Wallet), giao dịch mang tính chất bất khả đảo ngược (irreversible). Vì vậy, hệ thống đăng nhập và xác nhận giao dịch đòi hỏi bảo mật mức độ tối đa (High-Security Standards) thông qua mô hình **Multi-Factor Authentication (MFA)** và thuật toán **TOTP (Time-based One-Time Password)** theo tiêu chuẩn RFC 6238.

---

## 2. Thiết Kế Kiến Trúc 3 Lớp Bảo Vệ (3-Factor MFA Architecture)

Mô hình MFA áp dụng 3 yếu tố cốt lõi:
1. **Something you know (Thứ bạn biết):** Master Password / PIN.
2. **Something you have (Thứ bạn có):** Thiết bị lưu giữ secret key (Google Authenticator / YubiKey / Hardware Token).
3. **Something you are (Thứ thuộc về bạn):** Sinh trắc học (Biometrics: FaceID / Fingerprint) trên thiết bị tin cậy.

### Sơ Đồ Luồng Dữ Liệu (Data Flow Diagram)

```
[ Hacker có Password ] 
       │
       ▼ 
┌─────────────────────────────────────────────────────────────────────────────────┐
│ Bước 1: Yếu tố 1 - Knowledge Factor                                              │
│ User gửi: Username + Password                                                   │
│ Server kiểm tra: Hash Password (Argon2id)                                        │
└───────────────────────────────────────┬─────────────────────────────────────────┘
                                        │ (Hợp lệ)
                                        ▼
┌─────────────────────────────────────────────────────────────────────────────────┐
│ Bước 2: Yếu tố 2 - Possession Factor (TOTP - RFC 6238)                           │
│ Server yêu cầu: Mã TOTP (6 chữ số)                                              │
│ Client tính toán: TOTP = HMAC-SHA1(K, C) với C = floor(CurrentTime / 30)        │
│ 🛑 Hacker bị chặn tại đây nếu không có Secret Key (K) trên thiết bị người dùng  │
└───────────────────────────────────────┬─────────────────────────────────────────┘
                                        │ (Hợp lệ)
                                        ▼
┌─────────────────────────────────────────────────────────────────────────────────┐
│ Bước 3: Yếu tố 3 - Inherence Factor (Biometrics / Secure Enclave)               │
│ App yêu cầu: Quét FaceID / Vân tay local trên thiết bị                          │
│ Device: Xác thực qua phần cứng Secure Enclave (Mã khóa private giải mã Session) │
│ 🛑 Hacker bị chặn tại đây nếu dùng SessionHijacking hoặc điều khiển từ xa      │
└───────────────────────────────────────┬─────────────────────────────────────────┘
                                        │ (Hợp lệ)
                                        ▼
                   ┌─────────────────────────────────────────┐
                   │   CẤP ACCESS TOKEN / GIẢI MÃ PRIVATE KEY │
                   │       ĐĂNG NHẬP THÀNH CÔNG VÀO VÍ       │
                   └─────────────────────────────────────────┘
```

### Cơ Chế Phối Hợp Ngăn Chặn Hacker Khi Lộ Mật Khẩu

* **Nếu Hacker có Password:** Chúng đăng nhập qua bước 1 thành công. Tuy nhiên, hệ thống chưa cấp Access Token mà trả về trạng thái `CHALLENGE_MFA_REQUIRED`. Hacker không thể vượt qua bước 2 vì mã TOTP được tạo cục bộ (client-side) dựa trên khóa bí mật ($K$) lưu an toàn trong thiết bị vật lý của nạn nhân.
* **Ngăn chặn Tấn công Man-in-the-Middle (MitM) & Replay Attack:** Mã TOTP chỉ có hiệu lực trong cửa sổ thời gian ngắn ($\Delta t = 30$ giây). Mỗi mã chỉ được chấp nhận **một lần duy nhất** (Server ghi nhớ mã đã dùng trong Time Step đó).

---

## 3. Xử Lý Bẫy Edge Case: Người Dùng Ngoại Tuyến (Trên Máy Bay / Không Có Internet)

### Câu hỏi đặt ra:
*Nếu người dùng đang ở trên máy bay (không có Wifi/4G/sóng di động), làm sao họ vẫn lấy được mã OTP để thực hiện giao dịch?*

### Lời giải kỹ thuật: Bản Chất Ngoại Tuyến Của Thuật Toán TOTP (RFC 6238)

Khác với **HOTP** (dựa trên counter) hoặc **SMS OTP** (phụ thuộc vào mạng viễn thông), **TOTP hoàn toàn không cần kết nối Internet hoặc sóng điện thoại** để sinh mã OTP.

#### 1. Nguyên lý hoạt động dựa trên thời gian thực (Time-based Counter)

Mã TOTP được tạo ra bằng công thức toán học:

$$T = \left\lfloor \frac{\text{CurrentUnixTime} - T_0}{X} \right\rfloor$$

$$\text{TOTP} = \text{HMAC-SHA-1}(K, T) \pmod{10^6}$$

Trong đó:
* $K$ (Secret Key): Đã được chia sẻ và lưu trữ từ bước Setup 2FA ban đầu (lưu trong Secure Element trên thiết bị).
* $T_0$: Thời điểm bắt đầu ($0$).
* $X$: Khoảng thời gian làm mới (thường là $30$ giây).
* $T$: Giá trị đếm số chu kỳ thời gian trôi qua.

#### 2. Kịch bản xác thực trên máy bay

1. **Sinh mã trên thiết bị (Client-side):**
   * Ứng dụng Google Authenticator / YubiKey / Ví Crypto đọc đồng hồ hệ thống nội bộ của điện thoại (Internal RTC Clock).
   * Dù không có Internet, đồng hồ điện thoại vẫn chạy chính xác. Ứng dụng dùng thời gian này và khóa $K$ có sẵn để tính ra mã 6 chữ số.
2. **Trường hợp thao tác Offline trên Ví Cold/Hardware Wallet:**
   * Nếu giao dịch được ký ngoại tuyến (Offline Signing / Air-gapped Transaction via QR Code), thiết bị Ví phần cứng tự tính TOTP và ký giao dịch bằng Private Key mà **không cần kết nối bất kỳ mạng nào**.
3. **Trường hợp kết nối Internet máy bay bị chập chờn / Lệch đồng hồ (Clock Skew):**
   * Để xử lý việc đồng hồ của điện thoại bị lệch vài giây khi tắt mạng lâu ngày, phía Server triển khai thuật toán **Time-Window Tolerance**:
   * Server không chỉ kiểm tra $T_{current}$, mà còn kiểm tra thêm các khung thời gian lân cận: $T - 1$ ($30$ giây trước) và $T + 1$ ($30$ giây sau).

---

## 4. Bảng Tổng Kết Đáp Ứng An Toàn Cho Crypto Wallet

| Tình huống tấn công / Rủi ro | Cơ chế phòng thủ của Hệ thống |
| ----- | ----- |
| **Lộ Mật khẩu (Credential Leak)** | Chặn tại Yếu tố 2 (Mã TOTP trên thiết bị vật lý). |
| **Mất kết nối Mạng / Chế độ máy bay** | Thuật toán TOTP tự sinh mã ngoại tuyến bằng đồng hồ RTC nội bộ + Secret Key $K$. |
| **Tấn công SIM Swap / Nhận lén SMS** | Loại bỏ hoàn toàn SMS OTP, bắt buộc dùng TOTP App / Hardware Token. |
| **Đồng hồ thiết bị bị lệch thời gian** | Cơ chế Server Drift Tolerance cho phép sai số $\pm 1$ Time Step ($30$ giây). |