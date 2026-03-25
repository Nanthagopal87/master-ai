# Enterprise AI Hub – Project Plan
**Project:** Centralized LLM Gateway (`pr-ai-hub-shared`)
**Architecture Pattern:** AI Gateway over a Single Control Plane (GCP) with Apigee as the Intelligent Ingress Layer
**Model:** Claude 3.5 Sonnet on Vertex AI (Managed Inference Endpoint)

---

## Table of Contents

1. [Executive Summary](#executive-summary)
2. [Billing Strategy: Pay-As-You-Go → Provisioned Throughput](#billing-strategy-pay-as-you-go--provisioned-throughput)
3. [Goals & Success Criteria](#goals--success-criteria)
4. [Architecture Overview](#architecture-overview)
5. [**Apigee Proxy Configuration — Implementation Guide**](#apigee-proxy-configuration--implementation-guide) ⭐ **NEW**
6. [Phases & Timeline](#phases--timeline)
7. [Risks & Mitigations](#risks--mitigations)
8. [Team & Responsibilities](#team--responsibilities)
9. [Key Architectural Decisions](#key-architectural-decisions)
10. [Security](#security)
11. [Summary Timeline](#summary-timeline)
12. [Local CLI (`ask-claude`) — Architecture & Setup Guide](#local-cli-ask-claude--architecture--setup-guide)
13. [Internal Package Distribution — GCP Artifact Registry (PyPI)](#internal-package-distribution--gcp-artifact-registry-pypi)

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

## Apigee Proxy Configuration — Implementation Guide

This section provides the complete **Apigee API Proxy** configuration for the AI Gateway. The proxy is the core enforcement layer where all AuthN, quota, traffic shaping, and observability policies are executed in a defined sequence before requests reach the Vertex AI inference endpoint.

---

### Proxy Architecture — Request Flow & Policy Chain

Every request to the AI Gateway traverses the following **policy execution pipeline**:

```
Incoming Request
    │
    ├─► 1. VerifyAPIKey           (AuthN — validate consumer identity)
    │       ├─ Valid   → Extract team metadata, continue
    │       └─ Invalid → 401 Unauthorized, terminate
    │
    ├─► 2. Quota Policy            (Per-consumer RPM limit enforcement)
    │       ├─ Under quota   → continue
    │       └─ Quota exceeded → 429 with retry-after, terminate
    │
    ├─► 3. SpikeArrest Policy      (Global TPM ceiling for PT alignment)
    │       ├─ Under ceiling → continue
    │       └─ At ceiling    → 429 with backpressure message, terminate
    │
    ├─► 4. AssignMessage (Pre-Request) — Construct Vertex AI payload
    │       └─ Transform consumer request → Vertex AI API contract
    │
    ├─► 5. ServiceCallout (Optional) — Health check or token pre-validation
    │
    ├─► 6. TargetEndpoint           → Forward to Vertex AI (Claude 3.5 Sonnet)
    │       └─ Inference execution
    │
    ├─► 7. AssignMessage (Post-Response) — Extract token counts from response
    │
    ├─► 8. MessageLogging Policy    (Observability — Pub/Sub)
    │       └─ Publish: team, tokens, latency, status → BigQuery pipeline
    │
    └─► Response returned to consumer
```

**Execution Characteristics:**
- Policies 1–3 are **pre-request gates** — failures here terminate the request before reaching Vertex AI
- Policy 8 (MessageLogging) is **non-blocking** — executed asynchronously to avoid adding latency to the inference path
- All policy failures return structured error responses with HTTP status codes, error messages, and retry guidance

---

### Directory Structure — Apigee Proxy Bundle

The Apigee proxy is defined as a **bundle** — a directory structure containing the proxy definition, policies, and target configuration. This bundle is deployed to Apigee via the Management API or the `apigeecli` tool.

```
claude-ai-gateway/
├── apiproxy/
│   ├── proxies/
│   │   └── default.xml                  # ProxyEndpoint — consumer-facing ingress
│   ├── targets/
│   │   └── vertex-ai.xml                # TargetEndpoint — Vertex AI backend
│   ├── policies/
│   │   ├── VerifyAPIKey.xml             # Policy 1 — AuthN
│   │   ├── Quota-PerTeam.xml            # Policy 2 — Per-consumer RPM quota
│   │   ├── SpikeArrest-Global.xml       # Policy 3 — Global TPM ceiling
│   │   ├── AssignMessage-Request.xml    # Policy 4 — Request transformation
│   │   ├── AssignMessage-Response.xml   # Policy 5 — Extract token metadata
│   │   ├── MessageLogging-Pubsub.xml    # Policy 6 — Observability pipeline
│   │   ├── RaiseFault-401.xml           # Error response — Unauthorized
│   │   ├── RaiseFault-429-Quota.xml     # Error response — Quota exceeded
│   │   └── RaiseFault-429-Spike.xml     # Error response — PT ceiling breach
│   ├── resources/
│   │   └── jsc/                         # JavaScript callouts (optional)
│   └── claude-ai-gateway.xml            # Root proxy descriptor
└── README.md
```

---

### 1. Root Proxy Descriptor — `claude-ai-gateway.xml`

This file defines the proxy metadata, base path, and references to the ProxyEndpoint and TargetEndpoint.

```xml
<?xml version="1.0" encoding="UTF-8" standalone="yes"?>
<APIProxy name="claude-ai-gateway" revision="1">
  <DisplayName>Enterprise AI Gateway — Claude 3.5 Sonnet</DisplayName>
  <Description>
    Centralized LLM Gateway for Claude on Vertex AI.
    Enforces AuthN, per-team quotas, global SpikeArrest, and token-level observability.
  </Description>
  <BasePaths>/v1/claude</BasePaths>
  <Policies>
    <Policy>VerifyAPIKey</Policy>
    <Policy>Quota-PerTeam</Policy>
    <Policy>SpikeArrest-Global</Policy>
    <Policy>AssignMessage-Request</Policy>
    <Policy>AssignMessage-Response</Policy>
    <Policy>MessageLogging-Pubsub</Policy>
    <Policy>RaiseFault-401</Policy>
    <Policy>RaiseFault-429-Quota</Policy>
    <Policy>RaiseFault-429-Spike</Policy>
  </Policies>
  <ProxyEndpoints>
    <ProxyEndpoint>default</ProxyEndpoint>
  </ProxyEndpoints>
  <TargetEndpoints>
    <TargetEndpoint>vertex-ai</TargetEndpoint>
  </TargetEndpoints>
</APIProxy>
```

---

### 2. ProxyEndpoint — `proxies/default.xml`

The **ProxyEndpoint** is the consumer-facing ingress. It defines the HTTP listener configuration and the sequence of policies executed on the **request path** (PreFlow) and **response path** (PostFlow).

```xml
<?xml version="1.0" encoding="UTF-8" standalone="yes"?>
<ProxyEndpoint name="default">
  <Description>Consumer-facing endpoint for AI Gateway</Description>
  
  <!-- HTTP Configuration -->
  <HTTPProxyConnection>
    <BasePath>/v1/claude</BasePath>
    <VirtualHost>secure</VirtualHost>
  </HTTPProxyConnection>
  
  <!-- Request Flow — Executed BEFORE forwarding to Vertex AI -->
  <PreFlow name="PreFlow">
    <Request>
      <!-- Step 1: Verify API Key -->
      <Step>
        <Name>VerifyAPIKey</Name>
        <Condition>(request.verb = "POST")</Condition>
      </Step>
      
      <!-- Step 2: Enforce Per-Team Quota -->
      <Step>
        <Name>Quota-PerTeam</Name>
        <Condition>(request.verb = "POST")</Condition>
      </Step>
      
      <!-- Step 3: Enforce Global SpikeArrest (PT Ceiling) -->
      <Step>
        <Name>SpikeArrest-Global</Name>
        <Condition>(request.verb = "POST")</Condition>
      </Step>
      
      <!-- Step 4: Transform request for Vertex AI -->
      <Step>
        <Name>AssignMessage-Request</Name>
      </Step>
    </Request>
    <Response/>
  </PreFlow>
  
  <!-- Response Flow — Executed AFTER Vertex AI responds -->
  <PostFlow name="PostFlow">
    <Request/>
    <Response>
      <!-- Step 5: Extract token counts from Vertex AI response -->
      <Step>
        <Name>AssignMessage-Response</Name>
      </Step>
      
      <!-- Step 6: Log telemetry to Pub/Sub (non-blocking) -->
      <Step>
        <Name>MessageLogging-Pubsub</Name>
      </Step>
    </Response>
  </PostFlow>
  
  <!-- Fault Handling — Executed if any policy raises a fault -->
  <FaultRules>
    <FaultRule name="InvalidAPIKey">
      <Condition>(fault.name = "InvalidApiKey")</Condition>
      <Step>
        <Name>RaiseFault-401</Name>
      </Step>
    </FaultRule>
    
    <FaultRule name="QuotaViolation">
      <Condition>(fault.name Matches "QuotaViolation")</Condition>
      <Step>
        <Name>RaiseFault-429-Quota</Name>
      </Step>
    </FaultRule>
    
    <FaultRule name="SpikeArrestViolation">
      <Condition>(fault.name = "SpikeArrestViolation")</Condition>
      <Step>
        <Name>RaiseFault-429-Spike</Name>
      </Step>
    </FaultRule>
  </FaultRules>
  
  <RouteRule name="default">
    <TargetEndpoint>vertex-ai</TargetEndpoint>
  </RouteRule>
</ProxyEndpoint>
```

**Key Concepts:**
- **PreFlow Request**: Policies execute in sequence before the request is forwarded to the TargetEndpoint
- **PostFlow Response**: Policies execute after the response is received from the TargetEndpoint
- **FaultRules**: Custom error handling — triggered when a policy raises a fault (e.g., quota exceeded)
- **Condition**: Each policy can have a conditional execution clause (e.g., only execute on POST requests)

---

### 3. TargetEndpoint — `targets/vertex-ai.xml`

The **TargetEndpoint** defines the backend connection to the Vertex AI inference endpoint. It includes the URL, authentication mechanism (Google Service Account), and HTTP configuration.

```xml
<?xml version="1.0" encoding="UTF-8" standalone="yes"?>
<TargetEndpoint name="vertex-ai">
  <Description>Backend connection to Vertex AI — Claude 3.5 Sonnet inference endpoint</Description>
  
  <!-- Backend HTTP Configuration -->
  <HTTPTargetConnection>
    <URL>https://us-central1-aiplatform.googleapis.com/v1/projects/pr-ai-hub-shared/locations/us-central1/publishers/anthropic/models/claude-3-5-sonnet@20241022:streamRawPredict</URL>
    
    <!-- Authentication — Google Service Account (handled by Apigee runtime) -->
    <Authentication>
      <GoogleAccessToken>
        <Scopes>
          <Scope>https://www.googleapis.com/auth/cloud-platform</Scope>
        </Scopes>
      </GoogleAccessToken>
    </Authentication>
    
    <!-- SSL Configuration -->
    <SSLInfo>
      <Enabled>true</Enabled>
      <ClientAuthEnabled>false</ClientAuthEnabled>
      <IgnoreValidationErrors>false</IgnoreValidationErrors>
    </SSLInfo>
    
    <!-- Connection Properties -->
    <Properties>
      <Property name="connect.timeout.millis">10000</Property>
      <Property name="io.timeout.millis">60000</Property>
      <Property name="keepalive.timeout.millis">60000</Property>
      <Property name="supports.http11">true</Property>
    </Properties>
  </HTTPTargetConnection>
  
  <!-- No policies on TargetEndpoint — all logic in ProxyEndpoint -->
  <PreFlow name="PreFlow">
    <Request/>
    <Response/>
  </PreFlow>
  
  <PostFlow name="PostFlow">
    <Request/>
    <Response/>
  </PostFlow>
</TargetEndpoint>
```

**Key Configuration Notes:**
- **URL**: Points to the Vertex AI streaming inference endpoint for Claude 3.5 Sonnet. Replace project ID, region, and model version as needed.
- **GoogleAccessToken**: Apigee automatically generates a Google OAuth2 access token using the Service Account attached to the Apigee environment. No manual token management required.
- **Timeouts**: `connect.timeout` = 10s, `io.timeout` = 60s. Adjust based on expected inference latency.

---

### 4. Policy Definitions

---

#### Policy 1: VerifyAPIKey — `policies/VerifyAPIKey.xml`

Validates the `x-api-key` header against Apigee's internal API Key store. On success, extracts the associated **Developer App** metadata (consumer identity, team name). On failure, raises an `InvalidApiKey` fault.

```xml
<?xml version="1.0" encoding="UTF-8" standalone="yes"?>
<VerifyAPIKey name="VerifyAPIKey" async="false" continueOnError="false">
  <DisplayName>Verify API Key — Consumer AuthN</DisplayName>
  
  <!-- API Key location in request -->
  <APIKey ref="request.header.x-api-key"/>
  
  <!-- Extract Developer App metadata on success -->
  <!-- These variables are available to downstream policies -->
  <!-- verifyapikey.VerifyAPIKey.app.name = Developer App name -->
  <!-- verifyapikey.VerifyAPIKey.developer.id = Developer ID -->
  <!-- verifyapikey.VerifyAPIKey.CustomAttribute.team_name = Team name from Developer App custom attribute -->
</VerifyAPIKey>
```

**Developer App Custom Attributes (set during onboarding):**
When creating a Developer App in Apigee, attach custom attributes to store team metadata:

```bash
# Example: Creating a Developer App for the Engineering team
curl -X POST \
  "https://apigee.googleapis.com/v1/organizations/{org}/developers/{developer_email}/apps" \
  -H "Authorization: Bearer $(gcloud auth print-access-token)" \
  -d '{
    "name": "engineering-ask-claude",
    "apiProducts": ["claude-ai-product"],
    "attributes": [
      {"name": "team_name", "value": "Engineering"},
      {"name": "cost_center", "value": "ENG-001"},
      {"name": "quota_rpm", "value": "1000"}
    ]
  }'
```

These attributes are extractable as: `verifyapikey.VerifyAPIKey.CustomAttribute.team_name`

---

#### Policy 2: Quota Policy (Per-Team) — `policies/Quota-PerTeam.xml`

Enforces a **Requests Per Minute (RPM)** limit per consumer. The quota value is read from the Developer App's custom attribute (`quota_rpm`).

```xml
<?xml version="1.0" encoding="UTF-8" standalone="yes"?>
<Quota name="Quota-PerTeam" async="false" continueOnError="false">
  <DisplayName>Quota Policy — Per-Team RPM Enforcement</DisplayName>
  
  <!-- Quota counter identifier — unique per team -->
  <Identifier ref="verifyapikey.VerifyAPIKey.CustomAttribute.team_name"/>
  
  <!-- Quota limit — read from Developer App custom attribute -->
  <Allow count="verifyapikey.VerifyAPIKey.CustomAttribute.quota_rpm" countRef="verifyapikey.VerifyAPIKey.CustomAttribute.quota_rpm"/>
  
  <!-- Time interval -->
  <Interval>1</Interval>
  <TimeUnit>minute</TimeUnit>
  
  <!-- Distributed quota counter (shared across Apigee instances) -->
  <Distributed>true</Distributed>
  <Synchronous>true</Synchronous>
  
  <!-- Start time for the quota window -->
  <StartTime>2025-01-01 00:00:00</StartTime>
</Quota>
```

**Quota Behavior:**
- If `Engineering` team sends 1,001 requests in a 60-second window (quota = 1,000 RPM), the 1,001st request raises a `QuotaViolation` fault
- The fault is caught by the `FaultRule` in the ProxyEndpoint and returns a `429` with retry guidance

**Important Note — API Keys and Token Metering:**
> ⚠️ **API Keys track RPM (request count) only — they have no visibility into token consumption per request.** Two requests from the same team could consume vastly different token volumes depending on prompt length and response size. For accurate **token-level chargeback**, token counts must be extracted from the Vertex AI response payload by the `MessageLogging` policy and streamed to BigQuery. The API Key serves as the consumer identity signal only — financial metering happens at the observability layer, not the quota layer.

---

#### Policy 3: SpikeArrest (Global TPM Ceiling) — `policies/SpikeArrest-Global.xml`

Enforces a **global Requests Per Minute** ceiling to align with Provisioned Throughput (PT) capacity. This prevents the enterprise from exceeding the PT commitment and ensures 100% capacity utilization without wastage.

```xml
<?xml version="1.0" encoding="UTF-8" standalone="yes"?>
<SpikeArrest name="SpikeArrest-Global" async="false" continueOnError="false">
  <DisplayName>SpikeArrest — Global TPM Ceiling (PT Alignment)</DisplayName>
  
  <!-- Global rate limit — all consumers combined -->
  <!-- Phase 1-4 (PAYG): Set conservatively (e.g., 10000pm = 10,000 requests/min) -->
  <!-- Phase 5 (PT active): Set to PT ceiling (e.g., 80000pm = 80K TPM) -->
  <Rate>10000pm</Rate>
  
  <!-- Identifier: global (all traffic) -->
  <Identifier ref="request.uri"/>
  
  <!-- Use message weight for advanced metering (optional) -->
  <!-- <MessageWeight ref="request.header.weight"/> -->
</SpikeArrest>
```

**SpikeArrest vs Quota:**
| Policy | Scope | Purpose | Behavior |
|---|---|---|---|
| Quota | Per-consumer (per team) | Workload isolation — prevent noisy neighbor | Raises fault when team exceeds their RPM limit |
| SpikeArrest | Global (all consumers) | PT capacity governance — prevent platform saturation | Raises fault when enterprise exceeds total capacity ceiling |

**Configuration Evolution:**
- **Weeks 1–8 (PAYG phase)**: Set SpikeArrest conservatively (e.g., 10,000 RPM) as a safety valve
- **Week 9+ (PT active)**: Reconfigure to the PT-committed TPM ceiling (e.g., 80,000 TPM). This ensures the enterprise uses 100% of reserved capacity without overage.

---

#### Policy 4: AssignMessage — Request Transformation — `policies/AssignMessage-Request.xml`

Transforms the consumer's request into the Vertex AI API contract. This policy constructs the JSON payload expected by the Claude 3.5 Sonnet endpoint.

```xml
<?xml version="1.0" encoding="UTF-8" standalone="yes"?>
<AssignMessage name="AssignMessage-Request" async="false" continueOnError="false">
  <DisplayName>AssignMessage — Transform Consumer Request to Vertex AI Payload</DisplayName>
  
  <!-- Set Content-Type header for Vertex AI -->
  <Set>
    <Headers>
      <Header name="Content-Type">application/json</Header>
    </Headers>
    
    <!-- Construct Vertex AI request payload -->
    <Payload contentType="application/json">
{
  "anthropic_version": "vertex-2023-10-16",
  "messages": [
    {
      "role": "user",
      "content": "{request.content}"
    }
  ],
  "max_tokens": {request.queryparam.max_tokens},
  "temperature": {request.queryparam.temperature},
  "stream": true
}
    </Payload>
  </Set>
  
  <!-- Preserve consumer metadata for downstream logging -->
  <AssignVariable>
    <Name>request.team</Name>
    <Ref>verifyapikey.VerifyAPIKey.CustomAttribute.team_name</Ref>
  </AssignVariable>
  
  <AssignVariable>
    <Name>request.start_time</Name>
    <Value>{system.timestamp}</Value>
  </AssignVariable>
</AssignMessage>
```

**Template Variables:**
- `{request.content}`: Extracted from the consumer's JSON payload (`{"prompt": "..."}`)
- `{request.queryparam.max_tokens}`: Optional query parameter (default: 1000)
- `{request.queryparam.temperature}`: Optional query parameter (default: 1.0)

**Vertex AI Payload Contract:**
The above example constructs a minimal Vertex AI request. For production, you may need to handle:
- Multi-turn conversations (message history)
- System prompts
- Tool use / function calling
- Vision inputs (images)

Refer to the [Anthropic Vertex AI documentation](https://docs.anthropic.com/en/api/claude-on-vertex-ai) for the complete API contract.

---

#### Policy 5: AssignMessage — Response Metadata Extraction — `policies/AssignMessage-Response.xml`

Extracts token counts and latency metadata from the Vertex AI response. These variables are used by the MessageLogging policy for observability.

```xml
<?xml version="1.0" encoding="UTF-8" standalone="yes"?>
<AssignMessage name="AssignMessage-Response" async="false" continueOnError="false">
  <DisplayName>AssignMessage — Extract Token Counts from Vertex AI Response</DisplayName>
  
  <!-- Extract token metadata from response JSON -->
  <AssignVariable>
    <Name>response.input_tokens</Name>
    <JSONPath>$.usage.input_tokens</JSONPath>
    <Template>{response.content}</Template>
  </AssignVariable>
  
  <AssignVariable>
    <Name>response.output_tokens</Name>
    <JSONPath>$.usage.output_tokens</JSONPath>
    <Template>{response.content}</Template>
  </AssignVariable>
  
  <!-- Calculate end-to-end latency -->
  <AssignVariable>
    <Name>response.latency_ms</Name>
    <Value>{system.timestamp - request.start_time}</Value>
  </AssignVariable>
  
  <!-- Extract HTTP status -->
  <AssignVariable>
    <Name>response.status_code</Name>
    <Ref>response.status.code</Ref>
  </AssignVariable>
</AssignMessage>
```

**Vertex AI Response Structure (Example):**
```json
{
  "id": "msg_01...",
  "type": "message",
  "role": "assistant",
  "content": [{"type": "text", "text": "..."}],
  "model": "claude-3-5-sonnet-20241022",
  "usage": {
    "input_tokens": 142,
    "output_tokens": 89
  }
}
```

The `JSONPath` extraction pulls `usage.input_tokens` and `usage.output_tokens` directly from this structure.

---

#### Policy 6: MessageLogging — Pub/Sub Observability — `policies/MessageLogging-Pubsub.xml`

Publishes request/response telemetry to a **Cloud Pub/Sub** topic. This data flows into BigQuery as the **AI Token Ledger** for chargeback and FinOps reporting.

```xml
<?xml version="1.0" encoding="UTF-8" standalone="yes"?>
<MessageLogging name="MessageLogging-Pubsub" async="true" continueOnError="true">
  <DisplayName>MessageLogging — Publish Telemetry to Pub/Sub (AI Token Ledger)</DisplayName>
  
  <!-- Target: Cloud Pub/Sub -->
  <GCPPubSub>
    <Topic>projects/pr-ai-hub-shared/topics/ai-gateway-telemetry</Topic>
  </GCPPubSub>
  
  <!-- Construct JSON payload for BigQuery ingestion -->
  <Message>
{
  "request_id": "{messageid}",
  "request_timestamp": "{system.timestamp}",
  "consumer_team": "{request.team}",
  "api_key_id": "{verifyapikey.VerifyAPIKey.client_id}",
  "model_version": "claude-3-5-sonnet-20241022",
  "input_tokens": {response.input_tokens},
  "output_tokens": {response.output_tokens},
  "latency_ms": {response.latency_ms},
  "status_code": {response.status_code},
  "error_type": "{fault.name}",
  "cli_version": "{request.header.x-cli-version}"
}
  </Message>
  
  <!-- Async execution — does not block response to consumer -->
  <Async>true</Async>
</MessageLogging>
```

**Pub/Sub → BigQuery Pipeline:**
1. MessageLogging policy publishes JSON to `ai-gateway-telemetry` Pub/Sub topic
2. Cloud Dataflow (or Pub/Sub BigQuery subscription) streams messages to BigQuery table `ai_token_ledger`
3. Looker Studio dashboard queries BigQuery for chargeback reports

**Schema for BigQuery `ai_token_ledger` table:**
```sql
CREATE TABLE `pr-ai-hub-shared.ai_hub.ai_token_ledger` (
  request_id STRING NOT NULL,
  request_timestamp TIMESTAMP NOT NULL,
  consumer_team STRING NOT NULL,
  api_key_id STRING,
  model_version STRING,
  input_tokens INT64,
  output_tokens INT64,
  latency_ms INT64,
  status_code INT64,
  error_type STRING,
  cli_version STRING,
  _partitiontime TIMESTAMP
)
PARTITION BY DATE(request_timestamp)
CLUSTER BY consumer_team, status_code;
```

---

#### Policy 7: RaiseFault — 401 Unauthorized — `policies/RaiseFault-401.xml`

Returns a structured error response when API Key validation fails.

```xml
<?xml version="1.0" encoding="UTF-8" standalone="yes"?>
<RaiseFault name="RaiseFault-401" async="false" continueOnError="false">
  <DisplayName>RaiseFault — 401 Unauthorized (Invalid API Key)</DisplayName>
  
  <FaultResponse>
    <Set>
      <StatusCode>401</StatusCode>
      <ReasonPhrase>Unauthorized</ReasonPhrase>
      
      <Headers>
        <Header name="Content-Type">application/json</Header>
      </Headers>
      
      <Payload contentType="application/json">
{
  "error": {
    "code": 401,
    "message": "Unauthorized. Invalid or missing API key.",
    "details": "Verify your API key is correct and included in the 'x-api-key' header.",
    "support": "Contact platform-team@yourcompany.com for assistance."
  }
}
      </Payload>
    </Set>
  </FaultResponse>
</RaiseFault>
```

---

#### Policy 8: RaiseFault — 429 Quota Exceeded — `policies/RaiseFault-429-Quota.xml`

Returns a structured error response when a team exceeds their RPM quota.

```xml
<?xml version="1.0" encoding="UTF-8" standalone="yes"?>
<RaiseFault name="RaiseFault-429-Quota" async="false" continueOnError="false">
  <DisplayName>RaiseFault — 429 Team Quota Exceeded</DisplayName>
  
  <FaultResponse>
    <Set>
      <StatusCode>429</StatusCode>
      <ReasonPhrase>Too Many Requests</ReasonPhrase>
      
      <Headers>
        <Header name="Content-Type">application/json</Header>
        <Header name="Retry-After">60</Header>
      </Headers>
      
      <Payload contentType="application/json">
{
  "error": {
    "code": 429,
    "message": "Team quota exceeded. Your team has reached its request limit.",
    "details": "Quota: {verifyapikey.VerifyAPIKey.CustomAttribute.quota_rpm} RPM. Retry after 60 seconds.",
    "support": "Request a quota increase: platform-team@yourcompany.com"
  }
}
      </Payload>
    </Set>
  </FaultResponse>
</RaiseFault>
```

---

#### Policy 9: RaiseFault — 429 Global Capacity Saturation — `policies/RaiseFault-429-Spike.xml`

Returns a structured error response when the enterprise exceeds the global PT ceiling.

```xml
<?xml version="1.0" encoding="UTF-8" standalone="yes"?>
<RaiseFault name="RaiseFault-429-Spike" async="false" continueOnError="false">
  <DisplayName>RaiseFault — 429 Enterprise Capacity Saturated (PT Ceiling Breach)</DisplayName>
  
  <FaultResponse>
    <Set>
      <StatusCode>429</StatusCode>
      <ReasonPhrase>Too Many Requests</ReasonPhrase>
      
      <Headers>
        <Header name="Content-Type">application/json</Header>
        <Header name="Retry-After">10</Header>
      </Headers>
      
      <Payload contentType="application/json">
{
  "error": {
    "code": 429,
    "message": "Enterprise AI capacity temporarily saturated. Please retry shortly.",
    "details": "The AI Hub is at its Provisioned Throughput ceiling. Your request will be processed if retried in 10 seconds.",
    "support": "If this persists, contact platform-team@yourcompany.com"
  }
}
      </Payload>
    </Set>
  </FaultResponse>
</RaiseFault>
```

---

### Deployment — Uploading the Proxy Bundle to Apigee

Once the proxy bundle is authored, deploy it to Apigee using the **apigeecli** tool:

```bash
# Install apigeecli
curl -L https://raw.githubusercontent.com/apigee/apigeecli/main/downloadLatest.sh | sh

# Set environment variables
export APIGEE_ORG="your-apigee-org"
export APIGEE_ENV="prod"
export APIGEE_TOKEN=$(gcloud auth print-access-token)

# Deploy the proxy bundle
apigeecli apis create bundle \
  --name claude-ai-gateway \
  --proxy-bundle ./claude-ai-gateway.zip \
  --org $APIGEE_ORG \
  --token $APIGEE_TOKEN

# Deploy to environment
apigeecli apis deploy \
  --name claude-ai-gateway \
  --env $APIGEE_ENV \
  --org $APIGEE_ORG \
  --token $APIGEE_TOKEN \
  --ovr
```

**Verification:**
```bash
# Test the deployed proxy
curl -X POST \
  "https://${APIGEE_ORG}-${APIGEE_ENV}.apigee.net/v1/claude" \
  -H "x-api-key: YOUR_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{
    "prompt": "Explain photosynthesis in one sentence.",
    "max_tokens": 100
  }'
```

Expected response:
```json
{
  "id": "msg_01...",
  "type": "message",
  "role": "assistant",
  "content": [{"type": "text", "text": "Photosynthesis is the process by which plants..."}],
  "usage": {"input_tokens": 12, "output_tokens": 24}
}
```

---

### API Products & Developer Apps — Consumer Onboarding

Before teams can use the AI Gateway, they must be onboarded via **API Products** and **Developer Apps**.

**Step 1 — Create API Product**

An API Product is a bundle of API proxies with quota and security settings.

```bash
curl -X POST \
  "https://apigee.googleapis.com/v1/organizations/${APIGEE_ORG}/apiproducts" \
  -H "Authorization: Bearer $APIGEE_TOKEN" \
  -d '{
    "name": "claude-ai-product",
    "displayName": "Claude AI Gateway Product",
    "approvalType": "manual",
    "attributes": [
      {"name": "access", "value": "internal"}
    ],
    "apiResources": ["/v1/claude/**"],
    "environments": ["prod"],
    "proxies": ["claude-ai-gateway"],
    "quota": "0",
    "quotaInterval": "1",
    "quotaTimeUnit": "minute"
  }'
```

**Step 2 — Create Developer**

A Developer represents an individual or team.

```bash
curl -X POST \
  "https://apigee.googleapis.com/v1/organizations/${APIGEE_ORG}/developers" \
  -H "Authorization: Bearer $APIGEE_TOKEN" \
  -d '{
    "email": "engineering-team@yourcompany.com",
    "firstName": "Engineering",
    "lastName": "Team",
    "userName": "engineering"
  }'
```

**Step 3 — Create Developer App (Issues API Key)**

```bash
curl -X POST \
  "https://apigee.googleapis.com/v1/organizations/${APIGEE_ORG}/developers/engineering-team@yourcompany.com/apps" \
  -H "Authorization: Bearer $APIGEE_TOKEN" \
  -d '{
    "name": "engineering-ask-claude",
    "apiProducts": ["claude-ai-product"],
    "attributes": [
      {"name": "team_name", "value": "Engineering"},
      {"name": "cost_center", "value": "ENG-001"},
      {"name": "quota_rpm", "value": "1000"}
    ],
    "callbackUrl": "https://internal.yourcompany.com/callback"
  }'
```

The response contains the **API Key** (`credentials[0].consumerKey`) — this is what the Engineering team uses in their `ask-claude` CLI.

---

### Configuration Management — GitOps & CI/CD

Store the Apigee proxy bundle in version control and automate deployments via CI/CD.

**Directory structure:**
```
ai-hub-infra/
├── apigee/
│   ├── claude-ai-gateway/          # Proxy bundle
│   │   └── apiproxy/
│   ├── scripts/
│   │   ├── deploy-proxy.sh         # Deployment script
│   │   └── create-api-product.sh   # Onboarding automation
│   └── config/
│       ├── quota-config.yaml       # Per-team quota definitions
│       └── spike-arrest-config.yaml # PT ceiling config
└── .github/workflows/
    └── deploy-apigee.yaml          # GitHub Actions CI/CD
```

**Example CI/CD workflow (GitHub Actions):**
```yaml
name: Deploy Apigee Proxy
on:
  push:
    branches: [main]
    paths: ['apigee/claude-ai-gateway/**']

jobs:
  deploy:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v3
      
      - name: Authenticate to GCP
        uses: google-github-actions/auth@v1
        with:
          credentials_json: ${{ secrets.GCP_SA_KEY }}
      
      - name: Install apigeecli
        run: |
          curl -L https://raw.githubusercontent.com/apigee/apigeecli/main/downloadLatest.sh | sh
      
      - name: Create proxy bundle
        run: |
          cd apigee/claude-ai-gateway
          zip -r ../claude-ai-gateway.zip apiproxy/
      
      - name: Deploy to Apigee
        run: |
          ./apigee/scripts/deploy-proxy.sh
        env:
          APIGEE_ORG: ${{ secrets.APIGEE_ORG }}
          APIGEE_ENV: prod
```

**Benefits:**
- Version control for all policy changes
- Automated testing and validation before deployment
- Audit trail for who changed what and when
- Rollback capability via Git history

---

### Monitoring & Alerting — Apigee Analytics & Cloud Monitoring

**Apigee Analytics Dashboard:**
- Navigate to Apigee UI → Analyze → API Metrics
- Filter by: `claude-ai-gateway` proxy
- View: Request volume, latency (P50/P95/P99), error rates, quota violations

**Cloud Monitoring Alerts:**

```yaml
# Example: Alert when 5xx error rate exceeds 1%
alerting_policy:
  display_name: "AI Gateway — High 5xx Error Rate"
  conditions:
    - display_name: "5xx Rate > 1%"
      condition_threshold:
        filter: |
          resource.type="api"
          resource.labels.api_proxy="claude-ai-gateway"
          metric.type="apigee.googleapis.com/proxy/response_code_5xx_count"
        comparison: COMPARISON_GT
        threshold_value: 0.01
        duration: 300s
  notification_channels:
    - projects/pr-ai-hub-shared/notificationChannels/slack-platform-team
```

**Key Metrics to Monitor:**
- **Quota Violations**: `apigee.googleapis.com/quota/exceeded_count`
- **SpikeArrest Violations**: Indicates PT ceiling breach
- **5xx Errors**: Backend (Vertex AI) failures
- **4xx Errors**: Client errors (invalid requests, AuthN failures)
- **Latency P95**: Should remain under 3s SLO

---

### Policy Optimization — Performance Tuning

**Policy Execution Order Matters:**
The sequence defined in the ProxyEndpoint's PreFlow determines execution order. Place the cheapest policies first to fail fast:

1. **VerifyAPIKey** (cheap — hash lookup)
2. **Quota** (cheap — counter increment)
3. **SpikeArrest** (cheap — counter increment)
4. **AssignMessage** (moderate — string templating)
5. **ServiceCallout** (expensive — external HTTP call) — only if needed

**Async Execution:**
- **MessageLogging** is `async="true"` — it executes in a background thread and does not block the response to the consumer
- Never make the MessageLogging policy synchronous — it would add latency to every request

**Caching (Optional):**
For frequently accessed data (e.g., model metadata), use Apigee's **ResponseCache** policy to avoid redundant Vertex AI calls.

---

### Summary — Apigee Proxy Configuration Checklist

| Component | Status | Notes |
|---|---|---|
| Root proxy descriptor (`claude-ai-gateway.xml`) | ✅ Defined | Metadata, base path, policy references |
| ProxyEndpoint (`default.xml`) | ✅ Defined | PreFlow, PostFlow, FaultRules, policy chain |
| TargetEndpoint (`vertex-ai.xml`) | ✅ Defined | Vertex AI URL, Google Service Account auth |
| VerifyAPIKey policy | ✅ Defined | AuthN — validates `x-api-key` header |
| Quota policy (per-team) | ✅ Defined | RPM limit enforcement per consumer |
| SpikeArrest policy (global) | ✅ Defined | PT ceiling enforcement (reconfigured in Phase 5) |
| AssignMessage (request) | ✅ Defined | Transform consumer request → Vertex AI payload |
| AssignMessage (response) | ✅ Defined | Extract token counts, latency metadata |
| MessageLogging (Pub/Sub) | ✅ Defined | Async telemetry publishing to AI Token Ledger |
| RaiseFault (401) | ✅ Defined | Structured error for invalid API Key |
| RaiseFault (429 quota) | ✅ Defined | Structured error for quota exceeded |
| RaiseFault (429 spike) | ✅ Defined | Structured error for PT ceiling breach |
| API Product | ✅ Created | `claude-ai-product` — bundles the proxy |
| Developer Apps | 🔄 Per-team | Created during Phase 4 consumer onboarding |
| Deployment automation | ✅ CI/CD | GitHub Actions + apigeecli |
| Monitoring & alerts | ✅ Configured | Cloud Monitoring + Apigee Analytics |

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
