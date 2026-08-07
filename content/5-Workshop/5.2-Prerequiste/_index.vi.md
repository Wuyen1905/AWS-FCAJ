---
title : "Các bước chuẩn bị"
date : 2026-08-06
weight : 2
chapter : false
pre : " <b> 5.2. </b> "
---

#### 1. Chuẩn bị tài khoản và quyền AWS

Sử dụng một tài khoản AWS dành cho học tập hoặc sandbox. Người thực hiện cần có quyền tạo và quản lý các tài nguyên sau trong phạm vi workshop:

- EC2 instance, EBS volume, Security Group và IAM Role/Instance Profile.
- S3 bucket, object và bucket policy.
- CloudWatch Log Group, metric, dashboard và alarm.
- AWS Budget và email notification.

Không sử dụng tài khoản root cho thao tác hằng ngày. Nếu tài khoản do đơn vị quản lý, cần xác nhận quota, Region được phép dùng và chính sách dọn dẹp trước khi bắt đầu.

Workshop minh họa với Region `ap-southeast-1`. Có thể dùng Region khác nhưng EC2, EBS và các tài nguyên liên quan phải được tạo nhất quán. Bucket S3 có thể cùng Region để giảm độ phức tạp.

Kiểm tra danh tính AWS CLI sau khi kết nối vào EC2:

```bash
aws sts get-caller-identity
aws configure get region
```

Nếu lệnh đầu trả về IAM Role của EC2 thì không cần cấu hình access key tĩnh trên máy chủ.

#### 2. Chuẩn bị thông số triển khai

Ghi lại các giá trị sau trước khi tạo tài nguyên:

| Biến | Giá trị gợi ý | Ghi chú |
|---|---|---|
| `AWS_REGION` | `ap-southeast-1` | Region của workshop |
| `INSTANCE_NAME` | `fcaj-rag-chat-demo` | Tên EC2 |
| `INSTANCE_TYPE` | `t3.medium` | Có thể dùng `t3.small` nếu chỉ kiểm tra nhẹ |
| `APP_PORT` | `80` | Ánh xạ cổng ngoài tới cổng 7860 của Gradio |
| `DATA_DIR` | `/opt/fcaj/ktem_app_data` | Điểm gắn dữ liệu bền vững |
| `BACKUP_BUCKET` | `fcaj-rag-chat-backup-<account-id>` | Tên bucket phải duy nhất toàn cầu |
| `BACKUP_PREFIX` | `ktem_app_data/` | Phạm vi sao lưu trong bucket |
| `BUDGET_LIMIT` | `15 USD/tháng` | Ngưỡng mục tiêu cho kịch bản demo |

Không dùng nguyên chuỗi `<account-id>`. Thay bằng mã tài khoản hoặc hậu tố duy nhất, viết thường và không chứa khoảng trắng.

#### 3. Chuẩn bị Gemini API key

Tạo khóa cho project Gemini dùng riêng cho workshop và kiểm tra quota hiện có. Khóa sẽ được đặt trong tệp `.env` trên EC2 với tên biến:

```dotenv
GOOGLE_API_KEY=gia-tri-khoa-cua-ban
APP_PORT=80
```

Yêu cầu bảo mật:

- Không commit `.env` vào Git.
- Không đưa khóa vào Dockerfile hoặc lệnh `docker build --build-arg`.
- Không chụp màn hình hiển thị giá trị khóa.
- Giới hạn quyền đọc tệp bằng `chmod 600 .env`.
- Xoay hoặc thu hồi khóa ngay nếu bị lộ.

#### 4. Chuẩn bị mã nguồn

Repository của nhóm: [ngocchau04/fcaj-rag-chat](https://github.com/ngocchau04/fcaj-rag-chat).

Trên máy cá nhân, nên đọc trước các tệp:

- `Dockerfile`: image nhiều giai đoạn; workshop dùng target `lite`.
- `docker-compose.yml`: cổng container 7860, health check và volume `/app/ktem_app_data`.
- `flowsettings.py`: cấu hình Gemini chat, embedding và thư mục dữ liệu.
- `.env.example`: mẫu biến môi trường; cần bổ sung `GOOGLE_API_KEY` cho cấu hình của nhóm.

Kiểm tra mã nguồn cục bộ:

```bash
git clone https://github.com/ngocchau04/fcaj-rag-chat.git
cd fcaj-rag-chat
git status
docker compose config
```

`docker compose config` không được in ra khóa thật trong ảnh chụp báo cáo. Nếu cần lưu bằng chứng, hãy che hoặc thay khóa bằng giá trị giả trước khi chụp.

#### 5. Quy tắc Security Group

Tạo Security Group theo nguyên tắc tối thiểu:

| Loại | Giao thức/cổng | Nguồn | Mục đích |
|---|---|---|---|
| Inbound | TCP 22 | IP quản trị `/32` | SSH; bỏ rule nếu dùng Session Manager |
| Inbound | TCP 80 | IP người kiểm thử hoặc phạm vi được duyệt | Truy cập ứng dụng demo |
| Outbound | HTTPS 443 | Theo chính sách môi trường | GitHub, Gemini API, S3 và dịch vụ AWS |

Không mở cổng 7860 nếu đã ánh xạ cổng 80 tới 7860. Không mở SSH cho toàn Internet.

#### 6. Chọn EC2 và EBS

- Hệ điều hành: Ubuntu Server 22.04/24.04 LTS hoặc Amazon Linux 2023; các lệnh trong workshop dùng Ubuntu.
- Kiến trúc: `x86_64` để phù hợp target `linux/amd64`.
- Instance type: `t3.medium` cho build và demo; dừng instance khi không dùng.
- Root volume: đủ cho hệ điều hành, source và Docker layers.
- Data volume: EBS gp3 60 GB, cùng Availability Zone với EC2.
- Bật termination protection nếu tài khoản học tập cho phép và nhóm cần tránh xóa nhầm trong khi thực hiện.

#### 7. Checklist trước khi bắt đầu

- [ ] Tài khoản AWS và Region đã được xác nhận.
- [ ] Email nhận cảnh báo ngân sách có thể truy cập.
- [ ] Gemini API key hoạt động và chưa được đưa vào mã nguồn.
- [ ] IP quản trị hiện tại đã được xác định.
- [ ] Tên bucket duy nhất đã được chọn.
- [ ] Repository có thể clone và `docker compose config` hợp lệ.
- [ ] Nhóm thống nhất người chịu trách nhiệm tạo tài nguyên và người dọn dẹp.

{{% notice warning %}}
Workshop có thể phát sinh chi phí EC2, EBS, public IPv4, S3 và CloudWatch. Dừng EC2 khi tạm nghỉ và hoàn thành mục 5.6 sau buổi thực hành.
{{% /notice %}}
