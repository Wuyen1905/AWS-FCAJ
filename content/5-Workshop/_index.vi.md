---
title: "Workshop"
date: 2026-08-06
weight: 5
chapter: false
pre: " <b> 5. </b> "
---

# Triển khai FCAJ RAG Chat trên AWS

#### Tổng quan

Workshop hướng dẫn triển khai một hệ thống hỏi đáp tài liệu dựa trên RAG từ mã nguồn của nhóm. Ứng dụng Kotaemon được đóng gói bằng Docker, chạy trên Amazon EC2, lưu dữ liệu bền vững trên Amazon EBS và sao lưu vào Amazon S3 riêng tư. Amazon CloudWatch và AWS Budgets được sử dụng để theo dõi trạng thái, lỗi và chi phí; Gemini API được gọi từ phía máy chủ để tạo embedding hoặc câu trả lời theo cấu hình.

Sau khi hoàn thành, người học có thể:

- Chuẩn bị EC2, Security Group, IAM Role và EBS cho một ứng dụng RAG.
- Cấu hình Gemini an toàn và chạy FCAJ RAG Chat bằng Docker.
- Kiểm tra tải tài liệu, lập chỉ mục, truy xuất và trích dẫn.
- Sao lưu, khôi phục dữ liệu bằng S3 và kiểm tra dữ liệu sau khi container khởi động lại.
- Theo dõi hệ thống bằng CloudWatch, đặt cảnh báo chi phí và dọn dẹp đúng tài nguyên.

Kiến trúc này phù hợp cho bản thử nghiệm học thuật và demo nội bộ. Trước khi cung cấp cho người dùng bên ngoài nhóm, cần bổ sung HTTPS, xác thực, quản lý secret chuyên dụng và thiết kế có tính sẵn sàng cao hơn.

#### Nội dung

1. [Giới thiệu và kiến trúc workshop](5.1-Workshop-overview/)
2. [Chuẩn bị tài khoản, mã nguồn và thông số triển khai](5.2-Prerequiste/)
3. [Triển khai ứng dụng RAG trên EC2](5.3-S3-vpc/)
   1. [Chuẩn bị EC2, Security Group và IAM Role](5.3-S3-vpc/5.3.1-create-gwe/)
   2. [Cài Docker, chạy ứng dụng và kiểm tra](5.3-S3-vpc/5.3.2-test-gwe/)
4. [Lưu trữ bền vững, sao lưu và giám sát](5.4-S3-onprem/)
   1. [Chuẩn bị và gắn EBS](5.4-S3-onprem/5.4.1-prepare/)
   2. [Tạo S3 bucket riêng tư và sao lưu](5.4-S3-onprem/5.4.2-create-interface-enpoint/)
   3. [Khôi phục dữ liệu và kiểm tra tính bền vững](5.4-S3-onprem/5.4.3-test-endpoint/)
   4. [Cấu hình CloudWatch và AWS Budgets](5.4-S3-onprem/5.4.4-dns-simulation/)
5. [Kiểm thử chất lượng, bảo mật và chi phí](5.5-Policy/)
6. [Dọn dẹp tài nguyên](5.6-Cleanup/)
