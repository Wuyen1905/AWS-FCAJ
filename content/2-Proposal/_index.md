---
title: "Proposal"
date: 2026-08-06
weight: 2
chapter: false
pre: " <b> 2. </b> "
---

# FCAJ RAG Chat
## A citation-grounded document question-answering system on AWS

### 1. Executive summary

FCAJ RAG Chat is a document question-answering application based on Retrieval-Augmented Generation (RAG). Users upload documents, the application chunks and indexes their contents, retrieves relevant passages for each question, and asks Gemini to generate an answer with source citations. The solution is developed from Kotaemon and packaged with Docker so that the team can run the same application locally and on AWS.

The project proposes a controlled pilot on Amazon EC2. Application data is persisted on Amazon EBS, backups are stored in a private Amazon S3 bucket, and metrics and logs are monitored with Amazon CloudWatch. AWS Budgets provides cost alerts. The Gemini API remains an external dependency and is called only by the server-side application.

The objective is not to deliver a highly available commercial service in the first phase. The required outcome is a stable and reproducible demonstration that protects secrets, preserves data across restarts, and includes documented backup, restore, monitoring, and cost-control procedures.

### 2. Problem statement

The study group works with many technical documents, reports, and workshop guides. Keyword search can locate files but does not provide a contextual answer. A general-purpose language model may answer without using the team's documents or may provide claims that users cannot verify.

The main challenges are:

- Documents are distributed across files, making information retrieval slow.
- Generated answers may not identify the passages used as evidence.
- Different local environments make defects difficult to reproduce and hand over.
- ChromaDB, LanceDB, and SQLite data can be lost when a container is recreated without persistent storage.
- Gemini keys, AWS credentials, and user data must remain outside the source code.
- AWS resources must be monitored and stopped when idle to avoid unexpected cost.

FCAJ RAG Chat addresses these challenges through one controlled workflow: document ingestion, indexing, contextual retrieval, citation-grounded answer generation, and persistent state management.

### 3. Objectives and success criteria

#### Functional objectives

- Upload supported documents, with PDF and text documents prioritized for the pilot.
- Index content and retrieve passages relevant to a user's question.
- Produce Vietnamese or English answers with citations that users can inspect.
- Create, select, and manage document collections within Kotaemon's supported scope.
- Preserve data when the container or EC2 instance is restarted.

#### Operational objectives

- Deploy the Dockerized application on one EC2 instance.
- Open only required ports and restrict SSH access to administrators.
- Keep API keys out of Git, Docker images, screenshots, and logs.
- Back up application data to S3 and complete at least one restore exercise.
- Monitor EC2 health, storage, application errors, and the agreed budget threshold.

#### Proposed acceptance targets

| Area | Target |
|---|---|
| Functionality | Complete upload, indexing, question-answering, and citation-opening flows |
| Retrieval quality | At least 80% of a 20–30 question evaluation set retrieves the correct supporting passage |
| Verifiability | At least 90% of evaluated answers contain citations that support the main answer |
| Demo performance | Median response time at or below 15 seconds under a small concurrent-user load |
| Durability | Data remains after container restart and can be restored from S3 |
| Security | No secrets in the repository, image, or logs; IAM follows least privilege |

These targets apply to an academic pilot and will be calibrated after the team measures a baseline on the actual document set.

### 4. Project scope

#### In scope

- Customize and configure Kotaemon for the team's RAG workflow.
- Use Gemini for chat and embedding according to the project configuration.
- Package the application with Docker and standardize environment variables.
- Deploy the pilot on EC2 with a 60 GB EBS volume for persistent data.
- Use a private S3 bucket for backups, CloudWatch for monitoring, and AWS Budgets for alerts.
- Test functionality, retrieval, citations, restarts, backup, and restore.
- Produce deployment, operations, and cleanup documentation.

#### Out of scope for the pilot

- A public service for a large user population.
- High-availability, automatic scaling, or multi-Region disaster-recovery commitments.
- Training a new foundation model or processing production-sensitive data.
- Billing, complex enterprise authorization, or formal compliance audits.

After the pilot, the Docker image can move to Amazon ECR, the workload to Amazon ECS Fargate, shared data to Amazon EFS, and traffic through an Application Load Balancer. This is a future evolution rather than a workshop acceptance requirement.

### 5. Solution architecture

#### Main request flow

1. A user opens the FCAJ RAG Chat web interface.
2. The Security Group permits only the configured application and administration traffic.
3. The Kotaemon container on EC2 receives documents and questions.
4. The application chunks content, creates embeddings, and stores indexes and metadata under `/app/ktem_app_data`.
5. This path is mounted from `/opt/fcaj/ktem_app_data` on EBS so that data outlives the container.
6. For each question, the application retrieves relevant context and calls the Gemini API to generate an answer.
7. A backup task copies required data from EBS to a private S3 prefix using the EC2 IAM Role.
8. CloudWatch collects metrics and logs, while CloudWatch Alarm and AWS Budgets notify the administrator.

#### Architecture views

The following diagrams present the same pilot at two levels of detail: a concise view of the currently deployed EC2 solution and a complete AWS view showing network boundaries, storage, observability, cost control, and the external Gemini dependency.

<figure class="proposal-architecture">
  <img src="/images/2-Proposal/current-ec2-deployment.png" alt="Current FCAJ RAG Chat deployment on Amazon EC2 with EBS, S3, CloudWatch, and Gemini API">
  <figcaption><strong>Figure 1.</strong> Current pilot deployment. Users access the Kotaemon application on Amazon EC2; EBS provides persistent application storage, S3 stores private backups, CloudWatch supports monitoring, and Gemini API supplies chat and embedding capabilities.</figcaption>
</figure>

<figure class="proposal-architecture">
  <img src="/images/2-Proposal/aws-deployment-architecture.png" alt="Detailed AWS deployment architecture for FCAJ RAG Chat">
  <figcaption><strong>Figure 2.</strong> Detailed AWS deployment view. The diagram identifies the Region, VPC, public subnet, Security Group, IAM Role, EC2 and EBS resources, S3 backup path, CloudWatch alarm flow, AWS Budgets notification, administrator, and external Gemini API.</figcaption>
</figure>

#### Components and responsibilities

| Component | Responsibility |
|---|---|
| Amazon EC2 | Runs the RAG application container; `t3.medium` is used for the main demo |
| Amazon EBS gp3 60 GB | Persists Kotaemon, ChromaDB, LanceDB, SQLite, and processed-file data |
| Amazon S3 | Stores backups and test evidence in a bucket with public access blocked |
| IAM Role | Allows EC2 to back up and restore only the required S3 prefix without local access keys |
| Security Group | Controls application and SSH traffic and reduces the public attack surface |
| Amazon CloudWatch | Monitors EC2 metrics, application logs, and basic alarms |
| AWS Budgets | Sends alerts as cost approaches the agreed threshold |
| Gemini API | Provides generative and/or embedding models; it is external to AWS |
| Docker | Standardizes the runtime and makes deployment reproducible |

#### Security principles

- Pass the Gemini API key through an environment variable or an appropriate secret store; never hardcode it.
- Enable S3 Block Public Access and server-side encryption, and grant the application role only the required prefix permissions.
- Restrict SSH to an administrative address and prefer AWS Systems Manager Session Manager where available.
- Never log API keys, credentials, complete documents, or unnecessary personal information.
- The academic pilot uses HTTP over a public IPv4 address for a controlled demo. A public release must add a domain, HTTPS, and authentication first.

### 6. Delivery plan

| Phase | Main work | Output |
|---|---|---|
| Week 1 – Analysis | Confirm document types, user flow, a 20–30 question test set, and team ownership | Scope, evaluation set, and acceptance criteria |
| Week 2 – Application baseline | Review the repository, Dockerfile, Compose file, Gemini configuration, volumes, and environment-variable handling | Reproducible local build |
| Week 3 – AWS deployment | Provision EC2, Security Group, IAM Role, and EBS; install Docker and run the application | Accessible AWS demo |
| Week 4 – Data and observability | Configure S3 backup, perform a restore, and add CloudWatch Logs, an alarm, and an AWS Budget | Basic operating procedure |
| Week 5 – Testing | Test upload, indexing, retrieval, citations, latency, restarts, invalid keys, and permission failures | Defect and measurement report |
| Week 6 – Completion | Fix defects, run the demo, and finalize deployment, operations, cost, and cleanup guides | Handover package and final report |

### 7. Cost estimate

The estimate assumes a 60 GB gp3 EBS volume, about 2 GB in S3, about 1 GB of CloudWatch Logs retained for seven days, one basic alarm, and one public IPv4 address while EC2 is running. Actual charges depend on Region, runtime, traffic, and current AWS pricing.

| Scenario | `t3.small` | `t3.medium` |
|---|---:|---:|
| 60 demo hours/month | USD 9.53 | USD 11.36 |
| 120 demo hours/month | USD 11.71 | USD 15.35 |
| Continuous 24/7 operation | USD 33.73 | USD 55.89 |

The figures include the team's 15% contingency. The recommended academic configuration is `t3.medium` for approximately 60 hours per month with a USD 15 monthly budget. EC2 should be stopped when idle; EBS and S3 storage continue to incur charges while EC2 is stopped.

Gemini API usage is monitored separately from AWS charges. Titan Text Embeddings V2 and Cohere Multilingual may be evaluated as AWS-based embedding alternatives, but only after checking current regional availability, quota, and pricing.

### 8. Test and acceptance plan

1. **Functional testing:** upload a valid file, reject an unsupported file, index content, ask a question, open a citation, and remove test data.
2. **RAG evaluation:** run questions with reference answers and record retrieved passages, generated answers, citations, and failure categories.
3. **Durability testing:** restart the container, stop and start EC2, and verify that documents and indexes remain available.
4. **Restore testing:** back up to S3, restore into an empty directory, and rerun known questions.
5. **Failure testing:** use an invalid key, remove data-write permission, upload a damaged document, trigger Gemini HTTP 429 handling, and simulate low disk space.
6. **Operations testing:** verify CloudWatch log delivery, an alarm state transition, and the AWS Budget confirmation email.

### 9. Risks and mitigations

| Risk | Impact | Mitigation |
|---|---|---|
| Gemini API key or AWS credential exposure | High | Keep secrets outside Git and images, use an IAM Role, review logs, and rotate affected keys |
| Data loss or unusable backups | High | Persistent EBS, private S3 backup, backup verification, and restore exercises |
| Gemini HTTP 429 or quota changes | Medium | Limit concurrency, use bounded exponential backoff with jitter, and log clear failure states |
| Ungrounded answers | High | Standardize documents, tune chunking, evaluate retrieval, and require citation review |
| EC2 saturation or slow response | Medium | Monitor CPU, memory, and latency; reduce concurrency or resize for the demo window |
| Unexpected cost | Medium | Budget alerts, an EC2 stop schedule, short log retention, and post-session cleanup checks |
| Public HTTP access | High if expanded | Keep the pilot controlled and add HTTPS, authentication, and an appropriate delivery layer before public use |

### 10. Deliverables and expected value

- Source code and configuration instructions without committed secrets.
- A reproducible Docker image and EC2 deployment procedure.
- An FCAJ RAG Chat demo that answers from documents with citations.
- An evaluation set and a report covering retrieval, citations, and latency.
- Backup, restore, monitoring, cost-control, and resource-cleanup procedures.
- A bilingual workshop that another team member can follow end to end.

The main value of the project is faster document retrieval, more verifiable answers, and practical understanding of how an AI application, persistent data, security, monitoring, and AWS cost control work together.

Team repository: [fcaj-rag-chat](https://github.com/ngocchau04/fcaj-rag-chat).
