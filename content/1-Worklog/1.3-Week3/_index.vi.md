---
title: "Worklog Tuần 3"
date: 2024-01-01
weight: 1
chapter: false
pre: " <b> 1.3. </b> "
---

### Mục tiêu tuần 3

* Triển khai quy trình monitoring và observability với Amazon CloudWatch thông qua Metrics, Logs, Alarms và Dashboards.
* Xây dựng mô hình Hybrid DNS kết nối môi trường mô phỏng on-premises với AWS bằng Route 53 Resolver và Microsoft Active Directory.
* Chuyển từ thao tác thủ công trên Console sang quản trị và tự động hóa tài nguyên bằng AWS CLI v2; biết cách chẩn đoán các lỗi CLI thường gặp.
* Tìm hiểu chuyên sâu Module 03 về Amazon EC2, từ lựa chọn instance, AMI và storage đến user data, metadata, Auto Scaling và migration.
* Triển khai an toàn một Linux EC2 web server, cấu hình Security Group, kết nối SSH và kiểm tra ứng dụng qua trình duyệt.

### Các công việc đã triển khai trong tuần

| Thứ | Công việc | Ngày bắt đầu | Ngày hoàn thành | Tài liệu tham khảo |
| --- | --------- | ------------ | --------------- | ------------------ |
| 2 | **Monitoring và observability với Amazon CloudWatch**<br>- Theo dõi CloudWatch Metrics, sử dụng search expression, metric math và dynamic label.<br>- Thu thập, truy vấn log bằng CloudWatch Logs Insights và tạo Metric Filter từ dữ liệu log.<br>- Tạo CloudWatch Alarm cho chỉ số CPUUtilization và cấu hình thông báo khi vượt ngưỡng.<br>- Xây dựng CloudWatch Dashboard để theo dõi tập trung trạng thái và hiệu năng của tài nguyên EC2.<br>- Kiểm tra kết quả và cleanup các tài nguyên của bài lab. | 06/07/2026 | 06/07/2026 | [AWS CloudWatch Workshop](https://000008.awsstudygroup.com/) |
| 3 | **Hybrid DNS với Route 53 Resolver**<br>- Khởi tạo hạ tầng bằng CloudFormation, chuẩn bị Key Pair và Security Group.<br>- Kết nối an toàn vào Remote Desktop Gateway (RDGW) và triển khai Microsoft Active Directory.<br>- Tạo Route 53 Resolver Outbound Endpoint, Inbound Endpoint và Resolver Rule cho domain cần chuyển tiếp.<br>- Kiểm tra phân giải DNS hai chiều giữa môi trường mô phỏng on-premises và tài nguyên trên AWS.<br>- Dọn dẹp endpoint, rule và các tài nguyên liên quan sau khi hoàn thành. | 07/07/2026 | 07/07/2026 | [Set up Hybrid DNS with Route 53 Resolver](https://000010.awsstudygroup.com/) |
| 4 | **Quản trị hạ tầng và troubleshooting bằng AWS CLI v2**<br>- Kiểm tra phiên bản, cấu hình named profile, Region mặc định và các định dạng đầu ra JSON, text, table.<br>- Thực hành quản lý S3 bucket/object, Amazon SNS topic/subscription, IAM user/role, VPC, Subnet, Internet Gateway và EC2 bằng dòng lệnh.<br>- Sử dụng filter/query để đọc kết quả và tìm hiểu cách tự động hóa chuỗi lệnh bằng script.<br>- Chẩn đoán các lỗi thường gặp: sai cú pháp, sai Region, AccessDenied, credentials không hợp lệ và JSON không đúng định dạng; sử dụng tùy chọn `--debug` khi cần.<br>- Cleanup các tài nguyên đã tạo bằng CLI. | 08/07/2026 | 08/07/2026 | [Getting Started with the AWS CLI](https://000011.awsstudygroup.com/)<br>[AWS CLI v2 User Guide](https://docs.aws.amazon.com/cli/latest/userguide/cli-chap-welcome.html)<br>[Troubleshooting AWS CLI](https://docs.aws.amazon.com/cli/latest/userguide/cli-chap-troubleshooting.html)<br>[Video giới thiệu AWS CLI v2](https://www.youtube.com/watch?v=U5y7JI_mHk8) |
| 5 | **Module 03 – Compute VM on AWS**<br>- Phân tích EC2 instance family và cách chọn instance type theo đặc điểm workload.<br>- Tìm hiểu AMI, backup, Key Pair, EBS, EC2 Instance Store, User Data và Instance Metadata.<br>- Ôn lại cơ chế EC2 Auto Scaling dưới góc nhìn capacity và workload.<br>- So sánh các lựa chọn lưu trữ EFS/FSx, dịch vụ Amazon Lightsail và quy trình migration bằng AWS Application Migration Service (MGN). | 09/07/2026 | 09/07/2026 | [03-01 – EC2 Instance Types](https://www.youtube.com/watch?v=-t5h4N6vfBs)<br>[03-02 – AMI, Backup, Key Pair](https://www.youtube.com/watch?v=e7XeKdOVq40)<br>[03-03 – Elastic Block Store](https://www.youtube.com/watch?v=yAR6QRT3N1k)<br>[03-04 – EC2 Instance Store](https://www.youtube.com/watch?v=hKr_TfGP7NY)<br>[03-05 – EC2 User Data](https://www.youtube.com/watch?v=6IHNDJ85aoQ)<br>[03-06 – EC2 Metadata](https://www.youtube.com/watch?si=7UPcYjyhBr5NtpZM&v=_v_43Wi7zjo&feature=youtu.be)<br>[03-07 – EC2 Auto Scaling](https://www.youtube.com/watch?v=Ew3QRaKJQSA)<br>[EFS/FSx và Lightsail](https://www.youtube.com/watch?si=gDfz13c9xm7z0cP2&v=bbLcPitXJSY&feature=youtu.be)<br>[AWS Application Migration Service – MGN](https://www.youtube.com/watch?v=hFVYG8WqfU0) |
| 6 | **Triển khai Linux EC2 web server**<br>- Tạo Security Group theo nguyên tắc least privilege, mở SSH, HTTP và HTTPS cho đúng nguồn truy cập; phân biệt Security Group với Network ACL.<br>- Khởi tạo Amazon Linux 2023 instance, chọn AMI, instance type, Key Pair, VPC và public subnet phù hợp.<br>- Kết nối từ máy local bằng SSH và private key `.pem`, kiểm tra trạng thái hệ điều hành và network connectivity.<br>- Cài đặt, khởi động Nginx và triển khai một trang frontend mẫu; kiểm tra truy cập bằng Public DNS hoặc Public IPv4 qua trình duyệt.<br>- Tổng hợp kết quả, cập nhật worklog và terminate instance sau khi hoàn thành. | 10/07/2026 | 10/07/2026 | [Introduction to Amazon EC2](https://000004.awsstudygroup.com/)<br>[Create a Security Group for Linux](https://000004.awsstudygroup.com/2-prerequiste/2.3-createsecuritygrouplinux/)<br>[Launch Amazon Linux Instance](https://000004.awsstudygroup.com/4-launchlinuxinstance/4.1-createlinuxinstance/)<br>[Connect to Amazon Linux Instance](https://000004.awsstudygroup.com/4-launchlinuxinstance/4.2-connectlinuxinstance/) |

### Kết quả đạt được tuần 3

* **Xây dựng khả năng quan sát hệ thống:** Biết cách kết hợp CloudWatch Metrics, Logs, Alarms và Dashboards thành một quy trình observability cơ bản. Tôi có thể theo dõi chỉ số EC2, truy vấn log bằng Logs Insights, chuyển mẫu log thành metric và cấu hình cảnh báo để phát hiện sớm tình trạng bất thường thay vì chỉ kiểm tra thủ công khi có sự cố.

* **Hiểu luồng xử lý cảnh báo:** Nắm được mối liên hệ giữa metric, threshold, alarm state và notification. Việc trực quan hóa dữ liệu trên dashboard giúp tôi theo dõi xu hướng hiệu năng theo thời gian và có thêm dữ liệu để xác định nguyên nhân khi tài nguyên hoạt động không ổn định.

* **Triển khai kiến trúc Hybrid DNS:** Hiểu vai trò của Route 53 Resolver Inbound Endpoint, Outbound Endpoint và Resolver Rule trong việc chuyển tiếp DNS query giữa AWS và on-premises. Tôi đã kiểm tra được luồng phân giải tên miền hai chiều và hiểu vì sao Security Group, network route và domain rule phải được cấu hình đồng bộ.

* **Nâng cao kỹ năng AWS CLI:** Có thể sử dụng AWS CLI v2 để quản lý tài nguyên thuộc nhiều nhóm dịch vụ như S3, SNS, IAM, VPC và EC2. Tôi biết dùng named profile, lựa chọn Region, thay đổi output format và lọc kết quả để các lệnh dễ tái sử dụng trong script tự động hóa.

* **Cải thiện khả năng troubleshooting:** Biết kiểm tra lần lượt cú pháp lệnh, phiên bản CLI, Region, credentials và IAM permissions khi lệnh thất bại. Tôi cũng biết sử dụng `--debug` để xem quá trình tìm credentials, dựng request và nhận response, từ đó khoanh vùng các lỗi như AccessDenied, invalid credentials hoặc sai JSON.

* **Hiểu sâu hơn về Amazon EC2:** Phân biệt được các nhóm instance theo đặc điểm workload, vai trò của AMI, EBS và Instance Store, cũng như cách User Data hỗ trợ bootstrap khi instance khởi động. Tôi đồng thời hiểu mục đích của Instance Metadata, Auto Scaling và các lựa chọn mở rộng như EFS, FSx, Lightsail và MGN.

* **Triển khai Linux web server an toàn:** Tạo và cấu hình Amazon Linux instance, kết nối SSH bằng private key, cài đặt Nginx và xác minh website hoạt động qua trình duyệt. Tôi hiểu rõ hơn inbound/outbound rule của Security Group, nguyên tắc least privilege và sự khác nhau cơ bản giữa Security Group dạng stateful với Network ACL dạng stateless.

* **Liên hệ kiến thức với đề tài:** Việc triển khai frontend mẫu trên EC2 giúp tôi hình dung rõ hơn cách presentation layer của nền tảng E-commerce có thể được host, bảo vệ và giám sát trên AWS. Đây là bước chuẩn bị để kết nối frontend với backend và database trong các giai đoạn tiếp theo.

* **Duy trì resource hygiene:** Chủ động terminate EC2 instance và xóa alarm, dashboard, Resolver endpoint cùng các tài nguyên lab không còn sử dụng. Thói quen cleanup giúp hạn chế chi phí ngoài dự kiến và giữ môi trường AWS gọn, dễ kiểm soát.

> **Kết quả tổng thể:** Sau tuần 3, tôi đã chuyển từ việc triển khai từng dịch vụ sang quan sát, tự động hóa và xử lý sự cố cho hạ tầng AWS. Tôi có thể dùng CloudWatch để theo dõi hệ thống, Route 53 Resolver để hiểu kết nối hybrid, AWS CLI để quản trị tài nguyên và EC2 để triển khai một Linux web server có cấu hình mạng, bảo mật rõ ràng.
