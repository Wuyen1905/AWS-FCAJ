---
title : "Restore data and verify durability"
date : 2026-08-06
weight : 3
chapter : false
pre : " <b> 5.4.3 </b> "
---

#### Goal

This section restores an archive into an isolated directory, verifies its checksum, and runs a test container on port 8080. The active dataset is not overwritten, making the process suitable for proving that a backup is usable.

#### Step 1: Select a backup

List archives:

```bash
export BACKUP_BUCKET=YOUR_BUCKET_NAME
aws s3 ls "s3://${BACKUP_BUCKET}/ktem_app_data/archives/" --human-readable
```

Choose a timestamp recorded in section 5.4.2 and set the file name, for example:

```bash
export BACKUP_FILE=ktem_app_data-20260806T080000Z.tar.gz
```

#### Step 2: Download and verify the checksum

```bash
aws s3 cp "s3://${BACKUP_BUCKET}/ktem_app_data/archives/${BACKUP_FILE}" "/tmp/${BACKUP_FILE}"
aws s3 cp "s3://${BACKUP_BUCKET}/ktem_app_data/archives/${BACKUP_FILE%.tar.gz}.sha256" "/tmp/${BACKUP_FILE%.tar.gz}.sha256"
cd /tmp
sha256sum -c "${BACKUP_FILE%.tar.gz}.sha256"
```

Continue only when the result is `OK`. For a mismatch, do not extract the archive; download again and verify that the archive and checksum use the same timestamp.

#### Step 3: Extract into the test area

```bash
sudo mkdir -p /opt/fcaj/restore-test
sudo tar --xattrs --acls -C /opt/fcaj/restore-test -xzf "/tmp/${BACKUP_FILE}"
sudo chown -R ubuntu:ubuntu /opt/fcaj/restore-test
find /opt/fcaj/restore-test/ktem_app_data -maxdepth 2 -type f | head -n 30
du -sh /opt/fcaj/restore-test/ktem_app_data
```

Missing files, unexpected size, or extraction errors must be recorded as restore failures.

#### Step 4: Run an isolated restored environment

Create `/opt/fcaj/app/docker-compose.restore.yml`:

```yaml
services:
  fcaj-rag-chat:
    volumes:
      - /opt/fcaj/restore-test/ktem_app_data:/app/ktem_app_data
```

Start a separate Compose project on port 8080 and reuse the built image:

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

Port 8080 does not need public exposure. Create an SSH tunnel from the local computer:

```bash
ssh -i /path/to/key.pem -L 8080:localhost:8080 ubuntu@PUBLIC_IP
```

Open `http://localhost:8080`, confirm that restored documents exist, and rerun the question recorded before backup. Compare the retrieved passage and citation rather than only checking that the interface opens.

#### Step 5: Stop the restore environment

```bash
cd /opt/fcaj/app
sudo env APP_PORT=8080 docker compose \
  -p fcaj-restore \
  -f docker-compose.yml \
  -f docker-compose.restore.yml \
  down
rm "/tmp/${BACKUP_FILE}" "/tmp/${BACKUP_FILE%.tar.gz}.sha256"
```

Keep `restore-test` until evidence has been recorded. When removing it later, verify that `/opt/fcaj/ktem_app_data` is not targeted.

#### Restore record

| Field | Required record |
|---|---|
| Backup ID | Archive name and UTC time |
| Integrity | SHA-256 result |
| Application version | Commit hash or image ID |
| Data check | Documents, collections, and important-file counts |
| RAG check | Question, retrieved passage, answer, and citation |
| Restore time | Time from download until the application is ready |
| Conclusion | Pass, conditional pass, or fail |

A backup passes only when the checksum is valid, the application reads the data, and the validation question returns appropriate evidence.
