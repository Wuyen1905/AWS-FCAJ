---
title: "Week 2 Worklog"
date: 2024-01-01
weight: 1
chapter: false
pre: " <b> 1.2. </b> "
---

### Week 2 Objectives

* Apply AWS Cloud9 and the AWS CLI to create, inspect, and manage AWS resources in a cloud-based environment.
* Complete five Explore AWS tasks involving Amazon EC2, Amazon Bedrock, AWS Budgets, AWS Lambda, and Amazon RDS.
* Host a static website on Amazon S3, distribute content with Amazon CloudFront, and protect data with Versioning and Cross-Region Replication.
* Deploy a relational database with Amazon RDS, connect an application, and practice backup and recovery.
* Build a highly available and scalable architecture with a Launch Template, Application Load Balancer, and Auto Scaling Group.

### Tasks Completed During the Week

| Day | Task | Start Date | Completion Date | Reference Material |
| --- | ---- | ---------- | --------------- | ------------------ |
| Monday | **AWS Cloud9, CLI, and cost management**<br>- Created an AWS Cloud9 environment and became familiar with the editor, terminal, and text-file operations.<br>- Used the AWS CLI in Cloud9 to inspect and interact with AWS resources.<br>- Completed the Explore AWS task for launching an EC2 instance, then terminated the resource.<br>- Created an AWS Budget and configured email-based cost alerts. | 29/06/2026 | 29/06/2026 | [Getting Started with AWS Cloud9](https://000049.awsstudygroup.com/)<br>[Guide to five Explore AWS tasks](https://000001.awsstudygroup.com/4-h%C6%B0%E1%BB%9Bng-d%E1%BA%ABn-chi-ti%E1%BA%BFt-5-nhi%E1%BB%87m-v%E1%BB%A5-ki%E1%BA%BFm-ti%E1%BB%81n/) |
| Tuesday | **Content storage and delivery with S3 and CloudFront**<br>- Created an S3 bucket, uploaded data, and configured static website hosting.<br>- Configured public access and a bucket policy to test the website.<br>- Created a CloudFront distribution for CDN delivery, then blocked direct public access to S3.<br>- Enabled S3 Versioning and practiced object movement and Cross-Region Replication.<br>- Verified the results and cleaned up the lab resources. | 30/06/2026 | 30/06/2026 | [Starting with Amazon S3](https://000057.awsstudygroup.com/) |
| Wednesday | **Database management with Amazon RDS**<br>- Prepared a VPC, Security Groups for EC2/RDS, and a DB Subnet Group.<br>- Launched an EC2 instance for the application and created an Amazon RDS database instance.<br>- Deployed the application, verified connectivity from EC2 to RDS, and completed the Explore AWS RDS task.<br>- Created a snapshot, practiced backup/restore, and reviewed point-in-time recovery.<br>- Deleted resources that were no longer required. | 01/07/2026 | 01/07/2026 | [Getting Started with Amazon RDS](https://000005.awsstudygroup.com/)<br>[Guide to five Explore AWS tasks](https://000001.awsstudygroup.com/4-h%C6%B0%E1%BB%9Bng-d%E1%BA%ABn-chi-ti%E1%BA%BFt-5-nhi%E1%BB%87m-v%E1%BB%A5-ki%E1%BA%BFm-ti%E1%BB%81n/) |
| Thursday | **Generative AI and a serverless application**<br>- Opened the Amazon Bedrock Playground, selected a foundation model, and submitted a prompt to review its response.<br>- Completed the required use-case information for model access.<br>- Created a Lambda web application from the HTTP blueprint and tested its Function URL and response.<br>- Completed the Bedrock and Lambda Explore AWS tasks, then deleted the Lambda function to prevent unnecessary charges. | 02/07/2026 | 02/07/2026 | [Guide to five Explore AWS tasks](https://000001.awsstudygroup.com/4-h%C6%B0%E1%BB%9Bng-d%E1%BA%ABn-chi-ti%E1%BA%BFt-5-nhi%E1%BB%87m-v%E1%BB%A5-ki%E1%BA%BFm-ti%E1%BB%81n/) |
| Friday | **High availability and scalability**<br>- Prepared the network, EC2, RDS, and web server for the application architecture.<br>- Created a Launch Template, Target Group, and Application Load Balancer; configured its listener and verified traffic distribution.<br>- Created an Auto Scaling Group integrated with the Load Balancer.<br>- Tested manual, scheduled, and dynamic scaling and observed metrics used for predictive scaling.<br>- Verified application availability and cleaned up all resources. | 03/07/2026 | 03/07/2026 | [Deploying an Application with Auto Scaling Group](https://000006.awsstudygroup.com/) |

### Week 2 Achievements

* **Using a cloud-based development environment:** Successfully set up AWS Cloud9 and became familiar with its browser-based IDE. I used the code editor, terminal, file operations, and AWS CLI directly in Cloud9 to inspect and manage resources without repeatedly switching to the Console.

* **Understanding the AWS resource lifecycle:** Through the five Explore AWS tasks, I launched and terminated an EC2 instance, tested a foundation model in Amazon Bedrock, created an AWS Budget, deployed a Lambda function, and created an RDS database. Completing the creation, validation, and cleanup stages helped me understand that cloud operations require control over resource state, cost, and lifecycle—not only resource provisioning.

* **Deploying object storage and a static website with Amazon S3:** Learned to create a bucket, upload objects, configure static website hosting, and use a bucket policy to control access. I also understood how S3 Versioning protects previous versions of data and how Cross-Region Replication (CRR) copies objects to another Region to improve recoverability.

* **Delivering content with Amazon CloudFront:** Created a CloudFront distribution with S3 as its origin and tested content delivery through the CDN. This exercise demonstrated how CloudFront reduces access latency, caches content at Edge Locations, and limits direct user access to the S3 origin.

* **Deploying and protecting an Amazon RDS database:** Prepared a VPC, Security Groups, and a DB Subnet Group before creating an RDS database instance. I connected an EC2-hosted application to RDS, learned how Security Groups control traffic between the application and database tiers, and practiced snapshots, restore operations, and point-in-time recovery concepts.

* **Practicing serverless computing and Generative AI:** Used the Amazon Bedrock Playground to select a foundation model, submit prompts, and review responses. I also created an HTTP web application with AWS Lambda, tested its Function URL, and learned how the serverless model runs code on demand without requiring direct server administration.

* **Building a highly available architecture:** Created a Launch Template to standardize EC2 configuration, a Target Group to manage backend instances, and an Application Load Balancer (ALB) to distribute requests. I configured health checks so that the ALB routed traffic only to healthy instances.

* **Practicing scalability with an Auto Scaling Group:** Integrated an Auto Scaling Group (ASG) with the ALB and tested manual, scheduled, and dynamic scaling. I learned how an ASG maintains desired capacity, replaces unhealthy instances, and adjusts capacity in response to workload demand, while historical metrics can support predictive scaling.

* **Controlling cost and maintaining resource hygiene:** Configured AWS Budgets and email notifications to monitor spending. After each lab, I removed EC2, RDS, Lambda, Load Balancer, Auto Scaling Group, and related resources that were no longer required to prevent unexpected charges.

> **Overall result:** By the end of Week 2, I could combine multiple AWS services into complete workloads instead of operating each service in isolation. I gained a clearer understanding of how compute, storage, databases, CDNs, load balancing, and auto scaling work together and developed the habit of reviewing security, cost, and resource cleanup in every deployment.
