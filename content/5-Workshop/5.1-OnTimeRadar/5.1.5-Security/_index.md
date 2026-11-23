---
title: "Dashboard, Login, and Security"
date: 2026-11-23
weight: 5
chapter: false
pre: " <b> 5.1.5. </b> "
---

# Dashboard, Login, and Security

## The Website

The dashboard is plain HTML, CSS, and JavaScript (about 90 KB, no build step), with Chart.js for charts. It has a light and a dark theme and also works on a phone. There are three pages:

| Page | Who sees it | What it shows |
|---|---|---|
| SLA overview | manager | on-time rate, late rate per week, per carrier, and the regions with the most late deliveries |
| Shipment lookup | customer service + manager | current status and the full timeline of one shipment |
| Needs attention | customer service + manager | late, failed-attempt, and overdue shipments, filtered by carrier and region |

![SLA overview page: on-time rate, weekly late-rate chart, late rate by carrier and by region](../../../images/5-Workshop/5.1-OnTimeRadar/overview.jpg)

*SLA overview, 19 November 2026. The signed-in email is replaced with an example address.*

![Shipment lookup in dark mode: current status and a timeline with a failed attempt and a late delivery](../../../images/5-Workshop/5.1-OnTimeRadar/lookup.jpg)

*Looking up one shipment. Blocked events are shown with the reason they did not change the status.*

![Needs attention page listing shipments that had a failed delivery attempt](../../../images/5-Workshop/5.1-OnTimeRadar/attention.jpg)

*The "needs attention" list. Clicking a row opens that shipment's timeline.*

## Login and Roles

Staff sign in through the Cognito hosted page, using the authorization code flow with PKCE, so the browser holds no client secret. The token is kept only in the browser tab (`sessionStorage`) and expires after one hour. Multi-factor authentication is optional, and each person can turn it on.

| Request | No token | Token, no group | `cskh` | `manager` |
|---|---|---|---|---|
| `GET /shipments...` | 401 | 403 | ✅ | ✅ |
| `GET /sla/summary`, `GET /sla/trend` | 401 | 403 | 403 | ✅ |

The checks happen in two layers:

1. The API Gateway JWT authorizer stops a missing, invalid, or expired token with **401**, before any Lambda runs.
2. API Gateway cannot check Cognito groups, so the Lambda does it and returns **403** for the wrong role. A unit test fails if any `GET` route except `/health` is left without a role check.

Carrier webhooks do not use Cognito. They must send the `X-Webhook-Secret` header.

## Protecting the Website

- The web files sit in their own private S3 bucket, separate from the order data. Only CloudFront can read it, through Origin Access Control.
- CloudFront forces HTTPS and adds security headers: Content-Security-Policy with `frame-ancestors 'none'`, HSTS, `X-Frame-Options`, `X-Content-Type-Options: nosniff`, and `Referrer-Policy: no-referrer`.
- Chart.js is loaded with Subresource Integrity, and every value from the API is escaped before it is shown.
- The API's CORS setting and Cognito's callback URLs accept only the CloudFront domain and `http://localhost:5500` for local testing.

## Protecting the API and the Account

- **Throttling:** each route accepts 10 requests per second, with bursts of 20. Extra requests get 429 at the gateway, so a flood cannot use up all 10 concurrent Lambdas that a new account has. The replayer retries 429 safely, because the same payload produces the same `eventId`.
- **Input validation:** tracking codes, query parameters, and payload shapes are all checked, and bad input returns 400.
- **Least privilege:** every Lambda role gets only the actions it needs on this stack's resources.
- **Permissions boundary:** every role in the stack carries the boundary `shipment-sla-boundary`. The deploy role may create or change a role only if that boundary is attached, and it is explicitly denied removing or editing the boundary. Even a stolen deploy token could not create an administrator role.
- **No long-lived keys:** GitHub Actions gets a temporary role through OIDC. The role trusts only this repository, matched by GitHub's fixed numeric owner and repository IDs, and only the approved `production` environment.

> 📷 *Screenshots to add: the response headers in the browser developer tools, and the web bucket's "Block all public access" setting.*
