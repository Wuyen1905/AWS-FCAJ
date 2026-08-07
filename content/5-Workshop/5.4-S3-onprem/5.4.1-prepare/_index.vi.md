---
title : "Chuẩn bị và gắn EBS"
date : 2026-08-06
weight : 1
chapter : false
pre : " <b> 5.4.1 </b> "
---

#### Bước 1: Tạo EBS volume

1. Trong EC2 Console, mở **Elastic Block Store → Volumes → Create volume**.
2. Loại volume: `gp3`; dung lượng: `60 GiB`.
3. Chọn đúng Availability Zone của EC2 `fcaj-rag-chat-demo`.
4. Bật mã hóa; dùng AWS managed key hoặc KMS key theo quy định tài khoản.
5. Gắn tag `Name=fcaj-rag-chat-data`.
6. Tạo volume, chọn **Actions → Attach volume** và gắn vào EC2.

Tên thiết bị nhập trên Console có thể là `/dev/sdf`, nhưng trên máy Nitro volume thường xuất hiện dưới dạng `/dev/nvme1n1`. Luôn dùng `lsblk` để xác định thiết bị thật, không đoán theo tên đã nhập.

#### Bước 2: Xác định volume và filesystem

```bash
lsblk -f
sudo file -s /dev/nvme1n1
```

Chỉ tạo filesystem khi đây là volume mới và `file -s` cho biết thiết bị chưa có filesystem:

```bash
sudo mkfs.ext4 /dev/nvme1n1
```

{{% notice warning %}}
`mkfs` xóa khả năng truy cập dữ liệu hiện có trên volume. Không chạy lệnh nếu volume chứa dữ liệu hoặc nếu chưa xác nhận đúng thiết bị.
{{% /notice %}}

#### Bước 3: Gắn volume

```bash
sudo mkdir -p /opt/fcaj/ktem_app_data
sudo mount /dev/nvme1n1 /opt/fcaj/ktem_app_data
sudo chown -R ubuntu:ubuntu /opt/fcaj/ktem_app_data
df -h /opt/fcaj/ktem_app_data
```

Lấy UUID:

```bash
sudo blkid /dev/nvme1n1
```

Mở `/etc/fstab` bằng `sudo nano /etc/fstab` và thêm một dòng, thay UUID bằng giá trị thật:

```text
UUID=UUID_CUA_VOLUME /opt/fcaj/ktem_app_data ext4 defaults,nofail 0 2
```

Kiểm tra cú pháp trước khi khởi động lại:

```bash
sudo mount -a
findmnt /opt/fcaj/ktem_app_data
```

#### Bước 4: Chuyển dữ liệu thử sang EBS

Dừng ứng dụng để không có tiến trình ghi trong lúc sao chép:

```bash
cd /opt/fcaj/app
sudo docker compose down
sudo rsync -aHAX --numeric-ids /opt/fcaj/app/ktem_app_data/ /opt/fcaj/ktem_app_data/
```

Nếu thư mục nguồn trống, lệnh vẫn hợp lệ. Không sao chép `.env` vì tệp này nằm ở thư mục ứng dụng, không nằm trong `ktem_app_data`.

#### Bước 5: Ghi đè volume của Compose

Tạo `/opt/fcaj/app/docker-compose.override.yml`:

```yaml
services:
  fcaj-rag-chat:
    volumes:
      - /opt/fcaj/ktem_app_data:/app/ktem_app_data
```

Kiểm tra cấu hình hợp nhất:

```bash
cd /opt/fcaj/app
sudo docker compose config
sudo docker compose up -d
sudo docker compose ps
```

Xác minh mount trong container:

```bash
sudo docker compose exec fcaj-rag-chat sh -c 'mount | grep /app/ktem_app_data || true; ls -la /app/ktem_app_data | head'
```

#### Bước 6: Kiểm tra tính bền vững

1. Tải một tài liệu thử nhỏ và tạo chỉ mục.
2. Ghi lại tên tài liệu cùng một câu hỏi có đáp án.
3. Khởi động lại container:

```bash
sudo docker compose restart fcaj-rag-chat
sudo docker compose ps
```

4. Mở lại ứng dụng và xác nhận tài liệu cùng phiên kiểm thử vẫn còn.
5. Nếu chính sách workshop cho phép, dừng rồi khởi động EC2; sau đó chạy `findmnt`, `docker compose ps` và kiểm tra lại dữ liệu.

Kết quả này chứng minh dữ liệu đã tách khỏi vòng đời container và EBS được gắn lại đúng cấu hình.
