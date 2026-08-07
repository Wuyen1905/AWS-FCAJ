---
title: "Week 7 Worklog"
date: 2024-01-01
weight: 1
chapter: false
pre: " <b> 1.7. </b> "
---

### Week 7 objectives

- Finalize the FCAJ RAG Chat pilot scope and architecture based on the Week 6 decision.
- Build cost scenarios for the demonstration and select a suitable budget threshold.
- Replace the unrelated proposal template with content based on the team project.
- Create a bilingual workshop covering deployment, testing, backup, restore, and cleanup.
- Check consistency among the Worklog, Proposal, Workshop, self-evaluation, and project description.

### Work completed during the week

| Day | Work | Start date | Completion date | Reference |
|---|---|---|---|---|
| Monday | Finalize pilot requirements and architecture<br><br>Consolidated source-code requirements, local test results, and the Week 6 design. Defined the flow from users to the EC2 public IPv4 address, Docker host port 80 mapped to Kotaemon port 7860, a 60 GB gp3 EBS volume for `ktem_app_data`, S3 for backups, an EC2 IAM Role, CloudWatch metrics and logs, AWS Budgets alerts, and outbound Gemini API calls. Recorded that this is an internal pilot rather than a highly available architecture. | 03/08/2026 | 03/08/2026 | [fcaj-rag-chat](https://github.com/ngocchau04/fcaj-rag-chat)<br>[Amazon EC2 documentation](https://docs.aws.amazon.com/ec2/) |
| Tuesday | Build the cost and operating model<br><br>Estimated `t3.small` and `t3.medium` under 60-hour, 120-hour, and continuous monthly scenarios. Added 60 GB gp3 EBS, 2 GB S3, a public IPv4 address, 1 GB of CloudWatch Logs retained for seven days, one alarm, and a 15% contingency. Compared reference embedding costs for Titan Text Embeddings V2, Cohere Multilingual, and Gemini. Selected `t3.medium` for approximately 60 hours per month with a USD 15 target budget and an idle-instance stop rule. | 04/08/2026 | 04/08/2026 | [EC2 On-Demand pricing](https://aws.amazon.com/ec2/pricing/on-demand/)<br>[Amazon EBS pricing](https://aws.amazon.com/ebs/pricing/)<br>[AWS Pricing Calculator](https://calculator.aws/) |
| Wednesday | Write the Vietnamese and English project proposal<br><br>Replaced the IoT Weather Platform template with FCAJ RAG Chat. Completed the context, problem, objectives, scope, architecture flow, component responsibilities, six-week plan, cost, acceptance criteria, risks, and deliverables. Proposed a 20–30 question evaluation set, retrieval accuracy of at least 80%, citation support of at least 90%, and median response time of 15 seconds or less during the demo. Clearly separated the current EC2 pilot from the later ECR, ECS Fargate, and EFS roadmap. | 05/08/2026 | 05/08/2026 | [AWS Well-Architected Framework](https://docs.aws.amazon.com/wellarchitected/latest/framework/welcome.html)<br>[fcaj-rag-chat](https://github.com/ngocchau04/fcaj-rag-chat) |
| Thursday | Build Workshop sections 5.1 through 5.6<br><br>Replaced the old VPC Endpoint workshop with the FCAJ RAG Chat deployment process. Covered account and parameter preparation, Security Group and IAM Role creation, Docker installation, image build, Compose startup, EBS mounting, private S3 backup, checksum validation, isolated restore, CloudWatch Agent, alarms, AWS Budgets, RAG testing, security checks, cost reconciliation, and cleanup. Verified commands against the repository Dockerfile, `docker-compose.yml`, `flowsettings.py`, and `.env.example`. | 06/08/2026 | 06/08/2026 | [fcaj-rag-chat](https://github.com/ngocchau04/fcaj-rag-chat)<br>[Amazon CloudWatch Agent](https://docs.aws.amazon.com/AmazonCloudWatch/latest/monitoring/Install-CloudWatch-Agent.html)<br>[AWS Budgets](https://docs.aws.amazon.com/cost-management/latest/userguide/budgets-managing-costs.html) |
| Friday | Complete the Worklog and verify the report website<br><br>Completed Weeks 6 and 7 and prepared an honest Week 8 plan. Updated the bilingual Worklog index and reduced unnecessary English wording in Vietnamese content. Built Hugo into a temporary directory and checked target URLs, menus, tables, code blocks, and localhost rendering. Reviewed links among the Proposal, Workshop, and Worklog while keeping unrelated report sections unchanged. | 07/08/2026 | 07/08/2026 | [Hugo documentation](https://gohugo.io/documentation/)<br>[AWS Workshop sample repository](https://github.com/aws-samples/aws-modernization-workshop-sample) |

### Week 7 outcomes

- The pilot architecture now uses EC2, EBS, S3, an IAM Role, a Security Group, CloudWatch, AWS Budgets, and the external Gemini API.
- The scope is explicit: an internal learning and demo environment without high-availability, multi-Region, or automatic-scaling commitments.
- The cost model contains three operating scenarios, two EC2 types, and a 15% contingency; the recommended option fits a USD 15 monthly limit.
- The Project Proposal now describes FCAJ RAG Chat rather than the original template and includes measurable goals, architecture, planning, risks, and value.
- Workshop 5.1–5.6 forms one continuous workflow from preparation and deployment to storage, backup, restore, monitoring, testing, and cleanup.
- Docker commands and environment names were checked against the actual repository, including `GOOGLE_API_KEY`, port 7860, and `ktem_app_data`.
- Vietnamese and English versions share the same structure, figures, and criteria, while Vietnamese wording avoids unnecessary English terms.
- The Hugo site builds and the target pages render correctly on localhost without changing unrelated report content.

### Important decisions

| Decision | Reason | Effect |
|---|---|---|
| Use EC2 and EBS for the pilot | Fewer components and lower operational complexity than ECS Fargate, EFS, and ALB | The workshop fits the remaining schedule; scaling moves to a later phase |
| Use checksum-protected archives | Synchronizing active database files may produce an inconsistent copy | Pause briefly, archive, compute SHA-256, and restore-test every accepted backup |
| Keep the service private to testers | The pilot uses HTTP and a public IPv4 address without production authentication or HTTPS | Restrict tester IPs and add security controls before wider access |
| Track Gemini outside the AWS estimate | Gemini API is external to AWS | Monitor its quota and cost separately from AWS Budgets |

### Work to continue

- Execute the pilot deployment from the Workshop after the account and budget are confirmed.
- Run the evaluation set on actual documents and record retrieval, citations, latency, and errors.
- Exercise backup, restore, and data checks after container and EC2 restarts.
- Complete evidence, final handover, demonstration, and resource cleanup.

> Overall result: Week 7 completed the architecture, cost model, and deployment documentation for FCAJ RAG Chat. The project now has a scope-appropriate design, measurable acceptance criteria, a reproducible workshop, and a bilingual report website ready for the final-week pilot and acceptance activities.
