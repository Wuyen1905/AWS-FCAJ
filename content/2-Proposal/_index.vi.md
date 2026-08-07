---
title: "Đề xuất dự án"
date: 2026-08-06
weight: 2
chapter: false
pre: " <b> 2. </b> "
---

# FCAJ RAG Chat
## Hệ thống hỏi đáp tài liệu có trích dẫn, triển khai trên AWS

### 1. Tóm tắt điều hành

FCAJ RAG Chat là ứng dụng hỏi đáp tài liệu dựa trên kỹ thuật Retrieval-Augmented Generation (RAG). Người dùng tải tài liệu lên hệ thống, ứng dụng chia nhỏ và lập chỉ mục nội dung, truy xuất các đoạn liên quan khi có câu hỏi, sau đó sử dụng Gemini để tạo câu trả lời kèm nguồn tham chiếu. Giải pháp được phát triển từ Kotaemon và đóng gói bằng Docker để nhóm có thể chạy thống nhất trên máy cá nhân cũng như trên AWS.

Trong phạm vi đồ án, nhóm đề xuất triển khai một bản thử nghiệm có kiểm soát trên Amazon EC2. Dữ liệu ứng dụng được duy trì trên Amazon EBS, bản sao lưu được lưu trong Amazon S3 riêng tư, còn chỉ số và nhật ký được theo dõi bằng Amazon CloudWatch. AWS Budgets hỗ trợ kiểm soát chi phí; Gemini API là dịch vụ bên ngoài AWS và chỉ được ứng dụng gọi ở phía máy chủ.

Mục tiêu của dự án không phải xây dựng ngay một hệ thống thương mại có tính sẵn sàng cao. Kết quả cần đạt là một bản trình diễn ổn định, có thể tái triển khai, bảo vệ khóa bí mật, giữ được dữ liệu sau khi khởi động lại và có quy trình sao lưu, khôi phục, giám sát rõ ràng.

### 2. Bối cảnh và vấn đề cần giải quyết

Nhóm học tập phải đọc nhiều tài liệu kỹ thuật, báo cáo và hướng dẫn thực hành. Cách tìm kiếm theo từ khóa thường trả về danh sách tài liệu nhưng không tổng hợp được câu trả lời theo ngữ cảnh. Khi sử dụng mô hình ngôn ngữ trực tiếp, câu trả lời có thể thiếu căn cứ hoặc không phản ánh đúng nội dung tài liệu của nhóm.

Các khó khăn chính gồm:

- Tài liệu nằm ở nhiều tệp, việc tìm lại nội dung mất thời gian.
- Câu trả lời từ mô hình ngôn ngữ có thể không chỉ ra đoạn tài liệu đã sử dụng.
- Môi trường chạy trên máy cá nhân khác nhau, khó tái hiện lỗi và bàn giao.
- Dữ liệu chỉ mục của ChromaDB, LanceDB và dữ liệu quản trị của SQLite có thể mất nếu container bị tạo lại mà không gắn vùng lưu trữ bền vững.
- Khóa Gemini, thông tin truy cập AWS và dữ liệu người dùng cần được tách khỏi mã nguồn.
- Tài nguyên AWS phải được giám sát và dừng khi không sử dụng để tránh phát sinh chi phí ngoài kế hoạch.

FCAJ RAG Chat giải quyết các vấn đề trên bằng một luồng xử lý tập trung: tiếp nhận tài liệu, tạo chỉ mục, truy xuất ngữ cảnh, sinh câu trả lời có trích dẫn và lưu trạng thái trên vùng dữ liệu bền vững.

### 3. Mục tiêu và tiêu chí thành công

#### Mục tiêu chức năng

- Cho phép tải lên các tài liệu được nhóm chấp nhận, ưu tiên PDF và tài liệu văn bản.
- Tạo chỉ mục và truy xuất các đoạn có liên quan đến câu hỏi.
- Sinh câu trả lời tiếng Việt hoặc tiếng Anh, kèm trích dẫn để người dùng kiểm tra nguồn.
- Hỗ trợ tạo, lựa chọn và quản lý bộ tài liệu theo phạm vi của Kotaemon.
- Duy trì dữ liệu khi container hoặc EC2 được khởi động lại.

#### Mục tiêu vận hành

- Triển khai được ứng dụng bằng Docker trên một EC2 instance.
- Chỉ mở các cổng thật sự cần thiết và giới hạn SSH theo địa chỉ quản trị.
- Không lưu API key trong Git, Docker image, ảnh chụp màn hình hoặc nhật ký.
- Sao lưu được thư mục dữ liệu lên S3 và thực hiện được ít nhất một lần khôi phục thử.
- Theo dõi trạng thái EC2, dung lượng lưu trữ, lỗi ứng dụng và ngưỡng ngân sách.

#### Chỉ số nghiệm thu dự kiến

| Nhóm tiêu chí | Mức cần đạt |
|---|---|
| Chức năng | Hoàn tất luồng tải tài liệu, lập chỉ mục, đặt câu hỏi và mở được nguồn trích dẫn |
| Chất lượng truy xuất | Ít nhất 80% câu hỏi trong bộ 20–30 câu lấy đúng đoạn tài liệu liên quan |
| Khả năng kiểm chứng | Ít nhất 90% câu trả lời trong bộ kiểm thử có trích dẫn hỗ trợ được nội dung chính |
| Hiệu năng bản trình diễn | Thời gian phản hồi trung vị không quá 15 giây trong điều kiện một vài người dùng đồng thời |
| Tính bền vững | Dữ liệu vẫn còn sau khi khởi động lại container và khôi phục được từ bản sao lưu S3 |
| Bảo mật | Không phát hiện khóa bí mật trong kho mã nguồn, image và log; quyền IAM tuân theo nguyên tắc tối thiểu |

Các ngưỡng trên được dùng cho bản thử nghiệm học thuật và sẽ được hiệu chỉnh sau khi nhóm đo đường cơ sở trên tập tài liệu thực tế.

### 4. Phạm vi dự án

#### Trong phạm vi

- Tùy chỉnh và cấu hình Kotaemon cho quy trình RAG của nhóm.
- Sử dụng Gemini cho mô hình hội thoại và embedding theo cấu hình dự án.
- Đóng gói ứng dụng bằng Docker và chuẩn hóa biến môi trường.
- Triển khai bản thử nghiệm trên EC2, sử dụng EBS 60 GB cho dữ liệu bền vững.
- Dùng S3 riêng tư để lưu bản sao lưu, CloudWatch để giám sát và AWS Budgets để cảnh báo chi phí.
- Kiểm thử chức năng, truy xuất, trích dẫn, khởi động lại, sao lưu và khôi phục.
- Xây dựng tài liệu triển khai, vận hành và dọn dẹp tài nguyên.

#### Ngoài phạm vi của bản thử nghiệm

- Cung cấp dịch vụ công cộng cho số lượng lớn người dùng.
- Cam kết tính sẵn sàng cao, tự động mở rộng hoặc khôi phục thảm họa đa vùng.
- Huấn luyện mô hình nền tảng mới hoặc xử lý dữ liệu nhạy cảm cấp sản xuất.
- Thanh toán, phân quyền doanh nghiệp phức tạp và kiểm toán tuân thủ chuyên sâu.

Khi cần mở rộng sau giai đoạn thử nghiệm, Docker image có thể được đưa lên Amazon ECR, workload chuyển sang Amazon ECS Fargate, dữ liệu dùng chung đặt trên Amazon EFS và lưu lượng đi qua Application Load Balancer. Đây là hướng phát triển tiếp theo, không phải điều kiện nghiệm thu của workshop hiện tại.

### 5. Kiến trúc giải pháp

#### Luồng xử lý chính

1. Người dùng truy cập giao diện web của FCAJ RAG Chat bằng trình duyệt.
2. Security Group chỉ cho phép lưu lượng ứng dụng và kết nối quản trị theo phạm vi đã cấu hình.
3. Container Kotaemon chạy trên EC2 tiếp nhận tài liệu và câu hỏi.
4. Ứng dụng tách đoạn, tạo embedding và lưu chỉ mục cùng dữ liệu quản trị trong thư mục `/app/ktem_app_data`.
5. Thư mục này được ánh xạ tới `/opt/fcaj/ktem_app_data` trên EBS để dữ liệu không phụ thuộc vòng đời container.
6. Khi xử lý câu hỏi, ứng dụng truy xuất ngữ cảnh phù hợp rồi gọi Gemini API qua kết nối ra ngoài để tạo câu trả lời.
7. Tiến trình sao lưu đồng bộ dữ liệu cần thiết từ EBS lên S3 riêng tư bằng IAM Role của EC2.
8. CloudWatch thu thập chỉ số và nhật ký; CloudWatch Alarm và AWS Budgets gửi cảnh báo cho người quản trị.

#### Thành phần và trách nhiệm

| Thành phần | Vai trò trong giải pháp |
|---|---|
| Amazon EC2 | Chạy Docker container của ứng dụng RAG; dùng `t3.medium` cho bản trình diễn chính |
| Amazon EBS gp3 60 GB | Lưu dữ liệu Kotaemon, ChromaDB, LanceDB, SQLite và tệp đã xử lý |
| Amazon S3 | Lưu bản sao lưu và bằng chứng kiểm thử; bucket chặn truy cập công cộng |
| IAM Role | Cho phép EC2 sao lưu và khôi phục đúng prefix S3 cần thiết mà không lưu access key trên máy chủ |
| Security Group | Kiểm soát cổng ứng dụng và SSH; giảm bề mặt truy cập công khai |
| Amazon CloudWatch | Theo dõi CPU, trạng thái EC2, nhật ký ứng dụng và cảnh báo cơ bản |
| AWS Budgets | Cảnh báo khi chi phí tiến gần ngưỡng đã thống nhất |
| Gemini API | Cung cấp mô hình tạo sinh và/hoặc embedding theo cấu hình; nằm ngoài AWS |
| Docker | Chuẩn hóa môi trường chạy và giúp tái triển khai nhất quán |

#### Nguyên tắc bảo mật

- Gemini API key chỉ được truyền qua biến môi trường hoặc kho bí mật phù hợp; không ghi trực tiếp vào mã nguồn.
- Bucket S3 bật Block Public Access, mã hóa phía máy chủ và chỉ cho IAM Role của ứng dụng truy cập phạm vi cần thiết.
- SSH chỉ mở cho địa chỉ quản trị; ưu tiên AWS Systems Manager Session Manager nếu môi trường cho phép.
- Nhật ký không chứa API key, credential, toàn bộ tài liệu hoặc dữ liệu cá nhân không cần thiết.
- Hệ thống thử nghiệm sử dụng HTTP qua public IPv4 để đơn giản hóa demo. Nếu mở cho người dùng ngoài nhóm, phải bổ sung tên miền, HTTPS và cơ chế xác thực trước khi vận hành.

### 6. Kế hoạch triển khai

| Giai đoạn | Công việc chính | Kết quả |
|---|---|---|
| Tuần 1 – Phân tích | Chốt loại tài liệu, luồng người dùng, bộ 20–30 câu hỏi và phân công trách nhiệm | Phạm vi, bộ kiểm thử và tiêu chí nghiệm thu |
| Tuần 2 – Chuẩn hóa ứng dụng | Kiểm tra repository, Dockerfile, Docker Compose, cấu hình Gemini, volume và cách quản lý biến môi trường | Bản chạy cục bộ có thể tái tạo |
| Tuần 3 – Triển khai AWS | Tạo EC2, Security Group, IAM Role, EBS; cài Docker và chạy ứng dụng | Bản demo truy cập được trên AWS |
| Tuần 4 – Dữ liệu và giám sát | Cấu hình S3 backup, thử khôi phục, CloudWatch Logs, Alarm và AWS Budget | Quy trình vận hành cơ bản |
| Tuần 5 – Kiểm thử | Kiểm thử tải tệp, lập chỉ mục, retrieval, citation, latency, restart, lỗi khóa và lỗi quyền | Báo cáo lỗi và kết quả đo |
| Tuần 6 – Hoàn thiện | Sửa lỗi, chạy demo, hoàn thiện tài liệu triển khai, vận hành, chi phí và dọn dẹp | Gói bàn giao và báo cáo cuối kỳ |

### 7. Ước tính chi phí

Bảng sau sử dụng giả định EBS gp3 60 GB, S3 khoảng 2 GB, CloudWatch Logs khoảng 1 GB với thời gian lưu 7 ngày, một alarm cơ bản và một public IPv4 trong thời gian EC2 chạy. Chi phí thực tế phụ thuộc khu vực, số giờ chạy, lưu lượng và chính sách giá tại thời điểm sử dụng.

| Kịch bản | `t3.small` | `t3.medium` |
|---|---:|---:|
| Demo 60 giờ/tháng | 9,53 USD | 11,36 USD |
| Demo 120 giờ/tháng | 11,71 USD | 15,35 USD |
| Chạy liên tục 24/7 | 33,73 USD | 55,89 USD |

Các con số đã gồm 15% dự phòng theo bảng ước tính của nhóm. Phương án khuyến nghị cho đồ án là `t3.medium`, chạy khoảng 60 giờ mỗi tháng và đặt ngân sách mục tiêu 15 USD/tháng. EC2 cần được dừng khi không sử dụng; EBS và S3 vẫn phát sinh chi phí lưu trữ khi EC2 đã dừng.

Gemini API được theo dõi tách khỏi chi phí AWS. Nếu thử nghiệm embedding trên AWS, Titan Text Embeddings V2 và Cohere Multilingual có thể được đánh giá như phương án thay thế, nhưng nhóm chỉ triển khai sau khi kiểm tra khu vực hỗ trợ, quota và giá hiện hành.

### 8. Kế hoạch kiểm thử và nghiệm thu

1. **Kiểm thử chức năng:** tải tệp hợp lệ, từ chối tệp không hỗ trợ, tạo chỉ mục, đặt câu hỏi, xem nguồn và xóa tài liệu thử nghiệm.
2. **Kiểm thử RAG:** sử dụng bộ câu hỏi có đáp án tham chiếu, ghi nhận đoạn truy xuất, câu trả lời, trích dẫn và lỗi sai.
3. **Kiểm thử bền vững:** khởi động lại container, dừng và khởi động EC2, sau đó xác nhận tài liệu và chỉ mục vẫn tồn tại.
4. **Kiểm thử khôi phục:** sao lưu lên S3, khôi phục vào thư mục trống và chạy lại một số câu hỏi đã biết.
5. **Kiểm thử lỗi:** dùng khóa không hợp lệ, thiếu quyền ghi dữ liệu, tài liệu lỗi, Gemini trả HTTP 429 và dung lượng gần đầy.
6. **Kiểm thử vận hành:** xác nhận CloudWatch nhận log, alarm chuyển trạng thái khi có điều kiện thử và AWS Budget gửi email xác nhận.

### 9. Rủi ro và biện pháp giảm thiểu

| Rủi ro | Ảnh hưởng | Biện pháp |
|---|---|---|
| Lộ Gemini API key hoặc AWS credential | Cao | Tách secret khỏi Git và image, dùng IAM Role, rà log và xoay khóa khi nghi ngờ |
| Mất dữ liệu hoặc bản sao lưu không dùng được | Cao | EBS bền vững, S3 riêng tư, sao lưu có kiểm tra và diễn tập khôi phục |
| Gemini trả HTTP 429 hoặc thay đổi quota | Trung bình | Giới hạn xử lý đồng thời, retry có exponential backoff và jitter, ghi nhận lỗi rõ ràng |
| Câu trả lời thiếu căn cứ | Cao | Chuẩn hóa tài liệu, điều chỉnh chunking, đánh giá retrieval và bắt buộc kiểm tra trích dẫn |
| EC2 quá tải hoặc phản hồi chậm | Trung bình | Theo dõi CPU, bộ nhớ, latency; giảm concurrency hoặc nâng cấu hình trong thời gian demo |
| Phát sinh chi phí ngoài dự kiến | Trung bình | Budget alert, lịch dừng EC2, giới hạn thời gian giữ log và kiểm tra tài nguyên sau mỗi buổi |
| Truy cập qua HTTP công khai | Cao nếu mở rộng | Giới hạn phạm vi demo; bổ sung HTTPS, xác thực và lớp phân phối phù hợp trước khi công khai |

### 10. Sản phẩm bàn giao và giá trị kỳ vọng

- Kho mã nguồn và hướng dẫn cấu hình không chứa khóa bí mật.
- Docker image và lệnh triển khai có thể tái tạo trên EC2.
- Bản demo FCAJ RAG Chat xử lý được tài liệu và trả lời có trích dẫn.
- Bộ câu hỏi đánh giá cùng báo cáo kết quả retrieval, citation và latency.
- Quy trình sao lưu, khôi phục, giám sát, kiểm soát chi phí và dọn dẹp tài nguyên.
- Tài liệu workshop song ngữ giúp thành viên khác có thể thực hiện lại toàn bộ quy trình.

Giá trị chính của đồ án là rút ngắn thời gian tra cứu, tăng khả năng kiểm chứng câu trả lời và giúp nhóm hiểu trọn vẹn mối liên hệ giữa ứng dụng AI, dữ liệu bền vững, bảo mật, giám sát và chi phí trên AWS.

Mã nguồn tham khảo của nhóm: [fcaj-rag-chat](https://github.com/ngocchau04/fcaj-rag-chat).
