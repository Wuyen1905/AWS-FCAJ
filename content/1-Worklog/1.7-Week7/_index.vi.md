---
title: "Worklog Tuần 7"
date: 2024-01-01
weight: 1
chapter: false
pre: " <b> 1.7. </b> "
---

### Mục tiêu tuần 7

- Chốt phạm vi và kiến trúc bản thử nghiệm FCAJ RAG Chat dựa trên quyết định của tuần 6.
- Lập bảng chi phí cho các kịch bản trình diễn và xác định ngưỡng ngân sách phù hợp.
- Viết mục Đề xuất dự án theo đúng đồ án của nhóm, thay toàn bộ nội dung mẫu không liên quan.
- Xây dựng Workshop song ngữ có thể dùng để triển khai, kiểm thử, sao lưu, khôi phục và dọn dẹp hệ thống.
- Rà soát tính nhất quán giữa Worklog, Proposal, Workshop, phần tự đánh giá và mô tả đồ án.

### Công việc thực hiện trong tuần

| Ngày | Công việc | Ngày bắt đầu | Ngày hoàn thành | Nguồn tài liệu |
|---|---|---|---|---|
| Thứ Hai | Chốt yêu cầu và kiến trúc bản thử nghiệm<br><br>Tổng hợp yêu cầu từ mã nguồn, kết quả kiểm thử trên máy cá nhân và bản thiết kế tuần 6. Xác định luồng người dùng truy cập địa chỉ IPv4 công khai của EC2, Docker ánh xạ cổng 80 vào cổng 7860 của Kotaemon, EBS gp3 60 GB lưu `ktem_app_data`, S3 lưu bản sao, IAM Role cấp quyền cho EC2, CloudWatch theo dõi chỉ số và nhật ký, AWS Budgets cảnh báo chi phí, Gemini API được gọi ra ngoài AWS. Ghi rõ đây là bản thử nghiệm nội bộ, chưa phải kiến trúc có tính sẵn sàng cao. | 03/08/2026 | 03/08/2026 | [fcaj-rag-chat](https://github.com/ngocchau04/fcaj-rag-chat)<br>[Amazon EC2 documentation](https://docs.aws.amazon.com/ec2/) |
| Thứ Ba | Xây dựng mô hình chi phí và phương án vận hành<br><br>Lập bảng ước tính cho `t3.small` và `t3.medium` theo ba mức sử dụng: 60 giờ, 120 giờ mỗi tháng và chạy liên tục. Bổ sung EBS gp3 60 GB, S3 2 GB, địa chỉ IPv4 công khai, CloudWatch Logs 1 GB lưu 7 ngày, một cảnh báo và 15% dự phòng. So sánh chi phí tạo embedding tham khảo giữa Titan Text Embeddings V2, Cohere Multilingual và Gemini. Chọn `t3.medium` khoảng 60 giờ mỗi tháng, ngân sách mục tiêu 15 USD; yêu cầu dừng EC2 khi không sử dụng. | 04/08/2026 | 04/08/2026 | [EC2 On-Demand pricing](https://aws.amazon.com/ec2/pricing/on-demand/)<br>[Amazon EBS pricing](https://aws.amazon.com/ebs/pricing/)<br>[AWS Pricing Calculator](https://calculator.aws/) |
| Thứ Tư | Viết mục Đề xuất dự án bằng tiếng Việt và tiếng Anh<br><br>Thay nội dung mẫu IoT Weather Platform bằng FCAJ RAG Chat. Hoàn thiện bối cảnh, vấn đề, mục tiêu, phạm vi, luồng kiến trúc, vai trò của từng dịch vụ, kế hoạch sáu tuần, chi phí, tiêu chí nghiệm thu, rủi ro và sản phẩm bàn giao. Đặt các ngưỡng đánh giá dự kiến: 20–30 câu hỏi, độ chính xác truy xuất từ 80%, tỷ lệ trích dẫn hỗ trợ từ 90% và thời gian phản hồi trung vị không quá 15 giây trong điều kiện trình diễn. Phân biệt rõ bản thử nghiệm EC2 hiện tại với lộ trình ECR, ECS Fargate, EFS về sau. | 05/08/2026 | 05/08/2026 | [AWS Well-Architected Framework](https://docs.aws.amazon.com/wellarchitected/latest/framework/welcome.html)<br>[fcaj-rag-chat](https://github.com/ngocchau04/fcaj-rag-chat) |
| Thứ Năm | Xây dựng nội dung Workshop từ 5.1 đến 5.6<br><br>Viết lại toàn bộ Workshop cũ về VPC Endpoint thành quy trình triển khai FCAJ RAG Chat. Nội dung gồm chuẩn bị tài khoản và thông số, tạo Security Group và IAM Role, cài Docker, tạo image, chạy Compose, gắn EBS, tạo S3 bucket riêng tư, sao lưu có mã kiểm tra, khôi phục bằng container riêng, cài CloudWatch Agent, tạo cảnh báo và AWS Budget, kiểm thử RAG, kiểm tra bảo mật, đối chiếu chi phí và dọn dẹp. Đối chiếu lệnh với Dockerfile, `docker-compose.yml`, `flowsettings.py` và `.env.example` của kho mã nguồn. | 06/08/2026 | 06/08/2026 | [fcaj-rag-chat](https://github.com/ngocchau04/fcaj-rag-chat)<br>[Amazon CloudWatch Agent](https://docs.aws.amazon.com/AmazonCloudWatch/latest/monitoring/Install-CloudWatch-Agent.html)<br>[AWS Budgets](https://docs.aws.amazon.com/cost-management/latest/userguide/budgets-managing-costs.html) |
| Thứ Sáu | Hoàn thiện Worklog và kiểm tra website báo cáo<br><br>Viết chi tiết Worklog tuần 6 và 7, xây dựng kế hoạch trung thực cho tuần 8. Cập nhật mục lục Nhật ký công việc ở cả hai ngôn ngữ và giảm các đoạn tiếng Anh không cần thiết trong bản tiếng Việt. Xây dựng website bằng Hugo vào thư mục tạm, kiểm tra đường dẫn, menu con, bảng, khối lệnh và khả năng hiển thị trên máy cá nhân. Rà lại liên kết giữa Proposal, Workshop và Worklog, đồng thời giữ nguyên các mục ngoài phạm vi chỉnh sửa. | 07/08/2026 | 07/08/2026 | [Hugo documentation](https://gohugo.io/documentation/)<br>[AWS Workshop sample repository](https://github.com/aws-samples/aws-modernization-workshop-sample) |

### Kết quả đạt được tuần 7

- Kiến trúc bản thử nghiệm được chốt với EC2, EBS, S3, IAM Role, Security Group, CloudWatch, AWS Budgets và Gemini API bên ngoài AWS.
- Phạm vi được mô tả rõ: phục vụ trình diễn và học tập nội bộ, chưa cam kết tự động mở rộng, đa vùng hoặc tính sẵn sàng cao.
- Bảng chi phí có ba kịch bản vận hành, hai loại EC2 và 15% dự phòng; phương án khuyến nghị phù hợp giới hạn 15 USD mỗi tháng.
- Mục Đề xuất dự án đã thay hoàn toàn nội dung mẫu, phản ánh đúng vấn đề, kiến trúc, kế hoạch, rủi ro và giá trị của FCAJ RAG Chat.
- Workshop 5.1–5.6 được xây dựng thành quy trình liên tục từ chuẩn bị, triển khai, lưu trữ, sao lưu, khôi phục, giám sát, kiểm thử đến dọn dẹp.
- Các lệnh Docker và tên biến môi trường được đối chiếu với repository thật, sử dụng `GOOGLE_API_KEY`, cổng 7860 và thư mục `ktem_app_data`.
- Bản tiếng Việt và tiếng Anh có cùng cấu trúc, số liệu và tiêu chí; bản tiếng Việt ưu tiên cách diễn đạt tự nhiên, chỉ giữ thuật ngữ chuyên ngành khi cần.
- Website được Hugo xây dựng thành công và các trang mục tiêu hiển thị đúng trên máy cá nhân mà không làm thay đổi những phần khác của báo cáo.

### Quyết định quan trọng trong tuần

| Quyết định | Lý do | Ảnh hưởng |
|---|---|---|
| Dùng EC2 và EBS cho bản thử nghiệm | Ít thành phần, dễ quan sát và phù hợp ngân sách hơn ECS Fargate, EFS, ALB | Workshop có thể thực hiện trong thời gian còn lại; khả năng mở rộng được chuyển sang giai đoạn sau |
| Sao lưu bằng gói nén có mã kiểm tra | Việc đồng bộ trực tiếp cơ sở dữ liệu đang ghi có thể tạo bản sao không nhất quán | Dừng ngắn ứng dụng, tạo gói nén, tính SHA-256 và khôi phục thử trước khi công nhận bản sao lưu |
| Không công khai dịch vụ ngay | Bản trình diễn đang dùng HTTP và địa chỉ IPv4 công khai, chưa có xác thực, tên miền và HTTPS | Chỉ cho IP kiểm thử truy cập; yêu cầu bổ sung bảo mật trước khi mở rộng |
| Tách chi phí Gemini khỏi AWS | Gemini API là dịch vụ bên ngoài AWS | Theo dõi quota và chi phí riêng, tránh nhầm vào bảng AWS Budget |

### Nội dung cần tiếp tục

- Thực hiện triển khai thử nghiệm theo Workshop trong tuần 8 sau khi xác nhận tài khoản và ngân sách.
- Chạy bộ câu hỏi đánh giá trên tài liệu thực tế, ghi kết quả truy xuất, trích dẫn, độ trễ và lỗi.
- Diễn tập sao lưu, khôi phục và kiểm tra dữ liệu sau khi khởi động lại container, EC2.
- Hoàn thiện bằng chứng, bàn giao, trình diễn cuối kỳ và dọn dẹp tài nguyên.

> Kết quả tổng quan: Tuần 7 hoàn thiện phần thiết kế, chi phí và tài liệu triển khai của FCAJ RAG Chat. Đồ án có kiến trúc phù hợp phạm vi, tiêu chí nghiệm thu có thể đo, Workshop có thể thực hiện lại và website báo cáo song ngữ đã sẵn sàng cho giai đoạn triển khai thử nghiệm, nghiệm thu trong tuần cuối.
