---
title : "Giới thiệu"
date : 2026-08-06
weight : 1
chapter : false
pre : " <b> 5.1. </b> "
---

#### Mục tiêu workshop

Workshop xây dựng một bản triển khai FCAJ RAG Chat có thể lặp lại từ đầu đến cuối. Thay vì chỉ chạy ứng dụng trên máy cá nhân, người học sẽ đưa Docker container lên Amazon EC2, tách dữ liệu khỏi vòng đời container bằng Amazon EBS, tạo bản sao lưu trên Amazon S3 và bổ sung giám sát cùng cảnh báo chi phí.

Kết quả cuối cùng phải chứng minh được bốn luồng:

1. Người dùng truy cập giao diện web và tải tài liệu lên.
2. Ứng dụng lập chỉ mục, truy xuất nội dung và trả lời kèm trích dẫn.
3. Dữ liệu còn nguyên sau khi container khởi động lại và có thể khôi phục từ S3.
4. Người quản trị xem được chỉ số, log, cảnh báo và biết cách dọn dẹp tài nguyên.

#### Kiến trúc thực hành

| Lớp | Thành phần | Trách nhiệm |
|---|---|---|
| Truy cập | Trình duyệt, public IPv4, Security Group | Cho phép người học truy cập cổng ứng dụng; giới hạn SSH theo IP quản trị |
| Tính toán | EC2 `t3.medium` | Chạy Docker Engine và container FCAJ RAG Chat |
| Ứng dụng | Kotaemon, Gemini, Docker | Tiếp nhận tài liệu, tạo chỉ mục, truy xuất ngữ cảnh và sinh câu trả lời |
| Dữ liệu | EBS gp3 60 GB | Duy trì `ktem_app_data`, bao gồm dữ liệu ChromaDB, LanceDB và SQLite |
| Sao lưu | S3 riêng tư | Lưu bản sao theo thời điểm và dữ liệu dùng cho thử nghiệm khôi phục |
| Quyền | IAM Role | Cho phép EC2 truy cập đúng bucket hoặc prefix mà không dùng access key tĩnh |
| Quan sát | CloudWatch | Theo dõi trạng thái EC2, CPU, dung lượng, application log và alarm |
| Chi phí | AWS Budgets | Gửi cảnh báo khi chi phí tiến gần ngưỡng của nhóm |
| Phụ thuộc ngoài AWS | Gemini API | Cung cấp mô hình hội thoại và embedding theo cấu hình dự án |

Luồng mạng của bản thử nghiệm là: người dùng → public IPv4 của EC2 → cổng ứng dụng → Kotaemon. Từ EC2, ứng dụng gọi Gemini API bằng kết nối đi ra; tiến trình vận hành gọi S3 và CloudWatch thông qua quyền của IAM Role. S3 không được mở công khai.

#### Phạm vi bảo mật

Workshop sử dụng một EC2 instance và HTTP để đơn giản hóa bản demo nội bộ. Đây không phải kiến trúc sản xuất. Không đưa tài liệu nhạy cảm lên hệ thống, không chia sẻ URL công khai, và không mở SSH cho `0.0.0.0/0`. Nếu phát triển tiếp, cần bổ sung HTTPS, xác thực, quản lý secret bằng SSM Parameter Store hoặc Secrets Manager và một lớp cân bằng tải phù hợp.

#### Thời lượng dự kiến

| Phần | Thời lượng |
|---|---:|
| Chuẩn bị và tạo tài nguyên | 30–45 phút |
| Build Docker image và chạy ứng dụng | 30–60 phút |
| Gắn EBS, sao lưu và khôi phục | 30–45 phút |
| Giám sát, kiểm thử và dọn dẹp | 30–45 phút |

Thời gian build image có thể thay đổi theo tốc độ mạng và cấu hình EC2. Nên dành thêm thời gian nếu đây là lần đầu Docker tải dependency.

#### Kết quả cần lưu làm bằng chứng

- Ảnh trang ứng dụng hoạt động và kết quả một câu hỏi có trích dẫn.
- Kết quả `docker compose ps` cho thấy container ở trạng thái khỏe.
- Kết quả kiểm tra mount EBS và dữ liệu còn lại sau khi restart.
- Danh sách đối tượng trong prefix sao lưu S3 và biên bản khôi phục thử.
- Ảnh CloudWatch metric, log/alarm và xác nhận AWS Budget.

{{% notice info %}}
Mọi giá trị như Region, bucket name, địa chỉ IP và API key trong workshop phải được thay bằng giá trị của tài khoản người học. Không đưa API key thật vào báo cáo hoặc ảnh chụp.
{{% /notice %}}
