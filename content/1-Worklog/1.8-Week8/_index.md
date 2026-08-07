---
title: "Week 8 Worklog"
date: 2024-01-01
weight: 1
chapter: false
pre: " <b> 1.8. </b> "
---

{{% notice info %}}
Week 8 is the approved plan as of August 7, 2026. Work scheduled for August 10–15 will be compared with actual results, logs, and evidence before it is marked complete.
{{% /notice %}}

### Week 8 objectives

- Deploy the FCAJ RAG Chat pilot on AWS with EC2, EBS, S3, CloudWatch, and AWS Budgets.
- Validate document upload, indexing, retrieval, citation-grounded answers, and persistent data.
- Complete at least one checksum-verified backup and restore rather than checking only that S3 objects exist.
- Measure RAG quality, response time, dependency failures, and baseline security requirements.
- Complete the bilingual report, demonstration, handover, and resource cleanup before the internship ends.

### Planned work

| Day | Planned work | Start date | Planned completion | Reference |
|---|---|---|---|---|
| Monday | Prepare deployment and freeze the version<br><br>Confirm the AWS account, Region, Availability Zone, notification email, and budget limit. Freeze the commit or image ID for the demo, confirm that Git ignores `.env`, and verify the Gemini API key. Prepare 20–30 evaluation questions, reference answers, source documents, and retrieval, citation, and latency criteria. Review the create, verify, and cleanup checklist for every resource. | 10/08/2026 | 10/08/2026 | [FCAJ RAG Chat Workshop](/5-workshop/)<br>[fcaj-rag-chat](https://github.com/ngocchau04/fcaj-rag-chat) |
| Tuesday | Provision infrastructure and run the application<br><br>Create an EC2 IAM Role limited to the required S3 bucket and prefix. Create a Security Group that allows SSH only from the administrator IP and port 80 only from the test range. Launch a `t3.medium` instance, attach a 60 GB gp3 EBS volume, install Docker, Git, and AWS CLI, and clone the repository. Create a mode-600 `.env`, build the `lite` target, start Docker Compose, check health, and open the application through the public IPv4 address. Record the instance, volume, image, and commit IDs. | 11/08/2026 | 11/08/2026 | [Deploy the RAG application on EC2](/5-workshop/5.3-s3-vpc/)<br>[Amazon EC2 User Guide](https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/concepts.html) |
| Wednesday | Complete storage, backup, and monitoring<br><br>After identifying the correct new device, format and mount EBS at `/opt/fcaj/ktem_app_data`, update `fstab` by UUID, and override the Compose volume. Upload a small document, restart the container and EC2, and verify data retention. Create an encrypted, versioned, private S3 bucket; briefly stop the application; create an archive and SHA-256; and restore into an isolated environment. Install CloudWatch Agent, create CPU, status-check, and disk alarms, and configure a USD 15 AWS Budget. | 12/08/2026 | 12/08/2026 | [Persistent storage, backup, and monitoring](/5-workshop/5.4-s3-onprem/)<br>[Amazon EBS User Guide](https://docs.aws.amazon.com/ebs/latest/userguide/what-is-ebs.html)<br>[Amazon S3 User Guide](https://docs.aws.amazon.com/AmazonS3/latest/userguide/Welcome.html) |
| Thursday | Run acceptance testing<br><br>Test valid PDFs, damaged files, unsupported formats, answerable questions, and out-of-corpus questions. Record retrieved passages, answers, citations, response times, and conclusions. Test a small number of concurrent users and monitor CPU, memory, latency, HTTP 429, and Gemini retries. Confirm that S3 is private, the IAM Role cannot access unrelated resources, `.env` has mode 600, port 7860 is not directly exposed, and logs contain no API key. | 13/08/2026 | 13/08/2026 | [Test quality, security, and cost](/5-workshop/5.5-policy/)<br>[Gemini API troubleshooting](https://ai.google.dev/gemini-api/docs/troubleshooting) |
| Friday | Fix defects, finish the report, and present the demo<br><br>Classify defects by application, document, model, permission, network, and storage. Fix in-scope issues, rerun affected tests, and document remaining limitations. Update Week 8 results, acceptance evidence, self-evaluation, Proposal, and Workshop if actual architecture or figures differ. Prepare a demonstration covering application access, document upload, citation-grounded answers, persistence after restart, backup, restore, CloudWatch, and Budget. Hand over the source, documentation, and resource inventory. | 14/08/2026 | 14/08/2026 | [Project Proposal](/2-proposal/)<br>[Hugo documentation](https://gohugo.io/documentation/) |
| Saturday | Clean up, reconcile cost, and complete the retrospective<br><br>Confirm that the archive, checksum, evidence, and restore record are stored correctly. Stop the container and CloudWatch Agent, then follow the team's decision to retain or fully remove resources. For removal, handle EC2, EBS, public IPv4, S3 versions, Log Groups, alarms, SNS, IAM Roles, policies, Security Groups, and the Budget without touching shared resources. Review Billing, record delayed charges, summarize lessons, and document the ECR, ECS Fargate, EFS, HTTPS, and authentication roadmap. | 15/08/2026 | 15/08/2026 | [Clean up resources](/5-workshop/5.6-cleanup/)<br>[AWS Billing and Cost Management](https://docs.aws.amazon.com/cost-management/latest/userguide/what-is-costmanagement.html) |

### Acceptance criteria

| Area | Required condition |
|---|---|
| Functionality | Complete upload, indexing, question-answering, and citation-opening flows |
| Retrieval | At least 80% of answerable questions retrieve a relevant passage in the leading results |
| Citations | At least 90% of evaluated answers cite evidence supporting the main response |
| Out-of-corpus questions | Prefer an insufficient-information response over an unsupported assertion |
| Performance | Median response time of 15 seconds or less under a small demo load |
| Durability | Documents and indexes survive restart; the S3 archive passes checksum and restores successfully |
| Security | No API key in Git, images, screenshots, or logs; S3 is private; IAM and Security Groups are scoped correctly |
| Cost | Resources match the plan, the USD 15 Budget works, and EC2 is stopped when idle |

These are academic pilot targets. If a target is missed, the report must record the actual result, cause, impact, and corrective action instead of adjusting the data to match the goal.

### Required deliverables

- A deployment or test record identifying source version, configuration, and known limitations.
- A 20–30 question evaluation set with reference answers, source passages, and scores.
- A backup archive, SHA-256, restore record, and restore time.
- CloudWatch metric, log, alarm, and AWS Budget evidence.
- A successfully built bilingual report with no unrelated template content in the updated sections.
- Deployment, operation, troubleshooting, testing, and cleanup instructions.
- A handover record naming the owners of the source, data, bucket, and retained resources.

### Risks and contingencies

| Risk | Contingency |
|---|---|
| Insufficient AWS permissions or quota | Complete locally verifiable work, retain configurations, document the blocked step, and do not use unapproved personal credentials |
| Slow build or low disk space | Use the `lite` target, monitor root storage, reuse a verified image, and remove only confirmed unused cache |
| Gemini HTTP 429 or outage | Reduce concurrency, use bounded exponential backoff with jitter, and record quota and error details |
| Low retrieval or citation score | Review source quality, chunking, embeddings, top-k, and the prompt, then rerun the same evaluation set |
| Insufficient time for the full AWS scope | Prioritize the application, durable data, backup, restore, and cost controls; record incomplete items as limitations |
| Charges after the internship | Assign one cleanup owner, verify IDs and tags, and review Billing again the following day |

> Status on August 7, 2026: The Week 8 plan, deployment guide, and acceptance criteria are ready. Actual results will be confirmed only after the August 10–15 work is completed with supporting evidence.
