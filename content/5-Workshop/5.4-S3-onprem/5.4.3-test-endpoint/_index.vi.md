---
title : "Khôi phục dữ liệu và kiểm tra tính bền vững"
date : 2026-08-06
weight : 3
chapter : false
pre : " <b> 5.4.3 </b> "
---

#### Mục tiêu

Phần này khôi phục archive vào một thư mục tách biệt, kiểm tra checksum và chạy một container thử trên cổng 8080. Cách làm này không ghi đè dữ liệu đang hoạt động, phù hợp để chứng minh bản sao lưu có thể sử dụng.

#### Bước 1: Chọn bản sao lưu

Liệt kê archive:

```bash
export BACKUP_BUCKET=TEN_BUCKET_CUA_BAN
aws s3 ls "s3://${BACKUP_BUCKET}/ktem_app_data/archives/" --human-readable
```

Chọn một timestamp đã ghi ở phần 5.4.2 và đặt tên file, ví dụ:

```bash
export BACKUP_FILE=ktem_app_data-20260806T080000Z.tar.gz
```

#### Bước 2: Tải và kiểm tra checksum

```bash
aws s3 cp "s3://${BACKUP_BUCKET}/ktem_app_data/archives/${BACKUP_FILE}" "/tmp/${BACKUP_FILE}"
aws s3 cp "s3://${BACKUP_BUCKET}/ktem_app_data/archives/${BACKUP_FILE%.tar.gz}.sha256" "/tmp/${BACKUP_FILE%.tar.gz}.sha256"
cd /tmp
sha256sum -c "${BACKUP_FILE%.tar.gz}.sha256"
```

Chỉ tiếp tục khi kết quả là `OK`. Nếu checksum sai, không giải nén; tải lại object và so sánh đúng cặp archive/checksum theo timestamp.

#### Bước 3: Giải nén vào khu vực thử

```bash
sudo mkdir -p /opt/fcaj/restore-test
sudo tar --xattrs --acls -C /opt/fcaj/restore-test -xzf "/tmp/${BACKUP_FILE}"
sudo chown -R ubuntu:ubuntu /opt/fcaj/restore-test
find /opt/fcaj/restore-test/ktem_app_data -maxdepth 2 -type f | head -n 30
du -sh /opt/fcaj/restore-test/ktem_app_data
```

Không thấy file, kích thước bất thường hoặc lỗi giải nén phải được ghi nhận là khôi phục thất bại.

#### Bước 4: Chạy môi trường khôi phục riêng

Tạo `/opt/fcaj/app/docker-compose.restore.yml`:

```yaml
services:
  fcaj-rag-chat:
    volumes:
      - /opt/fcaj/restore-test/ktem_app_data:/app/ktem_app_data
```

Khởi động một Compose project khác trên cổng 8080, dùng lại image đã build:

```bash
cd /opt/fcaj/app
sudo env APP_PORT=8080 docker compose \
  -p fcaj-restore \
  -f docker-compose.yml \
  -f docker-compose.restore.yml \
  up -d --no-build
sudo env APP_PORT=8080 docker compose \
  -p fcaj-restore \
  -f docker-compose.yml \
  -f docker-compose.restore.yml \
  ps
curl --fail --head http://localhost:8080/
```

Không cần mở cổng 8080 công khai. Có thể tạo SSH tunnel từ máy cá nhân:

```bash
ssh -i /duong-dan/key.pem -L 8080:localhost:8080 ubuntu@PUBLIC_IP
```

Sau đó mở `http://localhost:8080` trên máy cá nhân, xác nhận tài liệu đã khôi phục và chạy lại câu hỏi đã ghi trước khi sao lưu. So sánh đoạn truy xuất và nguồn trích dẫn, không chỉ kiểm tra giao diện có mở được hay không.

#### Bước 5: Kết thúc môi trường thử

```bash
cd /opt/fcaj/app
sudo env APP_PORT=8080 docker compose \
  -p fcaj-restore \
  -f docker-compose.yml \
  -f docker-compose.restore.yml \
  down
rm "/tmp/${BACKUP_FILE}" "/tmp/${BACKUP_FILE%.tar.gz}.sha256"
```

Giữ thư mục `restore-test` đến khi ghi nhận xong bằng chứng. Sau đó có thể xóa theo quy trình đã xác nhận, không xóa nhầm `/opt/fcaj/ktem_app_data` đang hoạt động.

#### Biên bản khôi phục

| Trường | Nội dung cần ghi |
|---|---|
| Backup ID | Tên archive và thời gian UTC |
| Integrity | Kết quả SHA-256 |
| Application version | Commit hash hoặc image ID |
| Data check | Tài liệu, collection và số lượng file quan trọng |
| RAG check | Câu hỏi, đoạn truy xuất, câu trả lời và citation |
| Restore time | Thời gian từ lúc tải đến khi ứng dụng sẵn sàng |
| Kết luận | Đạt, đạt có điều kiện hoặc không đạt |

Một bản sao lưu chỉ được đánh dấu “đạt” khi checksum đúng, ứng dụng đọc được dữ liệu và câu hỏi kiểm tra trả về nguồn phù hợp.
