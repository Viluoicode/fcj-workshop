---
title: "Project 1 - Job Market AWS Pipeline"
date: 2026-11-12
weight: 2
chapter: false
pre: " <b> 5.2. </b> "
---

# Job Market AWS Pipeline

## Overview

Job Market AWS Pipeline is an AWS data-engineering workshop that collects job postings from public APIs, preserves traceable observations in a data lake, transforms them through Bronze, Silver, and Gold layers, and exposes the published results through Amazon Athena and a local Streamlit dashboard.

The project addresses a practical problem: public job platforms use different schemas, sources can fail or return incomplete results, and a posting may change or disappear over time. A successful download script alone cannot prove that the resulting data is current, complete, deduplicated, or safe to publish.

## Project History

A first prototype was started in late June 2026, before the internship began. During the internship (Weeks 13-24), it was re-scoped, redesigned, hardened, and deployed on AWS with Terraform, Data Quality gates, orchestration, monitoring, and tests.

## Project Goals

- Collect a configured sample of job postings every day without depending on a personal computer.
- Preserve source observations and a committed manifest for traceability.
- Normalize data and track posting lifecycle in the Silver layer.
- Enforce data-quality checks before advancing published state.
- Produce queryable Gold data products for job, role, company, location, and remote analysis.
- Automate the workflow and prevent overlapping writers.
- Monitor freshness and source coverage independently of the main pipeline.
- Provide a dashboard that refuses stale, incomplete, or in-progress publication.

## Scope

The configured sample contains 36 boards across Greenhouse, Lever, Ashby, and Arbeitnow. The system is designed as an educational, portfolio-scale batch pipeline. It does not claim to represent the whole job market, recommend jobs to an individual, or determine whether every posting is still available on the employer website.

## Main AWS Services

- Amazon S3
- AWS Glue Python Shell and AWS Glue Spark
- AWS Step Functions
- Amazon EventBridge
- Amazon DynamoDB
- AWS Glue Data Catalog (Gold tables in Apache Iceberg format)
- Amazon Athena
- AWS Lambda
- Amazon CloudWatch
- Amazon SNS
- AWS IAM

## AI-Assisted Development

AI tools supported research, implementation suggestions, code review, testing ideas, and documentation. I remained responsible for defining the project requirements, reviewing architectural decisions, validating the proposed code, checking AWS behavior, troubleshooting failures, testing the pipeline, and documenting verified results.

## Workshop Sections

1. Architecture and design decisions
2. Prerequisites and environment preparation
3. Multi-source ingestion and Bronze storage
4. Silver lifecycle, Data Quality, and Gold marts
5. Orchestration, security, and monitoring
6. Dashboard, testing, and verified results
7. Cost, cleanup, limitations, and reflection
8. Troubleshooting and future work

## Current Evidence

From 15 September to 21 November 2026, all 68 scheduled nightly runs succeeded. The snapshot of 21 November held 8,672 open postings from 467 employers, collected from 36 of 36 boards. These numbers describe a past point in time; section 5.2.6 explains how to check the pipeline's current state.

