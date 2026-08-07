---
title : "Tạo S3 bucket riêng tư và sao lưu"
date : 2026-08-06
weight : 2
chapter : false
pre : " <b> 5.4.2 </b> "
---

#### Bước 1: Tạo bucket

1. Mở **Amazon S3 → Create bucket**.
2. Nhập tên duy nhất, ví dụ `fcaj-rag-chat-backup-<account-id>`.
3. Chọn Region đã thống nhất.
4. Giữ **Block all public access** ở trạng thái bật.
5. Bật **Bucket Versioning** để giảm rủi ro ghi đè nhầm.
6. Bật mã hóa mặc định bằng SSE-S3 hoặc AWS KMS theo quy định tài khoản.
7. Tạo bucket và xác nhận mục Permissions không có public access.

Có thể thêm lifecycle rule để chuyển hoặc xóa archive cũ sau thời gian được nhóm phê duyệt. Không đặt vòng đời quá ngắn trước khi hoàn thành báo cáo và kiểm thử khôi phục.

#### Bước 2: Xác minh quyền từ EC2

Thay tên bucket trong các lệnh:

```bash
export BACKUP_BUCKET=TEN_BUCKET_CUA_BAN
aws s3api get-bucket-location --bucket "$BACKUP_BUCKET"
aws s3 ls "s3://$BACKUP_BUCKET/ktem_app_data/"
```

Nếu nhận `AccessDenied`, kiểm tra IAM Role, ARN bucket, điều kiện prefix và bucket policy. Không xử lý bằng cách cấp `AmazonS3FullAccess` cho role của ứng dụng.

#### Bước 3: Tạo bản sao lưu nhất quán

Tạm dừng ứng dụng để SQLite và các kho chỉ mục không tiếp tục ghi:

```bash
cd /opt/fcaj/app
sudo docker compose stop fcaj-rag-chat
BACKUP_TIME=$(date -u +%Y%m%dT%H%M%SZ)
sudo tar --xattrs --acls -C /opt/fcaj -czf "/tmp/ktem_app_data-${BACKUP_TIME}.tar.gz" ktem_app_data
sudo chown ubuntu:ubuntu "/tmp/ktem_app_data-${BACKUP_TIME}.tar.gz"
sha256sum "/tmp/ktem_app_data-${BACKUP_TIME}.tar.gz" > "/tmp/ktem_app_data-${BACKUP_TIME}.sha256"
```

Tải archive và checksum lên S3:

```bash
aws s3 cp "/tmp/ktem_app_data-${BACKUP_TIME}.tar.gz" "s3://${BACKUP_BUCKET}/ktem_app_data/archives/"
aws s3 cp "/tmp/ktem_app_data-${BACKUP_TIME}.sha256" "s3://${BACKUP_BUCKET}/ktem_app_data/archives/"
sudo docker compose start fcaj-rag-chat
```

Không đưa `/opt/fcaj/app/.env` vào archive. Gói sao lưu chỉ chứa thư mục dữ liệu ứng dụng.

#### Bước 4: Kiểm tra bản sao

```bash
aws s3 ls "s3://${BACKUP_BUCKET}/ktem_app_data/archives/" --recursive --human-readable
aws s3api head-object \
  --bucket "$BACKUP_BUCKET" \
  --key "ktem_app_data/archives/ktem_app_data-${BACKUP_TIME}.tar.gz"
```

Ghi lại:

- Thời gian UTC của bản sao lưu.
- Commit hash của ứng dụng.
- Kích thước archive và SHA-256.
- Người thực hiện và kết quả khôi phục ở phần 5.4.3.

Sau khi tải thành công và ghi nhận bằng chứng, xóa file tạm trên EC2 để tránh đầy root volume:

```bash
rm "/tmp/ktem_app_data-${BACKUP_TIME}.tar.gz" "/tmp/ktem_app_data-${BACKUP_TIME}.sha256"
```

{{% notice info %}}
Versioning giúp giữ các phiên bản object nhưng không thay thế việc kiểm thử khôi phục. Tiếp tục phần 5.4.3 trước khi xem bản sao lưu là hoàn chỉnh.
{{% /notice %}}
