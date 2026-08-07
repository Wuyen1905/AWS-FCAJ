---
title : "Dọn dẹp tài nguyên"
date : 2026-08-06
weight : 6
chapter : false
pre : " <b> 5.6. </b> "
---

Chúc mừng bạn đã hoàn thành workshop FCAJ RAG Chat. Dọn dẹp là một phần của bài thực hành vì EC2 đã dừng vẫn có thể để lại chi phí EBS, S3, public IPv4, CloudWatch và các tài nguyên liên quan.

#### 1. Xác nhận trước khi xóa

Trước khi thay đổi tài nguyên, ghi lại:

- EC2 instance ID, EBS volume ID, Security Group ID và IAM Role.
- Tên S3 bucket và prefix backup.
- Log Group, Alarm, SNS topic và AWS Budget.
- Bằng chứng đã bàn giao: ảnh demo, kết quả kiểm thử, archive, checksum và biên bản restore.

Thống nhất một trong hai phương án:

- **Tạm dừng:** dừng EC2 nhưng giữ EBS và backup để demo tiếp. Phương án này vẫn phát sinh chi phí lưu trữ và public IPv4 tùy cấu hình.
- **Xóa hoàn toàn:** chỉ thực hiện sau khi nhóm xác nhận dữ liệu cần giữ đã được tải về hoặc chuyển sang vị trí bàn giao.

#### 2. Dừng workload

Trên EC2:

```bash
cd /opt/fcaj/app
sudo docker compose down
sudo env APP_PORT=8080 docker compose \
  -p fcaj-restore \
  -f docker-compose.yml \
  -f docker-compose.restore.yml \
  down 2>/dev/null || true
sudo systemctl stop amazon-cloudwatch-agent
```

Xác nhận không còn container workshop chạy:

```bash
sudo docker ps --format 'table {{.Names}}\t{{.Status}}\t{{.Ports}}'
```

#### 3. Dọn tài nguyên quan sát

Trong CloudWatch và SNS:

1. Xóa các alarm có prefix `fcaj-rag-chat` sau khi lưu ảnh bằng chứng.
2. Xóa dashboard riêng của workshop nếu có.
3. Xóa Log Group `/fcaj/rag-chat/application` nếu không cần giữ log.
4. Xóa SNS subscription và topic chỉ dùng cho workshop.
5. Xóa AWS Budget `fcaj-rag-chat-monthly` khi không còn theo dõi dự án.

Không xóa topic, Log Group hoặc Budget đang dùng chung với workload khác.

#### 4. Xử lý S3 backup

Kiểm tra chính xác bucket và số phiên bản object. Nếu cần giữ sản phẩm bàn giao, không xóa bucket. Nếu xóa hoàn toàn:

1. Xác nhận tên bucket thuộc workshop.
2. Xóa mọi object version và delete marker vì bucket đã bật Versioning.
3. Xác nhận bucket trống.
4. Xóa bucket.

Thao tác này không thể khôi phục nếu không còn bản sao khác. Không dùng wildcard hoặc tên bucket được suy ra từ biến chưa kiểm tra.

#### 5. Xóa EC2 và EBS

1. Trong EC2 Console, chọn đúng instance `fcaj-rag-chat-demo` và **Terminate instance** nếu nhóm đã chọn xóa hoàn toàn.
2. Chờ trạng thái chuyển sang terminated.
3. Mở **Volumes**, xác định volume theo Volume ID đã ghi và tag `fcaj-rag-chat-data`.
4. Kiểm tra volume ở trạng thái `available` và không gắn với instance khác.
5. Xóa EBS volume 60 GB nếu dữ liệu không cần giữ.
6. Kiểm tra Elastic IP; giải phóng địa chỉ chỉ được cấp riêng cho workshop. Public IPv4 tự động thường được thu hồi khi terminate.

#### 6. Xóa quyền và mạng

Sau khi EC2 đã bị terminate và instance profile không còn được dùng:

1. Gỡ policy khỏi `fcaj-rag-chat-ec2-role`.
2. Xóa role và instance profile dành riêng cho workshop.
3. Xóa customer managed policy sao lưu sau khi xác nhận không có role khác sử dụng.
4. Xóa Security Group `fcaj-rag-chat-sg` khi không còn network interface tham chiếu.

Không xóa VPC, subnet hoặc IAM policy dùng chung.

#### 7. Kiểm tra chi phí sau dọn dẹp

- EC2: không còn instance running hoặc stopped của workshop.
- EBS: không còn volume và snapshot ngoài danh sách cần giữ.
- EC2 networking: không còn Elastic IP hoặc network interface mồ côi.
- S3: bucket đã xóa hoặc có chủ sở hữu và chính sách lưu giữ rõ ràng.
- CloudWatch/SNS: alarm, log, dashboard và topic riêng đã xử lý.
- IAM: role, instance profile và policy riêng đã xóa.
- Budgets: giữ lại nếu nhóm tiếp tục dự án; nếu không, đã xóa.

Billing và Cost Explorer có thể cập nhật trễ. Kiểm tra lại vào ngày hôm sau và tiếp tục theo dõi email ngân sách cho đến khi chắc chắn không còn tài nguyên phát sinh phí.

{{% notice warning %}}
Không xóa dữ liệu hoặc tài nguyên dùng chung chỉ vì tên gần giống. Luôn đối chiếu ID, tag, Region và người sở hữu trước mỗi thao tác xóa.
{{% /notice %}}
