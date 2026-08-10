---
title: "Week 3 Worklog"
date: 2024-01-01
weight: 1
chapter: false
pre: " <b> 1.3. </b> "
---

### Week 3 Objectives

* Implement monitoring and observability with Amazon CloudWatch through Metrics, Logs, Alarms, and Dashboards.
* Build a Hybrid DNS model connecting a simulated on-premises environment to AWS with Route 53 Resolver and Microsoft Active Directory.
* Move from manual Console operations to AWS CLI v2 resource management and automation and learn how to diagnose common CLI errors.
* Study the advanced Amazon EC2 topics in Module 03, from instance selection, AMIs, and storage to user data, metadata, Auto Scaling, and migration.
* Securely deploy a Linux EC2 web server, configure its Security Group, connect through SSH, and verify the application in a browser.

### Tasks Completed During the Week

| Day | Task | Start Date | Completion Date | Reference Material |
| --- | ---- | ---------- | --------------- | ------------------ |
| Monday | **Monitoring and observability with Amazon CloudWatch**<br>- Reviewed CloudWatch Metrics and used search expressions, metric math, and dynamic labels.<br>- Collected and queried logs with CloudWatch Logs Insights and created a Metric Filter from log data.<br>- Created a CloudWatch Alarm for CPUUtilization and configured a notification when the threshold was exceeded.<br>- Built a CloudWatch Dashboard for centralized monitoring of EC2 health and performance.<br>- Verified the results and cleaned up the lab resources. | 06/07/2026 | 06/07/2026 | [AWS CloudWatch Workshop](https://000008.awsstudygroup.com/) |
| Tuesday | **Hybrid DNS with Route 53 Resolver**<br>- Initialized the infrastructure with CloudFormation and prepared a Key Pair and Security Group.<br>- Connected securely through a Remote Desktop Gateway (RDGW) and deployed Microsoft Active Directory.<br>- Created Route 53 Resolver Outbound and Inbound Endpoints and a Resolver Rule for the forwarded domain.<br>- Tested bidirectional DNS resolution between the simulated on-premises environment and AWS resources.<br>- Removed the endpoints, rules, and related resources after completing the lab. | 07/07/2026 | 07/07/2026 | [Set up Hybrid DNS with Route 53 Resolver](https://000010.awsstudygroup.com/) |
| Wednesday | **Infrastructure management and troubleshooting with AWS CLI v2**<br>- Checked the CLI version and configured named profiles, the default Region, and JSON, text, and table output formats.<br>- Practiced managing S3 buckets/objects, SNS topics/subscriptions, IAM users/roles, VPCs, Subnets, Internet Gateways, and EC2 from the command line.<br>- Used filters and queries to process results and reviewed how command sequences can be automated with scripts.<br>- Diagnosed common issues involving syntax, Regions, AccessDenied responses, invalid credentials, and malformed JSON and used `--debug` when required.<br>- Cleaned up the created resources through the CLI. | 08/07/2026 | 08/07/2026 | [Getting Started with the AWS CLI](https://000011.awsstudygroup.com/)<br>[AWS CLI v2 User Guide](https://docs.aws.amazon.com/cli/latest/userguide/cli-chap-welcome.html)<br>[Troubleshooting AWS CLI](https://docs.aws.amazon.com/cli/latest/userguide/cli-chap-troubleshooting.html)<br>[AWS CLI v2 overview video](https://www.youtube.com/watch?v=U5y7JI_mHk8) |
| Thursday | **Module 03 – Compute VM on AWS**<br>- Analyzed EC2 instance families and selected instance types according to workload characteristics.<br>- Studied AMIs, backups, Key Pairs, EBS, EC2 Instance Store, User Data, and Instance Metadata.<br>- Reviewed EC2 Auto Scaling from capacity and workload perspectives.<br>- Compared EFS/FSx storage options, Amazon Lightsail, and migration with AWS Application Migration Service (MGN). | 09/07/2026 | 09/07/2026 | [Module 03-01 – Compute VM on AWS](https://www.youtube.com/watch?v=-t5h4N6vfBs)<br>[Module 03-01-01 – EC2 Instance Types](https://www.youtube.com/watch?v=e7XeKdOVq40)<br>[Module 03-01-02 – AMI, Backup, Key Pair](https://www.youtube.com/watch?v=yAR6QRT3N1k)<br>[Module 03-01-03 – Elastic Block Store](https://www.youtube.com/watch?v=hKr_TfGP7NY)<br>[Module 03-01-04 – EC2 Instance Store](https://www.youtube.com/watch?v=6IHNDJ85aoQ)<br>[Module 03-01-05 – EC2 User Data](https://www.youtube.com/watch?si=7UPcYjyhBr5NtpZM&v=_v_43Wi7zjo&feature=youtu.be)<br>[Module 03-01-06 – EC2 Metadata](https://www.youtube.com/watch?v=Ew3QRaKJQSA)<br>[Module 03-01-07 – EC2 Auto Scaling](https://www.youtube.com/watch?si=gDfz13c9xm7z0cP2&v=bbLcPitXJSY&feature=youtu.be)<br>[Module 03-02 – EC2 Auto Scaling, EFS/FSx, Lightsail, and MGN](https://www.youtube.com/watch?v=hFVYG8WqfU0) |
| Friday | **Deploying a Linux EC2 web server**<br>- Created a Security Group according to the principle of least privilege, allowing SSH, HTTP, and HTTPS only from the required sources, and compared Security Groups with Network ACLs.<br>- Launched an Amazon Linux 2023 instance and selected an appropriate AMI, instance type, Key Pair, VPC, and public subnet.<br>- Connected from the local machine through SSH with a `.pem` private key and checked the operating system and network connectivity.<br>- Installed and started Nginx, deployed a sample frontend page, and verified access through the Public DNS or Public IPv4 address in a browser.<br>- Summarized the results, updated the worklog, and terminated the instance. | 10/07/2026 | 10/07/2026 | [Introduction to Amazon EC2](https://000004.awsstudygroup.com/)<br>[Create a Security Group for Linux](https://000004.awsstudygroup.com/2-prerequiste/2.3-createsecuritygrouplinux/)<br>[Launch Amazon Linux Instance](https://000004.awsstudygroup.com/4-launchlinuxinstance/4.1-createlinuxinstance/)<br>[Connect to Amazon Linux Instance](https://000004.awsstudygroup.com/4-launchlinuxinstance/4.2-connectlinuxinstance/) |

### Week 3 Achievements

* **Establishing system observability:** Learned to combine CloudWatch Metrics, Logs, Alarms, and Dashboards into a basic observability workflow. I can monitor EC2 metrics, query logs with Logs Insights, convert log patterns into metrics, and configure alarms to identify abnormal conditions before relying on manual incident checks.

* **Understanding the alerting flow:** Understood the relationship between a metric, threshold, alarm state, and notification. Visualizing data on a dashboard also helped me review performance trends over time and collect evidence for diagnosing unstable resource behavior.

* **Implementing a Hybrid DNS architecture:** Understood the roles of Route 53 Resolver Inbound Endpoints, Outbound Endpoints, and Resolver Rules in forwarding DNS queries between AWS and on-premises environments. I tested bidirectional name resolution and learned why Security Groups, network routes, and domain rules must be configured consistently.

* **Advancing AWS CLI skills:** Used AWS CLI v2 to manage resources across S3, SNS, IAM, VPC, and EC2. I learned to work with named profiles, select Regions, change output formats, and filter results so that commands can be reused in automation scripts.

* **Improving troubleshooting skills:** Learned to verify command syntax, CLI version, Region, credentials, and IAM permissions when a command fails. I also used `--debug` to inspect credential discovery, request construction, and responses and to narrow down issues such as AccessDenied, invalid credentials, and malformed JSON.

* **Deepening Amazon EC2 knowledge:** Distinguished instance families by workload characteristics and understood the roles of AMIs, EBS, Instance Store, and User Data during instance startup. I also learned the purpose of Instance Metadata, Auto Scaling, and related options such as EFS, FSx, Lightsail, and MGN.

* **Deploying a secure Linux web server:** Created and configured an Amazon Linux instance, connected through SSH with a private key, installed Nginx, and verified the website in a browser. I gained a clearer understanding of Security Group inbound/outbound rules, the principle of least privilege, and the basic difference between stateful Security Groups and stateless Network ACLs.

* **Applying the knowledge to the project:** Deploying a sample frontend on EC2 helped me understand how the presentation layer of the E-commerce platform could be hosted, protected, and monitored on AWS. This prepared the project for later integration between the frontend, backend, and database layers.

* **Maintaining resource hygiene:** Proactively terminated the EC2 instance and removed alarms, dashboards, Resolver endpoints, and other lab resources that were no longer required. This cleanup practice reduces unexpected charges and keeps the AWS environment organized and easier to control.

> **Overall result:** By the end of Week 3, I had progressed from deploying individual services to observing, automating, and troubleshooting AWS infrastructure. I could use CloudWatch to monitor systems, Route 53 Resolver to understand hybrid connectivity, AWS CLI to manage resources, and EC2 to deploy a Linux web server with clear networking and security controls.
