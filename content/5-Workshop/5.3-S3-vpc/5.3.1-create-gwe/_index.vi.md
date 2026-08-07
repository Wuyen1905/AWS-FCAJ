---
title : "Chuẩn bị EC2, Security Group và IAM Role"
date : 2026-08-06
weight : 1
chapter : false
pre : " <b> 5.3.1 </b> "
---

#### Bước 1: Tạo chính sách sao lưu S3

Trong IAM, tạo customer managed policy dành riêng cho prefix sao lưu. Thay `TEN_BUCKET_CUA_BAN` bằng bucket đã chọn ở phần 5.2:

```json
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Sid": "ReadBucketLocation",
      "Effect": "Allow",
      "Action": "s3:GetBucketLocation",
      "Resource": "arn:aws:s3:::TEN_BUCKET_CUA_BAN"
    },
    {
      "Sid": "ListBackupPrefix",
      "Effect": "Allow",
      "Action": "s3:ListBucket",
      "Resource": "arn:aws:s3:::TEN_BUCKET_CUA_BAN",
      "Condition": {
        "StringLike": {
          "s3:prefix": ["ktem_app_data", "ktem_app_data/*"]
        }
      }
    },
    {
      "Sid": "ReadWriteBackupObjects",
      "Effect": "Allow",
      "Action": ["s3:GetObject", "s3:PutObject"],
      "Resource": "arn:aws:s3:::TEN_BUCKET_CUA_BAN/ktem_app_data/*"
    }
  ]
}
```

Chính sách không cho phép xóa object. Nếu quy trình lưu giữ sau này cần xóa phiên bản cũ, hãy bổ sung quyền cho một vai trò vận hành riêng thay vì mở rộng quyền mặc định của ứng dụng.

#### Bước 2: Tạo IAM Role cho EC2

1. Mở **IAM → Roles → Create role**.
2. Chọn trusted entity là **AWS service**, use case **EC2**.
3. Gắn policy sao lưu vừa tạo.
4. Đặt tên, ví dụ `fcaj-rag-chat-ec2-role`.
5. Kiểm tra Trust relationship chỉ cho phép dịch vụ `ec2.amazonaws.com` đảm nhận role.

CloudWatch Agent sẽ được cấu hình ở phần 5.4.4. Khi đến bước đó, gắn thêm managed policy `CloudWatchAgentServerPolicy` hoặc một policy tối thiểu tương đương.

#### Bước 3: Tạo Security Group

Tạo Security Group `fcaj-rag-chat-sg` trong VPC dùng cho EC2:

- Inbound TCP 22 từ địa chỉ IP quản trị `/32`.
- Inbound TCP 80 từ địa chỉ người kiểm thử hoặc dải mạng đã được mentor duyệt.
- Không tạo inbound cho 7860 vì Docker sẽ ánh xạ cổng 80 của máy chủ vào 7860 của container.
- Giữ outbound HTTPS 443 theo chính sách môi trường để tải source, gọi Gemini và truy cập API AWS.

Sau khi xác nhận có thể dùng Session Manager, có thể xóa inbound TCP 22.

#### Bước 4: Khởi tạo EC2

1. Mở **EC2 → Instances → Launch instances**.
2. Name: `fcaj-rag-chat-demo`.
3. AMI: Ubuntu Server 22.04 hoặc 24.04 LTS, kiến trúc `x86_64`.
4. Instance type: `t3.medium`.
5. Chọn key pair đang được quản lý an toàn; nếu dùng Session Manager và chính sách cho phép, có thể không dùng key pair.
6. Chọn VPC/subnet có đường ra Internet cho bản demo và bật public IPv4.
7. Chọn Security Group vừa tạo.
8. Root volume cần đủ chỗ cho Docker image và source; dữ liệu dài hạn sẽ đặt trên EBS riêng ở phần 5.4.
9. Trong **Advanced details → IAM instance profile**, chọn `fcaj-rag-chat-ec2-role`.
10. Khởi tạo instance và chờ cả hai status check đều đạt.

#### Bước 5: Kết nối và xác minh

Kết nối SSH từ IP đã cho phép:

```bash
ssh -i /duong-dan/key.pem ubuntu@PUBLIC_IP
```

Kiểm tra hệ điều hành, dung lượng và IAM Role:

```bash
uname -m
df -h
aws sts get-caller-identity
```

Kết quả mong đợi:

- `uname -m` trả về `x86_64`.
- Root volume còn đủ dung lượng cho quá trình build.
- ARN từ STS thể hiện assumed role của EC2, không phải IAM user có access key lưu trên máy.

{{% notice warning %}}
Không chạy `aws configure` với access key cá nhân trên EC2. Nếu STS chưa hoạt động, hãy kiểm tra IAM instance profile, Region và kết nối mạng.
{{% /notice %}}
