---
title: "Week 6 Worklog"
date: 2024-01-01
weight: 1
chapter: false
pre: " <b> 1.6. </b> "
---

### Week 6 objectives

- Turn the Week 5 blueprint into configurations that can be reviewed and tested for FCAJ RAG Chat.
- Complete the Docker image workflow and prepare image storage in Amazon ECR.
- Clarify how ECS Fargate uses EFS, IAM Roles, SSM Parameter Store, and CloudWatch Logs.
- Evaluate whether the ECS Fargate, EFS, and ALB design fits the remaining time, budget, and project scope.
- Select a pilot architecture that can be deployed, tested, and handed over during the internship.

### Work completed during the week

| Day | Work | Start date | Completion date | Reference |
|---|---|---|---|---|
| Monday | Review the project and standardize the build<br><br>Rechecked the multi-stage Dockerfile, the `lite` target, Docker Compose, health check, and port 7860. Compared dependencies in `pyproject.toml`, `uv.lock`, and the Gemini configuration so that the image can be reproduced. Reviewed `.gitignore`, `.env.example`, and `GOOGLE_API_KEY` handling to keep the real key out of source code and the image. Recorded the commit, build command, image size, and remaining errors. | 27/07/2026 | 27/07/2026 | [fcaj-rag-chat](https://github.com/ngocchau04/fcaj-rag-chat) |
| Tuesday | Complete the image build and distribution workflow<br><br>Standardized `buildspec.yml` for Amazon ECR login, build, tag, and push steps. Distinguished the CodeBuild service role from the ECS task execution role and identified the minimum ECR permissions for each role. Defined repository names, image tags, and commit-hash tagging for traceability. Kept the Gemini API key out of buildspec and build-time variables. | 28/07/2026 | 28/07/2026 | [Docker sample for CodeBuild](https://docs.aws.amazon.com/codebuild/latest/userguide/sample-docker.html)<br>[Amazon ECR getting started](https://docs.aws.amazon.com/AmazonECR/latest/userguide/getting-started-cli.html) |
| Wednesday | Analyze persistent storage with Amazon EFS<br><br>Identified the data under `ktem_app_data` that must survive container replacement, including uploaded documents, ChromaDB, LanceDB, SQLite, and user settings. Designed EFS mount targets, an access point, a POSIX identity, and the `/app/ktem_app_data` path. Reviewed the TCP 2049 Security Group flow from ECS tasks to EFS. Recorded file-locking and consistency risks when multiple tasks share SQLite or vector stores and limited the pilot design to one task. | 29/07/2026 | 29/07/2026 | [Amazon EFS volumes with ECS](https://docs.aws.amazon.com/AmazonECS/latest/developerguide/efs-volumes.html)<br>[Amazon EFS access points](https://docs.aws.amazon.com/efs/latest/ug/efs-access-points.html) |
| Thursday | Complete the ECS Fargate and secret-management design<br><br>Outlined a task definition with container port 7860, CPU, memory, an EFS volume, health check, and the `awslogs` log driver. Distinguished the task role from the task execution role. Prepared a reference to the Gemini API key in SSM Parameter Store instead of placing the value in the task definition. Reviewed the network flow from a public-subnet ALB to private-subnet tasks and limited Security Group rules to the required sources and ports. | 30/07/2026 | 30/07/2026 | [ECS task definitions](https://docs.aws.amazon.com/AmazonECS/latest/developerguide/task_definitions.html)<br>[SSM Parameter Store secrets in ECS](https://docs.aws.amazon.com/AmazonECS/latest/developerguide/secrets-envvar-ssm-paramstore.html)<br>[CloudWatch Logs for ECS](https://docs.aws.amazon.com/AmazonECS/latest/developerguide/using_awslogs.html) |
| Friday | Review the architecture and adjust the pilot scope<br><br>Compared ALB, ECS Fargate, and EFS with an EC2 and EBS design. ECS provides stronger container orchestration but requires more networking, permissions, storage configuration, and baseline cost. For an internal demo with one operator and limited internship time, the team selected EC2 with Docker, EBS for active data, S3 for backup, CloudWatch for monitoring, and AWS Budgets for cost alerts. ECR, ECS Fargate, and EFS remain the future scaling path. | 31/07/2026 | 31/07/2026 | [Amazon EC2](https://docs.aws.amazon.com/ec2/)<br>[Amazon EBS](https://docs.aws.amazon.com/ebs/)<br>[AWS Well-Architected Framework](https://docs.aws.amazon.com/wellarchitected/latest/framework/welcome.html) |

### Week 6 outcomes

- The Docker configuration was reviewed for reproducibility with the `lite` target, port 7860, and a health check consistent with the source.
- The ECR workflow now covers build, tag, and push steps without placing the Gemini key in buildspec.
- The CodeBuild service role, ECS task execution role, and task role have distinct responsibilities, avoiding one overly broad role.
- ChromaDB, LanceDB, and SQLite persistence requirements were mapped to `ktem_app_data`, and concurrent-write risks were documented.
- The EFS access-point and TCP 2049 Security Group design was completed at design level, without claiming readiness for multiple ECS tasks.
- The team defined how SSM Parameter Store supplies the Gemini key and how container logs reach CloudWatch Logs.
- Architecture choices were compared by complexity, operability, and cost rather than following the first design without review.
- EC2, EBS, S3, CloudWatch, and AWS Budgets were selected for the academic pilot, while ECR, ECS Fargate, and EFS became the scaling roadmap.

### Issues and responses

| Issue | Analysis | Response |
|---|---|---|
| Slow and large image build | Kotaemon dependencies, PDF.js, and document-processing libraries increase image size | Use the `lite` target, lock dependencies, use controlled caching, and record the commit hash |
| Data is not ready for immediate horizontal scaling | SQLite and some index stores require file-locking and consistency validation | Use one container in the pilot and scale only after concurrency tests |
| ECS architecture is large for the remaining schedule | It requires ALB, subnets, roles, EFS, logging, and task troubleshooting | Use EC2 and EBS now and retain the ECS design for a later phase |
| API-key exposure | A key may leak through Git, buildspec, task definitions, or logs | Keep it outside source code and use a restricted environment file or secret store |

> Overall result: Week 6 turned an ECS-oriented blueprint into an evidence-based architecture decision. The team retained and improved its ECR, ECS Fargate, and EFS knowledge, but selected EC2 and EBS for the pilot so deployment, testing, backup, and handover could be completed during the internship.
