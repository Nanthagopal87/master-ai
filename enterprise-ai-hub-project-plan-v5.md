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

> ⚠️ **API Key Limitation — Request Count vs Token Count**
> GCP API Keys (and Apigee Quota Policies backed by API Keys) can only enforce and track **Requests Per Minute (RPM)** — the number of API calls made. They have **no visibility into token consumption** (input or output tokens) per request. Two requests from the same team could consume vastly different token volumes depending on prompt length and response size. For accurate **token-level chargeback and FinOps attribution**, token counts must be extracted directly from the **Vertex AI response payload** by the Apigee **MessageLogging policy** and streamed to the **AI Token Ledger** (BigQuery). The API Key serves as the consumer identity signal only — all financial metering happens at the observability layer, not the quota layer.

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
| API Key used for identity, not token metering | GCP API Keys track RPM (request count) only — token-level metering requires Apigee MessageLogging to extract counts from the Vertex AI response payload |
| PT deferred to Phase 5 | Right-sizing requires 8 weeks of real baseline TPM data from the observability pipeline |
| Buy PT for steady-state baseline, not burst ceiling | Peak-sized PT leaves capacity idle 90% of the day; Apigee SpikeArrest absorbs burst safely |
| Zero-Trust consumer access model | No team has direct IAM access to the inference endpoint — all traffic is gateway-mediated |
| VPC Service Controls on inference endpoint | Prevents data exfiltration and enforces network perimeter around the AI control plane |


---

## Security

Security is a first-class concern across every layer of the AI Hub. The architecture follows a **Zero-Trust, Defence-in-Depth** model — no team or service is implicitly trusted, every access path is explicitly authorized, and all activity is logged for audit.

---

### 1. Zero-Trust Consumer Access Model

No internal team or developer has direct IAM access to the **Managed Inference Endpoint** (Vertex AI). All inference traffic must flow through the **AI Gateway (Apigee)** — the single authorized ingress path.

- Team-level IAM bindings to `pr-ai-hub-shared` Vertex AI are explicitly revoked after onboarding
- The only principal with `Vertex AI User` role is the **Apigee Service Account** — scoped under least-privilege
- Validated via GCP **Audit Log** review at the end of Phase 4

> This means even if a developer's laptop or credentials are compromised, they cannot invoke the inference endpoint directly — the API Key and gateway policy chain remain the mandatory control path.

---

### 2. API Key Management & Consumer Identity

API Keys issued via **Apigee Developer Apps** serve as the consumer identity mechanism at the gateway layer.

- Each team receives a unique API Key tied to their **Apigee Developer App**
- Keys are scoped to specific **Apigee API Products** — a team's key cannot be used to impersonate another team
- API Keys should be rotated on a defined schedule (recommended: every 90 days) or immediately upon suspected compromise
- Key revocation is instant at the Apigee layer — no model-layer changes required

> ⚠️ **Important:** API Keys track **request count (RPM) only** — they carry no token-level metering capability. Token consumption is metered separately via the **Apigee MessageLogging policy** extracting counts from the Vertex AI response payload. Do not rely on API Key quotas for financial chargeback — use the AI Token Ledger in BigQuery.

---

### 3. Network Perimeter — VPC Service Controls

The Vertex AI inference endpoint is protected behind **VPC Service Controls**, enforcing a strict network perimeter around the AI control plane.

- All Apigee → Vertex AI traffic is **private** — no public egress from the inference endpoint
- VPC Service Controls prevent data exfiltration — responses cannot be routed outside the defined perimeter
- External or unauthorized IPs are blocked at the network layer, even if they possess valid GCP credentials
- Private networking enforced from Day 1 in Phase 1 — never exposed to the public internet

---

### 4. IAM — Least-Privilege Principle

Every principal in the AI Hub is granted only the minimum permissions required to perform its function.

| Principal | Role | Scope |
|---|---|---|
| Apigee Service Account | `roles/aiplatform.user` | `pr-ai-hub-shared` Vertex AI only |
| CI/CD Service Account | `roles/artifactregistry.writer` | `internal-pypi` repository only |
| Developer group | `roles/artifactregistry.reader` | `internal-pypi` repository only |
| Finance / Team Leads | Read-only Looker Studio access | BigQuery AI Token Ledger views only |
| No team or developer | ~~`roles/aiplatform.user`~~ | Direct Vertex AI access revoked |

---

### 5. Audit Logging & Compliance

All activity across the AI Hub is captured in GCP **Cloud Audit Logs** — providing a tamper-evident trail for compliance and incident response.

- **Admin Activity logs** — IAM policy changes, Service Account modifications, quota updates
- **Data Access logs** — Vertex AI inference calls, BigQuery reads/writes
- **Apigee Access logs** — every API Key request, policy evaluation, and error response
- Logs are retained in **Cloud Logging** and optionally exported to BigQuery for long-term compliance storage
- Audit log review is a mandatory gate at the end of **Phase 4** before sign-off

---

### 6. Prompt & Response Data Handling

Inference requests (prompts) and responses are transient — they flow through Apigee and Vertex AI but are **not persisted in their raw form** by default.

- The **AI Token Ledger** (BigQuery) stores metadata only: token counts, latency, status codes, timestamps — not the prompt or response content
- If prompt logging is required for compliance or debugging, it must be explicitly designed with data classification, retention policies, and access controls in place
- Sensitive data (PII, financial data) must not be included in prompts without prior data governance review

---

### 7. Security Review Checkpoints

| Phase | Security Gate |
|---|---|
| Phase 1 | VPC Service Controls configured; Apigee Service Account scoped to least-privilege |
| Phase 2 | API Key rotation policy defined; error response contracts reviewed for information leakage |
| Phase 4 | Audit Log review confirms zero direct Vertex AI access paths remain |
| Phase 5 | Full Zero-Trust Security Review: IAM bindings, key rotation, VPC controls, audit coverage |

---

## Summary Timeline

| Week | Milestone |
|---|---|
| 1–2 | AI Hub Control Plane provisioned; Vertex AI inference endpoint live |
| 3–4 | AI Gateway (Apigee) deployed with per-consumer Quota Policies and SpikeArrest |
| 5–6 | Observability Pipeline live; AI Token Ledger and FinOps Dashboard operational |
| 7–8 | All consumers onboarded; direct inference access paths decommissioned |
| 9–10 | Provisioned Throughput active; platform hardened for production SLOs |

---

## Local CLI (`ask-claude`) — Architecture & Setup Guide

### What is `ask-claude`?

`ask-claude` is a lightweight **enterprise CLI client** distributed internally via GCP Artifact Registry. It acts as the developer-facing **Consumer Interface Layer** — abstracting the AI Gateway URL, API Key management, and HTTP request construction behind a simple terminal command. The CLI is intentionally thin; all intelligence (quota enforcement, AuthN, token logging) lives in the **AI Gateway Layer**, not the client.

```bash
ask-claude "Summarise the Q3 financial report"
> Claude: The Q3 financial report highlights a 12% revenue increase...
```

---

### End-to-End Request Flow

```
Developer Laptop
  └── ask-claude "prompt"
        └── HTTP POST + API Key header
              └── Apigee AI Gateway
                    ├── AuthN Check          (invalid key → 401)
                    ├── Per-Team Quota Check (quota exceeded → 429)
                    ├── SpikeArrest Check    (PT ceiling breach → 429)
                    ├── Token Logging        (Pub/Sub → BigQuery)
                    └── Forward to Vertex AI (Claude 3.5 Sonnet)
                          └── Response streamed back → Terminal
```

**Step-by-Step Walkthrough:**

**Step 1 – Developer installs the CLI** via the internal PyPI server (Artifact Registry). One-time setup stores the Gateway URL and team API Key locally:
```bash
pip install ask-claude
ask-claude configure
# Enter Gateway URL: https://api.yourcompany.com/claude
# Enter your API Key: ****************************
# Config saved to ~/.ask-claude/config.json
```

**Step 2 – Developer sends a prompt.** The CLI constructs a JSON payload and fires an HTTP POST to the AI Gateway with the API Key in the request header.

**Step 3 – Apigee runs its policy chain** in sequence before the request reaches the model:
- **AuthN Policy** — validates the API Key. Fails → `401 Unauthorized`
- **Quota Policy** — checks team's remaining RPM budget. Exceeded → `429` with retry hint
- **SpikeArrest Policy** — checks global TPM against PT ceiling. Saturated → graceful back-pressure response
- **MessageLogging Policy** — publishes request metadata to Pub/Sub (non-blocking, off the hot path)

**Step 4 – Request forwarded to Vertex AI.** Apigee rewrites the request using the central **Service Account** credentials. The developer's API Key never directly touches the inference endpoint — enforcing the **Zero-Trust Consumer Access** model.

**Step 5 – Claude processes and responds.** Vertex AI runs inference and streams the response back through Apigee to the CLI.

**Step 6 – Post-response token logging.** Apigee extracts input/output token counts, latency, and status from the response and publishes to the **AI Token Ledger** (BigQuery) for chargeback attribution.

**Step 7 – Developer sees the output** printed to the terminal.

---

### Error Response Contracts

| Scenario | HTTP Status | Message shown in CLI |
|---|---|---|
| Invalid or missing API Key | `401 Unauthorized` | `Error 401: Unauthorized. Check your API key.` |
| Team quota exceeded | `429 Too Many Requests` | `Error 429: Team quota reached. Retry in 60 seconds.` |
| Enterprise PT ceiling breached | `429 Too Many Requests` | `Error 429: Enterprise AI capacity saturated. Retry in 10 seconds.` |
| Vertex AI inference unavailable | `503 Service Unavailable` | `Error 503: Upstream inference endpoint unavailable.` |

---

### CLI Package Structure

```
ask-claude/
├── ask_claude/
│   ├── __init__.py
│   ├── cli.py          # Typer entry point — command definitions
│   ├── config.py       # Reads ~/.ask-claude/config.json
│   ├── gateway.py      # HTTP client — calls Apigee AI Gateway
│   └── formatter.py    # Rich terminal output formatting
├── pyproject.toml
└── README.md
```

`pyproject.toml`:
```toml
[project]
name = "ask-claude"
version = "1.0.0"
description = "Enterprise CLI for Claude via the AI Gateway"
dependencies = [
    "httpx>=0.27.0",
    "typer>=0.12.0",
    "rich>=13.0.0",
]

[project.scripts]
ask-claude = "ask_claude.cli:app"

[build-system]
requires = ["hatchling"]
build-backend = "hatchling.build"
```

Minimal `cli.py`:
```python
import typer
import httpx
from ask_claude.config import load_config
from ask_claude.formatter import print_response

app = typer.Typer()

@app.command()
def ask(prompt: str):
    config = load_config()
    response = httpx.post(
        config["gateway_url"],
        headers={"x-api-key": config["api_key"]},
        json={"prompt": prompt, "max_tokens": 1000},
        timeout=60.0
    )
    if response.status_code == 200:
        print_response(response.json())
    else:
        typer.echo(f"Error {response.status_code}: {response.json().get('message')}")
        raise typer.Exit(code=1)
```

---

## Internal Package Distribution — GCP Artifact Registry (PyPI)

### Why Artifact Registry over Self-Hosted PyPI?

| Dimension | Self-Hosted (devpi / pypiserver) | GCP Artifact Registry |
|---|---|---|
| Infrastructure | Self-managed VM, storage, HA | Fully managed by GCP |
| Authentication | Custom username/password | Native GCP IAM + Service Accounts |
| GCP Integration | Manual CI/CD wiring | Native Cloud Build, GKE, GCE support |
| Availability | Self-owned uptime | Google SLA-backed |
| Operational cost | VM + storage + ongoing ops overhead | Pay-per-GB storage + egress only |

Artifact Registry is the **recommended distribution plane** for `ask-claude` — it lives entirely within the GCP perimeter, uses the same IAM model as the rest of the AI Hub, and requires zero additional infrastructure to manage.

---

### Step 1 — Create the Python Repository

```bash
gcloud artifacts repositories create internal-pypi \
  --repository-format=python \
  --location=us-central1 \
  --description="Enterprise internal PyPI — ask-claude and internal tooling" \
  --project=pr-ai-hub-shared
```

Verify:
```bash
gcloud artifacts repositories list --project=pr-ai-hub-shared
```

---

### Step 2 — Configure IAM Access

**Developers (read-only — install only):**
```bash
gcloud artifacts repositories add-iam-policy-binding internal-pypi \
  --location=us-central1 \
  --member="group:developers@yourcompany.com" \
  --role="roles/artifactregistry.reader" \
  --project=pr-ai-hub-shared
```

**CI/CD pipeline Service Account (publish packages):**
```bash
gcloud artifacts repositories add-iam-policy-binding internal-pypi \
  --location=us-central1 \
  --member="serviceAccount:cicd-sa@pr-ai-hub-shared.iam.gserviceaccount.com" \
  --role="roles/artifactregistry.writer" \
  --project=pr-ai-hub-shared
```

---

### Step 3 — Authenticate on Developer Machines

One-time setup per developer:
```bash
gcloud auth login
gcloud auth application-default login
pip install keyring keyrings.google-artifactregistry-auth
```

Configure pip to resolve from the internal registry by default (`~/.config/pip/pip.conf`):
```ini
[global]
index-url = https://us-central1-python.pkg.dev/pr-ai-hub-shared/internal-pypi/simple/
```

From this point, `pip install ask-claude` resolves from Artifact Registry automatically — no additional credentials required.

---

### Step 4 — Build and Publish via CI/CD (Cloud Build)

`cloudbuild.yaml`:
```yaml
steps:
  - name: python:3.12
    entrypoint: pip
    args: ["install", "build", "twine", "keyrings.google-artifactregistry-auth"]

  - name: python:3.12
    entrypoint: python
    args: ["-m", "build"]

  - name: python:3.12
    entrypoint: twine
    args:
      - upload
      - --repository-url
      - https://us-central1-python.pkg.dev/pr-ai-hub-shared/internal-pypi/
      - dist/*
```

Every merge to `main` triggers Cloud Build, which builds and publishes the new version automatically.

---

### How It All Fits Together

```
GCP Project: pr-ai-hub-shared
├── Artifact Registry (internal-pypi)   ← CLI distribution plane
│     └── ask-claude v1.x.x
├── Apigee                              ← AI Gateway / Intelligent Ingress Layer
│     └── Claude API Proxy
├── Vertex AI                           ← Managed Inference Endpoint
│     └── Claude 3.5 Sonnet
└── BigQuery                            ← AI Token Ledger
      └── inference_logs table

Developer Laptop
  ├── pip install ask-claude            (pulls from Artifact Registry)
  └── ask-claude "prompt"              (routes through Apigee → Vertex AI)
```

Everything — the CLI distribution, the gateway, the model, and the token ledger — lives within a single GCP control plane (`pr-ai-hub-shared`), under a unified IAM and VPC perimeter.

> **Operational tip:** Log the CLI version (`ask-claude --version`) as a field in the Apigee MessageLogging policy alongside the API Key. This enriches the AI Token Ledger with client version metadata — invaluable for debugging regressions when a new CLI release introduces unexpected behaviour.

---

## Looker Studio FinOps Dashboard — Chart Reference

The Looker Studio dashboard is built entirely on **BigQuery as the AI Token Ledger** — a single source of truth for all inference telemetry streamed from Apigee via Pub/Sub. Every chart below maps directly to a BigQuery field, making the dashboard reproducible and auditable.

---

### Pillar 1 — Technical & Platform Health

| # | Chart Title | Chart Type | BigQuery Source | Looker Studio Config |
|---|---|---|---|---|
| 1 | Daily Request Volume — All Teams | Stacked Area (Time Series) | `ai_token_ledger` | Dimension: `request_date`, Breakdown: `consumer_team`, Metric: `COUNT(request_id)`. Enable smoothing + stack. |
| 2 | P50 / P95 / P99 Latency Trend | Grouped Bar | `ai_token_ledger` | X-axis: week, Metrics: `PERCENTILE(latency_ms, 50/95/99)` as separate series. Add 3,000ms SLO reference line. |
| 3 | 5xx Error Rate vs 429 Rate | Dual-Line (Time Series) | `apigee_access_logs` | Two metrics: `5xx_error_rate` and `quota_429_rate` on same axis. Mark 1% SLO threshold. |
| 4 | Vertex AI Quota Utilization — TPM vs PT Ceiling | Area + Reference Lines | `ai_token_ledger` | Metric: `SUM(tokens_per_minute)`. Add constant lines for PT ceiling (green dashed) and warn threshold at 75% (amber dashed). |
| 5 | AI Gateway Availability % | Bar (Conditional Colour) | `cloud_monitoring_export` | Weekly availability %. Conditional formatting: green ≥99.9%, red below. Reference line at 99.9%. |

---

### Pillar 2 — FinOps & Cost Optimisation

| # | Chart Title | Chart Type | BigQuery Source | Looker Studio Config |
|---|---|---|---|---|
| 6 | Monthly Token Burn Rate by Team | Stacked Bar | `ai_token_ledger` | Dimension: `consumer_team`, Metrics: `SUM(input_tokens)` + `SUM(output_tokens)` stacked. Drill-down to daily for chargeback invoicing. |
| 7 | Cost Share by Team | Donut Chart | `ai_token_ledger` | Dimension: `team_name`, Metric: `SUM(estimated_cost_usd)`. Show % and absolute labels. |
| 8 | TPM Baseline Profiling — Hourly Heatmap | Pivot Table / Heatmap Bar | `ai_token_ledger` | Rows: `HOUR(request_timestamp)`, Columns: `DAYOFWEEK`, Value: `AVG(tokens_per_minute)`. Colour intensity = TPM floor for PT sizing target. |
| 9 | PAYG vs Provisioned Throughput Cost Projection | Dual-Line (Forecast) | `gcp_billing_export` + PT rate | PAYG actual/projected (red) vs PT flat rate (green). Shade saving gap between lines. Annotate PT activation month. |

---

### Pillar 3 — Security & Compliance

| # | Chart Title | Chart Type | BigQuery Source | Looker Studio Config |
|---|---|---|---|---|
| 10 | 401 Unauthorized Events — Daily | Bar (Conditional Colour) | `apigee_access_logs` | Daily `COUNT` of HTTP 401 events. Conditional colour: amber for moderate spikes, red if > threshold. Click-through drill to `consumer_api_key`. |
| 11 | API Key Age Distribution | Histogram (Bar) | `apigee_developer_apps` | Buckets: 0–30d, 31–60d, 61–90d, 91–120d, >120d. Highlight >90-day bucket in red — policy breach indicator. |
| 12 | Security Review Gate Status | Static Table | Manual / phase tracker | Columns: Phase, Gate, Status, Sign-off Date. Conditional formatting: green (Passed), amber (Pending), red (Failed). |

---

### Pillar 4 — Consumer Adoption

| # | Chart Title | Chart Type | BigQuery Source | Looker Studio Config |
|---|---|---|---|---|
| 13 | Weekly Active API Keys — 8-Week Growth | Smooth Line | `ai_token_ledger` | X: week, Y: `COUNT(DISTINCT api_key)` with ≥1 request. Add Phase 4 annotation marker. Tracks adoption velocity post-onboarding. |
| 14 | Requests per Team — Ranked | Horizontal Bar | `ai_token_ledger` | Sorted descending by `COUNT(request_id)`. Overlay quota limit as a reference mark per team to show headroom. |
| 15 | Per-Team Quota — Allocated vs Peak RPM | Grouped Bar | `ai_token_ledger` + `apigee_quota_config` | Two series: `allocated_rpm` (grey) vs `peak_rpm_used` (blue). Gap = headroom. Flag teams >80% utilization for quota review. |
| 16 | Team Onboarding Funnel | Funnel / Progress Table | `onboarding_tracker` | Milestones: API Key Issued → First Request → Fully Onboarded → Old Access Revoked → Dashboard Access. Each step as a calculated field with completion rate. |

---

### BigQuery Schema Reference — `ai_token_ledger`

The core table powering the majority of dashboard charts:

| Field | Type | Description |
|---|---|---|
| `request_id` | STRING | Unique identifier per inference request |
| `request_timestamp` | TIMESTAMP | UTC time of request (from Apigee) |
| `consumer_team` | STRING | Team name derived from API Key lookup |
| `consumer_api_key` | STRING | Hashed API Key (consumer identity) |
| `model_version` | STRING | Vertex AI model endpoint identifier |
| `input_tokens` | INTEGER | Input token count from Vertex AI response |
| `output_tokens` | INTEGER | Output token count from Vertex AI response |
| `latency_ms` | INTEGER | Gateway-to-model round-trip latency (ms) |
| `http_status` | INTEGER | HTTP response code (200, 429, 5xx) |
| `estimated_cost_usd` | FLOAT | Calculated cost: `(input_tokens × input_rate) + (output_tokens × output_rate)` |
| `cli_version` | STRING | `ask-claude` client version (from Apigee log enrichment) |

---

## Architecture Comparison — This Approach vs LiteLLM

LiteLLM is a popular open-source proxy for routing LLM traffic across multiple providers. It is a legitimate tool for lightweight, developer-centric setups. The table below compares it honestly against the **GCP AI Hub (Apigee + Vertex AI)** architecture.

---

### Head-to-Head Comparison

| Dimension | GCP AI Hub (Apigee + Vertex AI) | LiteLLM Proxy |
|---|---|---|
| **Architecture pattern** | Enterprise AI Gateway on managed GCP control plane | Open-source LLM proxy, self-hosted or cloud-hosted |
| **Deployment model** | Fully managed — Apigee, Vertex AI, BigQuery, Pub/Sub | Self-managed — you own the server, HA, storage, updates |
| **Infrastructure overhead** | Near-zero — all GCP-managed services with SLA backing | High — VM/container provisioning, uptime, patching, scaling |
| **Multi-model support** | Single model (Claude on Vertex AI) — intentional simplicity | Multi-provider routing (OpenAI, Anthropic, Azure, Gemini, etc.) |
| **AuthN / AuthZ** | Apigee API Keys + IAM — enterprise-grade, auditable | API Key pass-through — lighter weight, less governance |
| **Quota & rate limiting** | Per-consumer Quota Policies + SpikeArrest in Apigee (policy-as-code) | Per-key rate limiting — functional but less granular |
| **Token-level observability** | Native — Apigee MessageLogging → Pub/Sub → BigQuery AI Token Ledger | Native — LiteLLM has built-in spend tracking and logs |
| **Financial chargeback** | Per-request, per-team chargeback via BigQuery + Looker Studio | Basic spend tracking — not designed for enterprise chargeback |
| **Provisioned Throughput** | Native GCP CUD / PT negotiation on `pr-ai-hub-shared` | No PT concept — always PAYG via the underlying provider API |
| **Statistical multiplexing** | Full — single control plane pools all teams' traffic for PT efficiency | Not applicable — no capacity commitment layer |
| **Network security** | VPC Service Controls, private networking, zero public egress | Depends on hosting — public internet exposure by default |
| **Audit logging** | GCP Cloud Audit Logs — tamper-evident, compliance-grade | Application-level logging — not compliance-grade by default |
| **Zero-Trust posture** | Enforced — no team has direct Vertex AI IAM access | Not enforced — teams can often access providers directly |
| **HA & resilience** | Apigee HA, Google SLA-backed — enterprise-grade | Self-managed HA — single point of failure unless architected carefully |
| **Cost at scale** | 20–50% reduction via PT + statistical multiplexing | PAYG only — no capacity discount model available |
| **Vendor lock-in** | GCP-native — higher lock-in to GCP ecosystem | Provider-agnostic — easy to switch models and providers |
| **Setup complexity** | Higher upfront — multi-phase, enterprise architecture | Lower — can be running in minutes |
| **Best fit** | Production enterprise AI Hub, regulated industries, FinOps maturity | Developer tooling, multi-provider experimentation, early-stage AI adoption |

---

### Where This Architecture Wins (Strongly)

**1. Provisioned Throughput & FinOps**
LiteLLM has no concept of Provisioned Throughput. Every token is billed at PAYG rates regardless of volume. The GCP AI Hub architecture enables a **20–50% cost reduction** at scale by pooling all enterprise traffic through a single control plane and negotiating a Committed Use Discount — a capability LiteLLM cannot replicate.

**2. Enterprise-Grade Security Posture**
LiteLLM relies on application-level controls. The GCP AI Hub enforces a **Zero-Trust perimeter** — VPC Service Controls, least-privilege IAM, and GCP Cloud Audit Logs that are tamper-evident and compliance-grade. No team can reach the inference endpoint outside the gateway, by design.

**3. Statistical Multiplexing**
In LiteLLM, each provider key is isolated. In the GCP AI Hub, all teams share one PT commitment. Engineering's quiet periods absorb Marketing's spikes — **one right-sized purchase, no idle waste**. This is architecturally impossible in LiteLLM.

**4. Financial Chargeback at Request Level**
LiteLLM offers basic spend tracking. The GCP AI Hub builds a **full AI Token Ledger in BigQuery** — every request attributed to a team, with input/output token counts, latency, cost, and CLI version. Finance can produce per-team invoices from a single SQL query.

**5. Policy-as-Code Governance**
Apigee Quota Policies, SpikeArrest, and AuthN are declarative, version-controlled, and auditable. LiteLLM configurations are application-level settings — harder to govern, audit, and enforce consistently across teams.

**6. GCP Ecosystem Integration**
The AI Hub lives entirely within one GCP control plane — same IAM, same VPC, same billing, same Artifact Registry for the CLI. LiteLLM sits outside this perimeter and requires additional integration work to achieve comparable observability.

---

### Where LiteLLM Has an Edge

**Multi-provider flexibility** is LiteLLM's strongest differentiator. If the enterprise needs to route requests across OpenAI, Anthropic (direct API), Azure OpenAI, and Gemini simultaneously — LiteLLM handles this natively with minimal configuration. The GCP AI Hub architecture is deliberately single-model (Claude on Vertex AI) and would require significant Apigee routing complexity to support multi-provider scenarios.

**Speed of setup** is also faster with LiteLLM — it can be running in under an hour. The GCP AI Hub is a multi-phase, 10-week enterprise rollout with significantly higher upfront investment.

---

### Recommendation

| Scenario | Recommended Approach |
|---|---|
| Enterprise production AI Hub, FinOps maturity, regulated industry | **GCP AI Hub (Apigee + Vertex AI)** |
| Single LLM provider (Claude), centralized governance, cost optimization at scale | **GCP AI Hub (Apigee + Vertex AI)** |
| Multi-provider experimentation, early-stage AI adoption, small team | **LiteLLM** |
| Need to switch providers frequently or benchmark models | **LiteLLM** |
| Compliance, audit logging, Zero-Trust required | **GCP AI Hub (Apigee + Vertex AI)** |
| Minimal infrastructure budget, fastest time-to-value | **LiteLLM** |

For an enterprise committing to Claude on Vertex AI at scale, the GCP AI Hub architecture is the correct long-term platform. LiteLLM is a useful prototyping and multi-provider tool — but it does not have the governance, security, or FinOps infrastructure required for production enterprise AI at the scale this architecture targets.
