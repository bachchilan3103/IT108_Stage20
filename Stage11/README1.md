# SESSION 10 – EX01: SỬA LỖI HEADING STYLES VÀ AUTO TOC TRONG WORD

## 1. Phân tích lỗi

### 1.1. Nguyên nhân mục lục trống

Nam sử dụng định dạng thủ công như phóng to chữ, in đậm nhưng vẫn để kiểu `Normal`. Word không nhận diện những đoạn văn này là tiêu đề khi tạo mục lục tự động, dẫn đến thông báo `No table of contents entries found`.

### 1.2. Nguyên nhân đoạn văn 5 trang xuất hiện trong mục lục

Nam đã gán `Heading 1` cho một đoạn văn dài 5 trang. Vì vậy, Word hiểu toàn bộ đoạn văn là tiêu đề cấp 1 và đưa vào mục lục. Đây là lỗi sử dụng sai Heading Styles. Chỉ các tiêu đề mới nên được gán Heading, còn nội dung thông thường phải sử dụng `Normal`.

## 2. Các bước sửa lỗi

### 2.1. Gán lại Heading Styles

- Chọn **1. Giới thiệu**, vào `Home → Styles → Heading 1`.
- Chọn **2. Yêu cầu chức năng**, vào `Home → Styles → Heading 1`.
- Các mục con như **2.1. Đăng ký tài khoản** sử dụng `Heading 2`.
- Các mục nhỏ hơn như **2.1.1. Nhập thông tin** sử dụng `Heading 3`.
- Chọn đoạn văn dài 5 trang và chuyển về kiểu `Normal`.

Việc phân cấp đúng giúp Word nhận diện chính xác cấu trúc tài liệu.

### 2.2. Chèn mục lục tự động

1. Đặt con trỏ tại vị trí cần chèn mục lục.
2. Chọn `References → Table of Contents`.
3. Chọn `Automatic Table 1` hoặc `Automatic Table 2`.
4. Kiểm tra các tiêu đề và cấp bậc trong mục lục.

### 2.3. Cập nhật mục lục và số trang

Sau khi thêm nội dung, thực hiện các bước sau:

1. Nhấp chuột phải vào mục lục.
2. Chọn `Update Field`.
3. Chọn `Update entire table` để cập nhật tiêu đề và số trang.
4. Nếu chỉ cần cập nhật số trang, chọn `Update page numbers only`.

### 2.4. Định dạng tiêu đề đồng bộ

Vào `Home → Styles`, nhấp chuột phải vào `Heading 1` hoặc `Heading 2`, chọn `Modify` để thay đổi phông chữ, kích thước và màu sắc. Các tiêu đề cùng cấp sẽ được cập nhật đồng bộ mà không cần chỉnh sửa thủ công từng trang.

## 3. Kết quả mong đợi

- Mục lục tự động hiển thị đầy đủ các tiêu đề.
- Tiêu đề chính và tiêu đề phụ được phân cấp bằng `Heading 1`, `Heading 2` và `Heading 3`.
- Đoạn văn dài 5 trang không còn xuất hiện trong mục lục.
- Số trang được cập nhật sau khi nội dung thay đổi.
- Định dạng tiêu đề được thống nhất trong toàn bộ tài liệu SRS.

## 4. Quy trình nộp bài lên GitHub

### 4.1. Tạo repository

Tạo repository với tên theo cú pháp:

`[Tên Lớp]_[Môn Học]_Session10_Ex01`

### 4.2. Đưa bài làm lên GitHub

Mở Git Bash tại thư mục chứa file bài làm và thực hiện:

```bash
git init
git add .
git commit -m "Complete Session10 Ex01"
git branch -M main
git remote add origin https://github.com/USERNAME/REPOSITORY.git
git push -u origin main
```

Thay `USERNAME` và `REPOSITORY` bằng tên tài khoản và repository thực tế. Nếu thư mục đã được kết nối với GitHub, không cần khởi tạo lại repository.

### 4.3. Nộp bài

Sau khi push thành công, sao chép đường dẫn repository GitHub và nộp lên hệ thống Portal theo yêu cầu.

## 5. Kết luận

Sử dụng Heading Styles đúng cấp bậc giúp Word nhận diện tiêu đề, tạo mục lục tự động chính xác và cập nhật số trang nhanh chóng. Đồng thời, việc định dạng bằng Styles giúp tài liệu SRS có cấu trúc khoa học, dễ chỉnh sửa và chuyên nghiệp hơn.