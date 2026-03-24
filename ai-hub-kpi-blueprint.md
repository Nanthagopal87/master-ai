<!DOCTYPE html>
<html lang="en">
<head>
<meta charset="UTF-8"/>
<meta name="viewport" content="width=device-width, initial-scale=1.0"/>
<title>Enterprise AI Hub — KPI Blueprint</title>
<link href="https://fonts.googleapis.com/css2?family=Syne:wght@400;600;700;800&family=DM+Mono:wght@300;400;500&family=DM+Sans:wght@300;400;500&display=swap" rel="stylesheet"/>
<style>
  :root {
    --bg:        #0a0c10;
    --surface:   #111318;
    --surface2:  #181c24;
    --border:    #1e2330;
    --accent:    #00e5ff;
    --accent2:   #7b61ff;
    --accent3:   #00ffb2;
    --warn:      #ffb800;
    --danger:    #ff4d6d;
    --text:      #e8eaf0;
    --muted:     #6b7280;
    --green:     #00ffb2;
    --radius:    12px;
  }

  *, *::before, *::after { box-sizing: border-box; margin: 0; padding: 0; }

  body {
    background: var(--bg);
    color: var(--text);
    font-family: 'DM Sans', sans-serif;
    font-size: 14px;
    line-height: 1.6;
    min-height: 100vh;
  }

  /* Background grid */
  body::before {
    content: '';
    position: fixed;
    inset: 0;
    background-image:
      linear-gradient(rgba(0,229,255,0.03) 1px, transparent 1px),
      linear-gradient(90deg, rgba(0,229,255,0.03) 1px, transparent 1px);
    background-size: 40px 40px;
    pointer-events: none;
    z-index: 0;
  }

  .page { position: relative; z-index: 1; max-width: 1200px; margin: 0 auto; padding: 40px 24px 80px; }

  /* Header */
  .header {
    display: flex;
    align-items: flex-start;
    justify-content: space-between;
    margin-bottom: 48px;
    padding-bottom: 32px;
    border-bottom: 1px solid var(--border);
    flex-wrap: wrap;
    gap: 16px;
  }
  .header-left { display: flex; flex-direction: column; gap: 8px; }
  .eyebrow {
    font-family: 'DM Mono', monospace;
    font-size: 11px;
    font-weight: 500;
    letter-spacing: 0.15em;
    color: var(--accent);
    text-transform: uppercase;
  }
  .title {
    font-family: 'Syne', sans-serif;
    font-size: 36px;
    font-weight: 800;
    line-height: 1.1;
    color: #fff;
    letter-spacing: -0.02em;
  }
  .title span { color: var(--accent); }
  .subtitle { color: var(--muted); font-size: 14px; margin-top: 4px; }
  .header-meta {
    display: flex;
    flex-direction: column;
    align-items: flex-end;
    gap: 6px;
  }
  .badge {
    font-family: 'DM Mono', monospace;
    font-size: 11px;
    padding: 4px 10px;
    border-radius: 20px;
    font-weight: 500;
    letter-spacing: 0.05em;
  }
  .badge-blue  { background: rgba(0,229,255,0.1);  color: var(--accent);  border: 1px solid rgba(0,229,255,0.2); }
  .badge-green { background: rgba(0,255,178,0.1);  color: var(--green);   border: 1px solid rgba(0,255,178,0.2); }
  .badge-purple{ background: rgba(123,97,255,0.1); color: var(--accent2); border: 1px solid rgba(123,97,255,0.2); }
  .badge-warn  { background: rgba(255,184,0,0.1);  color: var(--warn);    border: 1px solid rgba(255,184,0,0.2); }
  .badge-red   { background: rgba(255,77,109,0.1); color: var(--danger);  border: 1px solid rgba(255,77,109,0.2); }

  /* Section headers */
  .section { margin-bottom: 48px; }
  .section-header {
    display: flex;
    align-items: center;
    gap: 12px;
    margin-bottom: 20px;
  }
  .section-icon {
    width: 36px; height: 36px;
    border-radius: 8px;
    display: flex; align-items: center; justify-content: center;
    font-size: 16px;
    flex-shrink: 0;
  }
  .icon-blue   { background: rgba(0,229,255,0.12);  }
  .icon-green  { background: rgba(0,255,178,0.12);  }
  .icon-purple { background: rgba(123,97,255,0.12); }
  .icon-warn   { background: rgba(255,184,0,0.12);  }

  .section-title {
    font-family: 'Syne', sans-serif;
    font-size: 18px;
    font-weight: 700;
    color: #fff;
  }
  .section-desc { color: var(--muted); font-size: 13px; margin-top: 2px; }

  /* KPI Cards grid */
  .cards-grid {
    display: grid;
    grid-template-columns: repeat(auto-fill, minmax(260px, 1fr));
    gap: 16px;
    margin-bottom: 24px;
  }

  .kpi-card {
    background: var(--surface);
    border: 1px solid var(--border);
    border-radius: var(--radius);
    padding: 20px;
    position: relative;
    overflow: hidden;
    transition: border-color 0.2s, transform 0.2s;
  }
  .kpi-card:hover { border-color: rgba(0,229,255,0.3); transform: translateY(-2px); }
  .kpi-card::before {
    content: '';
    position: absolute;
    top: 0; left: 0; right: 0;
    height: 2px;
  }
  .card-blue::before   { background: linear-gradient(90deg, var(--accent), transparent); }
  .card-green::before  { background: linear-gradient(90deg, var(--green), transparent); }
  .card-purple::before { background: linear-gradient(90deg, var(--accent2), transparent); }
  .card-warn::before   { background: linear-gradient(90deg, var(--warn), transparent); }

  .kpi-label {
    font-family: 'DM Mono', monospace;
    font-size: 11px;
    letter-spacing: 0.08em;
    color: var(--muted);
    text-transform: uppercase;
    margin-bottom: 10px;
  }
  .kpi-value {
    font-family: 'Syne', sans-serif;
    font-size: 28px;
    font-weight: 800;
    line-height: 1;
    margin-bottom: 6px;
  }
  .kpi-value.blue   { color: var(--accent); }
  .kpi-value.green  { color: var(--green); }
  .kpi-value.purple { color: var(--accent2); }
  .kpi-value.warn   { color: var(--warn); }

  .kpi-target {
    font-size: 12px;
    color: var(--muted);
    margin-bottom: 12px;
  }
  .kpi-target strong { color: var(--text); }

  .kpi-desc {
    font-size: 12px;
    color: var(--muted);
    line-height: 1.5;
    padding-top: 12px;
    border-top: 1px solid var(--border);
  }

  /* Progress bar */
  .progress-wrap { margin-bottom: 12px; }
  .progress-bar {
    height: 4px;
    background: var(--border);
    border-radius: 4px;
    overflow: hidden;
    margin-top: 6px;
  }
  .progress-fill {
    height: 100%;
    border-radius: 4px;
    transition: width 1s ease;
  }
  .fill-blue   { background: linear-gradient(90deg, var(--accent), rgba(0,229,255,0.4)); }
  .fill-green  { background: linear-gradient(90deg, var(--green), rgba(0,255,178,0.4)); }
  .fill-purple { background: linear-gradient(90deg, var(--accent2), rgba(123,97,255,0.4)); }
  .fill-warn   { background: linear-gradient(90deg, var(--warn), rgba(255,184,0,0.4)); }

  /* Detail table */
  .detail-table-wrap {
    background: var(--surface);
    border: 1px solid var(--border);
    border-radius: var(--radius);
    overflow: hidden;
  }
  .detail-table-wrap table { width: 100%; border-collapse: collapse; }
  .detail-table-wrap thead tr {
    background: var(--surface2);
    border-bottom: 1px solid var(--border);
  }
  .detail-table-wrap th {
    font-family: 'DM Mono', monospace;
    font-size: 11px;
    letter-spacing: 0.08em;
    color: var(--muted);
    text-transform: uppercase;
    padding: 12px 16px;
    text-align: left;
    font-weight: 500;
  }
  .detail-table-wrap td {
    padding: 12px 16px;
    font-size: 13px;
    color: var(--text);
    border-bottom: 1px solid var(--border);
    vertical-align: top;
  }
  .detail-table-wrap tr:last-child td { border-bottom: none; }
  .detail-table-wrap tr:hover td { background: rgba(255,255,255,0.02); }

  .td-kpi { font-weight: 500; color: #fff; }
  .td-target { font-family: 'DM Mono', monospace; font-size: 12px; }
  .td-source { font-family: 'DM Mono', monospace; font-size: 11px; color: var(--muted); }
  .td-owner { font-size: 12px; color: var(--muted); }
  .td-freq { }

  /* Divider */
  .divider {
    height: 1px;
    background: var(--border);
    margin: 48px 0;
  }

  /* Legend */
  .legend {
    display: flex;
    flex-wrap: wrap;
    gap: 16px;
    margin-top: 12px;
  }
  .legend-item {
    display: flex;
    align-items: center;
    gap: 8px;
    font-size: 12px;
    color: var(--muted);
  }
  .legend-dot {
    width: 8px; height: 8px;
    border-radius: 50%;
    flex-shrink: 0;
  }

  /* Phase timeline strip */
  .phase-strip {
    display: grid;
    grid-template-columns: repeat(5, 1fr);
    gap: 8px;
    margin-bottom: 32px;
  }
  .phase-item {
    background: var(--surface);
    border: 1px solid var(--border);
    border-radius: 8px;
    padding: 12px 14px;
    text-align: center;
  }
  .phase-num {
    font-family: 'DM Mono', monospace;
    font-size: 10px;
    color: var(--muted);
    letter-spacing: 0.08em;
    margin-bottom: 4px;
  }
  .phase-label {
    font-size: 11px;
    font-weight: 500;
    color: var(--text);
    line-height: 1.3;
  }
  .phase-weeks {
    font-family: 'DM Mono', monospace;
    font-size: 10px;
    color: var(--accent);
    margin-top: 4px;
  }

  /* Footer */
  .footer {
    margin-top: 64px;
    padding-top: 24px;
    border-top: 1px solid var(--border);
    display: flex;
    justify-content: space-between;
    align-items: center;
    flex-wrap: wrap;
    gap: 12px;
  }
  .footer-left { font-size: 12px; color: var(--muted); }
  .footer-right { font-family: 'DM Mono', monospace; font-size: 11px; color: var(--muted); }

  /* Animate on load */
  @keyframes fadeUp {
    from { opacity: 0; transform: translateY(16px); }
    to   { opacity: 1; transform: translateY(0); }
  }
  .section { animation: fadeUp 0.5s ease both; }
  .section:nth-child(1) { animation-delay: 0.05s; }
  .section:nth-child(2) { animation-delay: 0.10s; }
  .section:nth-child(3) { animation-delay: 0.15s; }
  .section:nth-child(4) { animation-delay: 0.20s; }
  .section:nth-child(5) { animation-delay: 0.25s; }
  .section:nth-child(6) { animation-delay: 0.30s; }

  @media (max-width: 640px) {
    .title { font-size: 26px; }
    .phase-strip { grid-template-columns: 1fr 1fr; }
    .cards-grid { grid-template-columns: 1fr; }
  }
</style>
</head>
<body>
<div class="page">

  <!-- Header -->
  <div class="header">
    <div class="header-left">
      <div class="eyebrow">// Enterprise AI Hub</div>
      <div class="title">KPI <span>Blueprint</span></div>
      <div class="subtitle">Centralized LLM Gateway · pr-ai-hub-shared · GCP + Apigee + Vertex AI</div>
    </div>
    <div class="header-meta">
      <span class="badge badge-blue">Claude 3.5 Sonnet</span>
      <span class="badge badge-green">10-Week Rollout</span>
      <span class="badge badge-purple">4 Pillars · 24 KPIs</span>
    </div>
  </div>

  <!-- Phase Timeline -->
  <div class="section">
    <div class="section-header">
      <div class="section-icon icon-blue">📅</div>
      <div>
        <div class="section-title">Delivery Phases</div>
        <div class="section-desc">KPIs are measured progressively — each phase unlocks new metrics</div>
      </div>
    </div>
    <div class="phase-strip">
      <div class="phase-item">
        <div class="phase-num">PHASE 01</div>
        <div class="phase-label">Control Plane Foundation</div>
        <div class="phase-weeks">Wks 1–2</div>
      </div>
      <div class="phase-item">
        <div class="phase-num">PHASE 02</div>
        <div class="phase-label">AI Gateway Configuration</div>
        <div class="phase-weeks">Wks 3–4</div>
      </div>
      <div class="phase-item">
        <div class="phase-num">PHASE 03</div>
        <div class="phase-label">Observability & FinOps</div>
        <div class="phase-weeks">Wks 5–6</div>
      </div>
      <div class="phase-item">
        <div class="phase-num">PHASE 04</div>
        <div class="phase-label">Consumer Onboarding</div>
        <div class="phase-weeks">Wks 7–8</div>
      </div>
      <div class="phase-item">
        <div class="phase-num">PHASE 05</div>
        <div class="phase-label">FinOps & Hardening</div>
        <div class="phase-weeks">Wks 9–10</div>
      </div>
    </div>
  </div>

  <!-- PILLAR 1: Technical / Platform Health -->
  <div class="section">
    <div class="section-header">
      <div class="section-icon icon-blue">⚡</div>
      <div>
        <div class="section-title">Pillar 1 — Technical & Platform Health</div>
        <div class="section-desc">Inference reliability, latency SLOs, and AI Gateway availability</div>
      </div>
    </div>

    <div class="cards-grid">
      <div class="kpi-card card-blue">
        <div class="kpi-label">AI Gateway Availability</div>
        <div class="kpi-value blue">99.9%</div>
        <div class="kpi-target">Target: <strong>≥ 99.9% uptime</strong></div>
        <div class="progress-wrap">
          <div class="progress-bar"><div class="progress-fill fill-blue" style="width:99%"></div></div>
        </div>
        <div class="kpi-desc">Apigee proxy uptime measured via Cloud Monitoring. Excludes planned maintenance windows.</div>
      </div>
      <div class="kpi-card card-blue">
        <div class="kpi-label">P95 Inference Latency</div>
        <div class="kpi-value blue">&lt; 3s</div>
        <div class="kpi-target">Target: <strong>≤ 3s end-to-end</strong></div>
        <div class="progress-wrap">
          <div class="progress-bar"><div class="progress-fill fill-blue" style="width:78%"></div></div>
        </div>
        <div class="kpi-desc">Gateway-to-model round-trip latency at P95. Measured via Apigee MessageLogging + BigQuery.</div>
      </div>
      <div class="kpi-card card-blue">
        <div class="kpi-label">P50 Inference Latency</div>
        <div class="kpi-value blue">&lt; 1.5s</div>
        <div class="kpi-target">Target: <strong>≤ 1.5s median</strong></div>
        <div class="progress-wrap">
          <div class="progress-bar"><div class="progress-fill fill-blue" style="width:85%"></div></div>
        </div>
        <div class="kpi-desc">Median response time across all consumer teams. Baseline established in Phase 3.</div>
      </div>
      <div class="kpi-card card-blue">
        <div class="kpi-label">Gateway Error Rate</div>
        <div class="kpi-value blue">&lt; 1%</div>
        <div class="kpi-target">Target: <strong>≤ 1% 5xx errors</strong></div>
        <div class="progress-wrap">
          <div class="progress-bar"><div class="progress-fill fill-blue" style="width:92%"></div></div>
        </div>
        <div class="kpi-desc">5xx error rate at Apigee layer. Excludes intentional 429s from quota enforcement.</div>
      </div>
      <div class="kpi-card card-blue">
        <div class="kpi-label">Quota 429 Rate</div>
        <div class="kpi-value blue">&lt; 2%</div>
        <div class="kpi-target">Target: <strong>≤ 2% across all teams</strong></div>
        <div class="progress-wrap">
          <div class="progress-bar"><div class="progress-fill fill-blue" style="width:88%"></div></div>
        </div>
        <div class="kpi-desc">Rate of 429 responses from per-consumer Quota Policies. High rate signals quota re-sizing is needed.</div>
      </div>
      <div class="kpi-card card-blue">
        <div class="kpi-label">Vertex AI Quota Utilization</div>
        <div class="kpi-value blue">75%</div>
        <div class="kpi-target">Target: <strong>Alert at &gt; 75%, Critical at &gt; 90%</strong></div>
        <div class="progress-wrap">
          <div class="progress-bar"><div class="progress-fill fill-blue" style="width:75%"></div></div>
        </div>
        <div class="kpi-desc">TPM utilization against the Vertex AI control plane quota ceiling. Monitored via Cloud Monitoring alerts.</div>
      </div>
    </div>

    <div class="detail-table-wrap">
      <table>
        <thead>
          <tr>
            <th>KPI</th>
            <th>Target</th>
            <th>Measurement Source</th>
            <th>Frequency</th>
            <th>Owner</th>
            <th>Active From</th>
          </tr>
        </thead>
        <tbody>
          <tr>
            <td class="td-kpi">AI Gateway Availability</td>
            <td class="td-target">≥ 99.9%</td>
            <td class="td-source">Cloud Monitoring / Apigee</td>
            <td>Real-time</td>
            <td class="td-owner">Platform / DevOps</td>
            <td><span class="badge badge-blue">Phase 2</span></td>
          </tr>
          <tr>
            <td class="td-kpi">P95 Inference Latency</td>
            <td class="td-target">≤ 3s</td>
            <td class="td-source">Apigee MessageLogging → BigQuery</td>
            <td>Daily</td>
            <td class="td-owner">Apigee Engineer</td>
            <td><span class="badge badge-blue">Phase 3</span></td>
          </tr>
          <tr>
            <td class="td-kpi">P50 Inference Latency</td>
            <td class="td-target">≤ 1.5s</td>
            <td class="td-source">Apigee MessageLogging → BigQuery</td>
            <td>Daily</td>
            <td class="td-owner">Apigee Engineer</td>
            <td><span class="badge badge-blue">Phase 3</span></td>
          </tr>
          <tr>
            <td class="td-kpi">Gateway Error Rate (5xx)</td>
            <td class="td-target">≤ 1%</td>
            <td class="td-source">Apigee Access Logs</td>
            <td>Real-time</td>
            <td class="td-owner">Platform / DevOps</td>
            <td><span class="badge badge-blue">Phase 2</span></td>
          </tr>
          <tr>
            <td class="td-kpi">Quota 429 Rate</td>
            <td class="td-target">≤ 2%</td>
            <td class="td-source">Apigee Access Logs</td>
            <td>Daily</td>
            <td class="td-owner">Apigee Engineer</td>
            <td><span class="badge badge-blue">Phase 2</span></td>
          </tr>
          <tr>
            <td class="td-kpi">Vertex AI Quota Utilization</td>
            <td class="td-target">Warn &gt;75%, Critical &gt;90%</td>
            <td class="td-source">Cloud Monitoring</td>
            <td>Real-time</td>
            <td class="td-owner">Cloud Architect</td>
            <td><span class="badge badge-blue">Phase 5</span></td>
          </tr>
        </tbody>
      </table>
    </div>
  </div>

  <div class="divider"></div>

  <!-- PILLAR 2: FinOps / Cost -->
  <div class="section">
    <div class="section-header">
      <div class="section-icon icon-green">💰</div>
      <div>
        <div class="section-title">Pillar 2 — FinOps & Cost Optimisation</div>
        <div class="section-desc">Token consumption, chargeback accuracy, and Provisioned Throughput efficiency</div>
      </div>
    </div>

    <div class="cards-grid">
      <div class="kpi-card card-green">
        <div class="kpi-label">PT Utilization</div>
        <div class="kpi-value green">&gt; 80%</div>
        <div class="kpi-target">Target: <strong>≥ 80% post-PT activation</strong></div>
        <div class="progress-wrap">
          <div class="progress-bar"><div class="progress-fill fill-green" style="width:80%"></div></div>
        </div>
        <div class="kpi-desc">Provisioned Throughput utilization against committed TPM. Below 80% = over-committed capacity.</div>
      </div>
      <div class="kpi-card card-green">
        <div class="kpi-label">Cost Reduction vs PAYG</div>
        <div class="kpi-value green">20–50%</div>
        <div class="kpi-target">Target: <strong>≥ 20% reduction at scale</strong></div>
        <div class="progress-wrap">
          <div class="progress-bar"><div class="progress-fill fill-green" style="width:70%"></div></div>
        </div>
        <div class="kpi-desc">Percentage cost savings post-PT adoption compared to equivalent PAYG spend. Validated by Finance.</div>
      </div>
      <div class="kpi-card card-green">
        <div class="kpi-label">Chargeback Coverage</div>
        <div class="kpi-value green">100%</div>
        <div class="kpi-target">Target: <strong>100% requests attributed</strong></div>
        <div class="progress-wrap">
          <div class="progress-bar"><div class="progress-fill fill-green" style="width:100%"></div></div>
        </div>
        <div class="kpi-desc">Percentage of inference requests with a valid team attribution in the AI Token Ledger (BigQuery).</div>
      </div>
      <div class="kpi-card card-green">
        <div class="kpi-label">Token Ledger Latency</div>
        <div class="kpi-value green">&lt; 60s</div>
        <div class="kpi-target">Target: <strong>≤ 60s ingestion lag</strong></div>
        <div class="progress-wrap">
          <div class="progress-bar"><div class="progress-fill fill-green" style="width:90%"></div></div>
        </div>
        <div class="kpi-desc">End-to-end latency from inference response to BigQuery record availability. Pub/Sub streaming pipeline.</div>
      </div>
      <div class="kpi-card card-green">
        <div class="kpi-label">Idle PT Waste</div>
        <div class="kpi-value green">&lt; 20%</div>
        <div class="kpi-target">Target: <strong>≤ 20% idle reserved capacity</strong></div>
        <div class="progress-wrap">
          <div class="progress-bar"><div class="progress-fill fill-green" style="width:82%"></div></div>
        </div>
        <div class="kpi-desc">Percentage of committed TPM sitting idle during business hours. Managed via SpikeArrest and baseline sizing.</div>
      </div>
      <div class="kpi-card card-green">
        <div class="kpi-label">Steady-State TPM Baseline</div>
        <div class="kpi-value green">50K+</div>
        <div class="kpi-target">Target: <strong>Established by end of Phase 3</strong></div>
        <div class="progress-wrap">
          <div class="progress-bar"><div class="progress-fill fill-green" style="width:60%"></div></div>
        </div>
        <div class="kpi-desc">Minimum sustained TPM across the enterprise during business hours. Used as the PT commitment target.</div>
      </div>
    </div>

    <div class="detail-table-wrap">
      <table>
        <thead>
          <tr>
            <th>KPI</th>
            <th>Target</th>
            <th>Measurement Source</th>
            <th>Frequency</th>
            <th>Owner</th>
            <th>Active From</th>
          </tr>
        </thead>
        <tbody>
          <tr>
            <td class="td-kpi">Provisioned Throughput Utilization</td>
            <td class="td-target">≥ 80%</td>
            <td class="td-source">BigQuery AI Token Ledger + PT ceiling</td>
            <td>Daily</td>
            <td class="td-owner">FinOps / Finance</td>
            <td><span class="badge badge-green">Phase 5</span></td>
          </tr>
          <tr>
            <td class="td-kpi">Cost Reduction vs PAYG Baseline</td>
            <td class="td-target">≥ 20%</td>
            <td class="td-source">GCP Billing + Token Ledger</td>
            <td>Monthly</td>
            <td class="td-owner">FinOps / Finance</td>
            <td><span class="badge badge-green">Phase 5</span></td>
          </tr>
          <tr>
            <td class="td-kpi">Chargeback Attribution Coverage</td>
            <td class="td-target">100%</td>
            <td class="td-source">BigQuery AI Token Ledger</td>
            <td>Daily</td>
            <td class="td-owner">Data Engineer</td>
            <td><span class="badge badge-green">Phase 3</span></td>
          </tr>
          <tr>
            <td class="td-kpi">Token Ledger Ingestion Latency</td>
            <td class="td-target">≤ 60s</td>
            <td class="td-source">Pub/Sub → BigQuery pipeline</td>
            <td>Real-time</td>
            <td class="td-owner">Data Engineer</td>
            <td><span class="badge badge-green">Phase 3</span></td>
          </tr>
          <tr>
            <td class="td-kpi">Idle PT Waste</td>
            <td class="td-target">≤ 20%</td>
            <td class="td-source">BigQuery + PT commitment data</td>
            <td>Weekly</td>
            <td class="td-owner">FinOps / Finance</td>
            <td><span class="badge badge-green">Phase 5</span></td>
          </tr>
          <tr>
            <td class="td-kpi">Steady-State TPM Baseline</td>
            <td class="td-target">Documented pre-PT</td>
            <td class="td-source">Looker Studio TPM profiling charts</td>
            <td>Weekly</td>
            <td class="td-owner">BI / Analytics</td>
            <td><span class="badge badge-green">Phase 3</span></td>
          </tr>
        </tbody>
      </table>
    </div>
  </div>

  <div class="divider"></div>

  <!-- PILLAR 3: Security & Compliance -->
  <div class="section">
    <div class="section-header">
      <div class="section-icon icon-warn">🔒</div>
      <div>
        <div class="section-title">Pillar 3 — Security & Compliance</div>
        <div class="section-desc">Zero-Trust posture, IAM hygiene, audit coverage, and access governance</div>
      </div>
    </div>

    <div class="cards-grid">
      <div class="kpi-card card-warn">
        <div class="kpi-label">Direct Vertex AI Access Paths</div>
        <div class="kpi-value warn">0</div>
        <div class="kpi-target">Target: <strong>Zero direct access paths</strong></div>
        <div class="progress-wrap">
          <div class="progress-bar"><div class="progress-fill fill-warn" style="width:100%"></div></div>
        </div>
        <div class="kpi-desc">Number of team-level IAM bindings with direct Vertex AI access. Must be zero after Phase 4 sign-off.</div>
      </div>
      <div class="kpi-card card-warn">
        <div class="kpi-label">API Key Rotation Compliance</div>
        <div class="kpi-value warn">100%</div>
        <div class="kpi-target">Target: <strong>All keys rotated ≤ 90 days</strong></div>
        <div class="progress-wrap">
          <div class="progress-bar"><div class="progress-fill fill-warn" style="width:100%"></div></div>
        </div>
        <div class="kpi-desc">Percentage of active consumer API Keys rotated within the 90-day policy window. Tracked via Apigee.</div>
      </div>
      <div class="kpi-card card-warn">
        <div class="kpi-label">Audit Log Coverage</div>
        <div class="kpi-value warn">100%</div>
        <div class="kpi-target">Target: <strong>All services emit audit logs</strong></div>
        <div class="progress-wrap">
          <div class="progress-bar"><div class="progress-fill fill-warn" style="width:100%"></div></div>
        </div>
        <div class="kpi-desc">Coverage of Cloud Audit Logs across Vertex AI, Apigee, BigQuery, and Artifact Registry.</div>
      </div>
      <div class="kpi-card card-warn">
        <div class="kpi-label">Unauthorized Access Attempts</div>
        <div class="kpi-value warn">0</div>
        <div class="kpi-target">Target: <strong>Zero unmitigated attempts</strong></div>
        <div class="progress-wrap">
          <div class="progress-bar"><div class="progress-fill fill-warn" style="width:100%"></div></div>
        </div>
        <div class="kpi-desc">401 events from invalid API Keys reviewed and triaged. Any spike triggers immediate key revocation.</div>
      </div>
      <div class="kpi-card card-warn">
        <div class="kpi-label">VPC Service Controls Status</div>
        <div class="kpi-value warn">Active</div>
        <div class="kpi-target">Target: <strong>Enforced from Phase 1</strong></div>
        <div class="progress-wrap">
          <div class="progress-bar"><div class="progress-fill fill-warn" style="width:100%"></div></div>
        </div>
        <div class="kpi-desc">VPC Service Controls perimeter status on Vertex AI endpoint. Must be active at all times — no public egress.</div>
      </div>
      <div class="kpi-card card-warn">
        <div class="kpi-label">Security Review Gates Passed</div>
        <div class="kpi-value warn">4 / 4</div>
        <div class="kpi-target">Target: <strong>All phase checkpoints cleared</strong></div>
        <div class="progress-wrap">
          <div class="progress-bar"><div class="progress-fill fill-warn" style="width:100%"></div></div>
        </div>
        <div class="kpi-desc">Security checkpoints across Phases 1, 2, 4, and 5. Each gate must be formally signed off before progression.</div>
      </div>
    </div>

    <div class="detail-table-wrap">
      <table>
        <thead>
          <tr>
            <th>KPI</th>
            <th>Target</th>
            <th>Measurement Source</th>
            <th>Frequency</th>
            <th>Owner</th>
            <th>Active From</th>
          </tr>
        </thead>
        <tbody>
          <tr>
            <td class="td-kpi">Direct Vertex AI IAM Bindings</td>
            <td class="td-target">= 0</td>
            <td class="td-source">GCP Audit Logs / IAM policy review</td>
            <td>Per phase gate</td>
            <td class="td-owner">Cloud Architect</td>
            <td><span class="badge badge-warn">Phase 4</span></td>
          </tr>
          <tr>
            <td class="td-kpi">API Key Rotation Compliance</td>
            <td class="td-target">100% within 90 days</td>
            <td class="td-source">Apigee Developer App metadata</td>
            <td>Monthly</td>
            <td class="td-owner">Apigee Engineer</td>
            <td><span class="badge badge-warn">Phase 2</span></td>
          </tr>
          <tr>
            <td class="td-kpi">Cloud Audit Log Coverage</td>
            <td class="td-target">100% services</td>
            <td class="td-source">Cloud Logging configuration audit</td>
            <td>Per phase gate</td>
            <td class="td-owner">Platform / DevOps</td>
            <td><span class="badge badge-warn">Phase 1</span></td>
          </tr>
          <tr>
            <td class="td-kpi">Unauthorized Access Attempts (401s)</td>
            <td class="td-target">0 unmitigated</td>
            <td class="td-source">Apigee Access Logs</td>
            <td>Real-time alert</td>
            <td class="td-owner">Platform / DevOps</td>
            <td><span class="badge badge-warn">Phase 2</span></td>
          </tr>
          <tr>
            <td class="td-kpi">VPC Service Controls Status</td>
            <td class="td-target">Always enforced</td>
            <td class="td-source">GCP VPC SC policy audit</td>
            <td>Continuous</td>
            <td class="td-owner">Cloud Architect</td>
            <td><span class="badge badge-warn">Phase 1</span></td>
          </tr>
          <tr>
            <td class="td-kpi">Security Review Gates Passed</td>
            <td class="td-target">4 / 4 phases</td>
            <td class="td-source">Phase sign-off documentation</td>
            <td>Per phase</td>
            <td class="td-owner">Cloud Architect</td>
            <td><span class="badge badge-warn">Phase 5</span></td>
          </tr>
        </tbody>
      </table>
    </div>
  </div>

  <div class="divider"></div>

  <!-- PILLAR 4: Consumer Adoption -->
  <div class="section">
    <div class="section-header">
      <div class="section-icon icon-purple">👥</div>
      <div>
        <div class="section-title">Pillar 4 — Consumer Adoption</div>
        <div class="section-desc">Team onboarding velocity, gateway traffic share, and CLI usage health</div>
      </div>
    </div>

    <div class="cards-grid">
      <div class="kpi-card card-purple">
        <div class="kpi-label">Teams Onboarded via Gateway</div>
        <div class="kpi-value purple">100%</div>
        <div class="kpi-target">Target: <strong>All teams by end of Phase 4</strong></div>
        <div class="progress-wrap">
          <div class="progress-bar"><div class="progress-fill fill-purple" style="width:100%"></div></div>
        </div>
        <div class="kpi-desc">Percentage of internal consumer teams routing all AI traffic through the Apigee AI Gateway.</div>
      </div>
      <div class="kpi-card card-purple">
        <div class="kpi-label">AI Gateway Traffic Share</div>
        <div class="kpi-value purple">100%</div>
        <div class="kpi-target">Target: <strong>100% North-South via Apigee</strong></div>
        <div class="progress-wrap">
          <div class="progress-bar"><div class="progress-fill fill-purple" style="width:100%"></div></div>
        </div>
        <div class="kpi-desc">Percentage of total Claude inference traffic flowing through the AI Gateway. Zero direct Vertex AI calls.</div>
      </div>
      <div class="kpi-card card-purple">
        <div class="kpi-label">Active CLI Users (weekly)</div>
        <div class="kpi-value purple">↑ Growth</div>
        <div class="kpi-target">Target: <strong>Week-on-week increase post Phase 4</strong></div>
        <div class="progress-wrap">
          <div class="progress-bar"><div class="progress-fill fill-purple" style="width:72%"></div></div>
        </div>
        <div class="kpi-desc">Unique API Keys making at least one request per week. Tracked via AI Token Ledger in BigQuery.</div>
      </div>
      <div class="kpi-card card-purple">
        <div class="kpi-label">Onboarding Time per Team</div>
        <div class="kpi-value purple">&lt; 2 days</div>
        <div class="kpi-target">Target: <strong>≤ 2 business days per team</strong></div>
        <div class="progress-wrap">
          <div class="progress-bar"><div class="progress-fill fill-purple" style="width:80%"></div></div>
        </div>
        <div class="kpi-desc">Time from onboarding request to first successful API call through the AI Gateway. Tracked per team.</div>
      </div>
      <div class="kpi-card card-purple">
        <div class="kpi-label">Quota Breach Rate per Team</div>
        <div class="kpi-value purple">&lt; 5%</div>
        <div class="kpi-target">Target: <strong>≤ 5% 429s per consumer</strong></div>
        <div class="progress-wrap">
          <div class="progress-bar"><div class="progress-fill fill-purple" style="width:88%"></div></div>
        </div>
        <div class="kpi-desc">Per-team rate of quota-related 429 rejections. Persistent breaches trigger quota policy review.</div>
      </div>
      <div class="kpi-card card-purple">
        <div class="kpi-label">Developer Portal Adoption</div>
        <div class="kpi-value purple">↑ Active</div>
        <div class="kpi-target">Target: <strong>Self-serve onboarding &gt; 80%</strong></div>
        <div class="progress-wrap">
          <div class="progress-bar"><div class="progress-fill fill-purple" style="width:75%"></div></div>
        </div>
        <div class="kpi-desc">Percentage of new teams onboarding via the Internal AI Developer Portal without manual intervention.</div>
      </div>
    </div>

    <div class="detail-table-wrap">
      <table>
        <thead>
          <tr>
            <th>KPI</th>
            <th>Target</th>
            <th>Measurement Source</th>
            <th>Frequency</th>
            <th>Owner</th>
            <th>Active From</th>
          </tr>
        </thead>
        <tbody>
          <tr>
            <td class="td-kpi">Teams Onboarded via AI Gateway</td>
            <td class="td-target">100% by Phase 4</td>
            <td class="td-source">Apigee Developer Apps registry</td>
            <td>Weekly</td>
            <td class="td-owner">Team Leads</td>
            <td><span class="badge badge-purple">Phase 4</span></td>
          </tr>
          <tr>
            <td class="td-kpi">AI Gateway Traffic Share</td>
            <td class="td-target">100% North-South</td>
            <td class="td-source">Apigee Access Logs + GCP Audit Logs</td>
            <td>Continuous</td>
            <td class="td-owner">Cloud Architect</td>
            <td><span class="badge badge-purple">Phase 4</span></td>
          </tr>
          <tr>
            <td class="td-kpi">Weekly Active CLI Users</td>
            <td class="td-target">WoW growth post Phase 4</td>
            <td class="td-source">BigQuery AI Token Ledger</td>
            <td>Weekly</td>
            <td class="td-owner">BI / Analytics</td>
            <td><span class="badge badge-purple">Phase 4</span></td>
          </tr>
          <tr>
            <td class="td-kpi">Team Onboarding Time</td>
            <td class="td-target">≤ 2 business days</td>
            <td class="td-source">Internal onboarding tracker</td>
            <td>Per onboarding</td>
            <td class="td-owner">Team Leads</td>
            <td><span class="badge badge-purple">Phase 4</span></td>
          </tr>
          <tr>
            <td class="td-kpi">Per-Team Quota Breach Rate</td>
            <td class="td-target">≤ 5% per consumer</td>
            <td class="td-source">Apigee Access Logs → BigQuery</td>
            <td>Weekly</td>
            <td class="td-owner">Apigee Engineer</td>
            <td><span class="badge badge-purple">Phase 2</span></td>
          </tr>
          <tr>
            <td class="td-kpi">Self-Serve Developer Portal Adoption</td>
            <td class="td-target">&gt; 80% self-serve</td>
            <td class="td-source">Developer Portal access logs</td>
            <td>Monthly</td>
            <td class="td-owner">Platform / DevOps</td>
            <td><span class="badge badge-purple">Phase 4</span></td>
          </tr>
        </tbody>
      </table>
    </div>
  </div>

  <div class="divider"></div>

  <!-- Summary Table -->
  <div class="section">
    <div class="section-header">
      <div class="section-icon icon-blue">📊</div>
      <div>
        <div class="section-title">KPI Summary — All Pillars</div>
        <div class="section-desc">Master reference across all 24 KPIs, owners, and phase activation</div>
      </div>
    </div>
    <div class="detail-table-wrap">
      <table>
        <thead>
          <tr>
            <th>Pillar</th>
            <th>KPI</th>
            <th>Target</th>
            <th>Phase</th>
            <th>Owner</th>
          </tr>
        </thead>
        <tbody>
          <tr><td><span class="badge badge-blue">Technical</span></td><td class="td-kpi">AI Gateway Availability</td><td class="td-target">≥ 99.9%</td><td><span class="badge badge-blue">P2</span></td><td class="td-owner">Platform / DevOps</td></tr>
          <tr><td><span class="badge badge-blue">Technical</span></td><td class="td-kpi">P95 Inference Latency</td><td class="td-target">≤ 3s</td><td><span class="badge badge-blue">P3</span></td><td class="td-owner">Apigee Engineer</td></tr>
          <tr><td><span class="badge badge-blue">Technical</span></td><td class="td-kpi">P50 Inference Latency</td><td class="td-target">≤ 1.5s</td><td><span class="badge badge-blue">P3</span></td><td class="td-owner">Apigee Engineer</td></tr>
          <tr><td><span class="badge badge-blue">Technical</span></td><td class="td-kpi">Gateway Error Rate (5xx)</td><td class="td-target">≤ 1%</td><td><span class="badge badge-blue">P2</span></td><td class="td-owner">Platform / DevOps</td></tr>
          <tr><td><span class="badge badge-blue">Technical</span></td><td class="td-kpi">Quota 429 Rate</td><td class="td-target">≤ 2%</td><td><span class="badge badge-blue">P2</span></td><td class="td-owner">Apigee Engineer</td></tr>
          <tr><td><span class="badge badge-blue">Technical</span></td><td class="td-kpi">Vertex AI Quota Utilization</td><td class="td-target">Alert &gt;75%</td><td><span class="badge badge-blue">P5</span></td><td class="td-owner">Cloud Architect</td></tr>
          <tr><td><span class="badge badge-green">FinOps</span></td><td class="td-kpi">Provisioned Throughput Utilization</td><td class="td-target">≥ 80%</td><td><span class="badge badge-green">P5</span></td><td class="td-owner">FinOps / Finance</td></tr>
          <tr><td><span class="badge badge-green">FinOps</span></td><td class="td-kpi">Cost Reduction vs PAYG</td><td class="td-target">≥ 20%</td><td><span class="badge badge-green">P5</span></td><td class="td-owner">FinOps / Finance</td></tr>
          <tr><td><span class="badge badge-green">FinOps</span></td><td class="td-kpi">Chargeback Attribution Coverage</td><td class="td-target">100%</td><td><span class="badge badge-green">P3</span></td><td class="td-owner">Data Engineer</td></tr>
          <tr><td><span class="badge badge-green">FinOps</span></td><td class="td-kpi">Token Ledger Ingestion Latency</td><td class="td-target">≤ 60s</td><td><span class="badge badge-green">P3</span></td><td class="td-owner">Data Engineer</td></tr>
          <tr><td><span class="badge badge-green">FinOps</span></td><td class="td-kpi">Idle PT Waste</td><td class="td-target">≤ 20%</td><td><span class="badge badge-green">P5</span></td><td class="td-owner">FinOps / Finance</td></tr>
          <tr><td><span class="badge badge-green">FinOps</span></td><td class="td-kpi">Steady-State TPM Baseline</td><td class="td-target">Documented pre-PT</td><td><span class="badge badge-green">P3</span></td><td class="td-owner">BI / Analytics</td></tr>
          <tr><td><span class="badge badge-warn">Security</span></td><td class="td-kpi">Direct Vertex AI IAM Bindings</td><td class="td-target">= 0</td><td><span class="badge badge-warn">P4</span></td><td class="td-owner">Cloud Architect</td></tr>
          <tr><td><span class="badge badge-warn">Security</span></td><td class="td-kpi">API Key Rotation Compliance</td><td class="td-target">100% ≤ 90 days</td><td><span class="badge badge-warn">P2</span></td><td class="td-owner">Apigee Engineer</td></tr>
          <tr><td><span class="badge badge-warn">Security</span></td><td class="td-kpi">Cloud Audit Log Coverage</td><td class="td-target">100% services</td><td><span class="badge badge-warn">P1</span></td><td class="td-owner">Platform / DevOps</td></tr>
          <tr><td><span class="badge badge-warn">Security</span></td><td class="td-kpi">Unauthorized Access Attempts</td><td class="td-target">0 unmitigated</td><td><span class="badge badge-warn">P2</span></td><td class="td-owner">Platform / DevOps</td></tr>
          <tr><td><span class="badge badge-warn">Security</span></td><td class="td-kpi">VPC Service Controls Status</td><td class="td-target">Always enforced</td><td><span class="badge badge-warn">P1</span></td><td class="td-owner">Cloud Architect</td></tr>
          <tr><td><span class="badge badge-warn">Security</span></td><td class="td-kpi">Security Review Gates Passed</td><td class="td-target">4 / 4</td><td><span class="badge badge-warn">P5</span></td><td class="td-owner">Cloud Architect</td></tr>
          <tr><td><span class="badge badge-purple">Adoption</span></td><td class="td-kpi">Teams Onboarded via AI Gateway</td><td class="td-target">100%</td><td><span class="badge badge-purple">P4</span></td><td class="td-owner">Team Leads</td></tr>
          <tr><td><span class="badge badge-purple">Adoption</span></td><td class="td-kpi">AI Gateway Traffic Share</td><td class="td-target">100%</td><td><span class="badge badge-purple">P4</span></td><td class="td-owner">Cloud Architect</td></tr>
          <tr><td><span class="badge badge-purple">Adoption</span></td><td class="td-kpi">Weekly Active CLI Users</td><td class="td-target">WoW growth</td><td><span class="badge badge-purple">P4</span></td><td class="td-owner">BI / Analytics</td></tr>
          <tr><td><span class="badge badge-purple">Adoption</span></td><td class="td-kpi">Team Onboarding Time</td><td class="td-target">≤ 2 days</td><td><span class="badge badge-purple">P4</span></td><td class="td-owner">Team Leads</td></tr>
          <tr><td><span class="badge badge-purple">Adoption</span></td><td class="td-kpi">Per-Team Quota Breach Rate</td><td class="td-target">≤ 5%</td><td><span class="badge badge-purple">P2</span></td><td class="td-owner">Apigee Engineer</td></tr>
          <tr><td><span class="badge badge-purple">Adoption</span></td><td class="td-kpi">Self-Serve Developer Portal Adoption</td><td class="td-target">&gt; 80%</td><td><span class="badge badge-purple">P4</span></td><td class="td-owner">Platform / DevOps</td></tr>
        </tbody>
      </table>
    </div>

    <div class="legend" style="margin-top: 16px;">
      <div class="legend-item"><div class="legend-dot" style="background:var(--accent)"></div>Technical / Platform Health</div>
      <div class="legend-item"><div class="legend-dot" style="background:var(--green)"></div>FinOps / Cost</div>
      <div class="legend-item"><div class="legend-dot" style="background:var(--warn)"></div>Security & Compliance</div>
      <div class="legend-item"><div class="legend-dot" style="background:var(--accent2)"></div>Consumer Adoption</div>
    </div>
  </div>

  <!-- Footer -->
  <div class="footer">
    <div class="footer-left">Enterprise AI Hub · KPI Blueprint · pr-ai-hub-shared · GCP + Apigee + Vertex AI (Claude 3.5 Sonnet)</div>
    <div class="footer-right">4 PILLARS · 24 KPIs · 10-WEEK ROLLOUT</div>
  </div>

</div>
</body>
</html>
