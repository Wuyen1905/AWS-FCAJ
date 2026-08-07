---
title: "Blogs Posted"
date: 2024-01-01
weight: 3
chapter: false
pre: " <b> 3. </b> "
---

This section introduces the articles that the team posted in the [AWS Study Group](https://www.facebook.com/groups/awsstudygroupfcj) community and summarizes the main content and lessons learned while studying AWS.

### [Blog 1 - EC2 Looks Simple, but It Is Not](3.1-Blog1/)
This post shares practical lessons from Amazon EC2 labs: distinguishing Stop from Terminate, reviewing EBS charges, tracking account-specific Free Tier benefits, automating instance shutdown with EventBridge and Lambda, and maintaining a post-lab cleanup checklist.

### [Blog 2 - Handling HTTP 429 When Calling the Gemini API on AWS](3.2-Blog2/)
This post explains why a RAG application can run correctly in local testing but receive HTTP 429 when multiple EC2 workers call the Gemini API concurrently. The team proposes bounded retries with exponential backoff and jitter, concurrency control, idempotent processing, conditional Amazon Bedrock fallback, and CloudWatch monitoring.

### [Blog 3 - Protecting an ECS API Key with Parameter Store](3.3-Blog3/)
This draft explains the mistake of hardcoding a Gemini API key in an ECS task definition and the remediation process using Parameter Store SecureString, AWS KMS, and least-privilege IAM. It also clarifies Task Execution Role versus Task Role, secret rotation, forced deployment, and when AWS Secrets Manager is a better option.
