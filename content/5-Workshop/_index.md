---
title: "Workshop"
date: 2026-08-06
weight: 5
chapter: false
pre: " <b> 5. </b> "
---

# Deploying FCAJ RAG Chat on AWS

#### Overview

This workshop deploys the team's RAG document question-answering system. The Kotaemon application is packaged with Docker, runs on Amazon EC2, persists its data on Amazon EBS, and backs up to a private Amazon S3 bucket. Amazon CloudWatch and AWS Budgets provide operational and cost visibility, while the server-side application calls the Gemini API for embeddings or answer generation according to the selected configuration.

After completing the workshop, learners will be able to:

- Prepare EC2, a Security Group, an IAM Role, and EBS for a RAG application.
- Configure Gemini safely and run FCAJ RAG Chat with Docker.
- Validate document upload, indexing, retrieval, and citations.
- Back up and restore data with S3 and verify persistence after container restarts.
- Monitor the system with CloudWatch, configure cost alerts, and clean up resources correctly.

This architecture is intended for an academic pilot and an internal demonstration. Before serving external users, add HTTPS, authentication, dedicated secret management, and a more highly available design.

#### Contents

1. [Workshop introduction and architecture](5.1-Workshop-overview/)
2. [Prepare the account, source code, and deployment parameters](5.2-Prerequiste/)
3. [Deploy the RAG application on EC2](5.3-S3-vpc/)
   1. [Prepare EC2, the Security Group, and the IAM Role](5.3-S3-vpc/5.3.1-create-gwe/)
   2. [Install Docker, run the application, and validate it](5.3-S3-vpc/5.3.2-test-gwe/)
4. [Persistent storage, backup, and monitoring](5.4-S3-onprem/)
   1. [Prepare and mount EBS](5.4-S3-onprem/5.4.1-prepare/)
   2. [Create a private S3 bucket and back up data](5.4-S3-onprem/5.4.2-create-interface-enpoint/)
   3. [Restore data and verify durability](5.4-S3-onprem/5.4.3-test-endpoint/)
   4. [Configure CloudWatch and AWS Budgets](5.4-S3-onprem/5.4.4-dns-simulation/)
5. [Test quality, security, and cost](5.5-Policy/)
6. [Clean up resources](5.6-Cleanup/)
