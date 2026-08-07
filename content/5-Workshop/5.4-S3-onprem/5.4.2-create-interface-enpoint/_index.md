---
title : "Create a private S3 bucket and back up data"
date : 2026-08-06
weight : 2
chapter : false
pre : " <b> 5.4.2 </b> "
---

#### Step 1: Create the bucket

1. Open **Amazon S3 → Create bucket**.
2. Enter a globally unique name such as `fcaj-rag-chat-backup-<account-id>`.
3. Select the agreed Region.
4. Keep **Block all public access** enabled.
5. Enable **Bucket Versioning** to reduce accidental overwrite risk.
6. Enable default SSE-S3 or AWS KMS encryption according to account policy.
7. Create the bucket and confirm that Permissions reports no public access.

A lifecycle rule may archive or expire old backups after an approved retention period. Do not use a short expiration before the report and restore test are complete.

#### Step 2: Verify access from EC2

Replace the bucket name:

```bash
export BACKUP_BUCKET=YOUR_BUCKET_NAME
aws s3api get-bucket-location --bucket "$BACKUP_BUCKET"
aws s3 ls "s3://$BACKUP_BUCKET/ktem_app_data/"
```

For `AccessDenied`, check the IAM Role, bucket ARN, prefix condition, and bucket policy. Do not solve it by attaching `AmazonS3FullAccess` to the application role.

#### Step 3: Create a consistent backup

Pause the application so SQLite and index stores are not writing:

```bash
cd /opt/fcaj/app
sudo docker compose stop fcaj-rag-chat
BACKUP_TIME=$(date -u +%Y%m%dT%H%M%SZ)
sudo tar --xattrs --acls -C /opt/fcaj -czf "/tmp/ktem_app_data-${BACKUP_TIME}.tar.gz" ktem_app_data
sudo chown ubuntu:ubuntu "/tmp/ktem_app_data-${BACKUP_TIME}.tar.gz"
sha256sum "/tmp/ktem_app_data-${BACKUP_TIME}.tar.gz" > "/tmp/ktem_app_data-${BACKUP_TIME}.sha256"
```

Upload the archive and checksum:

```bash
aws s3 cp "/tmp/ktem_app_data-${BACKUP_TIME}.tar.gz" "s3://${BACKUP_BUCKET}/ktem_app_data/archives/"
aws s3 cp "/tmp/ktem_app_data-${BACKUP_TIME}.sha256" "s3://${BACKUP_BUCKET}/ktem_app_data/archives/"
sudo docker compose start fcaj-rag-chat
```

The archive contains only application data; `/opt/fcaj/app/.env` is excluded.

#### Step 4: Verify the copy

```bash
aws s3 ls "s3://${BACKUP_BUCKET}/ktem_app_data/archives/" --recursive --human-readable
aws s3api head-object \
  --bucket "$BACKUP_BUCKET" \
  --key "ktem_app_data/archives/ktem_app_data-${BACKUP_TIME}.tar.gz"
```

Record:

- Backup UTC time.
- Application commit hash.
- Archive size and SHA-256.
- Operator and the section 5.4.3 restore result.

After successful upload and evidence capture, remove temporary root-volume files:

```bash
rm "/tmp/ktem_app_data-${BACKUP_TIME}.tar.gz" "/tmp/ktem_app_data-${BACKUP_TIME}.sha256"
```

{{% notice info %}}
Versioning retains object versions but does not replace restore testing. Complete section 5.4.3 before treating the backup as valid.
{{% /notice %}}
