---
title : "Introduction"
date : 2026-08-06
weight : 1
chapter : false
pre : " <b> 5.1. </b> "
---

#### Workshop goal

This workshop creates a reproducible FCAJ RAG Chat deployment from start to finish. Instead of running only on a local computer, the Docker container is moved to Amazon EC2, its data is separated from the container lifecycle with Amazon EBS, backups are stored in Amazon S3, and monitoring and cost alerts are added.

The completed workshop must demonstrate four flows:

1. A user opens the web interface and uploads a document.
2. The application indexes and retrieves content and returns an answer with citations.
3. Data survives a container restart and can be restored from S3.
4. An administrator can inspect metrics, logs, alarms, and clean up the resources.

#### Lab architecture

| Layer | Component | Responsibility |
|---|---|---|
| Access | Browser, public IPv4, Security Group | Exposes the application port and restricts SSH to the administrator's IP |
| Compute | EC2 `t3.medium` | Runs Docker Engine and the FCAJ RAG Chat container |
| Application | Kotaemon, Gemini, Docker | Ingests documents, creates indexes, retrieves context, and generates answers |
| Data | 60 GB gp3 EBS | Persists `ktem_app_data`, including ChromaDB, LanceDB, and SQLite data |
| Backup | Private S3 bucket | Stores point-in-time copies and restore-test data |
| Authorization | IAM Role | Allows EC2 to use only the required bucket or prefix without static access keys |
| Observability | CloudWatch | Monitors EC2 health, CPU, disk, application logs, and alarms |
| Cost | AWS Budgets | Notifies the team as cost approaches the agreed threshold |
| External dependency | Gemini API | Provides chat and embedding models according to project configuration |

The pilot network flow is: user → EC2 public IPv4 → application port → Kotaemon. EC2 makes outbound calls to Gemini, while operational tasks call S3 and CloudWatch through the instance IAM Role. S3 is never exposed publicly.

#### Security boundary

The workshop uses one EC2 instance and HTTP to keep the internal demonstration simple. It is not a production architecture. Do not upload sensitive documents, publish the URL, or open SSH to `0.0.0.0/0`. A later production design should add HTTPS, authentication, secrets in SSM Parameter Store or Secrets Manager, and an appropriate load-balancing layer.

#### Estimated duration

| Part | Time |
|---|---:|
| Preparation and resource provisioning | 30–45 minutes |
| Docker image build and application launch | 30–60 minutes |
| EBS mount, backup, and restore | 30–45 minutes |
| Monitoring, testing, and cleanup | 30–45 minutes |

Image build time varies with network speed and instance size. Allow extra time for the first dependency download.

#### Evidence to retain

- The running application and one citation-grounded answer.
- `docker compose ps` showing a healthy container.
- EBS mount verification and data retained after restart.
- The S3 backup prefix and a restore-test record.
- CloudWatch metrics, logs/alarm, and AWS Budget confirmation.

{{% notice info %}}
Replace every example Region, bucket name, IP address, and API key with values from your account. Never include a real API key in the report or screenshots.
{{% /notice %}}
