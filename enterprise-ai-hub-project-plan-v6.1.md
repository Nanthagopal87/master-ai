To evolve your **Claude-only** plan into a **Multi-Model AI Hub** (Claude, Gemini, Llama 3 on Vertex, etc.), you need to refactor three specific areas: **The CLI**, **The Apigee Routing Logic**, and **The BigQuery Pricing Model**.

Here is your refactoring guide:

---

### 1. Refactor the CLI (`ask-claude` → `ai-hub` CLI)
The CLI needs to allow users to choose their "engine."

*   **Change:** Add a `--model` (or `-m`) flag.
*   **Default:** Set a sensible default (e.g., `gemini-1.5-flash` for cheap tasks or `claude-3-5-sonnet` for coding).
*   **Payload Change:** The CLI should send a header `x-model-id` (e.g., `gemini-pro`, `claude-sonnet`, `claude-opus`).

```bash
# Example Commands
ai-hub "Quick summary of this" --model gemini-flash  # Fast & Cheap
ai-hub "Write this complex app" --model claude-sonnet # High Intelligence
```

---

### 2. Refactor Apigee: The "Model Router" Pattern
Currently, your `TargetEndpoint` is hardcoded to Claude. You need to move to **Dynamic Routing**.

#### A. Define Multiple Target Servers
In Apigee, define "Target Servers" for the different model families because Gemini and Claude have different base URLs.

#### B. Conditional Policy Execution
You must "switch" your logic based on the model requested. Gemini and Claude return token counts in different JSON paths.

**Refactored `AssignMessage-Response` (Multi-Model Version):**
```xml
<AssignVariables name="Extract-Tokens">
    <!-- IF CLAUDE -->
    <JSONPayload condition='request.header.x-model-id ~~ "claude*"'>
        <Variable name="tokens.input" type="integer">
            <JSONPath>$.usage.input_tokens</JSONPath>
        </Variable>
        <Variable name="tokens.output" type="integer">
            <JSONPath>$.usage.output_tokens</JSONPath>
        </Variable>
    </JSONPayload>

    <!-- IF GEMINI -->
    <JSONPayload condition='request.header.x-model-id ~~ "gemini*"'>
        <Variable name="tokens.input" type="integer">
            <JSONPath>$.usage.promptTokenCount</JSONPath>
        </Variable>
        <Variable name="tokens.output" type="integer">
            <JSONPath>$.usage.candidatesTokenCount</JSONPath>
        </Variable>
    </JSONPayload>
</AssignVariables>
```

#### C. Dynamic Target Routing
In your `RouteRule`, decide which backend to hit:
```xml
<RouteRule name="route-to-gemini">
    <Condition>request.header.x-model-id ~~ "gemini*"</Condition>
    <TargetEndpoint>gemini-backend</TargetEndpoint>
</RouteRule>

<RouteRule name="route-to-claude">
    <Condition>request.header.x-model-id ~~ "claude*"</Condition>
    <TargetEndpoint>claude-backend</TargetEndpoint>
</RouteRule>
```

---

### 3. Refactor BigQuery: The "Inference Pricing Table"
With multiple models, you can no longer hardcode costs in SQL because Gemini Flash costs $0.07 per 1M tokens while Claude Opus costs $15.00 per 1M tokens.

**The Refactor:** Create a `model_pricing` lookup table in BigQuery.

| model_id | input_price_per_1M | output_price_per_1M | provider |
| :--- | :--- | :--- | :--- |
| claude-3-5-sonnet | $3.00 | $15.00 | Anthropic |
| claude-3-opus | $15.00 | $75.00 | Anthropic |
| gemini-1.5-flash | $0.075 | $0.30 | Google |
| gemini-1.5-pro | $3.50 | $10.50 | Google |

**Refactored Looker Studio SQL:**
```sql
SELECT 
    l.consumer_team,
    l.model_version,
    SUM(l.input_tokens) as total_in,
    SUM(l.output_tokens) as total_out,
    -- JOIN with Pricing Table
    SUM(l.input_tokens / 1000000 * p.input_price_per_1M) + 
    SUM(l.output_tokens / 1000000 * p.output_price_per_1M) as total_cost_usd
FROM `ai_token_ledger` l
JOIN `model_pricing` p ON l.model_version = p.model_id
GROUP BY 1, 2
```

---

### 4. Refactor Provisioned Throughput (PT)
*   **Gemini:** Uses "Provisioned Throughput" units (purchased in blocks of 100 or 1,000 TPM).
*   **Claude:** In Vertex AI, this is often handled as "Dedicated Capacity."
*   **The Hub Strategy:** You can now have a **Mixed Billing Strategy**. 
    *   Use **PT** for your high-volume, "workhorse" model (like Gemini Flash or Claude Sonnet).
    *   Use **PAYG** for your low-volume, high-intelligence model (like Claude Opus).
    *   Apigee will route the traffic seamlessly, and your users don't need to know which is which.

---

### 5. Critical Technical Challenge: Payload Abstraction
This is the biggest decision for your refactor:

*   **Option A (Passthrough):** The CLI user must send "Claude-formatted JSON" or "Gemini-formatted JSON." 
    *   *Pros:* Users get full access to model-specific features.
    *   *Cons:* CLI code gets messy; users have to learn two different API formats.
*   **Option B (Standardized Hub Schema):** You define a simple **Internal AI Hub Schema** (e.g., `{"prompt": "...", "context": "..."}`). Apigee uses a **JavaScript Policy** to "translate" your internal schema into the Gemini format or the Claude format.
    *   *Pros:* Extremely easy for teams to switch models. They just change one word in the CLI flag.
    *   *Cons:* Advanced features (like Gemini's "System Instructions" or Claude's "Tool Use") might get lost in translation.

**My Recommendation:** Start with **Option A (Passthrough)** but use a common header for the Model ID. As you grow, build the translation layer in Apigee for the most common use cases.

### Summary Checklist for Multi-Model:
1.  [ ] **GCP:** Enable Gemini 1.5 Pro/Flash APIs in `pr-ai-hub-shared`.
2.  [ ] **Apigee:** Create separate TargetEndpoints for Google (Gemini) and Anthropic (Claude).
3.  [ ] **Apigee:** Use `Condition` tags to switch token-extraction logic (Claude `usage` vs Gemini `usageMetadata`).
4.  [ ] **BigQuery:** Add a `model_id` column to your ledger and a separate `pricing` lookup table.
5.  [ ] **CLI:** Add the `--model` flag.