# SESSION 10 – EX03: TRA CỨU DỮ LIỆU BẰNG VLOOKUP TRONG EXCEL

## 1. Phân tích bài toán

Sheet `Don_Hang` chứa 100 mã linh kiện cần điền tên và đơn giá. Các thông tin này được lấy từ Sheet `Danh_Muc` bằng hàm `VLOOKUP`, giúp tự động tra cứu dữ liệu thay vì nhập thủ công.

### 1.1. Bốn tham số của hàm VLOOKUP

Cú pháp:

```excel
=VLOOKUP(lookup_value, table_array, col_index_num, [range_lookup])
```

Ý nghĩa các tham số trong bài toán:

- **Lookup_value:** Mã linh kiện cần tra cứu, ví dụ `LK01`, nằm trong cột A của Sheet `Don_Hang`.
- **Table_array:** Vùng dữ liệu tra cứu trên Sheet `Danh_Muc`, ví dụ `$A$2:$C$101`. Dấu `$` giúp cố định vùng dữ liệu khi kéo công thức xuống.
- **Col_index_num:** Số thứ tự cột cần lấy kết quả trong vùng tra cứu. Nếu cột A là Mã linh kiện, cột B là Tên linh kiện và cột C là Đơn giá thì giá trị lần lượt là `2` và `3`.
- **Range_lookup:** Sử dụng `FALSE` để tìm kiếm chính xác mã linh kiện, tránh trả về kết quả gần đúng.

### 1.2. Nguyên nhân lỗi #N/A

Lỗi `#N/A` có thể xảy ra khi mã linh kiện không tồn tại trong bảng tra cứu hoặc mã ở hai Sheet không khớp do khoảng trắng thừa. Ngoài ra, nếu quên dấu `$`, vùng tra cứu sẽ bị dịch chuyển khi kéo công thức xuống, khiến Excel không tìm thấy dữ liệu cần thiết.

## 2. Triển khai công thức VLOOKUP

Giả sử cấu trúc dữ liệu như sau:

- Sheet `Don_Hang`: cột A là Mã linh kiện, cột B là Tên linh kiện, cột C là Đơn giá.
- Sheet `Danh_Muc`: cột A là Mã linh kiện, cột B là Tên linh kiện, cột C là Đơn giá.

### 2.1. Tra cứu tên linh kiện

Tại ô B2 của Sheet `Don_Hang`, nhập công thức:

```excel
=IFERROR(VLOOKUP(TRIM(A2),Danh_Muc!$A$2:$C$101,2,FALSE),"Không tìm thấy mã linh kiện")
```

Công thức sử dụng `TRIM(A2)` để loại bỏ khoảng trắng thừa ở đầu và cuối mã cần tìm, `FALSE` để tìm chính xác và `IFERROR` để hiển thị thông báo dễ hiểu khi xảy ra lỗi.

### 2.2. Tra cứu đơn giá

Tại ô C2 của Sheet `Don_Hang`, nhập công thức:

```excel
=IFERROR(VLOOKUP(TRIM(A2),Danh_Muc!$A$2:$C$101,3,FALSE),"Không tìm thấy đơn giá")
```

Tham số `3` cho biết cần lấy dữ liệu ở cột thứ ba trong vùng tra cứu, tương ứng với cột Đơn giá.

### 2.3. Hoàn thành tra cứu cho 100 đơn hàng

1. Nhập công thức vào ô B2 và C2.
2. Chọn các ô chứa công thức.
3. Kéo chốt điền ở góc dưới bên phải xuống đến hàng 101 hoặc nhấp đúp vào chốt điền nếu dữ liệu liền kề phù hợp.
4. Kiểm tra kết quả tên linh kiện và đơn giá.
5. Nếu xuất hiện thông báo không tìm thấy, kiểm tra mã linh kiện ở cả hai Sheet.

**Lưu ý:** `TRIM(A2)` chỉ làm sạch mã ở Sheet `Don_Hang`. Nếu mã trong Sheet `Danh_Muc` cũng có khoảng trắng thừa, cần làm sạch cột mã trong bảng danh mục bằng `TRIM()` rồi thay thế bằng giá trị đã chuẩn hóa trước khi tra cứu.

## 3. Kết quả mong đợi

- Tên linh kiện và đơn giá được tự động điền từ Sheet `Danh_Muc`.
- Vùng tra cứu được cố định bằng dấu `$`, không bị dịch chuyển khi kéo công thức.
- `FALSE` đảm bảo tìm kiếm chính xác.
- `TRIM()` hỗ trợ xử lý khoảng trắng thừa ở mã cần tra cứu.
- `IFERROR()` thay lỗi `#N/A` bằng thông báo dễ hiểu.

## 4. Kết luận

Hàm `VLOOKUP` kết hợp với `IFERROR`, `TRIM()` và tham chiếu tuyệt đối giúp tự động hóa việc tra cứu dữ liệu giữa hai Sheet. Cách làm này tiết kiệm thời gian, giảm sai sót khi nhập liệu và giúp quản lý đơn hàng hiệu quả hơn.