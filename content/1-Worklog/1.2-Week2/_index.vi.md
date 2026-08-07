---
title: "Worklog Tuần 2"
date: 2024-01-01
weight: 1
chapter: false
pre: " <b> 1.2. </b> "
---

### Mục tiêu tuần 2

* Áp dụng AWS Cloud9 và AWS CLI để tạo, kiểm tra và quản lý tài nguyên AWS trên môi trường cloud.
* Hoàn thành năm nhiệm vụ Explore AWS với Amazon EC2, Amazon Bedrock, AWS Budgets, AWS Lambda và Amazon RDS.
* Thực hành lưu trữ website tĩnh trên Amazon S3, phân phối nội dung bằng Amazon CloudFront và bảo vệ dữ liệu bằng Versioning, Cross-Region Replication.
* Triển khai cơ sở dữ liệu quan hệ bằng Amazon RDS, kết nối ứng dụng và thực hành sao lưu, khôi phục.
* Xây dựng kiến trúc có tính sẵn sàng cao và khả năng mở rộng với Launch Template, Application Load Balancer và Auto Scaling Group.

### Các công việc đã triển khai trong tuần

| Thứ | Công việc | Ngày bắt đầu | Ngày hoàn thành | Tài liệu tham khảo |
| --- | --------- | ------------ | --------------- | ------------------ |
| 2 | **AWS Cloud9, CLI và quản lý chi phí**<br>- Tạo môi trường AWS Cloud9, làm quen với trình soạn thảo, terminal và thao tác với tệp văn bản.<br>- Sử dụng AWS CLI trong Cloud9 để kiểm tra và tương tác với tài nguyên AWS.<br>- Hoàn thành nhiệm vụ Explore AWS về khởi tạo EC2 instance, sau đó terminate tài nguyên.<br>- Tạo AWS Budget và thiết lập email cảnh báo chi phí. | 29/06/2026 | 29/06/2026 | [Getting Started with AWS Cloud9](https://000049.awsstudygroup.com/)<br>[Hướng dẫn 5 nhiệm vụ Explore AWS](https://000001.awsstudygroup.com/4-h%C6%B0%E1%BB%9Bng-d%E1%BA%ABn-chi-ti%E1%BA%BFt-5-nhi%E1%BB%87m-v%E1%BB%A5-ki%E1%BA%BFm-ti%E1%BB%81n/) |
| 3 | **Lưu trữ và phân phối nội dung với S3, CloudFront**<br>- Tạo S3 bucket, tải dữ liệu và cấu hình static website hosting.<br>- Cấu hình quyền truy cập công khai và bucket policy để kiểm thử website.<br>- Tạo CloudFront distribution để phân phối nội dung qua CDN, sau đó chặn lại public access trực tiếp trên S3.<br>- Bật S3 Versioning, thực hành di chuyển object và Cross-Region Replication.<br>- Kiểm tra kết quả và dọn dẹp tài nguyên sau bài lab. | 30/06/2026 | 30/06/2026 | [Starting with Amazon S3](https://000057.awsstudygroup.com/) |
| 4 | **Quản lý cơ sở dữ liệu với Amazon RDS**<br>- Chuẩn bị VPC, Security Group cho EC2/RDS và DB Subnet Group.<br>- Khởi tạo EC2 instance cho ứng dụng và tạo Amazon RDS database instance.<br>- Triển khai ứng dụng, kiểm tra kết nối từ EC2 đến RDS và hoàn thành nhiệm vụ Explore AWS về RDS.<br>- Tạo snapshot, thực hành backup/restore và tìm hiểu point-in-time recovery.<br>- Xóa các tài nguyên không còn sử dụng. | 01/07/2026 | 01/07/2026 | [Getting Started with Amazon RDS](https://000005.awsstudygroup.com/)<br>[Hướng dẫn 5 nhiệm vụ Explore AWS](https://000001.awsstudygroup.com/4-h%C6%B0%E1%BB%9Bng-d%E1%BA%ABn-chi-ti%E1%BA%BFt-5-nhi%E1%BB%87m-v%E1%BB%A5-ki%E1%BA%BFm-ti%E1%BB%81n/) |
| 5 | **AI tạo sinh và ứng dụng serverless**<br>- Mở Amazon Bedrock Playground, chọn foundation model và gửi prompt để kiểm tra phản hồi.<br>- Hoàn thành thông tin use case cần thiết cho quyền truy cập model.<br>- Tạo Lambda web application từ blueprint HTTP, kiểm tra Function URL và phản hồi của ứng dụng.<br>- Hoàn thành hai nhiệm vụ Explore AWS về Bedrock và Lambda, sau đó xóa Lambda function để tránh phát sinh chi phí. | 02/07/2026 | 02/07/2026 | [Hướng dẫn 5 nhiệm vụ Explore AWS](https://000001.awsstudygroup.com/4-h%C6%B0%E1%BB%9Bng-d%E1%BA%ABn-chi-ti%E1%BA%BFt-5-nhi%E1%BB%87m-v%E1%BB%A5-ki%E1%BA%BFm-ti%E1%BB%81n/) |
| 6 | **Tính sẵn sàng cao và khả năng mở rộng**<br>- Chuẩn bị network, EC2, RDS và web server cho kiến trúc ứng dụng.<br>- Tạo Launch Template, Target Group và Application Load Balancer; cấu hình listener và kiểm tra phân phối lưu lượng.<br>- Tạo Auto Scaling Group gắn với Load Balancer.<br>- Kiểm thử manual scaling, scheduled scaling và dynamic scaling; quan sát các chỉ số phục vụ predictive scaling.<br>- Xác minh tính sẵn sàng của ứng dụng và dọn dẹp toàn bộ tài nguyên. | 03/07/2026 | 03/07/2026 | [Deploying an Application with Auto Scaling Group](https://000006.awsstudygroup.com/) |

### Kết quả đạt được tuần 2

* **Sử dụng môi trường phát triển trên cloud:** Thiết lập thành công AWS Cloud9 và làm quen với IDE chạy trực tiếp trên trình duyệt. Tôi có thể sử dụng code editor, terminal, thao tác với tệp và thực thi AWS CLI ngay trong môi trường Cloud9 để kiểm tra hoặc quản lý tài nguyên mà không cần chuyển đổi liên tục sang Console.

* **Hiểu rõ hơn vòng đời tài nguyên AWS:** Thông qua năm nhiệm vụ Explore AWS, tôi đã thực hành khởi tạo và terminate EC2 instance, chạy thử foundation model trong Amazon Bedrock, tạo AWS Budget, triển khai Lambda function và tạo RDS database. Việc thực hiện đầy đủ cả bước tạo, kiểm thử và cleanup giúp tôi hiểu rằng quản trị cloud không chỉ là triển khai tài nguyên mà còn phải kiểm soát trạng thái, chi phí và vòng đời của chúng.

* **Triển khai lưu trữ đối tượng và website tĩnh với Amazon S3:** Biết cách tạo bucket, tải object, cấu hình static website hosting và sử dụng bucket policy để kiểm soát quyền truy cập. Tôi cũng hiểu vai trò của S3 Versioning trong việc bảo vệ các phiên bản dữ liệu và Cross-Region Replication (CRR) trong việc sao chép object sang Region khác để tăng khả năng phục hồi.

* **Phân phối nội dung bằng Amazon CloudFront:** Tạo CloudFront distribution với S3 làm origin và kiểm tra việc phân phối nội dung qua mạng lưới CDN. Qua bài lab, tôi hiểu CloudFront giúp giảm độ trễ truy cập, cache nội dung tại Edge Location và hạn chế việc người dùng truy cập trực tiếp vào S3 origin.

* **Triển khai và bảo vệ cơ sở dữ liệu Amazon RDS:** Chuẩn bị VPC, Security Group và DB Subnet Group trước khi tạo RDS database instance. Tôi đã kết nối ứng dụng trên EC2 với RDS, hiểu cách Security Group kiểm soát luồng kết nối giữa application tier và database tier, đồng thời thực hành snapshot, restore và tìm hiểu point-in-time recovery.

* **Thực hành Serverless và Generative AI:** Sử dụng Amazon Bedrock Playground để lựa chọn foundation model, gửi prompt và đánh giá phản hồi. Bên cạnh đó, tôi đã tạo một HTTP web application bằng AWS Lambda, kiểm thử Function URL và hiểu được mô hình serverless cho phép chạy code theo yêu cầu mà không phải trực tiếp quản trị máy chủ.

* **Xây dựng kiến trúc có tính sẵn sàng cao:** Tạo Launch Template để chuẩn hóa cấu hình EC2, Target Group để quản lý các backend instance và Application Load Balancer (ALB) để phân phối request. Tôi cũng cấu hình health check để ALB chỉ chuyển lưu lượng đến các instance đang hoạt động bình thường.

* **Thực hành khả năng mở rộng với Auto Scaling Group:** Gắn Auto Scaling Group (ASG) với ALB và kiểm thử các cơ chế manual scaling, scheduled scaling và dynamic scaling. Qua đó, tôi hiểu ASG duy trì số lượng instance mong muốn, thay thế instance không khỏe mạnh và điều chỉnh capacity dựa trên nhu cầu tải; các chỉ số lịch sử cũng có thể được dùng để hỗ trợ predictive scaling.

* **Kiểm soát chi phí và đảm bảo resource hygiene:** Thiết lập AWS Budgets cùng email notification để theo dõi mức sử dụng chi phí. Sau mỗi bài lab, tôi chủ động xóa EC2, RDS, Lambda, Load Balancer, Auto Scaling Group và các tài nguyên liên quan không còn cần thiết nhằm tránh phát sinh chi phí ngoài dự kiến.

> **Kết quả tổng thể:** Sau tuần 2, tôi đã có thể kết hợp nhiều dịch vụ AWS thành các workload hoàn chỉnh thay vì thao tác riêng lẻ trên từng dịch vụ. Tôi hiểu rõ hơn mối liên hệ giữa compute, storage, database, CDN, load balancing và auto scaling, đồng thời hình thành thói quen kiểm tra bảo mật, chi phí và cleanup tài nguyên trong mỗi lần triển khai.
