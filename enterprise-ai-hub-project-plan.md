# Enterprise AI Hub – Project Plan
**Project:** Centralized Claude API Gateway (`pr-ai-hub-shared`)
**Architecture:** Option A – Single GCP Project + Apigee API Gateway

---

## Executive Summary

Deploy a centralized GCP project as the enterprise AI hub, exposing Claude (via Vertex AI) to all internal teams through Apigee. Chargebacks will be handled via BigQuery + Looker Studio, eliminating the need for per-team GCP projects.

---

## Billing Strategy: Pay-As-You-Go → Provisioned Throughput

### What is Provisioned Throughput?

Most companies begin with **Pay-As-You-Go (PAYG)** billing — paying per 1,000 tokens consumed from a shared public GPU pool. This is flexible but comes with two risks at scale: unpredictable costs and potential `429 Resource Exhausted` errors during peak demand.

**Provisioned Throughput (PT)** is a committed capacity model where the enterprise purchases a dedicated block of compute (measured in Tokens Per Minute) for a fixed period (typically 1–6 months). Think of it as the difference between hailing an Uber every day versus leasing a dedicated fleet of company cars — the per-trip cost drops significantly when you drive them consistently.

### Why PT Matters for This Architecture

| | Pay-As-You-Go | Provisioned Throughput |
|---|---|---|
| Cost model | Per token consumed | Flat committed rate |
| Capacity | Shared public pool | Dedicated to your project |
| Latency | Variable (can spike) | Consistent and predictable |
| `429` errors | Possible at peak | Eliminated |
| Best for | Early / low usage | Scaled enterprise usage |

**Projected savings: 20–50% reduction in AI spend** once PT is sized correctly against real usage data.

### The 3-Step PT Strategy (Do Not Buy on Day 1)

**Step 1 – Gather Data (Months 1–2)**
Run all teams on PAYG via Apigee. Use the BigQuery + Looker Studio dashboard to identify your **baseline Tokens Per Minute (TPM)** — the floor of usage during business hours. For example: the company rarely drops below 50,000 TPM between 9 AM and 5 PM, but spikes to 150,000 TPM occasionally.

**Step 2 – Buy PT for the Baseline, Not the Peak**
The golden rule of cloud economics: purchase PT to cover your steady baseline (e.g., 50,000 TPM), not your peak. Buying for the peak means paying for 100,000 TPM sitting idle 90% of the day. Contact your Google Cloud Account Manager to negotiate a Committed Use Discount or dedicated backend deployment for Claude on Vertex AI.

**Step 3 – Use Apigee to Shape Traffic at the PT Ceiling**
Configure a global **SpikeArrest policy** in Apigee capped at your PT limit. When traffic exceeds the committed capacity, Apigee queues requests gracefully or returns a friendly message to users: *"Company AI capacity is temporarily full. Please retry in 10 seconds."* This ensures 100% utilization of purchased capacity without hard GCP errors.

### Why Option A Makes PT Viable

With a multi-project setup, each team would need its own PT commitment — Marketing's idle capacity cannot offset Engineering's spikes. By routing all teams through a single GCP project via Apigee, usage from different teams stacks together. Engineering's quiet periods cancel out Marketing's peaks, enabling **one optimised PT purchase** that serves the entire enterprise efficiently.

---

## Goals & Success Criteria

| Goal | Success Metric |
|---|---|
| Single point of access for all teams | All teams onboarded via Apigee API keys |
| Cost optimization via pooled throughput | Provisioned Throughput utilization > 80% |
| Per-team chargeback visibility | Looker Studio dashboard live with token-level billing |
| Noisy neighbor prevention | Apigee quota policies enforced per team |
| Quota simplicity | Single GCP project quota managed centrally |

---

## Phases & Timeline

### Phase 1 – Foundation (Weeks 1–2)

**Objective:** Set up the central GCP project and core infrastructure.

- Create GCP project `pr-ai-hub-shared`
- Enable Vertex AI API and request Claude 3.5 Sonnet access
- Create a dedicated Service Account for Apigee with `Vertex AI User` role
- Request elevated Vertex AI quota (RPM + TPM) via GCP support ticket
- Set up VPC and networking for Apigee → Vertex AI connectivity
- Define and document naming conventions and tagging strategy

**Deliverables:** GCP project live, Claude model accessible, Service Account configured

---

### Phase 2 – Apigee Gateway Setup (Weeks 3–4)

**Objective:** Configure Apigee as the unified gateway for all AI traffic.

- Create Apigee API Proxy pointing to Vertex AI endpoint in `pr-ai-hub-shared`
- Configure a single Target Endpoint using the central Service Account
- Implement API Key authentication for all consuming teams
- Build Apigee Quota Policies per team (e.g., Engineering: 1,000 RPM, Marketing: 100 RPM)
- Set up error handling for `429 Too Many Requests` with informative responses
- Create Apigee developer apps and products per team

**Deliverables:** Apigee proxy live, API keys issued, quota policies enforced

---

### Phase 3 – Observability & Chargeback (Weeks 5–6)

**Objective:** Implement logging, monitoring, and financial chargeback reporting.

- Configure Apigee to extract and log per-request metadata:
  - Team / API Key
  - Model version
  - Input token count
  - Output token count
  - Latency (ms)
  - HTTP status / error codes
  - Timestamp
- Stream logs from Apigee to BigQuery (via Pub/Sub or Apigee MessageLogging policy)
- Build Looker Studio dashboard with:
  - Token usage by team (daily / monthly)
  - Cost attribution per team (down-to-the-penny chargeback)
  - Error rate and latency trends
  - Quota utilization per team
- Share dashboard access with Finance and team leads

**Deliverables:** BigQuery logging live, Looker Studio chargeback dashboard operational

---

### Phase 4 – Team Onboarding (Weeks 7–8)

**Objective:** Migrate all teams from direct or ad-hoc access to the central gateway.

- Publish internal developer documentation (API usage guide, quota limits, error codes)
- Onboard pilot team (Engineering) and validate end-to-end flow
- Onboard remaining teams in batches
- Conduct quota review per team and adjust Apigee policies as needed
- Deprecate any direct Vertex AI access outside the gateway

**Deliverables:** All teams onboarded, old access paths removed

---

### Phase 5 – Cost Optimisation & Hardening (Weeks 9–10)

**Objective:** Transition to Provisioned Throughput and harden the architecture.

- Analyse 8+ weeks of BigQuery traffic data to establish baseline and peak TPM
- Identify the steady-state baseline TPM during business hours (the PT purchase target)
- Engage GCP Account Manager to negotiate Provisioned Throughput / Committed Use Discount for Claude 3.5 Sonnet on `pr-ai-hub-shared`
- Configure Apigee global **SpikeArrest policy** capped at the PT limit to ensure 100% utilization
- Set up user-friendly capacity messages in Apigee for requests exceeding the PT ceiling
- Implement retry logic and fallback policies in Apigee for resilience
- Set up GCP alerting for quota utilization thresholds (>80%)
- Conduct a security review: IAM roles, API key rotation policy, audit logs
- Document runbook for quota increases, PT renegotiation, team offboarding, and incident response

**Deliverables:** Provisioned Throughput active, SpikeArrest policy live, monitoring alerts configured, security review complete

---

## Risks & Mitigations

| Risk | Likelihood | Mitigation |
|---|---|---|
| Noisy neighbor causes quota exhaustion | Medium | Apigee quota policies per team enforced at gateway level |
| GCP quota increase takes time | High | Raise support ticket in Week 1 (can take 5–10 business days) |
| Teams bypass Apigee gateway | Low | Remove direct Vertex AI IAM access from team accounts |
| Provisioned Throughput underutilised | Low | Use 8+ weeks of traffic data before committing; buy for baseline not peak |
| PT purchase timing too early | Medium | Enforce PAYG-only for first 2 months; gate PT negotiation on Looker data |
| Single point of failure at Apigee | Low | Enable Apigee HA and configure retry logic |

---

## Team & Responsibilities

| Role | Responsibility |
|---|---|
| Cloud Architect | GCP project setup, Vertex AI, IAM, networking |
| Apigee Engineer | Proxy config, quota policies, API key management |
| Data Engineer | BigQuery logging pipeline, schema design |
| BI / Analytics | Looker Studio dashboard, chargeback reports |
| Finance | Chargeback model validation, cost sign-off |
| Team Leads | Onboarding, quota requirement inputs |

---

## Key Decisions Log

| Decision | Rationale |
|---|---|
| Option A (Single Project) over Option B | Pooled throughput, simpler Apigee routing, single quota to manage |
| Apigee for quota enforcement | Prevents noisy neighbor without needing project isolation |
| BigQuery + Looker for chargebacks | Eliminates need for per-project GCP billing separation |
| Provisioned Throughput (deferred to Phase 5) | Need 8 weeks of real traffic data to size baseline correctly; buying early wastes spend |
| Buy PT for baseline TPM, not peak | Buying for peak leaves capacity idle 90% of the day; Apigee SpikeArrest handles overflow |

---

## Summary Timeline

| Week | Milestone |
|---|---|
| 1–2 | GCP project and Vertex AI foundation live |
| 3–4 | Apigee gateway with quota policies operational |
| 5–6 | BigQuery logging and Looker chargeback dashboard live |
| 7–8 | All teams onboarded via Apigee |
| 9–10 | Provisioned Throughput active, architecture hardened |
