---
title: "Week 4 Worklog"
date: 2024-01-01
weight: 1
chapter: false
pre: " <b> 1.4. </b> "
---

### Week 4 Objectives:

* Begin exploring the fcaj-rag-chat project, understand the Kotaemon architecture, and study the processing flow of a RAG application.
* Clone the repository, prepare the Docker environment, and establish a local baseline with persistent data.
* Extend AWS knowledge through storage services and the VM Import/Export workflow.
* Draft the project deployment architecture around three core services: Amazon ECR, Amazon ECS Fargate, and Amazon EFS.

### Tasks to be carried out this week:

| Day | Task | Start date | Completion date | Reference |
| --- | --- | --- | --- | --- |
| Monday | **Review the repository and RAG workflow**<br>- Clone fcaj-rag-chat and study its run instructions.<br>- Inspect the Kotaemon structure, Dockerfile, Docker Compose, buildspec, and model configuration.<br>- Analyze the flow: PDF upload → content extraction → embedding → indexing → retrieval → Gemini-generated answer with citations.<br>- Identify ChromaDB as the vector store, LanceDB as the document store, and SQLite as the store for metadata, users, and application settings. | 13/07/2026 | 13/07/2026 | [fcaj-rag-chat](https://github.com/ngocchau04/fcaj-rag-chat) |
| Tuesday | **Set up and examine the application locally**<br>- Prepare environment variables and pass the Gemini API key without committing the secret to source code.<br>- Start the lite Docker image with Docker Compose and access the application at <code>localhost:7860</code>.<br>- Mount <code>ktem_app_data</code> so uploaded documents, indexes, vector data, and settings persist after a container restart.<br>- Prepare a checklist for login, PDF upload, document indexing, question answering, and citation display. | 14/07/2026 | 14/07/2026 | [fcaj-rag-chat](https://github.com/ngocchau04/fcaj-rag-chat) |
| Wednesday | **Study Module 04 – AWS Storage Services**<br>- 04-01: Amazon S3, Access Points, and Storage Classes.<br>- 04-02: S3 static website hosting, CORS, access control, object keys, performance, and S3 Glacier.<br>- 04-03: AWS Snow Family, Storage Gateway, and AWS Backup.<br>- Compare object storage, hybrid storage, and backup approaches to select an appropriate service for each workload. | 15/07/2026 | 15/07/2026 | [04-01](https://www.youtube.com/watch?v=_yunukwcAwc)<br>[04-02](https://www.youtube.com/watch?si=OITIt1x3d7OZ4ei_&v=mPBjB6Ltl_Q&feature=youtu.be)<br>[04-03](https://www.youtube.com/watch?v=YXn8Q_Hpsu4) |
| Thursday | **Practice the AWS VM Import/Export workflow**<br>- Review the prerequisites: a virtualization environment, AWS CLI, an S3 bucket, and an IAM service role.<br>- Follow the workflow for exporting an on-premises VM, uploading the image to S3, importing it as an AMI, and launching an EC2 instance.<br>- Study the reverse workflow for exporting an EC2 instance or AMI to S3 for on-premises use.<br>- Record format limitations, required permissions, and resource cleanup steps. | 16/07/2026 | 16/07/2026 | [AWS VM Import/Export Workshop](https://000014.awsstudygroup.com/) |
| Friday | **Draft the AWS architecture for the RAG project**<br>- Map the Docker image to Amazon ECR, the container runtime to ECS Fargate, and <code>/app/ktem_app_data</code> to Amazon EFS.<br>- Identify planned supporting components: Application Load Balancer, VPC, public/private subnets, Security Groups, IAM, SSM Parameter Store, and CloudWatch Logs.<br>- Analyze the limitations of sharing ChromaDB, LanceDB, and SQLite over EFS; prefer one ECS task for the initial release.<br>- Review the architecture with the team and prepare the Week 5 configuration and testing plan. | 17/07/2026 | 17/07/2026 | [fcaj-rag-chat](https://github.com/ngocchau04/fcaj-rag-chat) |

### Week 4 Achievements:

* **Understood the project structure:** Identified the role of Kotaemon, the important configuration files, and the relationships among the web interface, RAG pipeline, Gemini models, and persistence layers.
* **Documented the RAG workflow:** Described the complete process from PDF upload, embedding, and indexing to context retrieval and citation-backed answer generation.
* **Established a local baseline:** Standardized the Docker Compose startup procedure, application port, and environment variables; kept the API key outside source code to reduce credential exposure.
* **Clarified persistent storage:** Understood how <code>ktem_app_data</code> preserves ChromaDB, LanceDB, SQLite, uploaded documents, and application settings when a container restarts or is recreated.
* **Strengthened AWS storage knowledge:** Distinguished S3 Storage Classes, Access Points, Glacier, Snow Family, Storage Gateway, and AWS Backup by object storage, hybrid storage, and backup use cases.
* **Understood VM migration:** Learned the VM image → S3 → AMI → EC2 workflow and the reverse export process, including the roles of IAM, image formats, and cleanup.
* **Defined an initial deployment architecture:** Selected ECR–ECS Fargate–EFS as the three core services and separated the necessary supporting services from the project's “three AWS services” scope.
* **Identified technical risks:** Recorded the concurrency limitations of using ChromaDB, LanceDB, and SQLite on shared EFS and avoided planning horizontal multi-task scaling before testing locking and data consistency.

**Overall result:** Week 4 established the technical foundation for moving from isolated AWS exercises to a real project. The team understood the application, prepared a local baseline and test checklist, and drafted the AWS architecture to be refined in Week 5 before cloud deployment.
