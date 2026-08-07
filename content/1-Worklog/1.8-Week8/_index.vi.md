---
title: "Worklog Tuần 8"
date: 2024-01-01
weight: 1
chapter: false
pre: " <b> 1.8. </b> "
---

{{% notice info %}}
Nội dung tuần 8 là kế hoạch đã chốt tại thời điểm cập nhật ngày 07/08/2026. Các công việc diễn ra từ 10/08 đến 15/08/2026 sẽ được đối chiếu với kết quả thực tế, nhật ký hệ thống và bằng chứng trước khi đánh dấu hoàn thành.
{{% /notice %}}

### Mục tiêu tuần 8

- Triển khai bản thử nghiệm FCAJ RAG Chat trên AWS theo kiến trúc EC2, EBS, S3, CloudWatch và AWS Budgets.
- Kiểm chứng toàn bộ luồng tải tài liệu, lập chỉ mục, truy xuất, trả lời có trích dẫn và lưu dữ liệu bền vững.
- Thực hiện ít nhất một lần sao lưu và khôi phục có mã kiểm tra, không chỉ kiểm tra đối tượng tồn tại trên S3.
- Đo chất lượng RAG, thời gian phản hồi, lỗi phụ thuộc và các yêu cầu bảo mật cơ bản.
- Hoàn thiện báo cáo song ngữ, buổi trình diễn, bàn giao và dọn dẹp tài nguyên trước khi kết thúc kỳ thực tập.

### Kế hoạch công việc trong tuần

| Ngày | Công việc dự kiến | Ngày bắt đầu | Ngày hoàn thành dự kiến | Nguồn tài liệu |
|---|---|---|---|---|
| Thứ Hai | Chuẩn bị triển khai và cố định phiên bản<br><br>Xác nhận tài khoản AWS, Region, Availability Zone, email nhận cảnh báo và giới hạn ngân sách. Chốt mã commit hoặc image ID dùng cho buổi trình diễn, kiểm tra `.env` không được Git theo dõi và xác nhận Gemini API key hoạt động. Chuẩn bị 20–30 câu hỏi đánh giá, đáp án tham chiếu, tài liệu nguồn cùng tiêu chí truy xuất, trích dẫn và thời gian phản hồi. Rà lại danh sách tạo, xác minh và dọn dẹp từng tài nguyên. | 10/08/2026 | 10/08/2026 | [FCAJ RAG Chat Workshop](/vi/5-workshop/)<br>[fcaj-rag-chat](https://github.com/ngocchau04/fcaj-rag-chat) |
| Thứ Ba | Tạo hạ tầng và chạy ứng dụng<br><br>Tạo IAM Role cho EC2 với quyền S3 giới hạn theo bucket và prefix; tạo Security Group chỉ mở SSH cho IP quản trị và cổng 80 cho phạm vi kiểm thử. Khởi tạo `t3.medium`, gắn EBS gp3 60 GB, cài Docker, Git, AWS CLI rồi sao chép kho mã nguồn. Tạo `.env` với quyền 600, tạo image từ target `lite`, chạy Docker Compose, kiểm tra trạng thái và truy cập ứng dụng qua địa chỉ IPv4 công khai. Ghi lại instance ID, volume ID, image ID và mã commit. | 11/08/2026 | 11/08/2026 | [Triển khai ứng dụng RAG trên EC2](/vi/5-workshop/5.3-s3-vpc/)<br>[Amazon EC2 User Guide](https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/concepts.html) |
| Thứ Tư | Hoàn thiện lưu trữ, sao lưu và giám sát<br><br>Định dạng volume mới sau khi xác nhận đúng thiết bị, gắn EBS tại `/opt/fcaj/ktem_app_data`, cập nhật `fstab` bằng UUID và ghi đè volume Compose. Tải một tài liệu thử, khởi động lại container và EC2 để kiểm tra dữ liệu còn nguyên. Tạo S3 bucket riêng tư có Versioning và mã hóa, dừng ngắn ứng dụng, tạo gói nén cùng SHA-256 rồi khôi phục vào môi trường tách biệt. Cài CloudWatch Agent, tạo cảnh báo CPU, kiểm tra trạng thái, dung lượng và cấu hình AWS Budget 15 USD. | 12/08/2026 | 12/08/2026 | [Lưu trữ, sao lưu và giám sát](/vi/5-workshop/5.4-s3-onprem/)<br>[Amazon EBS User Guide](https://docs.aws.amazon.com/ebs/latest/userguide/what-is-ebs.html)<br>[Amazon S3 User Guide](https://docs.aws.amazon.com/AmazonS3/latest/userguide/Welcome.html) |
| Thứ Năm | Chạy kiểm thử nghiệm thu<br><br>Thực hiện các trường hợp tải PDF hợp lệ, tệp lỗi, tài liệu không hỗ trợ, câu hỏi có đáp án và câu hỏi ngoài tập tài liệu. Ghi đoạn truy xuất, câu trả lời, trích dẫn, thời gian phản hồi và kết luận cho từng câu. Thử một số ít người dùng đồng thời, theo dõi CPU, bộ nhớ, độ trễ, HTTP 429 và số lần thử lại Gemini. Kiểm tra bucket không công khai, IAM Role không truy cập ngoài phạm vi, `.env` có quyền 600, cổng 7860 không mở trực tiếp và nhật ký không chứa API key. | 13/08/2026 | 13/08/2026 | [Kiểm thử chất lượng, bảo mật và chi phí](/vi/5-workshop/5.5-policy/)<br>[Gemini API troubleshooting](https://ai.google.dev/gemini-api/docs/troubleshooting) |
| Thứ Sáu | Sửa lỗi, hoàn thiện báo cáo và trình bày bản thử nghiệm<br><br>Phân loại lỗi theo ứng dụng, tài liệu, mô hình, quyền, mạng và dung lượng. Sửa các lỗi có thể xử lý trong phạm vi đồ án, chạy lại trường hợp kiểm thử bị ảnh hưởng và ghi rõ giới hạn còn tồn tại. Cập nhật kết quả thực tế vào Worklog tuần 8, bảng nghiệm thu, phần tự đánh giá, Proposal và Workshop nếu kiến trúc hoặc số liệu thay đổi. Chuẩn bị luồng trình diễn: mở ứng dụng, tải tài liệu, hỏi đáp có trích dẫn, khởi động lại để kiểm tra dữ liệu, trình bày sao lưu, khôi phục, CloudWatch và Budget. Bàn giao mã nguồn, tài liệu và danh sách tài nguyên cho nhóm. | 14/08/2026 | 14/08/2026 | [Đề xuất dự án](/vi/2-proposal/)<br>[Hugo documentation](https://gohugo.io/documentation/) |
| Thứ Bảy | Dọn dẹp, đối chiếu chi phí và tổng kết<br><br>Xác nhận gói sao lưu, mã kiểm tra, ảnh minh chứng và biên bản khôi phục đã được lưu đúng nơi. Dừng container và CloudWatch Agent, sau đó chọn giữ tạm hoặc xóa hoàn toàn theo quyết định của nhóm. Nếu xóa, xử lý đúng EC2, EBS, địa chỉ IPv4 công khai, các phiên bản đối tượng S3, Log Group, cảnh báo, SNS, IAM Role, chính sách, Security Group và Budget; không xóa tài nguyên dùng chung. Kiểm tra trang thanh toán, ghi các khoản có thể cập nhật trễ, tổng kết bài học và hướng phát triển ECR, ECS Fargate, EFS, HTTPS cùng xác thực. | 15/08/2026 | 15/08/2026 | [Dọn dẹp tài nguyên](/vi/5-workshop/5.6-cleanup/)<br>[AWS Billing and Cost Management](https://docs.aws.amazon.com/cost-management/latest/userguide/what-is-costmanagement.html) |

### Tiêu chí nghiệm thu

| Nhóm tiêu chí | Điều kiện cần đạt |
|---|---|
| Chức năng | Hoàn tất tải tài liệu, lập chỉ mục, đặt câu hỏi và mở được nguồn trích dẫn |
| Truy xuất | Ít nhất 80% câu hỏi có đáp án lấy được đoạn tài liệu phù hợp trong tập kết quả đầu |
| Trích dẫn | Ít nhất 90% câu trả lời được đánh giá có nguồn hỗ trợ cho nội dung chính |
| Câu hỏi ngoài tài liệu | Hệ thống ưu tiên trả lời không đủ thông tin, không khẳng định dữ kiện không có nguồn |
| Hiệu năng | Thời gian phản hồi trung vị không quá 15 giây trong điều kiện trình diễn có ít người dùng |
| Tính bền vững | Tài liệu và chỉ mục còn sau khi khởi động lại; gói sao lưu S3 có mã kiểm tra đúng và khôi phục được |
| Bảo mật | Không có API key trong Git, image, ảnh chụp hoặc nhật ký; S3 chặn truy cập công khai; IAM và Security Group đúng phạm vi |
| Chi phí | Tài nguyên nằm trong kế hoạch, Budget 15 USD hoạt động và EC2 được dừng khi không sử dụng |

Các ngưỡng là mục tiêu của bản thử nghiệm học thuật. Nếu chưa đạt, báo cáo phải ghi kết quả thật, nguyên nhân, mức ảnh hưởng và hướng xử lý thay vì thay đổi số liệu để phù hợp mục tiêu.

### Sản phẩm cần hoàn tất

- Bản triển khai hoặc biên bản thử nghiệm ghi rõ phiên bản mã nguồn, cấu hình và giới hạn.
- Bộ 20–30 câu hỏi đánh giá kèm đáp án, đoạn nguồn và kết quả chấm.
- Gói sao lưu, SHA-256, biên bản khôi phục và thời gian khôi phục.
- Ảnh chỉ số, nhật ký, cảnh báo CloudWatch và xác nhận AWS Budget.
- Báo cáo song ngữ được Hugo xây dựng thành công, không còn nội dung mẫu ở các phần liên quan.
- Hướng dẫn triển khai, vận hành, xử lý lỗi, kiểm thử và dọn dẹp.
- Biên bản bàn giao gồm người sở hữu mã nguồn, dữ liệu, bucket và tài nguyên được giữ lại.

### Rủi ro và phương án dự phòng

| Rủi ro | Phương án |
|---|---|
| Không đủ quyền hoặc quota AWS | Hoàn thiện phần có thể kiểm tra trên máy cá nhân, lưu cấu hình và ghi rõ bước bị chặn; không dùng credential cá nhân ngoài quy định |
| Quá trình tạo image quá lâu hoặc hết dung lượng | Dùng target `lite`, theo dõi root volume, tái sử dụng image đã xác minh và chỉ xóa bộ nhớ đệm không còn cần thiết |
| Gemini trả HTTP 429 hoặc gián đoạn | Giảm số request đồng thời, thử lại có giới hạn theo exponential backoff và jitter, ghi lại quota và lỗi |
| Kết quả truy xuất hoặc trích dẫn chưa đạt | Rà chất lượng tài liệu, cách chia đoạn, embedding, top-k và prompt; chạy lại đúng tập câu hỏi |
| Không kịp triển khai đầy đủ trên AWS | Ưu tiên luồng quan trọng: ứng dụng, dữ liệu bền vững, sao lưu, khôi phục và chi phí; ghi phần chưa hoàn tất vào giới hạn |
| Nguy cơ phát sinh phí sau kỳ thực tập | Phân công một người dọn dẹp, đối chiếu ID và tag, kiểm tra Billing lại vào ngày hôm sau |

> Trạng thái tại ngày 07/08/2026: Kế hoạch tuần 8 đã hoàn thiện, tài liệu triển khai và tiêu chí nghiệm thu đã sẵn sàng. Kết quả thực tế chỉ được xác nhận sau khi các công việc từ ngày 10/08 đến 15/08 hoàn thành và có bằng chứng tương ứng.
