---
title : "Clean up resources"
date : 2026-08-06
weight : 6
chapter : false
pre : " <b> 5.6. </b> "
---

Congratulations on completing the FCAJ RAG Chat workshop. Cleanup is part of the lab because a stopped EC2 instance may still leave EBS, S3, public IPv4, CloudWatch, and related charges.

#### 1. Confirm before deletion

Record:

- EC2 instance ID, EBS volume ID, Security Group ID, and IAM Role.
- S3 bucket and backup prefix.
- Log Group, alarms, SNS topic, and AWS Budget.
- Handover evidence: demo captures, test results, archive, checksum, and restore record.

Agree on one option:

- **Pause:** stop EC2 but retain EBS and backup for a future demo. Storage and public IPv4 charges may continue.
- **Full removal:** proceed only after required data has been downloaded or moved to an approved handover location.

#### 2. Stop the workload

On EC2:

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

Verify that no workshop container is running:

```bash
sudo docker ps --format 'table {{.Names}}\t{{.Status}}\t{{.Ports}}'
```

#### 3. Remove observability resources

In CloudWatch and SNS:

1. Delete `fcaj-rag-chat` alarms after saving required evidence.
2. Delete the dedicated dashboard, if created.
3. Delete `/fcaj/rag-chat/application` if logs do not require retention.
4. Delete workshop-only SNS subscriptions and topics.
5. Delete `fcaj-rag-chat-monthly` if the project no longer needs tracking.

Do not delete shared topics, Log Groups, or Budgets.

#### 4. Handle the S3 backup

Verify the exact bucket and number of object versions. Retain it when handover data is required. For full removal:

1. Confirm that the bucket belongs to this workshop.
2. Remove all object versions and delete markers because Versioning is enabled.
3. Confirm that the bucket is empty.
4. Delete the bucket.

This cannot be recovered without another copy. Never use a wildcard or an unverified variable to select the bucket.

#### 5. Remove EC2 and EBS

1. Select `fcaj-rag-chat-demo` and **Terminate instance** for full removal.
2. Wait for the terminated state.
3. Under **Volumes**, locate the recorded Volume ID tagged `fcaj-rag-chat-data`.
4. Confirm that it is `available` and not attached elsewhere.
5. Delete the 60 GB volume when its data is no longer required.
6. Release only an Elastic IP dedicated to the workshop. An automatically assigned public IPv4 is normally released on termination.

#### 6. Remove permissions and networking

After EC2 termination and when the instance profile is unused:

1. Detach policies from `fcaj-rag-chat-ec2-role`.
2. Delete the dedicated role and instance profile.
3. Delete the backup customer managed policy after confirming no other role uses it.
4. Delete `fcaj-rag-chat-sg` after its network-interface references disappear.

Do not delete a shared VPC, subnet, or IAM policy.

#### 7. Post-cleanup cost check

- EC2: no running or stopped workshop instance.
- EBS: no volume or snapshot outside the retention list.
- EC2 networking: no orphaned Elastic IP or network interface.
- S3: the bucket is deleted or has a named owner and retention policy.
- CloudWatch/SNS: dedicated alarms, logs, dashboards, and topics are handled.
- IAM: dedicated role, instance profile, and policy are deleted.
- Budgets: retained only if the project continues.

Billing and Cost Explorer may update with delay. Check again the next day and monitor budget email until no charge-generating resource remains.

{{% notice warning %}}
Never delete shared data or resources because a name looks similar. Verify ID, tag, Region, and owner before every deletion.
{{% /notice %}}
