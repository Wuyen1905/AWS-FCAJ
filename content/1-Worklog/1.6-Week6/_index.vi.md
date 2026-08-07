---
title: "Worklog Tuần 6"
date: 2024-01-01
weight: 1
chapter: false
pre: " <b> 1.6. </b> "
---

### Mục tiêu tuần 6

- Chuyển từ bản thiết kế tuần 5 sang các cấu hình có thể kiểm tra được cho đồ án FCAJ RAG Chat.
- Hoàn thiện quy trình tạo Docker image và chuẩn bị phương án lưu trữ trên Amazon ECR.
- Làm rõ cách ECS Fargate sử dụng EFS, IAM Role, SSM Parameter Store và CloudWatch Logs.
- Đánh giá tính phù hợp của kiến trúc ECS Fargate, EFS, ALB so với phạm vi, ngân sách và thời gian còn lại.
- Chọn kiến trúc bản thử nghiệm có thể triển khai, kiểm thử và bàn giao trong kỳ thực tập.

### Công việc thực hiện trong tuần

| Ngày | Công việc | Ngày bắt đầu | Ngày hoàn thành | Nguồn tài liệu |
|---|---|---|---|---|
| Thứ Hai | Rà soát trạng thái đồ án và chuẩn hóa quá trình đóng gói<br><br>Kiểm tra lại Dockerfile nhiều giai đoạn, target `lite`, Docker Compose, cơ chế kiểm tra trạng thái và cổng 7860. Đối chiếu các thư viện phụ thuộc trong `pyproject.toml`, `uv.lock` và tệp cấu hình Gemini để bảo đảm Docker image có thể tái tạo. Rà soát `.gitignore`, `.env.example` và cách truyền `GOOGLE_API_KEY`, tránh đưa khóa thật vào mã nguồn hoặc image. Ghi lại mã commit, lệnh tạo image, dung lượng và các lỗi còn tồn tại. | 27/07/2026 | 27/07/2026 | [fcaj-rag-chat](https://github.com/ngocchau04/fcaj-rag-chat) |
| Thứ Ba | Hoàn thiện quy trình tạo và phân phối image<br><br>Chuẩn hóa `buildspec.yml` cho các bước đăng nhập Amazon ECR, tạo image, gắn thẻ và đẩy image. Phân biệt CodeBuild service role với ECS task execution role; xác định quyền ECR tối thiểu cho từng vai trò. Kiểm tra tên kho lưu trữ, thẻ image và cách dùng mã commit để truy vết phiên bản. Không đặt Gemini API key trong `buildspec.yml` hoặc biến môi trường của giai đoạn tạo image. | 28/07/2026 | 28/07/2026 | [Docker sample for CodeBuild](https://docs.aws.amazon.com/codebuild/latest/userguide/sample-docker.html)<br>[Amazon ECR getting started](https://docs.aws.amazon.com/AmazonECR/latest/userguide/getting-started-cli.html) |
| Thứ Tư | Phân tích lưu trữ bền vững bằng Amazon EFS<br><br>Xác định dữ liệu trong `ktem_app_data` cần giữ qua vòng đời container, gồm tài liệu tải lên, ChromaDB, LanceDB, SQLite và cấu hình người dùng. Thiết kế EFS mount target, access point, định danh POSIX và đường dẫn `/app/ktem_app_data`. Rà soát Security Group cho TCP 2049 giữa ECS task và EFS. Ghi nhận rủi ro khóa tệp và tính nhất quán khi nhiều task cùng truy cập SQLite hoặc kho vector; giới hạn bản thử nghiệm ở một task. | 29/07/2026 | 29/07/2026 | [Amazon EFS volumes with ECS](https://docs.aws.amazon.com/AmazonECS/latest/developerguide/efs-volumes.html)<br>[Amazon EFS access points](https://docs.aws.amazon.com/efs/latest/ug/efs-access-points.html) |
| Thứ Năm | Hoàn thiện cấu hình ECS Fargate và quản lý thông tin bí mật<br><br>Phác thảo task definition với cổng container 7860, CPU, bộ nhớ, EFS volume, cơ chế kiểm tra trạng thái và log driver `awslogs`. Phân biệt task role với task execution role. Chuẩn bị cách tham chiếu Gemini API key từ SSM Parameter Store thay vì ghi trực tiếp trong task definition. Rà soát luồng mạng từ ALB ở mạng con công khai đến ECS task ở mạng con riêng, đồng thời giới hạn Security Group theo đúng nguồn và cổng cần thiết. | 30/07/2026 | 30/07/2026 | [ECS task definitions](https://docs.aws.amazon.com/AmazonECS/latest/developerguide/task_definitions.html)<br>[SSM Parameter Store secrets in ECS](https://docs.aws.amazon.com/AmazonECS/latest/developerguide/secrets-envvar-ssm-paramstore.html)<br>[CloudWatch Logs for ECS](https://docs.aws.amazon.com/AmazonECS/latest/developerguide/using_awslogs.html) |
| Thứ Sáu | Đánh giá kiến trúc và điều chỉnh phạm vi bản thử nghiệm<br><br>So sánh hai phương án: ALB, ECS Fargate, EFS với EC2, EBS. Phương án ECS có khả năng quản lý container tốt hơn nhưng cần nhiều tài nguyên mạng, quyền, cấu hình lưu trữ và chi phí nền. Với mục tiêu trình diễn nội bộ, một người vận hành và thời gian thực tập còn ngắn, nhóm chọn EC2 chạy Docker, EBS lưu dữ liệu, S3 sao lưu, CloudWatch giám sát và AWS Budgets cảnh báo chi phí. Kiến trúc ECR, ECS Fargate, EFS được giữ làm hướng mở rộng sau khi bản thử nghiệm ổn định. | 31/07/2026 | 31/07/2026 | [Amazon EC2](https://docs.aws.amazon.com/ec2/)<br>[Amazon EBS](https://docs.aws.amazon.com/ebs/)<br>[AWS Well-Architected Framework](https://docs.aws.amazon.com/wellarchitected/latest/framework/welcome.html) |

### Kết quả đạt được tuần 6

- Cấu hình Docker được rà soát theo hướng có thể tái tạo, dùng target `lite`, cổng 7860 và cơ chế kiểm tra trạng thái nhất quán với mã nguồn.
- Quy trình ECR được mô tả rõ từ tạo image, gắn thẻ đến đẩy image; `buildspec.yml` không chứa khóa Gemini.
- Vai trò của CodeBuild service role, ECS task execution role và task role được phân biệt, tránh cấp một IAM Role quá rộng cho toàn bộ quy trình.
- Yêu cầu lưu trữ của ChromaDB, LanceDB và SQLite được ánh xạ vào `ktem_app_data`; rủi ro khi nhiều task ghi đồng thời đã được ghi nhận.
- Phương án sử dụng EFS access point và Security Group TCP 2049 được hoàn thiện ở mức thiết kế, chưa được xem là đã sẵn sàng cho nhiều ECS task.
- Cách lấy Gemini API key từ SSM Parameter Store và gửi nhật ký container tới CloudWatch Logs được xác định rõ.
- Nhóm đã so sánh độ phức tạp, khả năng vận hành và chi phí của hai kiến trúc thay vì tiếp tục dùng phương án ban đầu một cách máy móc.
- Kiến trúc EC2, EBS, S3, CloudWatch và AWS Budgets được chọn cho bản thử nghiệm học thuật; ECR, ECS Fargate và EFS trở thành lộ trình mở rộng.

### Vấn đề và hướng xử lý

| Vấn đề | Phân tích | Hướng xử lý |
|---|---|---|
| Quá trình tạo image tốn thời gian và dung lượng | Các thư viện phụ thuộc của Kotaemon, PDF.js và bộ xử lý tài liệu làm image lớn | Dùng target `lite`, cố định phiên bản thư viện, tận dụng bộ nhớ đệm có kiểm soát và ghi rõ mã commit |
| Dữ liệu chưa phù hợp để mở rộng theo chiều ngang | SQLite và một số kho chỉ mục cần kiểm tra khóa tệp, tính nhất quán | Dùng một container trong bản thử nghiệm; chỉ mở rộng sau khi có kiểm thử đồng thời |
| Kiến trúc ECS có nhiều thành phần so với thời gian còn lại | Cần ALB, mạng con, IAM Role, EFS, nhật ký và xử lý lỗi task | Chọn EC2, EBS cho giai đoạn hiện tại; giữ tài liệu ECS làm phương án sau này |
| Nguy cơ lộ API key | Khóa có thể xuất hiện trong Git, `buildspec.yml`, task definition hoặc nhật ký | Tách khỏi mã nguồn; dùng tệp môi trường có quyền hạn chế hoặc kho bí mật phù hợp |

> Kết quả tổng quan: Tuần 6 chuyển đồ án từ một bản thiết kế thiên về ECS sang một quyết định kiến trúc có căn cứ. Nhóm vẫn hoàn thiện kiến thức và cấu hình liên quan đến ECR, ECS Fargate, EFS, nhưng lựa chọn EC2 và EBS cho bản thử nghiệm để bảo đảm có thể triển khai, kiểm thử, sao lưu và bàn giao trong thời gian thực tập.
