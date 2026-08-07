---
title : "Triển khai ứng dụng RAG trên EC2"
date : 2026-08-06
weight : 3
chapter : false
pre : " <b> 5.3. </b> "
---

#### Mục tiêu

Phần này tạo máy chủ cho bản demo, gắn IAM Role, cài Docker và chạy FCAJ RAG Chat từ repository của nhóm. Ở cuối phần, giao diện phải truy cập được qua public IPv4 và container phải vượt qua health check.

Chưa tải bộ tài liệu chính thức ở bước này. Dữ liệu thử ban đầu nằm trên root volume; trong phần 5.4, ứng dụng sẽ được dừng để chuyển thư mục dữ liệu sang EBS 60 GB trước khi kiểm thử đầy đủ.

#### Nội dung

1. [Chuẩn bị EC2, Security Group và IAM Role](5.3.1-create-gwe/)
2. [Cài Docker, chạy ứng dụng và kiểm tra](5.3.2-test-gwe/)

#### Điều kiện hoàn thành

- EC2 ở trạng thái `running` và có public IPv4.
- IAM Role được gắn vào instance, `aws sts get-caller-identity` không dùng access key tĩnh.
- Chỉ IP quản trị được SSH; cổng ứng dụng chỉ mở trong phạm vi kiểm thử.
- `docker compose ps` hiển thị dịch vụ `fcaj-rag-chat` đang chạy hoặc khỏe.
- Trình duyệt mở được giao diện và không lộ nội dung tệp `.env`.
