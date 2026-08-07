---
title : "Cài Docker, chạy ứng dụng và kiểm tra"
date : 2026-08-06
weight : 2
chapter : false
pre : " <b> 5.3.2 </b> "
---

#### Bước 1: Cài công cụ cần thiết

Trên Ubuntu EC2:

```bash
sudo apt-get update
sudo apt-get install -y docker.io git awscli curl
sudo apt-get install -y docker-compose-v2 || sudo apt-get install -y docker-compose-plugin
sudo systemctl enable --now docker
sudo docker version
sudo docker compose version
```

Nếu tài khoản Ubuntu cần chạy Docker không có `sudo`, thêm người dùng vào nhóm `docker` rồi đăng xuất và đăng nhập lại. Không thay đổi quyền socket Docker thành chế độ mọi người đều truy cập.

#### Bước 2: Clone mã nguồn

```bash
sudo mkdir -p /opt/fcaj
sudo chown ubuntu:ubuntu /opt/fcaj
cd /opt/fcaj
git clone https://github.com/ngocchau04/fcaj-rag-chat.git app
cd app
git rev-parse --short HEAD
```

Ghi lại commit hash để bản triển khai có thể tái tạo. Khi cập nhật mã nguồn, đọc thay đổi trước khi chạy `git pull`; không ghi đè các tệp cấu hình cục bộ ngoài ý muốn.

#### Bước 3: Tạo tệp môi trường

Sao chép tệp mẫu rồi chỉnh bằng trình soạn thảo trên máy chủ:

```bash
cp .env.example .env
chmod 600 .env
nano .env
```

Thêm hoặc cập nhật:

```dotenv
GOOGLE_API_KEY=gia-tri-khoa-cua-ban
APP_PORT=80
```

Không dùng `echo` để in API key ra terminal vì giá trị có thể đi vào shell history hoặc ảnh chụp. Kiểm tra `.gitignore` và xác nhận `.env` không được Git theo dõi:

```bash
git status --short
git check-ignore .env
```

#### Bước 4: Build Docker image

Tệp Compose của nhóm sử dụng target `lite`. Build lần đầu có thể mất nhiều thời gian vì phải cài dependency và tải PDF.js:

```bash
cd /opt/fcaj/app
sudo docker compose build fcaj-rag-chat
sudo docker image ls fcaj-rag-chat
```

Nếu build dừng vì thiếu dung lượng, kiểm tra `df -h` và `sudo docker system df`. Chỉ xóa cache không còn dùng sau khi đã xác định đúng nguyên nhân.

#### Bước 5: Chạy ứng dụng

```bash
sudo docker compose up -d
sudo docker compose ps
sudo docker compose logs --tail=100 fcaj-rag-chat
```

Compose ánh xạ `${APP_PORT}:7860`, vì vậy với `APP_PORT=80`, ứng dụng được truy cập tại cổng 80. Health check gọi `http://localhost:7860/` bên trong container.

Chờ container khởi tạo rồi kiểm tra:

```bash
curl --fail --head http://localhost:80/
sudo docker inspect --format='{{json .State.Health}}' fcaj-rag-chat-fcaj-rag-chat-1
```

Tên container có thể khác nếu Compose project name thay đổi. Dùng `sudo docker compose ps --format json` hoặc `sudo docker ps` để lấy đúng tên.

#### Bước 6: Kiểm tra từ trình duyệt

1. Mở `http://PUBLIC_IP/` từ địa chỉ đã được Security Group cho phép.
2. Xác nhận trang Kotaemon hiển thị đầy đủ.
3. Tạo một phiên hỏi đáp thử nhưng chưa tải bộ tài liệu chính thức.
4. Kiểm tra log để đảm bảo không có stack trace liên tục hoặc API key bị ghi ra.

#### Xử lý lỗi thường gặp

| Hiện tượng | Kiểm tra | Hướng xử lý |
|---|---|---|
| Trình duyệt timeout | Security Group, public IPv4, container port | Chỉ mở TCP 80 cho IP kiểm thử và xác nhận Compose đã publish `0.0.0.0:80` |
| Container `unhealthy` | `docker compose logs`, thời gian `start_period` | Chờ khởi tạo, kiểm tra dependency và lỗi cấu hình Gemini |
| `GOOGLE_API_KEY` không được nhận | Tên biến trong `.env`, quyền đọc tệp | Dùng đúng tên biến và chạy lại `docker compose up -d --force-recreate` |
| HTTP 429 từ Gemini | Quota, số request đồng thời | Giảm concurrency, thử lại có giới hạn với exponential backoff và jitter |
| Build hết dung lượng | Root volume, Docker cache | Mở rộng volume hoặc xóa đúng cache/image không còn sử dụng |

{{% notice info %}}
Khi giao diện đã hoạt động, chuyển sang phần 5.4 để đặt `ktem_app_data` trên EBS trước khi nhập tài liệu và thực hiện bộ kiểm thử chính thức.
{{% /notice %}}
