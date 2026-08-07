---
title: "Blog 3"
date: 2024-01-01
weight: 1
chapter: false
pre: " <b> 3.3. </b> "
---

# I Hardcoded an API Key in an ECS Task Definition – How We Fixed It with Parameter Store

### Article Information

* **Status:** AWS Study Group VN community
* **Planned platform:** AWS Study Group VN community
* **Publication date:**  August 5, 2026
* **Post link:** https://www.facebook.com/groups/awsstudygroupfcj/permalink/2235479093883717/?rdid=0EI9A3H3rdsnAmIa#
* **Topic:** Securely managing a Gemini API key when deploying a container on Amazon ECS Fargate
* **Keywords:** Amazon ECS, AWS Fargate, Task Definition, Parameter Store, SecureString, AWS KMS, IAM, secret management

### Context of the Post

While deploying **fcaj-rag-chat**, a RAG chatbot customized from Kotaemon, to Amazon ECS Fargate, the backend container required a Gemini API key for its chat and embedding models. The quickest configuration was to place the key directly in the container definition's <code>environment</code> section:

    {
      "name": "GEMINI_API_KEY",
      "value": "actual-secret-value"
    }

This configuration made the application run, but it created a serious security issue: the secret became plaintext data in the ECS task definition. Anyone allowed to view the task definition or related configuration could read the key, and the value could also be copied into a repository, CI/CD log, screenshot, or report.

The team removed the secret from the task definition and used AWS Systems Manager Parameter Store with a SecureString parameter. The task definition now stores only the parameter ARN, while ECS retrieves the actual value when the task starts.

### Why Should Secrets Not Be Hardcoded?

#### 1. Broad Exposure

A task definition is not a secret store. Plaintext environment values can appear when a task definition is described through the Console, API, or CLI. Sharing the JSON for review also shares the key.

#### 2. Difficult and Error-Prone Rotation

ECS task definitions are revisioned resources. A key change requires a new revision and service update. Unless handled correctly, the old value can remain in previous revisions, configuration files, or logs.

#### 3. Weak Least-Privilege Separation and Auditability

When a secret is embedded in configuration, it is difficult to separate permission to deploy an application from permission to read the secret. Parameter Store allows access to be controlled by parameter ARN or path and makes AWS API activity available for auditing.

#### 4. Security Is More Than Networking

Security Groups, private subnets, and an Application Load Balancer protect the network flow, but they do not protect an API key stored as plaintext configuration. Security must cover networking, identity, secrets, images, logs, and data.

### Immediate Response When a Key Was Hardcoded

Moving a key into Parameter Store does not make the previous exposure disappear. Before deploying the new configuration, the team should:

1. Revoke or rotate the old Gemini API key at the provider.
2. Create a replacement key and store it only in an approved secret store.
3. Remove the key from source code, committed <code>.env</code> files, insecure CI/CD variables, and shared documents.
4. Review Git history, build logs, CloudWatch Logs, screenshots, and ECS task definition revisions that may contain the old value.
5. Stop tasks using the old key and deploy new tasks.
6. Monitor quota and request activity for signs that the old key was misused.

If a secret was committed to Git, deleting it in a later commit is not sufficient. Rotate the key first and then remediate repository history according to the team's process.

### Architecture After the Change

![Comparison of a hardcoded API key and AWS Systems Manager Parameter Store](/images/3-BlogsPosted/3.3-Blog3/ecs-parameter-store-securestring.png)

*Left: the API key is stored as plaintext in the task definition. Right: the task definition references a Parameter Store SecureString protected with AWS KMS.*

The new flow is:

1. The Gemini API key is stored as a Parameter Store <code>SecureString</code>.
2. Parameter Store uses AWS KMS to encrypt the value at rest.
3. The ECS Task Definition references the parameter ARN through the <code>secrets</code> field and contains no plaintext value.
4. At task startup, ECS/Fargate uses the Task Execution Role to retrieve and decrypt the parameter.
5. ECS injects the value into the container as the <code>GEMINI_API_KEY</code> environment variable.

### Detailed Configuration Process

#### Step 1: Create a SecureString Parameter

The team uses a hierarchical name that identifies the application, environment, and purpose:

<code>/fcaj-rag-chat/prod/gemini-api-key</code>

Important settings include:

* **Type:** <code>SecureString</code>.
* **Tier:** Standard is suitable for a small API key; current pricing and quotas should still be reviewed.
* **KMS key:** Use the AWS managed <code>aws/ssm</code> key or a customer managed symmetric KMS key when tighter key-policy and audit control is required.
* **Description/Tags:** Record the owner, environment, application, and purpose without including the secret in tags or descriptions.

The real value should not be placed in shell history, pipeline logs, or screenshots during parameter creation. Automated workflows should obtain the secret through a secure CI/CD input mechanism.

#### Step 2: Grant the Task Execution Role

When ECS injects a Parameter Store value through <code>secrets</code>, the Task Execution Role is used by the ECS agent/Fargate infrastructure to retrieve the secret before the container starts. The role needs permission to read the exact parameter:

    {
      "Effect": "Allow",
      "Action": "ssm:GetParameters",
      "Resource": "arn:aws:ssm:<region>:<account-id>:parameter/fcaj-rag-chat/prod/gemini-api-key"
    }

When the parameter uses a customer managed KMS key, the role also needs <code>kms:Decrypt</code> on that key, and the key policy must allow the role to use it:

    {
      "Effect": "Allow",
      "Action": "kms:Decrypt",
      "Resource": "arn:aws:kms:<region>:<account-id>:key/<key-id>"
    }

Avoid <code>Resource: "*"</code> when access can be limited to a specific parameter ARN and KMS key ARN.

#### Step 3: Distinguish the Task Execution Role from the Task Role

| Role | Used by | Purpose in this scenario |
| --- | --- | --- |
| **Task Execution Role** | ECS agent/Fargate infrastructure | Pull the image, deliver logs, and retrieve Parameter Store secrets before container startup |
| **Task Role** | Application code inside the container | Call AWS APIs while the application is running |

For injection through <code>secrets</code>, <code>ssm:GetParameters</code> belongs to the Task Execution Role. If the application is designed to retrieve Parameter Store values programmatically at runtime, the permission belongs to the Task Role instead.

#### Step 4: Reference the Parameter in the Task Definition

The team removes the API key from <code>environment</code> and adds it under <code>secrets</code>:

    {
      "name": "GEMINI_API_KEY",
      "valueFrom": "arn:aws:ssm:<region>:<account-id>:parameter/fcaj-rag-chat/prod/gemini-api-key"
    }

When the parameter is in the same Region as the task, ECS accepts its name or ARN; a complete ARN makes the configuration explicit. Parameter Store is Regional, so the task, parameter, and KMS key Regions must be reviewed.

#### Step 5: Register a Revision and Deploy New Tasks

After updating the task definition:

1. Register a new task definition revision.
2. Update the ECS service to use the new revision.
3. Select Force new deployment to launch new tasks.
4. Monitor ECS service events and CloudWatch Logs.
5. Confirm that health checks and Gemini requests succeed.
6. Stop old tasks after the new tasks become healthy.

The parameter is injected when the container starts. If the SecureString value is updated, running tasks do not receive the new value automatically. Launch a new task or force a new service deployment.

### Results After the Change

* The task definition displays only the parameter ARN and no longer exposes the plaintext Gemini API key.
* Permission to read the secret is separated from permission to modify the task definition and constrained through IAM.
* Key rotation requires one Parameter Store update followed by a new deployment instead of placing a real value in a task definition.
* The task definition JSON is safer to share for review because it does not contain the secret value.
* Hierarchical parameter naming simplifies management by application and environment.
* KMS and IAM provide a stronger basis for auditing and access control.

### Limitations and Security Considerations

Parameter Store SecureString significantly improves secret storage, but it does not make the value invisible at every stage:

* After injection, the value exists as a plaintext environment variable inside the container and can be read by processes or tools with sufficient access.
* The application must never log the complete environment, request headers, or exceptions containing the API key.
* ECS Exec, container-debugging permissions, and log access must be restricted.
* ECS Fargate must use a platform version that supports secret injection; use an appropriate current version.
* A Systems Manager interface VPC endpoint can allow private tasks to reach the service without traversing the public internet.
* KMS protects data at rest, while IAM, networking, logging, and runtime isolation must still be configured correctly.

For stricter requirements, AWS recommends considering a sidecar that writes a secret to a task-scoped volume or having the application retrieve the secret programmatically at runtime instead of retaining it in an environment variable.

### Parameter Store or Secrets Manager?

| Criterion | Parameter Store SecureString | AWS Secrets Manager |
| --- | --- | --- |
| Encrypted key/config storage | Yes | Yes |
| Hierarchical parameter naming | Yes | Not through the same path model |
| Integrated automatic rotation | Not its primary feature | Yes |
| Random secret generation | Not its primary feature | Yes |
| Cross-account secret sharing | More limited | Designed to support it more directly |
| Suitable for | Simple secrets/configuration with manual updates | Secrets requiring advanced lifecycle and rotation |

Parameter Store SecureString can meet the needs of a Gemini API key that is stored, read, and rotated manually. Secrets Manager is more appropriate for automatic rotation, advanced version/lifecycle management, or cross-account sharing. Current pricing and quotas should be reviewed before making the final choice.

### Testing Checklist

* Confirm that the new task definition contains no plaintext API key.
* Verify that the Task Execution Role has <code>ssm:GetParameters</code> only for the required parameter.
* When using a customer managed KMS key, verify <code>kms:Decrypt</code> and the key policy.
* Confirm that the new task starts successfully without <code>AccessDeniedException</code>.
* Verify that the container receives <code>GEMINI_API_KEY</code> without the application printing it to logs.
* Rotate a test parameter, force a new deployment, and confirm that new tasks receive the updated value.
* Confirm that an old task retains the old value until it is replaced.
* Test service rollback and health checks when the parameter ARN is wrong or the role lacks permission.
* Review the repository, task definition revisions, CI/CD logs, and documentation for previous exposure.
* Revoke the old key and monitor for unexpected usage.

### Lessons Learned

* **“It runs” does not mean “it is secure”:** secret management must be reviewed before configurations are shared or deployed.
* **Task Execution Role and Task Role serve different purposes:** attaching permission to the wrong role commonly prevents startup or grants excessive access.
* **Encryption must be combined with access control:** KMS protects data at rest, while IAM determines who can decrypt it.
* **Rotate exposed secrets:** moving a key to a safe store does not revoke plaintext copies that appeared previously.
* **Secret updates require redeployment:** running containers do not automatically receive a new Parameter Store value.
* **Choose a service based on the secret lifecycle:** Parameter Store suits simple needs; Secrets Manager is more suitable for automatic rotation and advanced secret management.

### Links and References

* [Pass sensitive data to an Amazon ECS container](https://docs.aws.amazon.com/AmazonECS/latest/developerguide/specifying-sensitive-data.html)
* [Pass Systems Manager parameters through ECS environment variables](https://docs.aws.amazon.com/AmazonECS/latest/developerguide/secrets-envvar-ssm-paramstore.html)
* [Parameter Store SecureString parameters](https://docs.aws.amazon.com/systems-manager/latest/userguide/sysman-paramstore-securestring.html)
* [KMS encryption for SecureString parameters](https://docs.aws.amazon.com/systems-manager/latest/userguide/secure-string-parameter-kms-encryption.html)
* [Creating a Parameter Store parameter using AWS CLI](https://docs.aws.amazon.com/systems-manager/latest/userguide/param-create-cli.html)
* [AWS Systems Manager Parameter Store overview](https://docs.aws.amazon.com/systems-manager/latest/userguide/systems-manager-parameter-store.html)

> **Conclusion:** Removing the API key from the ECS task definition is a small but important change. Parameter Store SecureString, AWS KMS, and least-privilege IAM reduce the risk of secret exposure, simplify updates, and make configuration safer to review. The solution is complete only when the old key is rotated, runtime access is restricted, secrets are excluded from logs, and new tasks are deployed whenever the value changes.
