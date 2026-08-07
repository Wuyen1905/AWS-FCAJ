---
title : "Persistent storage, backup, and monitoring"
date : 2026-08-06
weight : 4
chapter : false
pre : " <b> 5.4. </b> "
---

#### Overview

Containers are recreated during updates and troubleshooting, so RAG data must not depend on a container's writable layer. This section places `ktem_app_data` on EBS, creates a consistent backup in S3, exercises the restore procedure, and adds system observability.

#### Data principles

- EBS stores the active data used by one EC2 instance.
- S3 stores backup copies; do not mount S3 as a direct filesystem replacement for ChromaDB, LanceDB, or SQLite.
- Stop the application while creating a backup to reduce database-file inconsistency risk.
- A backup is valid only after checksum verification and a restore exercise.
- Never include `.env`, API keys, SSH keys, or AWS credentials in the backup archive.

#### Contents

1. [Prepare and mount EBS](5.4.1-prepare/)
2. [Create a private S3 bucket and back up data](5.4.2-create-interface-enpoint/)
3. [Restore data and verify durability](5.4.3-test-endpoint/)
4. [Configure CloudWatch and AWS Budgets](5.4.4-dns-simulation/)

#### Required result

- The container uses `/opt/fcaj/ktem_app_data:/app/ktem_app_data`.
- EBS mounts again after an EC2 restart.
- S3 blocks public access, uses encryption, and contains an archive and checksum.
- Restored data can answer at least one known question.
- CloudWatch displays required metrics/logs and the AWS Budget email is confirmed.
