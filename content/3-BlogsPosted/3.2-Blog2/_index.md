---
title: "Blog 2"
date: 2024-01-01
weight: 1
chapter: false
pre: " <b> 3.2. </b> "
---

# Handling HTTP 429 When Calling the Gemini API on AWS – How Our Team Fixed It

### Article Information

* **Published in:** AWS Study Group VN community
* **Posted on behalf of the team by:** Phan Thi Hai Van
* **Publication time:** 22:42, August 1, 2026
* **Post link:** [View the original post in AWS Study Group VN](https://www.facebook.com/groups/awsstudygroupfcj/permalink/2230866751011618/)
* **Topic:** Handling HTTP 429 when an AWS-hosted RAG application calls the Gemini API
* **Keywords:** HTTP 429, Gemini API, rate limiting, exponential backoff, jitter, Amazon EC2, Amazon Bedrock, CloudWatch

### Context of the Post

The post describes a RAG chatbot running on Amazon EC2 and calling the Gemini API for AI processing. During local testing with one user, the request rate was low and the application worked normally. After the application was placed on EC2 for concurrent team testing, multiple workers sent requests within a short period and the Gemini API began returning:

<code>HTTP 429 – Too Many Requests / RESOURCE_EXHAUSTED</code>

This response does not mean that EC2 has failed. It indicates that the API rejected a request because the project or API key reached an applicable limit, such as request volume, token usage, or usage-tier capacity. Because multiple workers share the same quota, concurrent traffic can reach the limit much faster than a single-user local test.

### Why Did Local Testing Work While EC2 Failed?

| Environment | Load characteristics | Likelihood of reaching a rate limit |
| --- | --- | --- |
| **Local** | One user, few workers, distributed requests | Lower |
| **Shared EC2 test** | Multiple users and parallel workers or processes | Higher |

Possible causes to investigate include:

* A sudden increase in concurrent requests when multiple users access the system.
* Multiple workers sharing one Gemini API key and quota.
* One RAG operation producing several model or embedding calls rather than one browser request.
* Immediate retries or retries at multiple layers amplifying traffic into a retry storm.
* Large prompts, contexts, or documents consuming tokens rapidly.
* A project quota or usage tier below the actual test load.

### Consequences of Incorrect Error Handling

If the application only catches a generic error or retries without limits, HTTP 429 can cause several secondary problems:

* A pipeline can stop between file reading, API invocation, embedding generation, and database writes.
* A worker can fail and lose the task state.
* Synchronized retries can overload the API again, increase latency, and consume EC2 resources.
* The same operation can produce duplicate data when retry behavior is not idempotent.
* Users may receive only a generic HTTP 500 response without knowing whether to wait or retry.
* Without logs and metrics, the team cannot determine whether the failure is in the provider, network, worker, or application layer.

### Solutions Proposed by the Team

#### 1. Bounded Retries with Exponential Backoff and Jitter

Instead of retrying immediately, the application increases the waiting time after each transient failure. The post illustrates these base delays:

| Failure | Base waiting time |
| --- | --- |
| First failure | 2 seconds |
| Second failure | 4 seconds |
| Third failure | 8 seconds |

A small random delay called jitter is added to each interval. Rather than having every worker retry at exactly four seconds, workers retry at slightly different times. This reduces the thundering herd effect in which synchronized retries produce another traffic spike.

A safe retry policy should:

* Retry only transient failures such as HTTP 429, timeouts, and selected 5xx errors.
* Avoid retrying authentication failures, invalid API keys, invalid payloads, or permission errors until the configuration is corrected.
* Honor <code>Retry-After</code> when the provider supplies retry timing.
* Set a maximum attempt count or a total time budget for the request.
* Implement retries at one appropriate layer instead of letting the SDK, worker, and API gateway all retry the same request.
* Record the attempt number, delay, error code, and request ID for troubleshooting.

Retries help with temporary failures only. When the quota is exhausted or sustained traffic exceeds the limit, additional retries make the situation worse.

#### 2. Control Load with Concurrency Limits, Batching, and Chunking

The team proposed controlling the workload before requests reach the API:

* **Concurrency limit:** Limit the number of workers allowed to call Gemini simultaneously through a semaphore, worker pool, or request queue.
* **Batching:** Combine small items into one request when supported by the API and use case, reducing the total number of calls.
* **Chunking:** Divide large data into sizes appropriate for input/token and processing limits.
* **Payload optimization:** Remove unnecessary fields, context, or conversation history.
* **Caching/deduplication:** Avoid invoking the model again for identical content with a valid cached result.

It is important to distinguish the two techniques: batching can reduce the number of requests, while chunking primarily controls data size. Excessive chunking without a concurrency limit can increase the request count and make HTTP 429 more frequent.

#### 3. Protect State and Ensure Idempotency

A RAG task can include document reading, chunking, embedding generation, model invocation, and data writes. The system should persist the state of each step so that a failed task can resume or retry without repeating the entire pipeline.

Data-writing operations should use an idempotency key, unique constraint, or state check before committing results. This ensures that repeated task execution does not create duplicate records, indexes, or outputs.

#### 4. Use Amazon Bedrock as a Conditional Fallback

After the retry limit is exceeded, the post proposes routing the request to Amazon Bedrock as a fallback provider:

![Architecture for retrying Gemini API and falling back to Amazon Bedrock](/images/3-BlogsPosted/3.2-Blog2/http-429-gemini-bedrock-fallback.png)

*EC2 calls the Gemini API; HTTP 429 responses are retried with exponential backoff. After retries are exhausted, the system can route the request to a fallback model in Amazon Bedrock.*

For a reliable fallback, the application must:

1. Define a common provider interface that normalizes input and output for Gemini and Bedrock.
2. Select a suitable Bedrock model and retest answer quality for the use case.
3. Configure the Region, model access, and least-privilege IAM permission such as <code>bedrock:InvokeModel</code>.
4. Transform prompts, inference parameters, and response formats for the target model.
5. Trigger fallback only for classified transient or provider-unavailable failures, not for bad data or programming errors.
6. Track fallback count, latency, cost, and result quality separately.

Amazon Bedrock also has quotas, limits, and costs. It is a resilience option that reduces dependence on one provider, not an unlimited service.

#### 5. Monitor Failures with Amazon CloudWatch

The team sends application logs from EC2 to CloudWatch Logs and creates metrics or Log Alarms to monitor:

* HTTP 429 and timeout counts.
* Total retries and requests that exceed the retry limit.
* The percentage of requests that succeed after retrying.
* The number of fallbacks to Bedrock.
* Gemini and Bedrock latency.
* Failed tasks or tasks remaining in a queue.

When a metric exceeds its threshold during a defined period, a CloudWatch Alarm can notify the team through Amazon SNS. Logs should be structured and contain a timestamp, provider, model, status code, retry count, latency, and correlation ID. API keys, credentials, complete documents, prompts, and sensitive data must not be written to logs.

#### 6. Return Clear Errors to Users

The application should not map every provider failure to HTTP 500. For temporary overload, it can return a clear message such as “The system is busy; please try again later,” together with an appropriate 429 or 503 status. Long-running work such as document indexing can be queued and exposed through task status instead of holding the request open.

### Resilience Testing Checklist

* Identify the active Gemini rate limits and usage tier in Google AI Studio.
* Run a controlled load test that reproduces HTTP 429 without affecting real data.
* Confirm that retries apply only to transient failures and stop after the configured limit.
* Verify that jitter prevents workers from retrying simultaneously.
* Limit concurrency and monitor queue depth.
* Confirm that task retries do not create duplicate data or indexes.
* Simulate Gemini unavailability and verify that Bedrock fallback activates only under the intended conditions.
* Verify that the IAM role can invoke only the required Bedrock model.
* Test CloudWatch metrics, alarms, and SNS notifications.
* Compare latency, quality, and cost before and after adding retry and fallback behavior.
* Confirm that logs contain no API keys, prompts, or sensitive documents.

### Lessons Learned

* **Working locally does not guarantee production behavior:** concurrency and user volume can change system behavior completely.
* **Retries are not a magic solution:** retry only transient failures with bounded attempts, backoff, and jitter.
* **Retries require idempotency:** otherwise, the application can produce duplicate data or inconsistent state.
* **Reduce load before retrying:** concurrency limits, queues, batching, and payload optimization address the cause instead of only the symptom.
* **Plan for external dependencies:** fallback, graceful degradation, or a clear user-facing response prevents total dependence on one provider.
* **Design observability from the beginning:** logs, metrics, and alarms reveal failures early and show whether the solution is effective.

### Links and References

* [Original post in AWS Study Group VN](https://www.facebook.com/groups/awsstudygroupfcj/permalink/2230866751011618/)
* [Gemini API – Rate limits](https://ai.google.dev/gemini-api/docs/rate-limits)
* [Gemini API – Troubleshooting](https://ai.google.dev/gemini-api/docs/troubleshooting)
* [Timeouts, retries, and backoff with jitter – Amazon Builders’ Library](https://aws.amazon.com/builders-library/timeouts-retries-and-backoff-with-jitter/)
* [REL05-BP03 – Control and limit retry calls](https://docs.aws.amazon.com/wellarchitected/latest/reliability-pillar/rel_mitigate_interaction_failure_limit_retries.html)
* [Retry behavior – AWS SDKs and Tools](https://docs.aws.amazon.com/sdkref/latest/guide/feature-retry-behavior.html)
* [Identity-based policy examples for Amazon Bedrock](https://docs.aws.amazon.com/bedrock/latest/userguide/security_iam_id-based-policy-examples.html)
* [InvokeModel API – Amazon Bedrock](https://docs.aws.amazon.com/bedrock/latest/APIReference/API_runtime_InvokeModel.html)
* [Alarming on logs – Amazon CloudWatch](https://docs.aws.amazon.com/AmazonCloudWatch/latest/monitoring/Alarm-On-Logs.html)
* [Creating a metric filter – Amazon CloudWatch Logs](https://docs.aws.amazon.com/AmazonCloudWatch/latest/logs/CreateMetricFilterProcedure.html)

> **Conclusion:** HTTP 429 is not merely an error that should be retried a few times; it is a signal that the system must manage load and external dependencies more carefully. Bounded retries, exponential backoff, jitter, concurrency control, idempotency, conditional fallback, and CloudWatch observability make the AI API workflow more resilient as it moves from local testing to concurrent use.
