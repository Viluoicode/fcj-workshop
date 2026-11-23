---
title: "Ingestion and Bronze Layer"
date: 2026-11-23
weight: 3
chapter: false
pre: " <b> 5.2.3. </b> "
---

# Ingestion and Bronze Layer

## The Boards

`ingestion/sources.json` lists 36 boards on four platforms:

| Platform | Boards | Examples |
|---|---:|---|
| Greenhouse | 20 | Stripe, Airbnb, Databricks, GitLab |
| Ashby | 11 | OpenAI, Notion, Ramp, Mistral AI |
| Lever | 4 | Spotify, Palantir |
| Arbeitnow | 1 | a feed that aggregates many companies |

None of them needs an API key. A test checks that the list is well formed: every platform is supported, no board appears twice, and no company is listed on two platforms. That last rule exists because Mistral AI moved from Lever to Ashby in November (see 5.2.8).

## Step 1 - Fetch the Boards in Parallel

The job runs five API calls at a time. Each call has a 20-second timeout and up to three attempts. It retries on network errors, HTTP 429, and 5xx responses, and it waits longer after each failure.

Results are processed in the same order as the list, whichever call finishes first. The job only fetches up to ten boards ahead of what it has already written. That limit matters because a single run produces about 150 MB of JSON, and Glue Python Shell has about 1 GB of memory.

## Step 2 - Turn Every Answer Into the Same Shape

Each platform names its fields differently. A small adapter per platform maps them into one record: source, board, the platform's job ID, company, title, location, work mode, application URL, posting date, and the original JSON.

Work mode has three values: remote, onsite or hybrid, and unknown. Each platform gives a different hint (Lever and Ashby have a workplace type, Arbeitnow has a remote flag, Greenhouse only has the location text). When no hint is present, the value stays unknown instead of being guessed. On 20 November, 47.5% of postings were unknown.

If a response does not look like a list of jobs, the board is marked as failed. It is never treated as a board that simply had no jobs.

## Step 3 - Write Bronze, Then the Manifest

Each board's records go to their own file, under a folder for the run:

```text
bronze/run_id=<run>/source=<platform>/board=<token>/jobs.json
```

Only after every board has been handled does the job write the manifest:

```text
control/ingestion/run_id=<run>/manifest.json
```

The manifest puts each board in one of four groups: success with records, success but empty, failed, or incomplete (Arbeitnow is read two pages deep, so it is always incomplete). The run succeeds only when at least 80% of boards succeed and at least one posting was collected.

The manifest works as the commit point. If the job crashes halfway, the files it already wrote are never read, because no manifest lists them. If S3 itself refuses a write, the whole run stops: a storage problem must not be recorded as a broken job board.

## Step 4 - Check It

After a run, open the manifest in S3 and compare the board counts with the `source-coverage-low` alarm in CloudWatch. 36 of 36 successful boards means full coverage. Arbeitnow is still reported as incomplete because of the page limit, which is expected.

> 📷 *Screenshot to add: a manifest file open in the S3 console, with the board counts visible.*
