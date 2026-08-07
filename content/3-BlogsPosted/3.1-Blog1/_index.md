---
title: "Blog 1"
date: 2024-01-01
weight: 1
chapter: false
pre: " <b> 3.1. </b> "
---

# EC2 Looks Simple, but It Is Not – Lessons Learned from AWS Labs

### Article Information

* **Published in:** AWS Study Group VN community
* **Posted on behalf of the team by:** Phan Thi Hai Van
* **Publication time:** 09:17, July 28, 2026
* **Post link:** [View the original post in AWS Study Group VN](https://www.facebook.com/groups/awsstudygroupfcj/permalink/2226675271430766/)
* **Topic:** Managing the Amazon EC2 lifecycle and reducing unexpected lab costs
* **Keywords:** Amazon EC2, Amazon EBS, AWS Free Tier, Amazon EventBridge, AWS Lambda, AWS Budgets, Cost Optimization
* **Hashtags:** #AWSStudyGroup #FCAJ #EC2 #CostOptimization

### Why the Team Chose This Topic

Amazon EC2 is often one of the first services encountered by AWS learners. Launching a virtual machine appears straightforward, but learners can easily leave resources behind when they do not fully understand instance states, attached storage, and the charging model.

The team wrote this post to share practical misunderstandings encountered during labs, especially the difference between Stop and Terminate, how to track AWS Free Tier eligibility, and how Amazon EventBridge and AWS Lambda can automate instance shutdown.

### Main Content of the Post

#### 1. Stop Is Not the Same as Terminate

When an instance is stopped, it remains in the account and can be started again. Compute usage charges stop after the instance reaches the stopped state, but associated resources may continue to incur charges, most notably Amazon EBS volumes. Retained networking resources or public IP allocations should also be reviewed separately.

When an instance is terminated, it is permanently deleted and cannot be restarted. The root EBS volume is normally deleted by default, but the actual behavior depends on the <code>DeleteOnTermination</code> attribute. Data volumes can remain and continue to incur charges when this attribute is <code>false</code>. Therefore, users should still review Volumes, Snapshots, Elastic IP addresses, and related resources after termination instead of assuming that everything has been removed.

| State | Can be started again | Instance compute charge | EBS volumes | Typical use |
| --- | --- | --- | --- | --- |
| Stopped | Yes | Not charged after the instance stops | Remain and may incur charges | Pause an instance for later use |
| Terminated | No | Instance charging stops | Deleted or retained according to <code>DeleteOnTermination</code> | Permanently remove an unused instance |

Lesson learned: Stopping an instance pauses compute; it does not clean up every associated resource.

#### 2. Free Tier Should Not Be Remembered as One Fixed Number

The original post emphasized that under the previous Free Tier model, eligible instance hours were calculated as the combined usage of all eligible instances, rather than giving every instance a separate allowance. Running several instances concurrently could therefore consume the available usage much faster than expected.

However, the current AWS Free Tier model depends on the account creation date:

* Accounts created before July 15, 2025, may use the previous EC2 Free Tier model with hourly limits during the first 12 months.
* New accounts created from July 15, 2025, use a credit-based model; the free plan lasts for up to six months or until the available credits are exhausted.
* In both cases, users should check the Free Tier and Billing and Cost Management pages and review the terms that apply to their own account instead of relying on a number remembered from older material.

**Lesson learned:** Free Tier can reduce learning costs, but it does not mean that every resource is free or that usage can be left unmonitored.

#### 3. Automate EC2 Shutdown Instead of Relying on Memory

Learners can forget to stop an instance after completing a short lab. The post proposes using a fixed schedule to invoke the <code>StopInstances</code> operation automatically. The team's diagram uses this flow:

**Amazon EventBridge → AWS Lambda → Amazon EC2**

![Architecture for automatically stopping EC2 with EventBridge and Lambda](/images/3-BlogsPosted/3.1-Blog1/eventbridge-lambda-stop-ec2.png)

*Amazon EventBridge runs at 23:00 every day, Lambda applies the required logic, and the EC2 StopInstances API stops the selected instances.*

The workflow can be understood as follows:

1. EventBridge triggers the schedule at 23:00 every day.
2. The schedule invokes a Lambda function.
3. Lambda identifies the instances to stop, preferably by tags such as <code>Environment=Lab</code> or <code>AutoStop=true</code>.
4. Lambda invokes the EC2 <code>StopInstances</code> API through an IAM role containing only the required permissions.
5. Execution results are written to CloudWatch Logs for troubleshooting.

For new implementations, AWS recommends **Amazon EventBridge Scheduler** instead of legacy scheduled rules. Scheduler can invoke AWS APIs on a schedule; Lambda is useful when additional logic is required, such as tag-based filtering, exclusion rules, notifications, or multiple conditions.

**Lesson learned:** Automation reduces dependence on manual action, but IAM should still follow least privilege and execution logs should be reviewed.

#### 4. Configure Cost Alerts

Automated EC2 shutdown addresses only part of the risk. Learners should also create an AWS Budget and configure alerts based on actual or forecasted cost thresholds. Alerts help identify forgotten resources, but billing data is not real-time, so alerts do not replace resource checks after every lab.

### Post-Lab Checklist

* Review the EC2 Dashboard and confirm that no unexpected instance remains in the <code>running</code> state.
* Apply clear names or tags that identify the purpose, owner, and creation date of each instance.
* Stop an instance when it will be reused; terminate it when it is no longer required.
* Review <code>DeleteOnTermination</code> and delete unnecessary EBS volumes or snapshots.
* Check Elastic IP addresses, Load Balancers, NAT Gateways, and other associated resources that may incur charges.
* Configure EventBridge Scheduler or EventBridge–Lambda automation for lab environments that require automatic shutdown.
* Enable AWS Budgets and regularly review the Billing Dashboard and Free Tier usage.
* Complete the cleanup procedure included in the workshop.

### Value and Lessons from the Post

The post helped the team move from the idea that “launching EC2 completes the task” to managing the complete resource lifecycle. A lab is only complete after its result has been validated, evidence has been recorded, and unused resources have been removed.

These lessons can be applied directly to later worklogs through naming conventions, resource tags, cost alerts, and cleanup checklists. They also introduce the basic mindset behind cloud FinOps and cost optimization.

### Links and References

* [Original post in AWS Study Group VN](https://www.facebook.com/groups/awsstudygroupfcj/permalink/2226675271430766/)
* [Amazon EC2 instance lifecycle](https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/ec2-instance-lifecycle.html)
* [Track Amazon EC2 Free Tier usage](https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/ec2-free-tier-usage.html)
* [Amazon EventBridge Scheduler](https://docs.aws.amazon.com/eventbridge/latest/userguide/using-eventbridge-scheduler.html)
* [Best practices for AWS Budgets](https://docs.aws.amazon.com/cost-management/latest/userguide/budgets-best-practices.html)

> **Conclusion:** Amazon EC2 is easy to start with but requires careful management. Understanding Stop versus Terminate, checking account-specific Free Tier terms, using controlled automation, and maintaining a cleanup habit are simple but important practices for learning AWS safely and efficiently.
