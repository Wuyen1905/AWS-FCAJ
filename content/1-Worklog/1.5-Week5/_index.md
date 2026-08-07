---
title: "Week 5 Worklog"
date: 2024-01-01
weight: 1
chapter: false
pre: " <b> 1.5. </b> "
---

### Week 5 Objectives:

* Continue customizing **fcaj-rag-chat** and standardize its Gemini, Docker, and application persistence configuration.
* Run end-to-end local RAG tests covering PDF upload, indexing, retrieval, citations, and persistence after a container restart.
* Study schema conversion and data migration with AWS SCT/AWS DMS while systematizing AWS security knowledge.
* Complete the ECR–ECS Fargate–EFS deployment blueprint in preparation for implementation from Week 6.

### Tasks to be carried out this week:

| Day | Task | Start date | Completion date | Reference |
| --- | --- | --- | --- | --- |
| Monday | **Standardize the project and container configuration**<br>- Configure Gemini <code>gemini-3.1-flash-lite</code> as the default chat model and <code>models/gemini-embedding-001</code> as the default embedding model.<br>- Review ChromaDB, LanceDB, and SQLite within the persistent data directory and ensure the mount path is consistent between local Docker and the planned EFS environment.<br>- Remove Cohere-dependent reranking to simplify the pipeline and avoid an unnecessary additional API key.<br>- Refine the Dockerfile, Docker Compose, sample environment file, and local run instructions. | 20/07/2026 | 20/07/2026 | [fcaj-rag-chat](https://github.com/ngocchau04/fcaj-rag-chat) |
| Tuesday | **Study and practice Database Schema Conversion & Migration**<br>- Distinguish the roles of AWS Schema Conversion Tool and AWS Database Migration Service in a heterogeneous migration.<br>- Study how to prepare Oracle/SQL Server sources and Amazon RDS/Aurora targets, create a schema conversion project, and handle objects that cannot be converted automatically.<br>- Configure a replication instance, source/target endpoints, and a migration task using full load with Change Data Capture (CDC).<br>- Monitor task status, table statistics, and logs; review DMS Serverless, troubleshooting, and cleanup.<br>- Record this as an independent AWS exercise, not as the RAG project's storage solution. | 21/07/2026 | 21/07/2026 | [Database Schema Conversion & Migration](https://000043.awsstudygroup.com/) |
| Wednesday | **Study Module 05 – AWS Security Services**<br>- 05-01: AWS Shared Responsibility Model.<br>- 05-02: AWS Identity and Access Management (IAM) and least privilege.<br>- 05-03: Amazon Cognito.<br>- 05-04: AWS Organizations.<br>- 05-05: AWS IAM Identity Center.<br>- 05-06: AWS Key Management Service (KMS).<br>- 05-07: AWS Security Hub.<br>- Relate the concepts to the project's task roles, ECR/EFS permissions, and Gemini API key management. | 22/07/2026 | 22/07/2026 | [05-01](https://www.youtube.com/watch?si=-xSAVT8MZReV10RP&v=tsobAlSg19g&feature=youtu.be)<br>[05-02](https://www.youtube.com/watch?v=N_vlJGAqZxo)<br>[05-03](https://www.youtube.com/watch?v=pZ2fgEFK3Vs)<br>[05-04](https://www.youtube.com/watch?v=5oQY8Rogz9Y)<br>[05-05](https://www.youtube.com/watch?v=NW1xrMkNMjU)<br>[05-06](https://www.youtube.com/watch?v=GMihNQojhZc)<br>[05-07](https://www.youtube.com/watch?v=clj2E0rNBEs) |
| Thursday | **Run end-to-end and local persistence tests**<br>- Execute the login → PDF upload → extract/chunk → embed/index → retrieve → answer-with-citations workflow.<br>- Restart and recreate the container to confirm that data in <code>ktem_app_data</code> is retained.<br>- Test basic failure cases: a missing or invalid Gemini API key, an unsupported document, a model error, and insufficient write permission on the data directory.<br>- Collect logs and evidence, compare expected and actual results, and update the RAG testing guide.<br>- Confirm local functionality only; do not claim that the application has already been publicly deployed on AWS. | 23/07/2026 | 23/07/2026 | [fcaj-rag-chat](https://github.com/ngocchau04/fcaj-rag-chat) |
| Friday | **Complete the deployment blueprint and Week 6 plan**<br>- Define the build/tag/push workflow for publishing the Docker image to ECR with buildspec and CodeBuild as a supporting component.<br>- Design the ECS Fargate task: container port 7860, task execution role, task role, environment variables, health check, and CloudWatch Logs.<br>- Design the EFS file system, mount targets, access point, and <code>/app/ktem_app_data</code> mount path for persistent data.<br>- Complete the network flow: users → ALB in public subnets → ECS tasks in private subnets → EFS mount targets; apply least-privilege Security Group rules.<br>- Plan to store the Gemini API key in SSM Parameter Store, assign responsibilities, and prepare the Week 6 deployment checklist. | 24/07/2026 | 24/07/2026 | [fcaj-rag-chat](https://github.com/ngocchau04/fcaj-rag-chat) |

### Week 5 Achievements:

* **Standardized the AI configuration:** Defined the default Gemini chat and embedding models, consistently passed the API key through environment variables, and avoided committing secrets to the repository.
* **Simplified the RAG pipeline:** Removed unnecessary Cohere reranking, reducing the number of credentials to manage and making the configuration easier to reproduce.
* **Completed the container baseline:** Organized the Dockerfile, Docker Compose, volume mount, and local instructions consistently, providing a basis for using the same image with ECR and ECS Fargate.
* **Validated the RAG workflow:** Systematically covered upload, indexing, retrieval, and citation behavior and added negative tests for model, credential, input-file, and data-permission failures.
* **Confirmed persistence requirements:** Identified all data that must survive the container lifecycle and mapped uploaded documents, ChromaDB, LanceDB, SQLite, and settings to <code>ktem_app_data</code>.
* **Understood database migration:** Distinguished schema conversion from data replication and learned replication instances, endpoints, full load, CDC, DMS Serverless, monitoring, troubleshooting, and cleanup without treating DMS as part of the RAG project.
* **Improved security design:** Understood the Shared Responsibility Model, least privilege, and the roles of IAM, Cognito, Organizations, IAM Identity Center, KMS, and Security Hub; applied them to task-role and secret-management decisions.
* **Completed the deployment blueprint:** Documented the users → ALB → ECS Fargate → EFS flow, the image → ECR workflow, and supporting CodeBuild, SSM Parameter Store, IAM, VPC, Security Group, and CloudWatch Logs components.
* **Prepared for implementation:** Produced a technical checklist, test criteria, known risks, and a task breakdown for starting provisioning and image builds in Week 6 without claiming production readiness before AWS testing.

**Overall result:** Week 5 completed the project discovery and preparation phase. The application now has a clear local configuration, a checklist-tested RAG and persistence flow, additional migration and security knowledge, and an AWS architecture detailed enough to begin implementing ECR, ECS Fargate, and EFS during the final weeks.
