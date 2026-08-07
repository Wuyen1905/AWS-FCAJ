---
title : "Prepare and mount EBS"
date : 2026-08-06
weight : 1
chapter : false
pre : " <b> 5.4.1 </b> "
---

#### Step 1: Create the EBS volume

1. In the EC2 Console, open **Elastic Block Store → Volumes → Create volume**.
2. Volume type: `gp3`; size: `60 GiB`.
3. Select the Availability Zone used by `fcaj-rag-chat-demo`.
4. Enable encryption with an AWS managed key or the KMS key required by the account.
5. Add `Name=fcaj-rag-chat-data`.
6. Create the volume, select **Actions → Attach volume**, and attach it to EC2.

The Console device name may be `/dev/sdf`, while a Nitro instance commonly exposes it as `/dev/nvme1n1`. Always use `lsblk` to identify the real device.

#### Step 2: Identify the volume and filesystem

```bash
lsblk -f
sudo file -s /dev/nvme1n1
```

Create a filesystem only when this is a new volume and `file -s` confirms that no filesystem exists:

```bash
sudo mkfs.ext4 /dev/nvme1n1
```

{{% notice warning %}}
`mkfs` makes existing volume data inaccessible. Never run it until the device and its empty state are confirmed.
{{% /notice %}}

#### Step 3: Mount the volume

```bash
sudo mkdir -p /opt/fcaj/ktem_app_data
sudo mount /dev/nvme1n1 /opt/fcaj/ktem_app_data
sudo chown -R ubuntu:ubuntu /opt/fcaj/ktem_app_data
df -h /opt/fcaj/ktem_app_data
```

Get the UUID:

```bash
sudo blkid /dev/nvme1n1
```

Open `/etc/fstab` with `sudo nano /etc/fstab` and add this line with the real UUID:

```text
UUID=YOUR_VOLUME_UUID /opt/fcaj/ktem_app_data ext4 defaults,nofail 0 2
```

Validate before rebooting:

```bash
sudo mount -a
findmnt /opt/fcaj/ktem_app_data
```

#### Step 4: Move test data to EBS

Stop the application so no process writes during the copy:

```bash
cd /opt/fcaj/app
sudo docker compose down
sudo rsync -aHAX --numeric-ids /opt/fcaj/app/ktem_app_data/ /opt/fcaj/ktem_app_data/
```

An empty source directory is acceptable. `.env` is not copied because it is stored in the application directory, not under `ktem_app_data`.

#### Step 5: Override the Compose volume

Create `/opt/fcaj/app/docker-compose.override.yml`:

```yaml
services:
  fcaj-rag-chat:
    volumes:
      - /opt/fcaj/ktem_app_data:/app/ktem_app_data
```

Validate the merged configuration:

```bash
cd /opt/fcaj/app
sudo docker compose config
sudo docker compose up -d
sudo docker compose ps
```

Verify the mount inside the container:

```bash
sudo docker compose exec fcaj-rag-chat sh -c 'mount | grep /app/ktem_app_data || true; ls -la /app/ktem_app_data | head'
```

#### Step 6: Test durability

1. Upload a small test document and index it.
2. Record its name and a known question.
3. Restart the container:

```bash
sudo docker compose restart fcaj-rag-chat
sudo docker compose ps
```

4. Reopen the application and verify that the document and test session remain.
5. If the workshop policy permits, stop and start EC2, then rerun `findmnt`, `docker compose ps`, and the data check.

This result demonstrates that data is independent of the container lifecycle and that EBS mounts correctly after startup.
