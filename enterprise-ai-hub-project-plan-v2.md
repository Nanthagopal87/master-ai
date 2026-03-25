# Enterprise AI Hub – Project Plan
**Project:** Centralized LLM Gateway (`pr-ai-hub-shared`)
**Architecture Pattern:** AI Gateway over a Single Control Plane (GCP) with Apigee as the Intelligent Ingress Layer
**Model:** Claude 3.5 Sonnet on Vertex AI (Managed Inference Endpoint)

---

## Executive Summary

Deploy a centralized **AI Gateway** on GCP that serves as the unified **LLM Inference Plane** for all enterprise teams. Apigee acts as the **AI Traffic Controller** — handling AuthN/AuthZ, rate limiting, traffic shaping, and observability — sitting in front of a single managed **Model Serving Endpoint** (Vertex AI). All AI consumption is routed through this single **North-South traffic** path, with token-level telemetry streamed to a **Data Lakehouse** (BigQuery) for financial chargebacks via Looker Studio. This eliminates per-team model deployments and consolidates capacity under one **AI Consumption Layer**.

---

## Billing Strategy: Pay-As-You-Go → Provisioned Throughput

### What is Provisioned Throughput?

Most enterprises begin on **Pay-As-You-Go (PAYG)** — a shared, multi-tenant inference pool billed per token. At low utilization this is cost-effective, but at enterprise scale it introduces two failure modes: **non-deterministic latency** from GPU pool contention, and hard **backpressure errors** (`429 Resource Exhausted`) during demand spikes.

**Provisioned Throughput (PT)** is a **dedicated inference capacity** commitment — a reserved slice of the model serving infrastructure measured in **Tokens Per Minute (TPM)**. It shifts billing from a variable operational expense to a predictable flat-rate model, delivering 20–50% cost reduction at scale with guaranteed **Service Level Objectives (SLOs)** for latency and availability.

### PT vs PAYG — Architecture Comparison

| Dimension | Pay-As-You-Go | Provisioned Throughput |
|---|---|---|
| Tenancy model | Shared multi-tenant GPU pool | Dedicated single-tenant capacity |
| Cost structure | Variable OpEx (per token) | Flat committed rate |
| Latency profile | Non-deterministic (contention-prone) | Deterministic, SLO-backed |
| Backpressure behaviour | Hard `429` errors at quota ceiling | Eliminated within committed TPM |
| Capacity governance | GCP-managed, shared | Enterprise-owned, reserved |
| Best fit | Proof-of-concept / low-volume workloads | Production enterprise AI Hub |

### The 3-Step PT Adoption Strategy

**Step 1 – Baseline Profiling (Months 1–2)**
Run all teams on PAYG through the AI Gateway. Use the **Observability Pipeline** (Apigee → Pub/Sub → BigQuery) to profile traffic patterns in Looker Studio. Identify the **steady-state TPM floor** during business hours — the minimum sustained inference throughput across the enterprise. For example: throughput rarely drops below 50,000 TPM between 9 AM–5 PM, with burst spikes to 150,000 TPM.

**Step 2 – Right-Size the PT Commitment (Baseline, Not Peak)**
The core principle of **FinOps for AI**: purchase PT to cover the **steady-state baseline** (e.g., 50,000 TPM), not the burst ceiling. Buying for peak leaves the majority of reserved capacity idle — wasted spend. Engage the GCP Account Manager to negotiate a **Committed Use Discount (CUD)** or a dedicated backend deployment for Claude on Vertex AI against the `pr-ai-hub-shared` control plane.

**Step 3 – Traffic Shaping at the AI Gateway Layer**
Configure a global **SpikeArrest policy** in Apigee set at the PT ceiling. Burst traffic beyond committed capacity is either gracefully queued or returned with a structured back-pressure response: *"Enterprise inference capacity is temporarily saturated. Please retry in 10 seconds."* This ensures **100% PT utilization** — no wasted reserved capacity, no hard model-layer failures.

### Why a Single Control Plane Unlocks PT Efficiency

In a multi-project **siloed architecture**, each business unit requires its own PT commitment. Idle capacity in one silo cannot absorb demand from another — there is no **statistical multiplexing** across workloads. By routing all teams through a single **AI Gateway** into one unified **Inference Plane**, traffic from diverse teams with different usage profiles naturally smooths. Engineering's off-peak hours offset Marketing's peak burst. The enterprise buys **one right-sized PT commitment** at the highest possible utilization efficiency.

---

## Goals & Success Criteria

| Goal | Success Metric |
|---|---|
| Unified AI Gateway as single ingress | All teams consuming Claude exclusively through the Apigee AI Gateway |
| Optimised inference capacity | Provisioned Throughput utilization > 80% post-PT adoption |
| Token-level financial observability | Looker Studio chargeback dashboard live with per-team cost attribution |
| Workload isolation without siloing | Apigee per-consumer Quota Policies preventing noisy-neighbor saturation |
| Centralized quota governance | Single Vertex AI control plane with one quota ceiling to manage |
| Deterministic latency at scale | P95 inference latency within SLO post-PT activation |

---

## Architecture Overview

```
Consumer Layer                AI Gateway Layer              Inference Plane
─────────────────     ───────────────────────────     ──────────────────────────
 Engineering CLI  ──►                               
 Marketing Tools  ──►  Apigee (AI Traffic Control)  ──►  Vertex AI (Claude 3.5)
 HR Applications  ──►    • AuthN / AuthZ                  pr-ai-hub-shared
 Finance Apps     ──►    • Per-consumer Quota Policy  
                         • SpikeArrest (PT Ceiling)   
                         • Token Logging               
                              │                       
                         Observability Pipeline        
                         Pub/Sub → BigQuery → Looker  
```

**Key Architectural Principles:**
- **Single Ingress Point:** All North-South AI traffic flows through one Apigee proxy
- **Policy-as-Code:** Quota, AuthN, and traffic shaping enforced declaratively at the gateway
- **Data Plane Separation:** Observability pipeline is decoupled from the inference path
- **Zero-Trust Consumer Access:** No team has direct IAM access to the Vertex AI endpoint

---

## Phases & Timeline

### Phase 1 – Control Plane Foundation (Weeks 1–2)

**Objective:** Provision the centralized GCP control plane and establish the core inference endpoint.

- Provision GCP project `pr-ai-hub-shared` as the **AI Hub Control Plane**
- Enable Vertex AI API and onboard Claude 3.5 Sonnet as the **Managed Inference Endpoint**
- Create a dedicated **Service Account** for Apigee with scoped `Vertex AI User` IAM role (least-privilege)
- Submit GCP support ticket to increase Vertex AI **RPM and TPM quota** on the control plane project
- Configure **VPC Service Controls** and private networking for Apigee → Vertex AI traffic (no public egress)
- Define resource naming conventions, label taxonomy, and tagging strategy for cost attribution

**Deliverables:** GCP control plane live, Claude inference endpoint accessible, Service Account provisioned

---

### Phase 2 – AI Gateway Configuration (Weeks 3–4)

**Objective:** Deploy and configure Apigee as the enterprise **AI Gateway** with per-consumer traffic policies.

- Deploy **Apigee API Proxy** with a single **Target Endpoint** pointing to the Vertex AI inference URL in `pr-ai-hub-shared`
- Implement **API Key-based AuthN** as the consumer identity mechanism at the gateway layer
- Configure per-consumer **Quota Policies** for workload isolation (e.g., Engineering: 1,000 RPM; Marketing: 100 RPM)
- Add global **SpikeArrest Policy** as the PAYG-phase safety valve to prevent control plane saturation
- Build structured **error response contracts** for `429` backpressure events with retry guidance
- Create **Apigee API Products** and **Developer Apps** as the consumer onboarding abstraction

**Deliverables:** AI Gateway live, consumer API keys issued, per-team quota enforcement active

---

### Phase 3 – Observability Pipeline & FinOps Layer (Weeks 5–6)

**Objective:** Build the end-to-end **AI Observability Pipeline** and financial chargeback reporting layer.

- Instrument Apigee **MessageLogging policy** to capture per-request telemetry:
  - Consumer identity (Team / API Key)
  - Model version (inference endpoint metadata)
  - Input token count / Output token count
  - End-to-end latency (ms) — gateway-to-model round trip
  - HTTP status codes and error classifications
  - Request timestamp (UTC)
- Build **Streaming Ingestion Pipeline**: Apigee → Cloud Pub/Sub → BigQuery (event-driven, low-latency)
- Design BigQuery schema as the **AI Token Ledger** — the single source of truth for all inference consumption
- Build **Looker Studio FinOps Dashboard** with:
  - Token burn rate by team (daily / monthly views)
  - Per-team cost attribution (chargeback down to the request level)
  - P50 / P95 latency trends and error rate by consumer
  - Quota headroom and utilization heatmaps per team
  - TPM baseline profiling charts (inputs for PT sizing)
- Grant Finance and team leads **read-only dashboard access**

**Deliverables:** Observability pipeline live, AI Token Ledger in BigQuery, FinOps chargeback dashboard operational

---

### Phase 4 – Consumer Onboarding & Access Migration (Weeks 7–8)

**Objective:** Migrate all internal consumers to the **AI Gateway** and decommission any direct model access paths.

- Publish **Internal AI Developer Portal** documentation: gateway URL, API key provisioning, quota limits, error contracts, retry guidance
- Onboard pilot consumer (Engineering team) — validate full data plane flow end-to-end
- Onboard remaining consumer teams in batches with dedicated support window
- Review per-team quota headroom post-onboarding and tune **Apigee Quota Policies** accordingly
- **Decommission direct Vertex AI access**: revoke team-level IAM bindings to the inference endpoint — all traffic must flow through the AI Gateway
- Validate zero direct-access paths via GCP **Audit Log** review

**Deliverables:** All consumers onboarded via AI Gateway, direct inference access paths closed, audit validation complete

---

### Phase 5 – FinOps Optimisation & Platform Hardening (Weeks 9–10)

**Objective:** Activate **Provisioned Throughput**, enforce PT-aligned traffic shaping, and harden the platform for production SLOs.

- Analyse 8+ weeks of **AI Token Ledger** data to establish steady-state baseline TPM and identify burst profiles
- Present TPM baseline analysis to Finance — validate PT sizing and commitment period
- Engage GCP Account Manager to negotiate **Committed Use Discount (CUD)** or dedicated backend for Claude 3.5 Sonnet on `pr-ai-hub-shared`
- Reconfigure Apigee global **SpikeArrest Policy** to the PT-committed TPM ceiling — ensuring 100% capacity utilization
- Implement **graceful degradation responses** at the gateway for burst traffic exceeding PT ceiling
- Configure **Cloud Monitoring alerting** on Vertex AI quota utilization thresholds (warn at >75%, critical at >90%)
- Conduct **Zero-Trust Security Review**: IAM bindings, API key rotation policy, VPC Service Controls, audit log coverage
- Author and publish **Platform Runbook**: PT renegotiation process, consumer quota changes, incident response, offboarding procedures

**Deliverables:** Provisioned Throughput active, PT-aligned SpikeArrest live, monitoring alerts configured, platform runbook published

---

## Risks & Mitigations

| Risk | Likelihood | Mitigation |
|---|---|---|
| Noisy-neighbor saturates shared quota | Medium | Per-consumer Quota Policies at the AI Gateway enforce hard workload isolation |
| Vertex AI quota increase SLA | High | Raise GCP support ticket in Week 1 — quota approvals can take 5–10 business days |
| Consumer teams bypass the AI Gateway | Low | Revoke direct Vertex AI IAM access; enforce via VPC Service Controls |
| PT commitment undersized or oversized | Low | Gate PT negotiation on 8+ weeks of baseline TPM data from the AI Token Ledger |
| PT purchased too early (insufficient data) | Medium | Enforce PAYG-only for Months 1–2; PT sizing is a Phase 5 gate |
| AI Gateway becomes single point of failure | Low | Enable Apigee HA deployment; configure retry policies and circuit breakers |
| Data plane latency introduced by gateway | Low | Benchmark Apigee proxy overhead in Phase 2; optimize policy execution order |

---

## Team & Responsibilities

| Role | Responsibility |
|---|---|
| Cloud / AI Architect | Control plane design, Vertex AI inference endpoint, IAM, VPC Service Controls |
| Apigee Engineer | AI Gateway proxy config, Quota Policies, SpikeArrest, API Product management |
| Data Engineer | Observability pipeline (Pub/Sub → BigQuery), AI Token Ledger schema |
| BI / Analytics Engineer | Looker Studio FinOps Dashboard, chargeback models, TPM baseline reporting |
| FinOps / Finance | PT commitment sizing sign-off, chargeback model validation, CUD negotiation |
| Platform / DevOps | Cloud Monitoring alerting, runbook authoring, incident response |
| Team Leads | Consumer onboarding, quota requirement inputs, pilot validation |

---

## Key Architectural Decisions

| Decision | Rationale |
|---|---|
| Single Control Plane (Option A) over siloed per-team projects | Enables statistical multiplexing, single quota governance, and simplified AI Gateway routing |
| Apigee as the AI Gateway / Intelligent Ingress Layer | Enforces workload isolation, AuthN, and traffic shaping without model-layer complexity |
| BigQuery as the AI Token Ledger | Decouples financial observability from GCP project-level billing; enables per-request chargeback |
| PT deferred to Phase 5 | Right-sizing requires 8 weeks of real baseline TPM data from the observability pipeline |
| Buy PT for steady-state baseline, not burst ceiling | Peak-sized PT leaves capacity idle 90% of the day; Apigee SpikeArrest absorbs burst safely |
| Zero-Trust consumer access model | No team has direct IAM access to the inference endpoint — all traffic is gateway-mediated |
| VPC Service Controls on inference endpoint | Prevents data exfiltration and enforces network perimeter around the AI control plane |

---

## Summary Timeline

| Week | Milestone |
|---|---|
| 1–2 | AI Hub Control Plane provisioned; Vertex AI inference endpoint live |
| 3–4 | AI Gateway (Apigee) deployed with per-consumer Quota Policies and SpikeArrest |
| 5–6 | Observability Pipeline live; AI Token Ledger and FinOps Dashboard operational |
| 7–8 | All consumers onboarded; direct inference access paths decommissioned |
| 9–10 | Provisioned Throughput active; platform hardened for production SLOs |
