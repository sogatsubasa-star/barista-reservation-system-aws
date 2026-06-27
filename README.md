# barista-reservation-system-aws
Serverless Booking System for Barista School on AWS
# Serverless Barista Course Booking System

A secure, cost-optimized, and highly available serverless booking infrastructure built on AWS. This platform serves as a production-ready reservation system for a local specialty coffee training school in Sydney, strictly aligned with the **AWS Well-Architected Framework**.

## 🏗️ Architecture Overview (Current Milestone)
The frontend is completely decoupled and hosted securely at the edge, ensuring sub-millisecond latency and maximum resilience against unauthorized access.

* **Storage**: Amazon S3 (Fully private bucket with all public access blocked)
* **Content Delivery Network (CDN)**: Amazon CloudFront
* **Origin Protection**: Origin Access Control (OAC) with fine-grained S3 bucket policies
* **Edge Security**: AWS WAF (Web Application Firewall) configured in Monitor Mode for L7 vulnerability protection

---

## 🛡️ Architectural Highlights & AWS Well-Architected Alignment

### 1. Security (Security Pillar)
* **Zero Public S3 Exposure**: The S3 bucket holding the static web assets is 100% private. No direct public internet traffic can reach the bucket.
* **Origin Access Control (OAC)**: Enforced a strict bucket policy restricting S3 read access (`s3:GetObject`) exclusively to our specific CloudFront distribution utilizing a conditioned `AWS:SourceArn`.
* **Identity Governance**: Adhered to the Principle of Least Privilege by completely restricting the AWS Root User and managing all infrastructure configurations through AWS IAM Identity Center with multi-factor authentication (MFA) enabled.
* **Audit & Compliance**: Configured a dedicated, multi-region AWS CloudTrail to ensure complete visibility and immutable logging of all administrative API operations.

### 2. Cost Optimization & Sustainability (Cost Pillar)
* **Zero-Idle Infrastructure Cost**: Utilized a completely serverless frontend stack. The infrastructure scales down to absolute zero when no trainees are visiting the site, dropping idle operation costs to virtually $0.
* **Unified Resource Tagging**: Enforced a strict tagging strategy (`Project: barista-reservation`) across all active resources to track, analyze, and budget project-specific expenses accurately inside AWS Cost Explorer.
* **Proactive Billing Guardrails**: Deployed AWS Budgets with an aggressive $1 actual cost threshold to prevent accidental cost overruns during development.

---

## 🚀 Upcoming Roadmap
* **Phase 3 (Backend)**: Integrating Amazon API Gateway, AWS Lambda, and Amazon DynamoDB to process live appointment slots and coordinate availability check queries.
* **Phase 4 (Third-Party Integrations)**: Implementing secure payment deposit workflows via Stripe API and syncing final bookings directly to the operational calendar using Google Calendar API integrations.
* **Phase 5 (Infrastructure as Code)**: Migrating this entire manual architecture into a declarative codebase using the AWS CDK (Cloud Development Kit) to demonstrate environment reproducibility and DevSecOps best practices.
