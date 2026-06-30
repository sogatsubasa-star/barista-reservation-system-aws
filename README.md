# barista-reservation-system-aws

Serverless Booking System for Barista School on AWS

# Serverless Barista Course Booking System

A secure, cost-optimized, and highly available serverless booking infrastructure built on AWS. This platform serves as a production-ready reservation system for a local specialty coffee training school in Sydney, strictly aligned with the **AWS Well-Architected Framework**.

---

## 🏗️ Architecture Overview

The frontend is completely decoupled and hosted securely at the edge, ensuring sub-millisecond latency and maximum resilience against unauthorized access.

* **Storage**: Amazon S3 (Fully private bucket with all public access blocked)
* **Content Delivery Network (CDN)**: Amazon CloudFront
* **Origin Protection**: Origin Access Control (OAC) with fine-grained S3 bucket policies
* **Edge Security**: AWS WAF (Web Application Firewall) configured in Monitor Mode for L7 vulnerability protection

The backend layer is now fully operational, providing a stateless, event-driven API that scales to zero when idle.

* **API Layer**: Amazon API Gateway (HTTP API) — chosen over REST API for ~30% lower cost and reduced latency at this scale
* **Compute**: AWS Lambda (Python 3.13 runtime) — `createReservation` function handling input validation, ID generation, and persistence
* **Persistence**: Amazon DynamoDB (On-Demand capacity mode) — single-table design with `reservationId` as the partition key
* **Observability**: Amazon CloudWatch Logs — automatic capture of every Lambda invocation, including request payloads and error traces
* **Identity for Compute**: A dedicated IAM execution role with scoped DynamoDB write permissions, eliminating long-lived credentials from application code

---

## 🛡️ Architectural Highlights & AWS Well-Architected Alignment

### 1. Security (Security Pillar)

* **Zero Public S3 Exposure**: The S3 bucket holding the static web assets is 100% private. No direct public internet traffic can reach the bucket.
* **Origin Access Control (OAC)**: Enforced a strict bucket policy restricting S3 read access (`s3:GetObject`) exclusively to our specific CloudFront distribution utilizing a conditioned `AWS:SourceArn`.
* **Identity Governance**: Adhered to the Principle of Least Privilege by completely restricting the AWS Root User and managing all infrastructure configurations through AWS IAM Identity Center with multi-factor authentication (MFA) enabled.
* **Audit & Compliance**: Configured a dedicated, multi-region AWS CloudTrail to ensure complete visibility and immutable logging of all administrative API operations.
* **No Embedded Credentials**: Lambda accesses DynamoDB through its assumed IAM role; STS issues short-lived credentials per invocation, eliminating any long-lived access keys from application code.

### 2. Cost Optimization & Sustainability (Cost Pillar)

* **Zero-Idle Infrastructure Cost**: Utilized a completely serverless stack across both frontend and backend. The infrastructure scales down to absolute zero when no trainees are visiting the site, dropping idle operation costs to virtually $0.
* **Unified Resource Tagging**: Enforced a strict tagging strategy (`Project: barista-reservation`) across all active resources to track, analyze, and budget project-specific expenses accurately inside AWS Cost Explorer.
* **Proactive Billing Guardrails**: Deployed AWS Budgets with an aggressive $1 actual cost threshold to prevent accidental cost overruns during development.
* **Right-Sized Service Selection**: Chose HTTP API over REST API, SSE-S3 over SSE-KMS, and Parameter Store over Secrets Manager where the additional features were unnecessary at this scale — each decision documented with explicit rationale rather than defaulting to the most feature-rich option.

---

## ⚙️ Phase 3 — Backend Implementation (Completed)

The backend pipeline implements a **fully decoupled, event-driven request flow**, demonstrating production-grade design patterns at every layer.

### Request Flow

```
Client → API Gateway (POST /reservations)
       → Lambda (validate → generate UUID → write)
       → DynamoDB (Reservations table)
       → JSON response with reservationId
```

### Key Engineering Decisions

| Decision | Choice | Rationale |
|---|---|---|
| API type | HTTP API | Lower cost, lower latency, sufficient feature set for this use case. REST API's advanced features (request validators, caching, API keys) were unnecessary at this stage. |
| DynamoDB capacity | On-Demand | Booking traffic is unpredictable and bursty (marketing campaigns, holiday surges). On-Demand scales seamlessly and avoids over-provisioning. |
| Lambda runtime | Python 3.13 | Native `boto3` SDK availability, fast cold starts, and SAA-aligned best practices. |
| Schema design | NoSQL single-table | DynamoDB's strength in single-item lookups by `reservationId`; secondary indexes can be added incrementally as access patterns emerge. |
| Error handling | Try/except with structured JSON responses | API Gateway and clients receive consistent `statusCode + body` shapes, never raw stack traces. |

### Security Posture for Phase 3

* **Least-Privilege IAM**: The Lambda execution role grants only the specific DynamoDB actions required (`PutItem` on the `Reservations` table). No wildcard resource ARNs in production-bound versions.
* **CORS Configuration**: Explicit `Access-Control-Allow-Origin` headers set at the Lambda response layer, enabling controlled cross-origin requests from the CloudFront-hosted frontend without exposing the API to arbitrary domains in future hardening passes.
* **Input Validation at the Edge of Compute**: Required fields are validated inside the Lambda handler, returning `400 Bad Request` for malformed payloads rather than allowing malformed writes to reach DynamoDB.

### Verification

The full pipeline was verified end-to-end via `curl` invocation against the deployed API Gateway endpoint, confirming successful item creation in DynamoDB and a structured success response from Lambda.

```bash
$ curl -X POST https://[API_ID].execute-api.ap-southeast-2.amazonaws.com/reservations \
       -H "Content-Type: application/json" \
       -d '{"courseId":"special","datetime":"2026-08-15T18:00:00","name":"Yamada","email":"yamada@example.com"}'

{"message": "Reservation created successfully", "reservationId": "resv-62d7e1c8"}
```

---

## 🚀 Roadmap & Project Status

### ✅ Completed Phases

* **Phase 0 — Foundations**: Account hardening (root MFA, IAM Identity Center), AWS Budgets guardrails, multi-region CloudTrail audit logging.
* **Phase 1 — Architecture Design**: Requirements definition, NoSQL schema design, serverless architecture diagram.
* **Phase 2 — Frontend Delivery**: Private S3 bucket, CloudFront distribution with OAC, WAF in Monitor Mode.
* **Phase 3 — Reservation Backend**: API Gateway (HTTP API) + Lambda (Python 3.13) + DynamoDB. End-to-end live and verified.

### 🔜 In Progress / Next

* **Phase 4 — Notifications & Identity**: Integrating Amazon SES for automated booking confirmation emails, and Amazon Cognito for an administrative dashboard with MFA-protected access.
* **Phase 5 — Third-Party Integrations**: Implementing secure deposit workflows via Stripe API, syncing confirmed bookings into Google Calendar for the operating team, and integrating LINE Messaging API for conversational booking (with HMAC-SHA256 signature verification via Parameter Store SecureString).
* **Phase 6 — Infrastructure as Code**: Migrating the entire manual architecture into a declarative AWS CDK codebase, demonstrating environment reproducibility and DevSecOps best practices.
* **Phase 7 — Hardening for Production**: Tightening Lambda execution role to scoped DynamoDB actions only, restricting CORS to the production CloudFront domain, enabling WAF in Block Mode, and adding API Gateway throttling and request validators.

---

## 🛠️ Tech Stack

**AWS**: S3, CloudFront (with OAC), WAF, API Gateway (HTTP API), Lambda, DynamoDB, IAM Identity Center, CloudTrail, AWS Budgets, Parameter Store, CloudWatch Logs

**Languages & Tools**: Python 3.13, HTML / CSS / JavaScript, Git

**Planned Integrations**: SES, Cognito, Stripe, Google Calendar API, LINE Messaging API, AWS CDK

---

## 👤 About

Built and maintained by **Sebastian** — a cybersecurity student at university in Sydney (graduating August 2026), and the operator of **@sebbarista**, a barista training school serving working-holiday and international students in Sydney.

This project is the intersection of two ongoing efforts: pursuing the **AWS Solutions Architect — Associate** certification, and rebuilding a real business on serverless AWS infrastructure. The goal is to demonstrate not "I have learned AWS," but "I have **built and operate** a real business on AWS."

Open to junior cloud / backend engineering roles in Sydney from August 2026.
