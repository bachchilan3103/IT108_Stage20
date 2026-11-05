# SESSION 10 – EX02: LÀM SẠCH DỮ LIỆU EXCEL VÀ SỬ DỤNG AUTOFILTER

## 1. Phân tích lỗi

Bảng Excel quản lý thiết bị IoT đang vi phạm các quy tắc quan trọng trong tư duy Database:

### 1.1. Vi phạm quy tắc tiêu đề cột

Các ô tiêu đề đang sử dụng `Merge Cell`. Điều này khiến cấu trúc bảng không đồng nhất và có thể làm AutoFilter hoạt động không chính xác. Mỗi cột cần có một ô tiêu đề riêng biệt, không gộp ô.

### 1.2. Vi phạm quy tắc dữ liệu liên tục

Bảng có hàng trống nằm giữa vùng dữ liệu. Khi sử dụng `Ctrl + Shift + L`, Excel có thể chỉ nhận diện một phần dữ liệu và bỏ qua các hàng phía dưới. Bảng cần được tổ chức thành một vùng dữ liệu liên tục.

### 1.3. Vi phạm quy tắc nhất quán dữ liệu

Cột `Trạng thái` có các giá trị như `Hỏng` và `Hỏng ` (có dấu cách ở cuối). Excel có thể coi chúng là hai giá trị khác nhau nên kết quả lọc không chính xác. Cần chuẩn hóa dữ liệu bằng hàm `TRIM()`.

### 1.4. Vi phạm quy tắc mỗi ô chứa một giá trị

Mỗi ô cần chứa một giá trị rõ ràng, không gộp nhiều ô hoặc sử dụng cấu trúc phức tạp làm ảnh hưởng đến việc lọc và xử lý dữ liệu.

## 2. Hoàn thiện bảng dữ liệu

### 2.1. Gỡ bỏ Merge Cell

1. Chọn toàn bộ bảng dữ liệu.
2. Vào tab `Home`.
3. Mở menu `Merge & Center`.
4. Chọn `Unmerge Cells`.
5. Đảm bảo mỗi cột có một tiêu đề riêng biệt như `Mã thiết bị`, `Tên thiết bị` và `Trạng thái`.

### 2.2. Xóa các hàng trống

1. Chọn vùng dữ liệu.
2. Xác định các hàng trống nằm giữa bảng.
3. Nhấp chuột phải vào số thứ tự hàng trống và chọn `Delete`.
4. Kiểm tra để đảm bảo dữ liệu được nối liền thành một vùng liên tục, không làm mất các bản ghi hợp lệ.

### 2.3. Sử dụng hàm TRIM để làm sạch dữ liệu

Giả sử cột `Trạng thái` nằm ở cột C, dữ liệu bắt đầu từ ô C2. Tạo một cột phụ tên `Trạng thái chuẩn hóa`, sau đó nhập công thức:

```excel
=TRIM(C2)
```

Kéo công thức xuống toàn bộ các hàng dữ liệu để loại bỏ khoảng trắng thừa ở đầu, cuối và rút gọn các khoảng trắng liên tiếp giữa các từ.

Sau khi làm sạch:

1. Sao chép toàn bộ cột kết quả.
2. Chọn cột `Trạng thái` ban đầu.
3. Chọn `Paste Special → Values` để thay thế dữ liệu cũ bằng giá trị đã chuẩn hóa.
4. Xóa cột phụ nếu không còn cần sử dụng.

**Lưu ý:** Nếu dữ liệu có khoảng trắng đặc biệt như dấu cách không ngắt dòng (`CHAR(160)`), có thể dùng công thức:

```excel
=TRIM(SUBSTITUTE(C2,CHAR(160)," "))
```

### 2.4. Bật AutoFilter

1. Chọn một ô bất kỳ trong vùng dữ liệu đã hoàn thiện.
2. Nhấn `Ctrl + Shift + L` để bật bộ lọc.
3. Nhấn nút lọc ở cột `Trạng thái`.
4. Chọn giá trị `Hỏng` để hiển thị các thiết bị bị hỏng.
5. Kiểm tra kết quả để đảm bảo tất cả bản ghi có trạng thái `Hỏng` đều được hiển thị.

## 3. Kết quả mong đợi

- Các ô tiêu đề không còn bị gộp.
- Không còn hàng trống ngắt quãng trong vùng dữ liệu.
- Cột `Trạng thái` được chuẩn hóa bằng `TRIM()`.
- AutoFilter áp dụng cho toàn bộ bảng dữ liệu.
- Các thiết bị có trạng thái `Hỏng` được lọc chính xác, không bị chia thành nhiều nhóm do khoảng trắng thừa.

## 4. Kết luận

Dữ liệu Excel cần có cấu trúc bảng rõ ràng, tiêu đề riêng biệt, không có hàng trống ngắt quãng và giá trị được chuẩn hóa. Việc sử dụng `TRIM()` kết hợp với gỡ `Merge Cell` và xóa hàng trống giúp AutoFilter hoạt động chính xác, đồng thời nâng cao chất lượng dữ liệu cho hệ thống quản lý thiết bị IoT.