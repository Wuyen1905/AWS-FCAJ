---
title : "Preparation"
date : 2026-08-06
weight : 2
chapter : false
pre : " <b> 5.2. </b> "
---

#### 1. Prepare the AWS account and permissions

Use an AWS learning or sandbox account. The participant needs permission to create and manage the following workshop resources:

- EC2 instances, EBS volumes, Security Groups, and IAM Roles/Instance Profiles.
- S3 buckets, objects, and bucket policies.
- CloudWatch Log Groups, metrics, dashboards, and alarms.
- AWS Budgets and email notifications.

Do not use the root user for daily work. For an organization-managed account, confirm quotas, allowed Regions, and cleanup policies before starting.

Examples use `ap-southeast-1`. Another Region is acceptable, but EC2, EBS, and related resources must be created consistently. Keeping S3 in the same Region reduces complexity.

After connecting to EC2, verify the AWS CLI identity:

```bash
aws sts get-caller-identity
aws configure get region
```

If the first command returns the EC2 IAM Role, do not configure static access keys on the server.

#### 2. Record deployment parameters

Choose these values before provisioning resources:

| Variable | Suggested value | Notes |
|---|---|---|
| `AWS_REGION` | `ap-southeast-1` | Workshop Region |
| `INSTANCE_NAME` | `fcaj-rag-chat-demo` | EC2 name |
| `INSTANCE_TYPE` | `t3.medium` | `t3.small` may be used for light validation |
| `APP_PORT` | `80` | Maps the host port to Gradio port 7860 |
| `DATA_DIR` | `/opt/fcaj/ktem_app_data` | Persistent-data mount point |
| `BACKUP_BUCKET` | `fcaj-rag-chat-backup-<account-id>` | The bucket name must be globally unique |
| `BACKUP_PREFIX` | `ktem_app_data/` | Backup scope in the bucket |
| `BUDGET_LIMIT` | `USD 15/month` | Target for the demo scenario |

Replace `<account-id>` with a unique lowercase suffix without spaces.

#### 3. Prepare the Gemini API key

Create a dedicated Gemini key for the workshop and review its current quota. The key is stored in the EC2 `.env` file as:

```dotenv
GOOGLE_API_KEY=your-key-value
APP_PORT=80
```

Security requirements:

- Never commit `.env` to Git.
- Never place the key in the Dockerfile or a `docker build --build-arg` command.
- Do not capture the key in screenshots.
- Restrict the file with `chmod 600 .env`.
- Rotate or revoke the key immediately if exposure is suspected.

#### 4. Prepare the source code

Team repository: [ngocchau04/fcaj-rag-chat](https://github.com/ngocchau04/fcaj-rag-chat).

Review these files locally:

- `Dockerfile`: a multi-stage image; the workshop uses the `lite` target.
- `docker-compose.yml`: container port 7860, a health check, and `/app/ktem_app_data` volume.
- `flowsettings.py`: Gemini chat, embedding, and data-directory configuration.
- `.env.example`: environment template; add `GOOGLE_API_KEY` for the team's configuration.

Validate the source:

```bash
git clone https://github.com/ngocchau04/fcaj-rag-chat.git
cd fcaj-rag-chat
git status
docker compose config
```

`docker compose config` may render environment values. Never include a real key in report screenshots.

#### 5. Security Group rules

Use least-privilege rules:

| Direction | Protocol/port | Source | Purpose |
|---|---|---|---|
| Inbound | TCP 22 | Administrator IP `/32` | SSH; omit when using Session Manager |
| Inbound | TCP 80 | Tester IP or approved range | Demo application access |
| Outbound | HTTPS 443 | Per environment policy | GitHub, Gemini API, S3, and AWS services |

Do not expose port 7860 when port 80 is mapped to it. Never open SSH to the entire Internet.

#### 6. Select EC2 and EBS

- Operating system: Ubuntu Server 22.04/24.04 LTS or Amazon Linux 2023; commands use Ubuntu.
- Architecture: `x86_64` for the `linux/amd64` target.
- Instance type: `t3.medium` for build and demo; stop it when idle.
- Root volume: enough capacity for the OS, source tree, and Docker layers.
- Data volume: 60 GB gp3 EBS in the same Availability Zone as EC2.
- Enable termination protection if the learning account supports it and accidental deletion is a concern.

#### 7. Pre-workshop checklist

- [ ] AWS account and Region confirmed.
- [ ] Budget-notification email is accessible.
- [ ] Gemini API key works and is not committed.
- [ ] Current administrator IP identified.
- [ ] Globally unique bucket name selected.
- [ ] Repository can be cloned and `docker compose config` is valid.
- [ ] The team has assigned resource provisioning and cleanup owners.

{{% notice warning %}}
EC2, EBS, public IPv4, S3, and CloudWatch may incur charges. Stop EC2 during breaks and complete section 5.6 after the lab.
{{% /notice %}}
