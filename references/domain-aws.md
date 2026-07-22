# Domain Playbook: AWS

Load when the change touches IAM, Lambda, SQS/SNS, DynamoDB, or Step
Functions.

- **IAM boundaries** — check the actual policy, not the intent in the PR
  description. Wildcard resources/actions, missing conditions (`aws:SourceArn`,
  `aws:SourceAccount`), and overly broad trust policies are the most common
  real vulnerabilities.
- **Retries** — AWS SDKs retry by default on throttling/5xx. Confirm the
  operation being retried is actually idempotent (e.g. `PutItem` without a
  conditional expression is not safe to retry blindly under a network
  timeout — you can't tell if the first attempt succeeded).
- **Idempotency** — for Lambda + SQS, at-least-once delivery is the default.
  Confirm handlers dedupe on a stable message attribute, not on
  `messageId` alone if the producer might legitimately resend the same
  logical event with a new `messageId`.
- **Consistency** — DynamoDB reads are eventually consistent unless
  `ConsistentRead: true` is set; GSIs are always eventually consistent.
  Confirm correctness-critical reads account for this.
- **Partial failure in batches** — `BatchWriteItem`/SQS batch operations can
  partially fail. Confirm unprocessed items/failed messages are actually
  handled, not just logged and dropped.
- **Step Functions** — check retry/catch blocks per state, not just at the
  top level; a state without a `Catch` fails the whole execution on any
  transient error.
