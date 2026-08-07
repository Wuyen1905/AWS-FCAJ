---
title: "Các bài blogs đã đăng"
date: 2024-01-01
weight: 3
chapter: false
pre: " <b> 3. </b> "
---

Phần này giới thiệu các bài viết mà nhóm đã đăng trên cộng đồng [AWS Study Group](https://www.facebook.com/groups/awsstudygroupfcj), đồng thời tổng hợp nội dung chính và những kiến thức rút ra trong quá trình tìm hiểu AWS.

### [Blog 1 - EC2 tưởng đơn giản mà không đơn giản](3.1-Blog1/)

Bài viết chia sẻ những bài học thực tế khi làm lab Amazon EC2: phân biệt Stop và Terminate, kiểm tra chi phí EBS, theo dõi quyền lợi Free Tier, tự động dừng instance bằng EventBridge–Lambda và duy trì checklist cleanup sau mỗi buổi lab.

### [Blog 2 - Xử lý lỗi HTTP 429 khi gọi Gemini API trên AWS](3.2-Blog2/)

Bài viết phân tích vì sao ứng dụng RAG chạy ổn trên local nhưng gặp HTTP 429 khi nhiều worker cùng gọi Gemini API trên EC2. Nhóm đề xuất retry có giới hạn với exponential backoff và jitter, kiểm soát concurrency, bảo vệ tính idempotent, fallback có điều kiện sang Amazon Bedrock và giám sát bằng CloudWatch.

### [Blog 3 - Bảo vệ API key trên ECS bằng Parameter Store](3.3-Blog3/)

Bản nháp này chia sẻ sai lầm khi hardcode Gemini API key trong ECS task definition và quy trình khắc phục bằng Parameter Store SecureString, AWS KMS cùng IAM least privilege. Bài viết cũng làm rõ Task Execution Role, Task Role, secret rotation, force new deployment và thời điểm nên cân nhắc AWS Secrets Manager.
