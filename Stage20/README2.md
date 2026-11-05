# Báo Cáo Phân Tích Lỗi Tái Sử Dụng Mật Khẩu & Độ An Toàn Của Mã Hóa AES-256

## 1. Cơ Chế Tấn Công Credential Stuffing (Hiệu Ứng Domino)

Khi một diễn đàn nhỏ bị tấn công và làm rò rỉ cơ sở dữ liệu (đặc biệt nếu diễn đàn lưu mật khẩu dạng dạng rõ - plaintext hoặc hash yếu), hacker sẽ kích hoạt chuỗi tấn công **Credential Stuffing** (Hiệu ứng Domino):

```
[Diễn đàn nhỏ bị hack] 
       │
       ▼ (Rò rỉ Email + Mật khẩu: "123456")
[Tạo danh sách tài khoản rò rỉ (Comb List)]
       │
       ▼ (Sử dụng Tool tự động hóa: Combo List + Proxy)
[Thử đăng nhập hàng loạt vào Gmail, Facebook, GitHub...]
       │
       ▼
[Chiếm đoạt tài khoản lớn ➔ Đổi mật khẩu ➔ Cắt quyền truy cập]
```

### Các bước diễn ra:
1. **Rò rỉ dữ liệu:** Hacker thu thập được cặp thông tin `Email/Username` và `Mật khẩu ("123456")` từ diễn đàn nhỏ.
2. **Khai thác thói quen:** Hacker dựa trên tâm lý người dùng thường dùng 1 email và 1 mật khẩu duy nhất cho nhiều dịch vụ.
3. **Thử nghiệm tự động hóa (Credential Stuffing):** Sử dụng các công cụ tự động để thử cặp tài khoản/mật khẩu này trên các dịch vụ quan trọng như Gmail, Facebook, GitHub, Ngân hàng...
4. **Chiếm quyền điều khiển:** Do sử dụng chung mật khẩu, hacker dễ dàng đăng nhập thành công vào các tài khoản lớn, sau đó chiếm quyền nhận mã OTP/Email khôi phục để khóa tài khoản nạn nhân vĩnh viễn.

---

## 2. So Sánh: Mật Khẩu Cảm Tính vs. Mật Khẩu Máy Tính Sinh Ra

### A. Mật khẩu cảm tính (Human-generated Password)
* **Đặc điểm:** Thường là các chuỗi dễ nhớ như `123456`, `password`, ngày tháng năm sinh, tên người yêu, hoặc các mẫu bàn phím như `qwerty`.
* **Điểm yếu:** 
  * **Không gian tìm kiếm nhỏ:** Nằm trong các danh sách mật khẩu phổ biến (Dictionary/Wordlist).
  * **Độ hỗn loạn (Entropy) cực thấp:** Dễ dàng bị bẻ khóa bằng các phương pháp tấn công Từ điển (Dictionary Attack) hoặc Tấn công vét cạn (Brute-force) chỉ trong vài miligiây.

### B. Mật khẩu máy tính sinh ra (Random Generated Password)
* **Đặc điểm:** Dãy ký tự ngẫu nhiên bao gồm chữ hoa, chữ thường, chữ số và ký tự đặc biệt (Ví dụ: `k9#mP$9vL!2xQZ@w`).
* **Tại sao khó bị bẻ khóa:**
  * **Entropy (Độ hỗn loạn) cao:** Với độ dài 16 ký tự bao gồm đủ các loại ký tự, không gian mật khẩu lên đến $94^{16}$ khả năng.
  * **Kháng tấn công từ điển:** Không có quy luật từ ngữ hay ý nghĩa ngôn ngữ để hacker lợi dụng dự đoán.
  * **Thời gian bẻ khóa cực lớn:** Siêu máy tính hiện đại có thể mất hàng triệu năm mới thử hết các khả năng.

---

## 3. Mã Hóa AES-256 và Sự An Toàn Của Password Manager vs. Ghi Chép Thủ Công

### A. Chuẩn mã hóa AES-256 (Advanced Encryption Standard)
AES-256 là tiêu chuẩn mã hóa khối đối xứng (Symmetric Block Cipher) được sử dụng rộng rãi bởi các chính phủ và tổ chức quân sự toàn cầu.

* **Độ dài khóa 256-bit:** Số lượng khóa khả thi là $2^{256} \approx 1.15 \times 10^{77}$ khả năng.
* **Độ an toàn tuyệt đối trước Brute-force:** Hiện tại, không có siêu máy tính nào trên thế giới (kể cả hệ thống máy tính toán lượng tử hiện tại) đủ khả năng bẻ khóa AES-256 bằng phương pháp vét cạn trong khoảng thời gian thực tế (mất nhiều tỷ năm).
* **Kiến trúc Zero-Knowledge:** Các Trình quản lý mật khẩu (như Bitwarden, 1Password) sử dụng Master Password của bạn để tạo ra khóa mã hóa AES-256 local ngay trên máy. Dữ liệu đẩy lên Cloud đã được mã hóa hoàn toàn. Nhà cung cấp dịch vụ cũng không thể đọc được mật khẩu của bạn.

### B. Bảng so sánh: Password Manager (AES-256) vs. Ghi chép thủ công

| Tiêu chí | Password Manager (AES-256) | Ghi chép thủ công (Sổ sách / Note phone) |
| :--- | :--- | :--- |
| **Bảo mật vật lý** | Chống trộm tuyệt đối (Dữ liệu bị lấy đi vẫn chỉ là chuỗi mã hóa AES vô nghĩa). | Dễ bị thất lạc, bị nhìn lén (shoulder surfing), hoặc bị người khác đọc trực tiếp. |
| **Bảo mật kỹ thuật** | Được bảo vệ bằng thuật toán mã hóa mạnh nhất thế giới hiện nay ($2^{256}$ khả năng). | Ghi chú trên điện thoại/file Excel không được mã hóa hoặc mã hóa yếu rất dễ bị Malware/Spyware thu thập. |
| **Sinh mật khẩu** | Tự động tạo mật khẩu ngẫu nhiên độ dài lớn, Entropy cao cho từng trang web riêng biệt. | Xu hướng người dùng sẽ ghi lại mật khẩu ngắn, dễ nhớ hoặc lặp lại mật khẩu do ngại viết dài. |
| **Khả năng tự động điền** | Chống lừa đảo (Anti-Phishing/Homograph): Không tự điền trên tên miền giả mạo. | Người dùng vẫn phải copy-paste hoặc gõ tay, dễ bị tấn công lừa đảo (Phishing) hoặcKeylogger ghi lại. |

### Kết luận
Ghi chép thủ công loại bỏ được rủi ro rò rỉ qua mạng nhưng lại tạo ra **lỗ hổng nghiêm trọng về bảo mật vật lý** và **kích thích thói quen dùng mật khẩu yếu**. Trong khi đó, **Trình quản lý mật khẩu dựa trên AES-256** vừa giải quyết bài toán mã hóa dữ liệu an toàn tuyệt đối, vừa giúp duy trì mỗi tài khoản một mật khẩu duy nhất có Entropy cao.