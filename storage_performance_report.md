# Báo Cáo Tối Ưu Hóa Tài Nguyên Bảng Posts - QuickFeed

## 1. Bản chất vấn đề (Over-Indexing)
Trước khi tối ưu, bảng `Posts` có tới 5 Secondary Index. Mỗi thao tác `INSERT` một bài viết mới buộc công cụ lưu trữ (InnoDB) phải chèn dữ liệu vào bảng chính (Clustered Index), đồng thời thực hiện thêm 5 thao tác ghi ngầm vào 5 cây B-Tree khác nhau. Điều này gây hiện tượng I/O bound trên ổ đĩa, dẫn đến timeout 5-10 giây khi người dùng đăng bài.

## 2. Quyết định loại bỏ Index
- **idx_content**: Cắt bỏ vì chuỗi ký tự độ dài 255 làm phình to kích thước các trang index (Index Pages).
- **idx_post_type & idx_is_visible**: Cắt bỏ vì **Cardinality** quá thấp (lần lượt 3 và 2 giá trị). Trong hầu hết trường hợp, MySQL Query Optimizer sẽ coi việc đọc index rồi quay lại tìm bản ghi (Bookmark Lookup) tốn kém hơn việc Full Table Scan.

## 3. Kết quả đạt được
- **Tốc độ Ghi (Write)**: Giảm 3 lần thao tác cập nhật cấu trúc B-Tree trên mỗi lệnh `INSERT`, giải quyết triệt để lỗi timeout.
- **Dung lượng Đĩa (Storage)**: Kích thước `index_length` giảm mạnh, giải phóng RAM (InnoDB Buffer Pool) và dung lượng ổ cứng cho máy chủ.
- **Tốc độ Đọc (Read)**: Các truy vấn quan trọng nhất (theo `user_id` và sắp xếp theo `created_at`) vẫn được duy trì hiệu năng cao nhất.