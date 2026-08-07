---
title : "Test quality, security, and cost"
date : 2026-08-06
weight : 5
chapter : false
pre : " <b> 5.5. </b> "
---

#### Goal

A working interface is not sufficient evidence that the project succeeds. This section evaluates the complete chain from upload, indexing, retrieval, answer generation, and citations to durable data, security, failure handling, and operating cost.

#### 1. Prepare the evaluation set

Select documents representative of the demonstration. Remove sensitive data, duplicates, and obsolete versions. Record each document's name, version, page count, language, and checksum where reproducibility matters.

Create 20–30 questions covering:

- Answers stated clearly in one passage.
- Answers requiring two passages from the same document.
- Similar terms or conflicting document versions.
- Questions with no answer in the corpus to test abstention.
- Vietnamese and English questions if bilingual support is claimed.

Each question needs a reference answer, source document, expected passage, and question type. Do not use a chatbot-generated answer as the ground truth.

#### 2. Functional tests

| ID | Scenario | Expected result |
|---|---|---|
| F01 | Upload a valid PDF | The file is accepted, indexing completes, and no silent error occurs |
| F02 | Upload an unsupported or damaged file | The system rejects it clearly without damaging the current collection |
| F03 | Ask an answerable question | The answer is relevant and its citation opens the supporting passage |
| F04 | Ask an out-of-corpus question | The system indicates insufficient information instead of inventing a fact |
| F05 | Restart the container | Documents, indexes, and required history remain |
| F06 | Stop and start EC2 | EBS mounts, the container starts, and data is accessible |
| F07 | Back up and restore | Checksum passes and the restore container answers a known question |

Record time, commit hash, model, document set, actual result, and evidence for every case. For a failure, identify the failing step rather than recording only “failed.”

#### 3. RAG evaluation

Score three layers for each question:

1. **Retrieval:** whether the expected passage appears in the top retrieved results.
2. **Answer:** whether the answer conveys the correct meaning without contradicting the corpus.
3. **Citation:** whether the cited source supports the main claim.

Suggested metrics:

```text
Retrieval accuracy = questions retrieving the correct passage / answerable questions
Citation support rate = answers with supporting citations / answered questions
Unsupported answer rate = asserted out-of-corpus answers / out-of-corpus questions
```

Pilot acceptance targets:

- Retrieval accuracy of at least 80%.
- Citation support rate of at least 90%.
- Out-of-corpus questions should produce an insufficient-information response; investigate every unsupported assertion.

For low scores, review source quality, chunking, embeddings, top-k, and the prompt before selecting a larger model.

#### 4. Performance and dependency-failure testing

Measure question receipt, retrieval completion, first response, and completed response. For the small demo, the initial target is a median response time of 15 seconds or less.

Test one, two, and a small number of concurrent users. Monitor:

- EC2 CPU, memory, and disk usage.
- Retrieval latency and end-to-end latency.
- Gemini HTTP 429, timeout, and retry counts.
- Indexing time by document size.

Retries must be bounded and use exponential backoff with jitter. Infinite bursts increase failures and make quota usage difficult to control.

#### 5. Security tests

- Search Git history, Docker image metadata, and logs for `GOOGLE_API_KEY`, access-key patterns, and `.env` contents.
- Confirm that S3 Block Public Access remains enabled and no object is public.
- Use IAM Policy Simulator or a controlled negative test to verify that the role cannot access an unrelated bucket.
- Confirm that SSH is restricted to the administrator IP and port 7860 is not publicly exposed.
- Use an invalid API key and verify a controlled error without key output in logs.
- Verify `.env` mode 600 and minimal operator access.

Example checks:

```bash
cd /opt/fcaj/app
stat -c '%a %U:%G %n' .env
sudo docker compose ps
sudo ss -lntp | grep -E ':80|:7860'
```

#### 6. Cost reconciliation

After the test session, compare billing data with the estimate:

- Actual EC2 runtime.
- Lifetime of the 60 GB gp3 volume.
- Time with a charged public IPv4 address.
- S3 size, object versions, and CloudWatch logs.
- Alarms, custom metrics, and data transfer.
- Gemini or other external-API charges tracked separately.

The target is `t3.medium` for approximately 60 hours per month, estimated at USD 11.36 including 15% contingency, with a USD 15 monthly Budget. Continuous operation is estimated at approximately USD 55.89 per month, so EC2 must be stopped when idle.

#### 7. Acceptance record

| Area | Result | Evidence | Conclusion |
|---|---|---|---|
| Functionality | Passed tests / total tests | Screenshots, logs, test sheet | Pass/Fail |
| Retrieval | Correct-passage rate | 20–30 question evaluation | Pass/Fail |
| Citations | Supporting-source rate | Source comparison | Pass/Fail |
| Performance | P50, maximum, and failures | Metrics and logs | Pass/Fail |
| Durability | Restart and restore | Archive, checksum, test evidence | Pass/Fail |
| Security | Secret scan, IAM, SG, and S3 | Checklist | Pass/Fail |
| Cost | Actual versus estimate | Billing and Budget | Pass/Fail |

For any critical failure, record the owner, corrective action, and retest date. Do not complete the workshop while serious data-protection or secret-management defects remain.
