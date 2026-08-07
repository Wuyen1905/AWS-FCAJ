---
title : "Lưu trữ bền vững, sao lưu và giám sát"
date : 2026-08-06
weight : 4
chapter : false
pre : " <b> 5.4. </b> "
---

#### Tổng quan

Container có thể bị tạo lại trong quá trình cập nhật hoặc xử lý lỗi. Vì vậy, dữ liệu RAG không được phụ thuộc vào writable layer của container. Phần này đặt thư mục `ktem_app_data` trên EBS, tạo bản sao lưu nhất quán lên S3, diễn tập khôi phục và bổ sung khả năng quan sát hệ thống.

#### Nguyên tắc dữ liệu

- EBS là nơi lưu dữ liệu đang hoạt động của một EC2 instance.
- S3 là nơi giữ bản sao lưu; không mount S3 thay thế trực tiếp cho filesystem của ChromaDB, LanceDB hoặc SQLite.
- Dừng ứng dụng trong lúc tạo bản sao để giảm nguy cơ file cơ sở dữ liệu không nhất quán.
- Một bản sao lưu chỉ được xem là hợp lệ sau khi kiểm tra checksum và khôi phục thử.
- Không đưa `.env`, API key, SSH key hoặc AWS credential vào gói sao lưu.

#### Nội dung

1. [Chuẩn bị và gắn EBS](5.4.1-prepare/)
2. [Tạo S3 bucket riêng tư và sao lưu](5.4.2-create-interface-enpoint/)
3. [Khôi phục dữ liệu và kiểm tra tính bền vững](5.4.3-test-endpoint/)
4. [Cấu hình CloudWatch và AWS Budgets](5.4.4-dns-simulation/)

#### Kết quả cần đạt

- Container sử dụng bind mount `/opt/fcaj/ktem_app_data:/app/ktem_app_data`.
- EBS tự gắn lại sau khi EC2 khởi động.
- S3 chặn truy cập công cộng, bật mã hóa và chứa archive cùng checksum.
- Dữ liệu khôi phục trả lời được ít nhất một câu hỏi đã biết.
- CloudWatch hiển thị chỉ số, log cần thiết và AWS Budget đã xác nhận email.
