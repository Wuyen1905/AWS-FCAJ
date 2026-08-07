---
title : "Prepare EC2, the Security Group, and the IAM Role"
date : 2026-08-06
weight : 1
chapter : false
pre : " <b> 5.3.1 </b> "
---

#### Step 1: Create the S3 backup policy

Create a customer managed IAM policy for the backup prefix. Replace `YOUR_BUCKET_NAME` with the bucket selected in section 5.2:

```json
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Sid": "ReadBucketLocation",
      "Effect": "Allow",
      "Action": "s3:GetBucketLocation",
      "Resource": "arn:aws:s3:::YOUR_BUCKET_NAME"
    },
    {
      "Sid": "ListBackupPrefix",
      "Effect": "Allow",
      "Action": "s3:ListBucket",
      "Resource": "arn:aws:s3:::YOUR_BUCKET_NAME",
      "Condition": {
        "StringLike": {
          "s3:prefix": ["ktem_app_data", "ktem_app_data/*"]
        }
      }
    },
    {
      "Sid": "ReadWriteBackupObjects",
      "Effect": "Allow",
      "Action": ["s3:GetObject", "s3:PutObject"],
      "Resource": "arn:aws:s3:::YOUR_BUCKET_NAME/ktem_app_data/*"
    }
  ]
}
```

This policy cannot delete objects. If retention automation later requires deletion, grant that action to a separate operations role instead of broadening the application's default permissions.

#### Step 2: Create the EC2 IAM Role

1. Open **IAM → Roles → Create role**.
2. Select **AWS service** and the **EC2** use case.
3. Attach the backup policy.
4. Name the role `fcaj-rag-chat-ec2-role` or a similar value.
5. Verify that the trust relationship permits only `ec2.amazonaws.com` to assume the role.

CloudWatch Agent is configured in section 5.4.4. At that point, attach `CloudWatchAgentServerPolicy` or an equivalent least-privilege policy.

#### Step 3: Create the Security Group

Create `fcaj-rag-chat-sg` in the EC2 VPC:

- Inbound TCP 22 from the administrator IP `/32`.
- Inbound TCP 80 from tester addresses or the range approved by the mentor.
- No inbound rule for 7860 because Docker maps host port 80 to container port 7860.
- Outbound HTTPS 443 according to the environment policy for source download, Gemini, and AWS APIs.

After Session Manager is confirmed, the TCP 22 rule may be removed.

#### Step 4: Launch EC2

1. Open **EC2 → Instances → Launch instances**.
2. Name: `fcaj-rag-chat-demo`.
3. AMI: Ubuntu Server 22.04 or 24.04 LTS, `x86_64`.
4. Instance type: `t3.medium`.
5. Select a safely managed key pair, or omit it when Session Manager is available and approved.
6. Select a VPC/subnet with outbound Internet access for the pilot and enable a public IPv4 address.
7. Select the Security Group created above.
8. Allocate enough root storage for the Docker image and source; long-lived data moves to a separate EBS volume in section 5.4.
9. Under **Advanced details → IAM instance profile**, select `fcaj-rag-chat-ec2-role`.
10. Launch and wait for both status checks to pass.

#### Step 5: Connect and verify

Connect from the allowed administrator address:

```bash
ssh -i /path/to/key.pem ubuntu@PUBLIC_IP
```

Verify the platform, storage, and IAM Role:

```bash
uname -m
df -h
aws sts get-caller-identity
```

Expected results:

- `uname -m` returns `x86_64`.
- The root volume has enough free space for the build.
- The STS ARN represents the EC2 assumed role, not an IAM user backed by locally stored keys.

{{% notice warning %}}
Do not run `aws configure` with personal access keys on EC2. If STS fails, check the instance profile, Region, and network connectivity.
{{% /notice %}}
