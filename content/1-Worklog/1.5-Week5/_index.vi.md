---
title: "Worklog Tuần 5"
date: 2024-01-01
weight: 1
chapter: false
pre: " <b> 1.5. </b> "
---

### Mục tiêu tuần 5:

* Tiếp tục tùy biến **fcaj-rag-chat**, chuẩn hóa cấu hình Gemini, Docker và cơ chế lưu dữ liệu của ứng dụng.
* Kiểm thử end-to-end luồng RAG trên local, bao gồm upload PDF, indexing, retrieval, citation và khả năng giữ dữ liệu sau khi container restart.
* Học quy trình chuyển đổi lược đồ và di chuyển dữ liệu bằng AWS SCT/AWS DMS; đồng thời hệ thống hóa kiến thức bảo mật AWS.
* Hoàn thiện deployment blueprint ECR–ECS Fargate–EFS để sẵn sàng bước vào giai đoạn triển khai từ tuần 6.

### Các công việc cần triển khai trong tuần này:

| Thứ | Công việc | Ngày bắt đầu | Ngày hoàn thành | Nguồn tài liệu |
| --- | --- | --- | --- | --- |
| 2 | **Chuẩn hóa cấu hình project và môi trường container**<br>- Cấu hình Gemini <code>gemini-3.1-flash-lite</code> làm chat model và <code>models/gemini-embedding-001</code> làm embedding model mặc định.<br>- Rà soát ChromaDB, LanceDB và SQLite trong thư mục dữ liệu persistent; xác nhận đường dẫn mount nhất quán giữa local và môi trường dự kiến trên EFS.<br>- Loại bỏ reranking phụ thuộc Cohere để đơn giản hóa pipeline và tránh thêm API key không cần thiết.<br>- Điều chỉnh Dockerfile, Docker Compose, file biến môi trường mẫu và tài liệu hướng dẫn chạy local. | 20/07/2026 | 20/07/2026 | [fcaj-rag-chat](https://github.com/ngocchau04/fcaj-rag-chat) |
| 3 | **Học và thực hành Database Schema Conversion & Migration**<br>- Phân biệt vai trò của AWS Schema Conversion Tool và AWS Database Migration Service trong heterogeneous migration.<br>- Tìm hiểu cách chuẩn bị source Oracle/SQL Server và target Amazon RDS/Aurora; tạo schema conversion project và xử lý các đối tượng không thể chuyển đổi tự động.<br>- Cấu hình replication instance, source/target endpoints và migration task theo chiến lược full load kết hợp Change Data Capture (CDC).<br>- Theo dõi task status, table statistics, logs; tìm hiểu DMS Serverless, troubleshooting và cleanup.<br>- Ghi chú đây là bài thực hành AWS độc lập, không phải phương án lưu trữ của project RAG. | 21/07/2026 | 21/07/2026 | [Database Schema Conversion & Migration](https://000043.awsstudygroup.com/) |
| 4 | **Học Module 05 – Dịch vụ bảo mật trên AWS**<br>- 05-01: AWS Shared Responsibility Model.<br>- 05-02: AWS Identity and Access Management (IAM) và nguyên tắc least privilege.<br>- 05-03: Amazon Cognito.<br>- 05-04: AWS Organizations.<br>- 05-05: AWS IAM Identity Center.<br>- 05-06: AWS Key Management Service (KMS).<br>- 05-07: AWS Security Hub.<br>- Liên hệ kiến thức với IAM task role, quyền truy cập ECR/EFS và quản lý Gemini API key của project. | 22/07/2026 | 22/07/2026 | [05-01](https://www.youtube.com/watch?si=-xSAVT8MZReV10RP&v=tsobAlSg19g&feature=youtu.be)<br>[05-02](https://www.youtube.com/watch?v=N_vlJGAqZxo)<br>[05-03](https://www.youtube.com/watch?v=pZ2fgEFK3Vs)<br>[05-04](https://www.youtube.com/watch?v=5oQY8Rogz9Y)<br>[05-05](https://www.youtube.com/watch?v=NW1xrMkNMjU)<br>[05-06](https://www.youtube.com/watch?v=GMihNQojhZc)<br>[05-07](https://www.youtube.com/watch?v=clj2E0rNBEs) |
| 5 | **Kiểm thử end-to-end và khả năng lưu dữ liệu local**<br>- Thực hiện luồng đăng nhập → upload PDF → extract/chunk → embedding/index → truy vấn → trả lời kèm citation.<br>- Restart và recreate container để kiểm tra dữ liệu trong <code>ktem_app_data</code> vẫn được giữ lại.<br>- Kiểm tra các trường hợp lỗi cơ bản: thiếu hoặc sai Gemini API key, tài liệu không hợp lệ, lỗi model và lỗi quyền ghi thư mục dữ liệu.<br>- Thu thập log, ảnh minh chứng, ghi nhận kết quả mong đợi/thực tế và cập nhật hướng dẫn kiểm thử RAG.<br>- Chỉ xác nhận chức năng local; chưa ghi nhận ứng dụng đã được triển khai công khai trên AWS. | 23/07/2026 | 23/07/2026 | [fcaj-rag-chat](https://github.com/ngocchau04/fcaj-rag-chat) |
| 6 | **Hoàn thiện deployment blueprint và kế hoạch tuần 6**<br>- Xác định quy trình build/tag/push Docker image lên ECR bằng buildspec và CodeBuild như một thành phần hỗ trợ.<br>- Thiết kế ECS Fargate task: container port 7860, task execution role, task role, environment variables, health check và CloudWatch Logs.<br>- Thiết kế EFS file system, mount target, access point và mount path <code>/app/ktem_app_data</code> để duy trì dữ liệu.<br>- Hoàn thiện network flow: người dùng → ALB ở public subnets → ECS tasks ở private subnets → EFS mount targets; cấu hình Security Group theo nguyên tắc tối thiểu.<br>- Lưu Gemini API key dự kiến trong SSM Parameter Store, phân công nhiệm vụ và chuẩn bị checklist triển khai tuần 6. | 24/07/2026 | 24/07/2026 | [fcaj-rag-chat](https://github.com/ngocchau04/fcaj-rag-chat) |

### Kết quả đạt được tuần 5:

* **Chuẩn hóa AI configuration:** Xác định rõ chat model và embedding model Gemini mặc định, thống nhất cách truyền API key qua biến môi trường và tránh lưu secret trực tiếp trong repository.
* **Đơn giản hóa RAG pipeline:** Loại bỏ phụ thuộc reranking Cohere không cần thiết, giúp giảm số lượng credential phải quản lý và làm luồng cấu hình dễ tái hiện hơn.
* **Hoàn thiện container baseline:** Dockerfile, Docker Compose, volume mount và hướng dẫn chạy local được tổ chức nhất quán, tạo tiền đề để đóng gói cùng một image cho ECR và ECS Fargate.
* **Kiểm tra chức năng RAG:** Thực hiện có hệ thống các bước upload, index, retrieval và citation; đồng thời bổ sung negative test để dễ chẩn đoán lỗi model, credential, tệp đầu vào hoặc quyền ghi dữ liệu.
* **Xác nhận yêu cầu persistent data:** Kiểm tra dữ liệu cần giữ qua vòng đời container và ánh xạ đầy đủ tài liệu upload, ChromaDB, LanceDB, SQLite cùng cấu hình vào <code>ktem_app_data</code>.
* **Nắm được quy trình database migration:** Phân biệt schema conversion và data replication; hiểu replication instance, endpoints, full load, CDC, DMS Serverless, monitoring, troubleshooting và cleanup mà không nhầm DMS là thành phần của project RAG.
* **Củng cố tư duy bảo mật:** Hiểu Shared Responsibility Model, least privilege và vai trò của IAM, Cognito, Organizations, IAM Identity Center, KMS và Security Hub; áp dụng trực tiếp vào thiết kế task role và quản lý secret.
* **Hoàn thiện deployment blueprint:** Mô tả rõ luồng người dùng → ALB → ECS Fargate → EFS, quy trình image → ECR và các thành phần hỗ trợ gồm CodeBuild, SSM Parameter Store, IAM, VPC, Security Group và CloudWatch Logs.
* **Sẵn sàng cho giai đoạn triển khai:** Có checklist kỹ thuật, tiêu chí kiểm thử, rủi ro cần theo dõi và phân rã công việc để bắt đầu provisioning/build image ở tuần 6 mà không tuyên bố hệ thống đã production-ready trước khi kiểm thử trên AWS.

**Kết quả tổng quan:** Tuần 5 hoàn tất giai đoạn tìm hiểu và chuẩn bị project. Ứng dụng có cấu hình local rõ ràng, luồng RAG và cơ chế lưu dữ liệu được kiểm thử theo checklist, kiến thức migration/security được bổ sung, và kiến trúc AWS đã đủ chi tiết để chuyển sang triển khai ECR, ECS Fargate và EFS trong các tuần cuối.
