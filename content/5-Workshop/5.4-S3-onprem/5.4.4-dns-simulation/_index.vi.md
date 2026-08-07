---
title : "Cấu hình CloudWatch và AWS Budgets"
date : 2026-08-06
weight : 4
chapter : false
pre : " <b> 5.4.4 </b> "
---

#### 1. Chuẩn bị quyền CloudWatch Agent

Gắn `CloudWatchAgentServerPolicy` vào IAM Role của EC2 hoặc tạo policy tối thiểu cho đúng Log Group và metric namespace. Chờ IAM cập nhật rồi kiểm tra lại STS trên instance.

Không cần lưu access key trong cấu hình agent; agent sử dụng IAM Role đã gắn với EC2.

#### 2. Cài CloudWatch Agent

Trên Ubuntu `x86_64`:

```bash
cd /tmp
curl -O https://amazoncloudwatch-agent.s3.amazonaws.com/ubuntu/amd64/latest/amazon-cloudwatch-agent.deb
sudo dpkg -i amazon-cloudwatch-agent.deb
/opt/aws/amazon-cloudwatch-agent/bin/amazon-cloudwatch-agent-ctl -a status
```

#### 3. Cấu hình metric và log

Tạo tệp `/opt/aws/amazon-cloudwatch-agent/etc/fcaj-rag-chat.json` với nội dung:

```json
{
  "agent": {
    "metrics_collection_interval": 60,
    "run_as_user": "root"
  },
  "metrics": {
    "namespace": "FCAJ/RAGChat",
    "append_dimensions": {
      "InstanceId": "${aws:InstanceId}"
    },
    "metrics_collected": {
      "mem": {
        "measurement": ["mem_used_percent"]
      },
      "disk": {
        "measurement": ["used_percent"],
        "resources": ["/opt/fcaj/ktem_app_data"]
      }
    }
  },
  "logs": {
    "logs_collected": {
      "files": {
        "collect_list": [
          {
            "file_path": "/var/lib/docker/containers/*/*.log",
            "log_group_name": "/fcaj/rag-chat/application",
            "log_stream_name": "{instance_id}",
            "retention_in_days": 7
          }
        ]
      }
    }
  }
}
```

Khởi động agent bằng cấu hình này:

```bash
sudo /opt/aws/amazon-cloudwatch-agent/bin/amazon-cloudwatch-agent-ctl \
  -a fetch-config \
  -m ec2 \
  -c file:/opt/aws/amazon-cloudwatch-agent/etc/fcaj-rag-chat.json \
  -s
sudo /opt/aws/amazon-cloudwatch-agent/bin/amazon-cloudwatch-agent-ctl -a status
```

Mở CloudWatch và xác nhận namespace `FCAJ/RAGChat`, metric bộ nhớ, metric dung lượng EBS cùng Log Group `/fcaj/rag-chat/application` đã xuất hiện.

{{% notice warning %}}
Docker log có thể chứa câu hỏi, tên tệp hoặc lỗi từ dịch vụ ngoài. Kiểm tra ứng dụng không ghi API key, credential hay toàn bộ nội dung tài liệu trước khi chuyển log lên CloudWatch.
{{% /notice %}}

#### 4. Tạo alarm

Tạo ít nhất các cảnh báo sau:

| Alarm | Điều kiện gợi ý | Mục đích |
|---|---|---|
| CPU cao | `CPUUtilization >= 80%` trong 3 khoảng 5 phút | Phát hiện instance quá tải kéo dài |
| EC2 status check | `StatusCheckFailed >= 1` trong 2 khoảng 1 phút | Phát hiện lỗi hệ thống hoặc instance |
| EBS gần đầy | `disk_used_percent >= 80%` trong 2 khoảng 5 phút | Tránh lỗi indexing do hết dung lượng |

Ngưỡng cần được điều chỉnh sau khi có đường cơ sở. Không đặt quá nhạy khiến nhóm bỏ qua cảnh báo do nhận quá nhiều email.

Nếu tài khoản cho phép, tạo Amazon SNS topic và xác nhận email subscription để Alarm gửi thông báo. Chuyển một alarm sang trạng thái thử, ghi nhận email, sau đó trả alarm về cấu hình bình thường.

#### 5. Tạo AWS Budget

1. Mở **Billing and Cost Management → Budgets → Create budget**.
2. Chọn **Cost budget** và chu kỳ **Monthly**.
3. Đặt tên `fcaj-rag-chat-monthly`.
4. Nhập giới hạn `15 USD` cho kịch bản demo khoảng 60 giờ/tháng.
5. Tạo cảnh báo ở 80% actual, 100% actual và 100% forecasted.
6. Nhập email nhóm hoặc người chịu trách nhiệm và hoàn tất xác nhận.

Budget không tự dừng EC2. Nhóm vẫn phải dừng instance khi không dùng và kiểm tra EBS, public IPv4, S3, log cùng alarm còn phát sinh chi phí hay không.

#### 6. Checklist quan sát

- [ ] EC2 Metrics hiển thị `CPUUtilization` và `StatusCheckFailed`.
- [ ] Namespace `FCAJ/RAGChat` có metric bộ nhớ và dung lượng.
- [ ] Log Group nhận log mới, retention là 7 ngày.
- [ ] Alarm có đúng metric, period, threshold và người nhận.
- [ ] Email SNS, nếu dùng, đã xác nhận.
- [ ] AWS Budget có giới hạn 15 USD và email đã xác nhận.
- [ ] Không có secret hoặc tài liệu nhạy cảm trong log.
