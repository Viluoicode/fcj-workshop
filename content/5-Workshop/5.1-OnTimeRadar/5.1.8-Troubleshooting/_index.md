---
title: "Troubleshooting and Future Work"
date: 2026-11-23
weight: 8
chapter: false
pre: " <b> 5.1.8. </b> "
---

# Troubleshooting and Future Work

These problems came up while building and running OnTime Radar. Each one lists the commit that fixed it in the `ontime-radar` repository.

## Problems I Hit

### 1. The API started answering 503 once real traffic arrived

- **Symptom:** with the Olist replay running, API Gateway answered some webhooks with 503.
- **Cause:** a new AWS account can run only 10 Lambdas at the same time. The FIFO processor scaled to one instance per shipment group and took all 10, leaving nothing for the API.
- **Fix:** I limited the processor's SQS trigger to 2 concurrent instances. Asking AWS for a higher quota is the next step.
- **Commit:** `e835dd5` (16/11).

### 2. A secret file was almost committed

- **Symptom:** `samconfig.toml`, which holds the webhook secret, showed up as an untracked file even though it was listed in `.gitignore`.
- **Cause:** the line had a comment after the pattern. In `.gitignore`, text after the pattern on the same line becomes part of the pattern, so it never matched.
- **Fix:** I moved the comment to its own line.
- **Commit:** `d77358c` (16/11).

### 3. The late-delivery alarm kept flipping

- **Symptom:** the first alarm fired at "5 or more late deliveries per hour". With about 80 deliveries an hour, it went into ALARM three times in one morning.
- **Cause:** a fixed count grows with traffic, so a busier hour looks like an incident.
- **Fix:** the alarm now uses the late *rate*: 20% or more in 2 of 3 windows, counting only windows with at least 10 deliveries. It also sends an "all clear" email.
- **Commit:** `b6c2dfa` (19/11).

### 4. GitHub could not assume the deploy role

- **Symptom:** the first OIDC deployment was denied at `AssumeRoleWithWebIdentity`.
- **Cause:** GitHub's token now carries the owner and repository IDs in its `sub` claim (`repo:owner@<id>/repo@<id>:...`). The trust policy expected names only.
- **Fix:** the trust policy matches the fixed IDs. This is also safer: a new repository created with the same name cannot use the role.
- **Commit:** `01b4e9c` (20/11).

### 5. The replayer could skip failed orders

- **Symptom:** when a few webhooks in a run failed, the cursor still moved forward, so those orders were never sent again.
- **Fix:** responses are now sorted into Sent, Rejected (bad data that resending cannot fix), and Failed (throttling, 5xx, network). The cursor moves only when Failed is 0. Resending is safe because duplicates are dropped.
- **Commit:** `81576a1` (20/11).

### 6. Broken payloads returned 500 instead of 400

- **Symptom:** the chaos generator found that a body that is not a JSON object, or a number sent as a string, made the adapters throw. The API returned 500, and a real carrier would keep retrying a 500 forever.
- **Fix:** every adapter now checks the JSON shape and value types first and returns 400. I added 7 regression tests.
- **Commit:** `ac243b0` (23/11).

### 7. Other problems (to be filled in)

*(Add problems that are not in the commit history, for example console errors or `AccessDenied` messages during the first deployment, with the same layout: symptom, cause, fix.)*

## Known Limitations

| Limitation | Effect today | Possible fix |
|---|---|---|
| **R1:** two events with exactly the same timestamp; the one that arrives later wins | 0 shipments affected: the replay data never has equal timestamps | Break ties with a fixed order (status rank, then source code) |
| **R2:** a promised date sent only on an event that arrives late is lost, so the shipment is not counted in the on-time rate | 0 shipments affected: the replayer sends the promised date on every webhook | Let a `Stale` event fill empty fields without changing the status |
| DynamoDB and S3 are not written in one transaction | If the Lambda stops between the two writes, the event is missing from Athena. Lookups and alarms are not affected | Write the lake from DynamoDB Streams |
| Lateness compares the UTC date with the promised date | 280 of 96,470 orders (0.29 percentage points) are counted late although they arrived in the evening of the promised day in Brazil | Compare in the delivery region's time zone |
| Carriers, tracking codes, and in-transit steps are simulated | Failed-attempt rates do not reflect real carriers | Connect real carrier APIs |
| The data is from Brazil | Regions are Brazilian states, not Vietnamese provinces | Use Vietnamese order data when available |

Each R1/R2 gap has a test named `Known_gap_...` that records today's behaviour, so fixing the rule forces the test to change too.

## Future Work

- Ask AWS for a higher Lambda concurrency quota (in a burst test, 80 requests at once still produced 47 errors of 503).
- Give each carrier its own secret, or sign webhooks with HMAC, stored in Secrets Manager.
- Narrow the S3 permissions of the processor and the replayer.
- Add a lifecycle rule for Athena results, and stop a query that times out.
- Write the event lake from DynamoDB Streams, to close the gap described above.
- Finish the cost report and record the demo video.
