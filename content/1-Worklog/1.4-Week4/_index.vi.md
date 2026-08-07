---
title: "Worklog Tuần 4"
date: 2024-01-01
weight: 1
chapter: false
pre: " <b> 1.4. </b> "
---

### Mục tiêu tuần 4:

* Bắt đầu tiếp cận project fcaj-rag-chat, hiểu kiến trúc Kotaemon và luồng xử lý của một ứng dụng RAG.
* Clone repository, thiết lập môi trường Docker và xây dựng baseline chạy local có khả năng lưu dữ liệu bền vững.
* Bổ sung kiến thức chuyên sâu về các dịch vụ lưu trữ AWS và quy trình di chuyển máy ảo bằng VM Import/Export.
* Phác thảo kiến trúc triển khai project với ba dịch vụ cốt lõi: Amazon ECR, Amazon ECS Fargate và Amazon EFS.

### Các công việc cần triển khai trong tuần này:

| Thứ | Công việc | Ngày bắt đầu | Ngày hoàn thành | Nguồn tài liệu |
| --- | --- | --- | --- | --- |
| 2 | **Khảo sát repository và luồng RAG**<br>- Clone project fcaj-rag-chat và đọc hướng dẫn chạy.<br>- Kiểm tra cấu trúc Kotaemon, Dockerfile, Docker Compose, buildspec và file cấu hình model.<br>- Phân tích luồng: upload PDF → trích xuất nội dung → embedding → lập chỉ mục → truy xuất → Gemini sinh câu trả lời kèm citation.<br>- Xác định ChromaDB là vector store, LanceDB là document store và SQLite lưu metadata, người dùng cùng cấu hình. | 13/07/2026 | 13/07/2026 | [fcaj-rag-chat](https://github.com/ngocchau04/fcaj-rag-chat) |
| 3 | **Thiết lập và kiểm tra ứng dụng trên local**<br>- Tạo file biến môi trường và truyền Gemini API key mà không lưu khóa trong source code.<br>- Khởi chạy Docker image lite bằng Docker Compose và truy cập ứng dụng tại <code>localhost:7860</code>.<br>- Mount thư mục <code>ktem_app_data</code> để duy trì tài liệu, index, vector data và cấu hình sau khi container khởi động lại.<br>- Xây dựng checklist kiểm tra đăng nhập, upload PDF, index tài liệu, đặt câu hỏi và hiển thị citation. | 14/07/2026 | 14/07/2026 | [fcaj-rag-chat](https://github.com/ngocchau04/fcaj-rag-chat) |
| 4 | **Học Module 04 – Dịch vụ lưu trữ trên AWS**<br>- 04-01: Amazon S3, Access Point và các Storage Class.<br>- 04-02: S3 Static Website, CORS, kiểm soát truy cập, object key, hiệu năng và S3 Glacier.<br>- 04-03: AWS Snow Family, Storage Gateway và AWS Backup.<br>- So sánh object storage, hybrid storage và backup để lựa chọn đúng dịch vụ theo yêu cầu workload. | 15/07/2026 | 15/07/2026 | [04-01](https://www.youtube.com/watch?v=_yunukwcAwc)<br>[04-02](https://www.youtube.com/watch?si=OITIt1x3d7OZ4ei_&v=mPBjB6Ltl_Q&feature=youtu.be)<br>[04-03](https://www.youtube.com/watch?v=YXn8Q_Hpsu4) |
| 5 | **Thực hành quy trình AWS VM Import/Export**<br>- Tìm hiểu các điều kiện cần có: môi trường ảo hóa, AWS CLI, S3 bucket và IAM service role.<br>- Theo dõi quy trình xuất VM từ môi trường on-premises, upload image lên S3, import thành AMI và khởi chạy EC2 instance.<br>- Tìm hiểu chiều ngược lại: export EC2 instance hoặc AMI về S3 để sử dụng tại chỗ.<br>- Ghi chú các giới hạn định dạng, quyền truy cập và quy trình dọn dẹp tài nguyên. | 16/07/2026 | 16/07/2026 | [AWS VM Import/Export Workshop](https://000014.awsstudygroup.com/) |
| 6 | **Thiết kế sơ bộ kiến trúc AWS cho project RAG**<br>- Ánh xạ Docker image vào Amazon ECR, container runtime vào ECS Fargate và thư mục <code>/app/ktem_app_data</code> vào Amazon EFS.<br>- Xác định các thành phần hỗ trợ dự kiến: Application Load Balancer, VPC, public/private subnet, Security Group, IAM, SSM Parameter Store và CloudWatch Logs.<br>- Phân tích giới hạn của ChromaDB, LanceDB và SQLite khi dùng chung EFS; ưu tiên một ECS task ở giai đoạn đầu.<br>- Trao đổi nhóm, hoàn thiện sơ đồ kiến trúc và lập kế hoạch cấu hình, kiểm thử cho tuần 5. | 17/07/2026 | 17/07/2026 | [fcaj-rag-chat](https://github.com/ngocchau04/fcaj-rag-chat) |

### Kết quả đạt được tuần 4:

* **Hiểu cấu trúc project:** Xác định được vai trò của Kotaemon, các file cấu hình quan trọng và mối liên hệ giữa giao diện web, pipeline RAG, model Gemini và các tầng lưu trữ.
* **Nắm rõ luồng xử lý RAG:** Mô tả được toàn bộ quá trình từ khi người dùng upload PDF, hệ thống tạo embedding và index cho đến khi truy xuất ngữ cảnh để sinh câu trả lời có citation.
* **Thiết lập baseline chạy local:** Chuẩn hóa cách chạy ứng dụng bằng Docker Compose, cổng truy cập và biến môi trường; API key được tách khỏi source code để hạn chế rò rỉ thông tin xác thực.
* **Xác định cơ chế persistent storage:** Hiểu vai trò của volume <code>ktem_app_data</code> đối với ChromaDB, LanceDB, SQLite, tài liệu upload và cấu hình khi container bị restart hoặc được tạo lại.
* **Củng cố kiến thức lưu trữ AWS:** Phân biệt được S3 Storage Class, Access Point, Glacier, Snow Family, Storage Gateway và AWS Backup theo mục đích lưu trữ, lưu trữ lai và sao lưu.
* **Hiểu quy trình di chuyển máy ảo:** Nắm được chuỗi thao tác VM image → S3 → AMI → EC2 và quy trình export ngược, đồng thời nhận biết vai trò của IAM, định dạng image và cleanup.
* **Hình thành kiến trúc triển khai ban đầu:** Chốt ba dịch vụ cốt lõi ECR–ECS Fargate–EFS và xác định rõ các dịch vụ hỗ trợ cần thiết mà không làm thay đổi phạm vi “ba dịch vụ AWS” của project.
* **Nhận diện rủi ro kỹ thuật:** Ghi nhận giới hạn về đồng thời khi ChromaDB, LanceDB và SQLite cùng sử dụng EFS; chưa đặt mục tiêu scale ngang nhiều task trước khi có kiểm thử locking và tính nhất quán dữ liệu.

**Kết quả tổng quan:** Tuần 4 tạo được nền tảng kỹ thuật để chuyển từ giai đoạn học các dịch vụ AWS riêng lẻ sang giai đoạn làm việc với project thực tế. Nhóm đã hiểu project, có baseline chạy local, có checklist kiểm thử và có bản thiết kế sơ bộ để tiếp tục cấu hình trong tuần 5 trước khi triển khai lên AWS.
