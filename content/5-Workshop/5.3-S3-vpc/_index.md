---
title : "Deploy the RAG application on EC2"
date : 2026-08-06
weight : 3
chapter : false
pre : " <b> 5.3. </b> "
---

#### Goal

This section provisions the demo server, attaches an IAM Role, installs Docker, and runs FCAJ RAG Chat from the team repository. At the end, the interface must be reachable through the public IPv4 address and the container must pass its health check.

Do not upload the official evaluation documents yet. Initial test data is stored on the root volume. In section 5.4, the application is stopped and its data directory is moved to the 60 GB EBS volume before full testing.

#### Contents

1. [Prepare EC2, the Security Group, and the IAM Role](5.3.1-create-gwe/)
2. [Install Docker, run the application, and validate it](5.3.2-test-gwe/)

#### Completion conditions

- EC2 is `running` and has a public IPv4 address.
- The IAM Role is attached and `aws sts get-caller-identity` works without static keys.
- SSH is limited to the administrator IP and application access is limited to the test scope.
- `docker compose ps` shows `fcaj-rag-chat` running or healthy.
- The browser opens the interface and the `.env` content is never exposed.
