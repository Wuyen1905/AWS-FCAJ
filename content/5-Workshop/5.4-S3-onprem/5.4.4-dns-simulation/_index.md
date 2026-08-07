---
title : "Configure CloudWatch and AWS Budgets"
date : 2026-08-06
weight : 4
chapter : false
pre : " <b> 5.4.4 </b> "
---

#### 1. Prepare CloudWatch Agent permissions

Attach `CloudWatchAgentServerPolicy` to the EC2 IAM Role or create a least-privilege policy for the required Log Group and metric namespace. Allow IAM changes to propagate and verify STS again on the instance.

The agent uses the EC2 IAM Role. No access key is stored in its configuration.

#### 2. Install CloudWatch Agent

On Ubuntu `x86_64`:

```bash
cd /tmp
curl -O https://amazoncloudwatch-agent.s3.amazonaws.com/ubuntu/amd64/latest/amazon-cloudwatch-agent.deb
sudo dpkg -i amazon-cloudwatch-agent.deb
/opt/aws/amazon-cloudwatch-agent/bin/amazon-cloudwatch-agent-ctl -a status
```

#### 3. Configure metrics and logs

Create `/opt/aws/amazon-cloudwatch-agent/etc/fcaj-rag-chat.json`:

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

Start the agent with this configuration:

```bash
sudo /opt/aws/amazon-cloudwatch-agent/bin/amazon-cloudwatch-agent-ctl \
  -a fetch-config \
  -m ec2 \
  -c file:/opt/aws/amazon-cloudwatch-agent/etc/fcaj-rag-chat.json \
  -s
sudo /opt/aws/amazon-cloudwatch-agent/bin/amazon-cloudwatch-agent-ctl -a status
```

In CloudWatch, confirm the `FCAJ/RAGChat` namespace, memory metric, EBS disk metric, and `/fcaj/rag-chat/application` Log Group.

{{% notice warning %}}
Docker logs may contain questions, file names, or external-service errors. Confirm that the application does not log API keys, credentials, or full document contents before shipping logs to CloudWatch.
{{% /notice %}}

#### 4. Create alarms

Create at least these alarms:

| Alarm | Suggested condition | Purpose |
|---|---|---|
| High CPU | `CPUUtilization >= 80%` for three 5-minute periods | Detect sustained instance saturation |
| EC2 status check | `StatusCheckFailed >= 1` for two 1-minute periods | Detect system or instance failure |
| EBS nearly full | `disk_used_percent >= 80%` for two 5-minute periods | Prevent indexing failure caused by low disk space |

Tune thresholds after collecting a baseline. Avoid noisy alarms that the team will ignore.

If permitted, create an Amazon SNS topic, confirm the email subscription, test one alarm notification, and return the alarm to its normal configuration.

#### 5. Create the AWS Budget

1. Open **Billing and Cost Management → Budgets → Create budget**.
2. Select a monthly **Cost budget**.
3. Name it `fcaj-rag-chat-monthly`.
4. Set a `USD 15` limit for the approximate 60-hour demo scenario.
5. Add notifications at 80% actual, 100% actual, and 100% forecasted.
6. Enter the team or owner email and confirm the subscription.

A Budget does not stop EC2. The team must still stop idle instances and review ongoing EBS, public IPv4, S3, log, and alarm charges.

#### 6. Observability checklist

- [ ] EC2 Metrics shows `CPUUtilization` and `StatusCheckFailed`.
- [ ] `FCAJ/RAGChat` contains memory and disk metrics.
- [ ] The Log Group receives new logs and retains them for seven days.
- [ ] Each alarm has the correct metric, period, threshold, and recipient.
- [ ] SNS email, if used, is confirmed.
- [ ] The USD 15 AWS Budget and email are confirmed.
- [ ] Logs contain no secrets or sensitive document content.
