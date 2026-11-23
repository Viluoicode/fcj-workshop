---
title: "Silver, Data Quality, and Gold"
date: 2026-11-23
weight: 4
chapter: false
pre: " <b> 5.2.4. </b> "
---

# Silver, Data Quality, and Gold

## Bronze to Silver

The Silver job reads only the files named in this run's manifest. Before it starts, it checks that the manifest belongs to the same run and UTC date, that the run succeeded, and that the manifest is recent.

### Giving Each Posting an Identity

- `job_id` is a SHA-256 hash of platform, board, and the platform's job ID. The board is part of the key because two employers on the same platform can reuse the same internal ID.
- `dedup_hash` is built from the normalized company, title, and location. Gold uses it to spot the same job posted on two platforms.

### Deciding Whether a Posting Is Still Open

The job compares today's observations with the previous Silver run:

| What happened today | What Silver records |
|---|---|
| The posting was seen | Keep the first-seen time, update last-seen, mark active |
| The board answered completely and the posting was missing | Closed: missing from a complete board |
| The board failed or was cut short | Keep the last observation: the board could not be verified |
| Not seen for seven days | Closed: the observation is too old |
| A closed posting appears again | Reopen it and keep the original first-seen time |

The key rule is that a broken board never closes anything. If Lever is down for a day, its postings keep their last observation, instead of looking like a wave of closures.

### Data Quality Gate

Before Silver is published, Glue Data Quality checks seven rules:

1. There is at least one row.
2. Every row has a `job_id`.
3. No `job_id` appears twice.
4. Every row has a `dedup_hash`.
5. At least 30% of rows have a known work mode. The threshold is low because many boards do not state it.
6. The platform is one of the four supported ones.
7. At least 90% of rows have a company name.

If any rule fails, the run stops and the previous Silver stays the published one. Only after the rules pass does the job write the new run and move `state/silver/latest.json` to point at it.

## Silver to Gold

Gold keeps postings that are active and were seen within the freshness window. Three tables are built:

| Table | One row per | Used for |
|---|---|---|
| `fact_job_posting` | unique open posting | browsing jobs, companies, locations, work mode, and apply links |
| `demand_by_role` | role family per day | comparing demand between roles |
| `role_opportunity` | role family per day | demand rank, remote share, and the top employer for each role |

When the same job appears on two platforms, Gold keeps the most recently seen copy, then the one with a posting date, then the lowest `job_id`, so the choice is always the same.

### Role Families

Each title is matched against 15 role families in a fixed priority order, for example Forward Deployed / Solutions, ML / AI Engineer, Data Engineer, DevOps / SRE, Backend Engineer, and finally Software Engineer (General). A posting gets at most one family, so the counts add up. Titles such as recruiter or account executive are excluded on purpose. On 20 November about 60% of postings matched no family, mostly sales, legal, and hardware jobs. They stay in the totals but not in the role charts.

### Writing Gold as Iceberg

Each table is an Apache Iceberg table under `gold_iceberg/` in the lake bucket. A run replaces exactly one `snapshot_date` per table in a single commit, even when the result is empty, so a reader sees either yesterday's version or today's, never a half-written day.

The job then writes a small record with the new snapshot ID of each table. Step Functions checks that this record belongs to the current run and copies the three IDs into the completion marker. Readers query `FOR VERSION AS OF <snapshot id>`, so all three tables are read at the same point in time.

Before switching, I ran Iceberg in "shadow" mode next to the old Parquet tables for several days and compared them row by row (see 5.2.8, problem 4). The old Parquet tables are kept, read-only, as history.

> 📷 *Screenshots to add: the Glue Data Quality result for a run, and an Athena query using `FOR VERSION AS OF`.*
