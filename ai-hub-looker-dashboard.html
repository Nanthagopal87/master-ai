<!DOCTYPE html>
<html lang="en">
<head>
<meta charset="UTF-8"/>
<meta name="viewport" content="width=device-width, initial-scale=1.0"/>
<title>Enterprise AI Hub — Looker Studio Dashboard</title>
<link href="https://fonts.googleapis.com/css2?family=Plus+Jakarta+Sans:wght@300;400;500;600;700;800&family=JetBrains+Mono:wght@300;400;500&display=swap" rel="stylesheet"/>
<script src="https://cdnjs.cloudflare.com/ajax/libs/Chart.js/4.4.1/chart.umd.min.js"></script>
<style>
:root {
  --bg:#f0f2f7; --surface:#ffffff; --surface2:#f8f9fc; --border:#e2e6ef;
  --text:#1a1f36; --muted:#6b7494;
  --blue:#2563eb; --blue-l:#eff4ff;
  --green:#059669; --green-l:#ecfdf5;
  --amber:#d97706; --amber-l:#fffbeb;
  --red:#dc2626; --red-l:#fef2f2;
  --purple:#7c3aed; --purple-l:#f5f3ff;
  --teal:#0891b2; --teal-l:#ecfeff;
  --radius:14px;
  --shadow:0 1px 3px rgba(0,0,0,.06),0 4px 16px rgba(0,0,0,.04);
  --shadow-md:0 4px 12px rgba(0,0,0,.08),0 8px 32px rgba(0,0,0,.06);
}
*,*::before,*::after{box-sizing:border-box;margin:0;padding:0}
body{background:var(--bg);color:var(--text);font-family:'Plus Jakarta Sans',sans-serif;font-size:13px;line-height:1.5}
.topbar{background:var(--surface);border-bottom:1px solid var(--border);padding:0 28px;height:56px;display:flex;align-items:center;justify-content:space-between;position:sticky;top:0;z-index:100;box-shadow:0 1px 4px rgba(0,0,0,.04)}
.topbar-left{display:flex;align-items:center;gap:12px}
.logo{width:30px;height:30px;background:var(--blue);border-radius:8px;display:flex;align-items:center;justify-content:center;font-size:15px}
.t-title{font-size:14px;font-weight:700;letter-spacing:-.01em}
.t-sub{font-size:11px;color:var(--muted);margin-top:1px}
.topbar-right{display:flex;align-items:center;gap:8px}
.chip{font-family:'JetBrains Mono',monospace;font-size:10px;padding:3px 9px;border-radius:20px;font-weight:500;letter-spacing:.03em}
.c-blue{background:var(--blue-l);color:var(--blue);border:1px solid #bfdbfe}
.c-green{background:var(--green-l);color:var(--green);border:1px solid #a7f3d0}
.c-amber{background:var(--amber-l);color:var(--amber);border:1px solid #fde68a}
.c-purple{background:var(--purple-l);color:var(--purple);border:1px solid #ddd6fe}
.c-red{background:var(--red-l);color:var(--red);border:1px solid #fecaca}
.c-teal{background:var(--teal-l);color:var(--teal);border:1px solid #a5f3fc}
.tabbar{background:var(--surface);border-bottom:1px solid var(--border);padding:0 28px;display:flex;gap:0;overflow-x:auto}
.tab{padding:12px 18px;font-size:12px;font-weight:600;color:var(--muted);cursor:pointer;border-bottom:2px solid transparent;transition:all .15s;white-space:nowrap}
.tab.active{color:var(--blue);border-bottom-color:var(--blue)}
.main{padding:24px 28px 60px;max-width:1360px;margin:0 auto}
.sec-lbl{font-family:'JetBrains Mono',monospace;font-size:10px;font-weight:500;letter-spacing:.12em;text-transform:uppercase;color:var(--muted);margin-bottom:12px;margin-top:32px;display:flex;align-items:center;gap:8px}
.sec-lbl::after{content:'';flex:1;height:1px;background:var(--border)}
.sc-row{display:grid;grid-template-columns:repeat(6,1fr);gap:12px;margin-bottom:4px}
.sc{background:var(--surface);border:1px solid var(--border);border-radius:var(--radius);padding:16px;box-shadow:var(--shadow);position:relative;overflow:hidden}
.sc::after{content:'';position:absolute;top:0;left:0;right:0;height:3px;border-radius:var(--radius) var(--radius) 0 0}
.sc-blue::after{background:var(--blue)}.sc-green::after{background:var(--green)}.sc-amber::after{background:var(--amber)}.sc-red::after{background:var(--red)}.sc-purple::after{background:var(--purple)}.sc-teal::after{background:var(--teal)}
.sc-lbl{font-size:10px;font-weight:600;color:var(--muted);text-transform:uppercase;letter-spacing:.06em;margin-bottom:8px}
.sc-val{font-size:22px;font-weight:800;color:var(--text);letter-spacing:-.02em;line-height:1}
.sc-d{font-size:11px;margin-top:6px;font-weight:500}
.up{color:var(--green)}.dn{color:var(--red)}.fl{color:var(--muted)}
.g2{display:grid;grid-template-columns:1fr 1fr;gap:16px}
.g3{display:grid;grid-template-columns:1fr 1fr 1fr;gap:16px}
.g21{display:grid;grid-template-columns:2fr 1fr;gap:16px}
.g32{display:grid;grid-template-columns:3fr 2fr;gap:16px}
.card{background:var(--surface);border:1px solid var(--border);border-radius:var(--radius);padding:20px 20px 16px;box-shadow:var(--shadow);margin-bottom:0;animation:fadeUp .4s ease both}
.card:hover{box-shadow:var(--shadow-md)}
.ch{display:flex;justify-content:space-between;align-items:flex-start;margin-bottom:16px}
.ct{font-size:13px;font-weight:700;color:var(--text)}
.cs{font-size:11px;color:var(--muted);margin-top:2px}
.cw{position:relative}.cw canvas{width:100%!important}
.hint{background:var(--blue-l);border:1px solid #bfdbfe;border-radius:10px;padding:9px 13px;font-size:11px;color:var(--blue);display:flex;align-items:flex-start;gap:7px;margin-top:10px}
.hint strong{font-weight:700}
.leg-row{display:flex;flex-wrap:wrap;gap:12px;margin-top:10px}
.leg{display:flex;align-items:center;gap:6px;font-size:11px;color:var(--muted)}
.ld{width:8px;height:8px;border-radius:50%;flex-shrink:0}
.pb-row{display:flex;align-items:center;gap:10px;margin-bottom:10px}
.pb-lbl{font-size:12px;width:130px;flex-shrink:0;font-weight:500;color:var(--text)}
.pb-bar{flex:1;height:8px;background:#e9ecf5;border-radius:4px;overflow:hidden}
.pb-fill{height:100%;border-radius:4px}
.pb-v{font-family:'JetBrains Mono',monospace;font-size:11px;color:var(--muted);width:42px;text-align:right;flex-shrink:0}
.mt{width:100%;border-collapse:collapse}
.mt th{font-size:10px;font-weight:600;color:var(--muted);text-transform:uppercase;letter-spacing:.06em;padding:6px 8px;text-align:left;border-bottom:1px solid var(--border)}
.mt td{padding:8px 8px;font-size:12px;border-bottom:1px solid #f3f4f8;color:var(--text)}
.mt tr:last-child td{border-bottom:none}
.mt tr:hover td{background:var(--surface2)}
.mb16{margin-bottom:16px}
@keyframes fadeUp{from{opacity:0;transform:translateY(12px)}to{opacity:1;transform:translateY(0)}}
@media(max-width:900px){.sc-row{grid-template-columns:repeat(3,1fr)}.g2,.g3,.g21,.g32{grid-template-columns:1fr}}
@media(max-width:560px){.sc-row{grid-template-columns:repeat(2,1fr)}.main{padding:16px 14px 40px}}
</style>
</head>
<body>
<div class="topbar">
  <div class="topbar-left">
    <div class="logo">🤖</div>
    <div>
      <div class="t-title">Enterprise AI Hub — Looker Studio Dashboard</div>
      <div class="t-sub">pr-ai-hub-shared · Claude 3.5 Sonnet · Apigee + Vertex AI · BigQuery Source</div>
    </div>
  </div>
  <div class="topbar-right">
    <span class="chip c-green">● Live</span>
    <span class="chip c-blue">Mar 2025</span>
    <span class="chip c-teal">BigQuery</span>
    <span class="chip c-purple">14 Charts · 4 Pillars</span>
  </div>
</div>

<div class="tabbar">
  <div class="tab active">📊 Overview</div>
  <div class="tab">⚡ Platform Health</div>
  <div class="tab">💰 FinOps</div>
  <div class="tab">🔒 Security</div>
  <div class="tab">👥 Adoption</div>
</div>

<div class="main">

<!-- SCORECARDS -->
<div class="sec-lbl">Scorecard — Key Metrics at a Glance</div>
<div class="sc-row">
  <div class="sc sc-blue"><div class="sc-lbl">Total Requests Today</div><div class="sc-val">284K</div><div class="sc-d up">↑ 12% vs yesterday</div></div>
  <div class="sc sc-green"><div class="sc-lbl">P95 Latency</div><div class="sc-val">2.1s</div><div class="sc-d up">↓ 0.3s vs last week</div></div>
  <div class="sc sc-teal"><div class="sc-lbl">Tokens This Month</div><div class="sc-val">4.2B</div><div class="sc-d up">↑ 23% MoM</div></div>
  <div class="sc sc-amber"><div class="sc-lbl">Est. Monthly Cost</div><div class="sc-val">$18.4K</div><div class="sc-d dn">↑ $2.1K vs budget</div></div>
  <div class="sc sc-purple"><div class="sc-lbl">Teams Onboarded</div><div class="sc-val">8 / 10</div><div class="sc-d fl">→ 2 in progress</div></div>
  <div class="sc sc-red"><div class="sc-lbl">Error Rate (5xx)</div><div class="sc-val">0.4%</div><div class="sc-d up">↓ 0.1% vs yesterday</div></div>
</div>

<!-- PILLAR 1 -->
<div class="sec-lbl">⚡ Pillar 1 — Technical & Platform Health</div>

<div class="g32 mb16">
  <div class="card">
    <div class="ch"><div><div class="ct">Daily Request Volume — All Teams (30 days)</div><div class="cs">Stacked Area · Source: Apigee Access Logs → BigQuery · ai_token_ledger table</div></div><span class="chip c-blue">TIME SERIES</span></div>
    <div class="cw" style="height:200px"><canvas id="c1"></canvas></div>
    <div class="hint">💡 <span><strong>Looker Studio:</strong> Time Series Chart — dimension: <code>request_date</code>, breakdown: <code>consumer_team</code>, metric: <code>COUNT(request_id)</code>. Enable stack + smoothing.</span></div>
  </div>
  <div class="card">
    <div class="ch"><div><div class="ct">P50 / P95 / P99 Latency (4-week)</div><div class="cs">Grouped Bar · Gateway-to-model round trip (ms)</div></div><span class="chip c-teal">LATENCY SLO</span></div>
    <div class="cw" style="height:200px"><canvas id="c2"></canvas></div>
    <div class="hint">💡 <span><strong>Looker Studio:</strong> Grouped Bar Chart — X: week, metrics: PERCENTILE(latency_ms, 50/95/99). Add reference line at 3000ms SLO.</span></div>
  </div>
</div>

<div class="g3 mb16">
  <div class="card">
    <div class="ch"><div><div class="ct">5xx Error Rate vs 429 Rate (14 days)</div><div class="cs">Dual-line · Daily trend</div></div><span class="chip c-red">ERRORS</span></div>
    <div class="cw" style="height:175px"><canvas id="c3"></canvas></div>
    <div class="hint">💡 <span><strong>Looker Studio:</strong> Time Series — two metrics: <code>5xx_rate</code> and <code>quota_429_rate</code>. Mark 1% SLO threshold line.</span></div>
  </div>
  <div class="card">
    <div class="ch"><div><div class="ct">Vertex AI Quota Utilization — TPM vs PT Ceiling</div><div class="cs">Area chart · Warn/critical thresholds</div></div><span class="chip c-amber">QUOTA</span></div>
    <div class="cw" style="height:175px"><canvas id="c4"></canvas></div>
    <div class="hint">💡 <span><strong>Looker Studio:</strong> Area Chart — <code>actual_tpm</code> vs PT ceiling constant. Shade warn band (75–90%) amber, critical band red.</span></div>
  </div>
  <div class="card">
    <div class="ch"><div><div class="ct">AI Gateway Availability % (weekly)</div><div class="cs">Bar chart · 99.9% SLO target</div></div><span class="chip c-green">UPTIME</span></div>
    <div class="cw" style="height:175px"><canvas id="c5"></canvas></div>
    <div class="hint">💡 <span><strong>Looker Studio:</strong> Bar Chart — weekly availability %. Conditional colour formatting: green ≥99.9%, red below. Add 99.9% reference line.</span></div>
  </div>
</div>

<!-- PILLAR 2 -->
<div class="sec-lbl">💰 Pillar 2 — FinOps & Cost Optimisation</div>

<div class="g21 mb16">
  <div class="card">
    <div class="ch"><div><div class="ct">Monthly Token Burn Rate by Team — Input vs Output</div><div class="cs">Stacked Bar · Source: AI Token Ledger (BigQuery) · input_tokens + output_tokens fields</div></div><span class="chip c-teal">CHARGEBACK</span></div>
    <div class="cw" style="height:210px"><canvas id="c6"></canvas></div>
    <div class="hint">💡 <span><strong>Looker Studio:</strong> Stacked Bar — dimension: <code>consumer_team</code>, metrics: <code>SUM(input_tokens)</code> and <code>SUM(output_tokens)</code>. Drill-down to daily for per-team chargeback invoicing.</span></div>
  </div>
  <div class="card">
    <div class="ch"><div><div class="ct">Cost Share by Team</div><div class="cs">Donut · This month's estimated spend</div></div><span class="chip c-purple">COST SPLIT</span></div>
    <div class="cw" style="height:170px"><canvas id="c7"></canvas></div>
    <div class="leg-row" id="donutLeg"></div>
    <div class="hint" style="margin-top:10px">💡 <span><strong>Looker Studio:</strong> Donut Chart — dimension: <code>team_name</code>, metric: <code>SUM(estimated_cost_usd)</code>. Show % + absolute labels.</span></div>
  </div>
</div>

<div class="g2 mb16">
  <div class="card">
    <div class="ch"><div><div class="ct">TPM Baseline Profiling — Hourly Average</div><div class="cs">Bar Heatmap · Identifies steady-state floor for PT sizing</div></div><span class="chip c-blue">PT SIZING INPUT</span></div>
    <div class="cw" style="height:210px"><canvas id="c8"></canvas></div>
    <div class="hint">💡 <span><strong>Looker Studio:</strong> Pivot Table with heatmap colouring — rows: <code>HOUR(request_timestamp)</code>, columns: <code>DAYOFWEEK</code>, value: <code>AVG(tpm)</code>. The TPM floor = PT baseline purchase target.</span></div>
  </div>
  <div class="card">
    <div class="ch"><div><div class="ct">PAYG vs Provisioned Throughput — Cost Projection</div><div class="cs">Dual-line · 6-month actual + forecast · Shows saving gap</div></div><span class="chip c-green">SAVINGS</span></div>
    <div class="cw" style="height:210px"><canvas id="c9"></canvas></div>
    <div class="hint">💡 <span><strong>Looker Studio:</strong> Dual-Line Chart — PAYG projected cost (red) vs PT flat rate (green). Shade saving gap between lines. Annotate PT activation month.</span></div>
  </div>
</div>

<!-- PILLAR 3 -->
<div class="sec-lbl">🔒 Pillar 3 — Security & Compliance</div>

<div class="g3 mb16">
  <div class="card">
    <div class="ch"><div><div class="ct">401 Unauthorized Events — Daily</div><div class="cs">Bar chart · Invalid or expired API Keys</div></div><span class="chip c-red">AUTH ANOMALY</span></div>
    <div class="cw" style="height:175px"><canvas id="c10"></canvas></div>
    <div class="hint">💡 <span><strong>Looker Studio:</strong> Bar Chart — daily <code>COUNT</code> of 401 events from Apigee logs. Conditional colour: red if &gt;threshold. Click-through drill to consumer API Key.</span></div>
  </div>
  <div class="card">
    <div class="ch"><div><div class="ct">API Key Age Distribution</div><div class="cs">Histogram · 90-day rotation policy compliance</div></div><span class="chip c-amber">KEY HYGIENE</span></div>
    <div class="cw" style="height:175px"><canvas id="c11"></canvas></div>
    <div class="hint">💡 <span><strong>Looker Studio:</strong> Bar Chart (histogram buckets by age range) — highlight bucket &gt;90 days in red as policy breach. Supports key rotation SLA tracking.</span></div>
  </div>
  <div class="card">
    <div class="ch"><div><div class="ct">Security Review Gate Status</div><div class="cs">Table · Phase-by-phase compliance tracker</div></div><span class="chip c-green">GATES</span></div>
    <div style="margin-top:4px">
      <table class="mt">
        <thead><tr><th>Phase</th><th>Security Gate</th><th>Status</th><th>Week</th></tr></thead>
        <tbody>
          <tr><td>Phase 1</td><td>VPC Service Controls</td><td><span class="chip c-green">✓ Passed</span></td><td>Wk 2</td></tr>
          <tr><td>Phase 2</td><td>API Key Rotation Policy</td><td><span class="chip c-green">✓ Passed</span></td><td>Wk 4</td></tr>
          <tr><td>Phase 4</td><td>Zero Direct IAM Bindings</td><td><span class="chip c-green">✓ Passed</span></td><td>Wk 8</td></tr>
          <tr><td>Phase 5</td><td>Zero-Trust Full Review</td><td><span class="chip c-amber">⏳ Pending</span></td><td>Wk 10</td></tr>
        </tbody>
      </table>
      <div class="hint" style="margin-top:10px">💡 <span><strong>Looker Studio:</strong> Table with conditional formatting on <code>status</code> field — green/amber/red. Filter by phase. Link to audit log evidence.</span></div>
    </div>
  </div>
</div>

<!-- PILLAR 4 -->
<div class="sec-lbl">👥 Pillar 4 — Consumer Adoption</div>

<div class="g2 mb16">
  <div class="card">
    <div class="ch"><div><div class="ct">Weekly Active API Keys — 8-Week Growth</div><div class="cs">Line chart · Post-Phase 4 onboarding momentum</div></div><span class="chip c-purple">WAU GROWTH</span></div>
    <div class="cw" style="height:195px"><canvas id="c12"></canvas></div>
    <div class="hint">💡 <span><strong>Looker Studio:</strong> Smooth Line Chart — X: week, Y: <code>COUNT(DISTINCT api_key)</code> with at least 1 request. Add Phase 4 annotation marker. Tracks adoption velocity.</span></div>
  </div>
  <div class="card">
    <div class="ch"><div><div class="ct">Requests per Team — This Month (Ranked)</div><div class="cs">Horizontal bar · Sorted descending by volume</div></div><span class="chip c-blue">USAGE RANK</span></div>
    <div class="cw" style="height:195px"><canvas id="c13"></canvas></div>
    <div class="hint">💡 <span><strong>Looker Studio:</strong> Horizontal Bar Chart — sorted by <code>COUNT(request_id)</code> DESC. Overlay quota limit as reference mark per team to visualise headroom.</span></div>
  </div>
</div>

<div class="g21 mb16">
  <div class="card">
    <div class="ch"><div><div class="ct">Per-Team Quota — Allocated RPM vs Peak RPM Used</div><div class="cs">Grouped bar · Headroom visibility per consumer</div></div><span class="chip c-amber">QUOTA HEADROOM</span></div>
    <div class="cw" style="height:200px"><canvas id="c14"></canvas></div>
    <div class="hint">💡 <span><strong>Looker Studio:</strong> Grouped Bar — <code>allocated_rpm</code> vs <code>peak_rpm_used</code> per team. Gap = headroom. Flag teams using &gt;80% for quota policy review.</span></div>
  </div>
  <div class="card">
    <div class="ch"><div><div class="ct">Team Onboarding Funnel</div><div class="cs">Progress bars · Phase 4 milestone completion</div></div><span class="chip c-green">ONBOARDING</span></div>
    <div style="margin-top:12px">
      <div class="pb-row"><div class="pb-lbl">API Key Issued</div><div class="pb-bar"><div class="pb-fill" style="width:100%;background:#2563eb"></div></div><div class="pb-v">10 / 10</div></div>
      <div class="pb-row"><div class="pb-lbl">First Request Made</div><div class="pb-bar"><div class="pb-fill" style="width:90%;background:#0891b2"></div></div><div class="pb-v">9 / 10</div></div>
      <div class="pb-row"><div class="pb-lbl">Fully Onboarded</div><div class="pb-bar"><div class="pb-fill" style="width:80%;background:#7c3aed"></div></div><div class="pb-v">8 / 10</div></div>
      <div class="pb-row"><div class="pb-lbl">Old Access Revoked</div><div class="pb-bar"><div class="pb-fill" style="width:80%;background:#059669"></div></div><div class="pb-v">8 / 10</div></div>
      <div class="pb-row"><div class="pb-lbl">Dashboard Access</div><div class="pb-bar"><div class="pb-fill" style="width:70%;background:#d97706"></div></div><div class="pb-v">7 / 10</div></div>
      <div class="hint" style="margin-top:14px">💡 <span><strong>Looker Studio:</strong> Funnel Chart or Scorecard Table — each milestone as a calculated field. Shows onboarding completion rate with drill-down to team name.</span></div>
    </div>
  </div>
</div>

</div><!-- /main -->

<script>
Chart.defaults.font.family="'Plus Jakarta Sans',sans-serif";
Chart.defaults.font.size=11;
Chart.defaults.color='#6b7494';
Chart.defaults.plugins.legend.display=false;

const d30=Array.from({length:30},(_,i)=>{const d=new Date('2025-02-01');d.setDate(d.getDate()+i);return d.toLocaleDateString('en',{month:'short',day:'numeric'})});
const w4=['Week 8','Week 9','Week 10','Week 11'];
const w8=['Wk 1','Wk 2','Wk 3','Wk 4','Wk 5','Wk 6','Wk 7','Wk 8'];
const teams=['Engineering','Marketing','Finance','HR','Data','Legal','Product','Ops'];
const gr=(o=.5)=>({color:`rgba(107,116,148,${o})`,lineWidth:.5});

// C1 — Stacked area: request volume
new Chart(document.getElementById('c1'),{type:'line',data:{labels:d30,datasets:[
  {label:'Engineering',data:d30.map((_,i)=>5000+Math.random()*3000+i*80),fill:true,backgroundColor:'rgba(37,99,235,.15)',borderColor:'#2563eb',borderWidth:1.5,tension:.4,pointRadius:0},
  {label:'Marketing',  data:d30.map((_,i)=>2000+Math.random()*1500+i*30),fill:true,backgroundColor:'rgba(124,58,237,.12)',borderColor:'#7c3aed',borderWidth:1.5,tension:.4,pointRadius:0},
  {label:'Finance',    data:d30.map((_,i)=>1200+Math.random()*800+i*15), fill:true,backgroundColor:'rgba(8,145,178,.12)', borderColor:'#0891b2',borderWidth:1.5,tension:.4,pointRadius:0},
  {label:'Others',     data:d30.map((_,i)=>800+Math.random()*600+i*10),  fill:true,backgroundColor:'rgba(5,150,105,.12)', borderColor:'#059669',borderWidth:1.5,tension:.4,pointRadius:0},
]},options:{responsive:true,maintainAspectRatio:false,interaction:{mode:'index',intersect:false},
  plugins:{legend:{display:true,position:'bottom',labels:{boxWidth:8,padding:14,font:{size:10}}},tooltip:{backgroundColor:'#1a1f36',padding:10}},
  scales:{x:{ticks:{maxTicksLimit:8,maxRotation:0},grid:gr()},y:{ticks:{callback:v=>(v/1000).toFixed(0)+'K'},grid:gr()}}}});

// C2 — Grouped bar: latency P50/P95/P99
new Chart(document.getElementById('c2'),{type:'bar',data:{labels:w4,datasets:[
  {label:'P50',data:[0.9,.85,.95,.88],backgroundColor:'rgba(8,145,178,.75)',borderRadius:4},
  {label:'P95',data:[2.4,2.2,2.5,2.1],backgroundColor:'rgba(37,99,235,.75)',borderRadius:4},
  {label:'P99',data:[3.8,3.5,4.1,3.3],backgroundColor:'rgba(220,38,38,.65)',borderRadius:4},
]},options:{responsive:true,maintainAspectRatio:false,
  plugins:{legend:{display:true,position:'bottom',labels:{boxWidth:8,padding:12,font:{size:10}}}},
  scales:{x:{grid:gr()},y:{min:0,max:5,ticks:{callback:v=>v+'s'},grid:gr()}}}});

// C3 — Dual line: error rates
const d14=d30.slice(0,14);
new Chart(document.getElementById('c3'),{type:'line',data:{labels:d14,datasets:[
  {label:'5xx %',data:[.8,.6,.9,.5,.4,.7,.5,.4,.6,.4,.3,.5,.4,.4],borderColor:'#dc2626',backgroundColor:'rgba(220,38,38,.08)',fill:true,tension:.4,borderWidth:2,pointRadius:3,pointBackgroundColor:'#dc2626'},
  {label:'429 %',data:[2.1,1.8,2.4,1.9,1.5,1.7,1.4,1.6,1.3,1.8,1.2,1.5,1.4,1.3],borderColor:'#d97706',backgroundColor:'rgba(217,119,6,.08)',fill:true,tension:.4,borderWidth:2,pointRadius:3,pointBackgroundColor:'#d97706'},
]},options:{responsive:true,maintainAspectRatio:false,interaction:{mode:'index',intersect:false},
  plugins:{legend:{display:true,position:'bottom',labels:{boxWidth:8,padding:12,font:{size:10}}},tooltip:{backgroundColor:'#1a1f36'}},
  scales:{x:{ticks:{maxRotation:0},grid:gr()},y:{ticks:{callback:v=>v+'%'},grid:gr()}}}});

// C4 — Area: quota utilization
new Chart(document.getElementById('c4'),{type:'line',data:{labels:d30.slice(0,20),datasets:[
  {label:'Actual TPM (K)',data:[38,42,45,50,48,55,62,58,65,70,68,72,66,74,71,75,69,73,78,76],fill:true,backgroundColor:'rgba(37,99,235,.1)',borderColor:'#2563eb',borderWidth:2,tension:.4,pointRadius:0},
  {label:'PT Ceiling 80K',data:Array(20).fill(80),borderColor:'#059669',borderWidth:1.5,borderDash:[5,4],pointRadius:0},
  {label:'Warn 75%',      data:Array(20).fill(60),borderColor:'#d97706',borderWidth:1,borderDash:[3,3],pointRadius:0},
]},options:{responsive:true,maintainAspectRatio:false,interaction:{mode:'index',intersect:false},
  plugins:{legend:{display:true,position:'bottom',labels:{boxWidth:8,padding:12,font:{size:10}}},tooltip:{backgroundColor:'#1a1f36'}},
  scales:{x:{ticks:{maxTicksLimit:6,maxRotation:0},grid:gr()},y:{ticks:{callback:v=>v+'K'},grid:gr()}}}});

// C5 — Bar: availability
new Chart(document.getElementById('c5'),{type:'bar',data:{labels:w8,datasets:[
  {label:'Availability %',data:[100,99.95,100,99.92,100,100,99.97,100],
    backgroundColor:[100,99.95,100,99.92,100,100,99.97,100].map(v=>v>=99.9?'rgba(5,150,105,.75)':'rgba(220,38,38,.75)'),borderRadius:5}
]},options:{responsive:true,maintainAspectRatio:false,
  plugins:{legend:{display:false},tooltip:{callbacks:{label:ctx=>`${ctx.raw.toFixed(2)}%`}}},
  scales:{x:{grid:gr()},y:{min:99.8,max:100.1,ticks:{callback:v=>v.toFixed(1)+'%'},grid:gr()}}}});

// C6 — Stacked bar: token burn
new Chart(document.getElementById('c6'),{type:'bar',data:{labels:teams,datasets:[
  {label:'Input Tokens (M)', data:[320,180,95,60,140,40,110,75],backgroundColor:'rgba(37,99,235,.75)',borderRadius:4},
  {label:'Output Tokens (M)',data:[280,150,80,50,120,35,95,65], backgroundColor:'rgba(8,145,178,.65)',borderRadius:4},
]},options:{responsive:true,maintainAspectRatio:false,
  plugins:{legend:{display:true,position:'bottom',labels:{boxWidth:8,padding:12,font:{size:10}}},tooltip:{backgroundColor:'#1a1f36'}},
  scales:{x:{stacked:true,grid:gr()},y:{stacked:true,ticks:{callback:v=>v+'M'},grid:gr()}}}});

// C7 — Donut: cost share
const costData=[5800,3200,2100,1400,2800,850,1600,650];
const costColors=['#2563eb','#7c3aed','#0891b2','#059669','#d97706','#dc2626','#6366f1','#84cc16'];
new Chart(document.getElementById('c7'),{type:'doughnut',data:{labels:teams,datasets:[
  {data:costData,backgroundColor:costColors,borderWidth:2,borderColor:'#fff',hoverOffset:6}
]},options:{responsive:true,maintainAspectRatio:false,cutout:'65%',
  plugins:{legend:{display:false},tooltip:{callbacks:{label:ctx=>`${ctx.label}: $${ctx.raw.toLocaleString()}`}}}}});
const dl=document.getElementById('donutLeg');
teams.forEach((t,i)=>{dl.innerHTML+=`<div class="leg"><div class="ld" style="background:${costColors[i]}"></div>${t}</div>`});

// C8 — Bar heatmap: hourly TPM
const hrs=['8AM','9AM','10AM','11AM','12PM','1PM','2PM','3PM','4PM','5PM','6PM'];
const tpmV=[28,52,68,74,58,62,71,76,65,42,18];
new Chart(document.getElementById('c8'),{type:'bar',data:{labels:hrs,datasets:[
  {label:'Avg TPM (K)',data:tpmV,backgroundColor:tpmV.map(v=>v<30?'rgba(5,150,105,.6)':v<60?'rgba(37,99,235,.65)':v<75?'rgba(217,119,6,.7)':'rgba(220,38,38,.75)'),borderRadius:5}
]},options:{responsive:true,maintainAspectRatio:false,
  plugins:{legend:{display:false},tooltip:{callbacks:{label:ctx=>`${ctx.raw}K TPM — ${ctx.raw>=75?'⚠ Near Ceiling':ctx.raw>=60?'High':'Normal'}`},backgroundColor:'#1a1f36'}},
  scales:{x:{grid:gr()},y:{ticks:{callback:v=>v+'K'},grid:gr()}}}});

// C9 — Dual line: PAYG vs PT
new Chart(document.getElementById('c9'),{type:'line',data:{labels:['Oct','Nov','Dec','Jan','Feb','Mar'],datasets:[
  {label:'PAYG (actual/projected)',data:[11200,13500,15800,18400,21000,24000],borderColor:'#dc2626',backgroundColor:'rgba(220,38,38,.08)',fill:true,tension:.3,borderWidth:2,pointRadius:4},
  {label:'Provisioned Throughput', data:[null,null,null,null,18000,18000],borderColor:'#059669',backgroundColor:'rgba(5,150,105,.08)',fill:true,tension:0,borderWidth:2.5,pointRadius:5},
]},options:{responsive:true,maintainAspectRatio:false,interaction:{mode:'index',intersect:false},
  plugins:{legend:{display:true,position:'bottom',labels:{boxWidth:8,padding:12,font:{size:10}}},tooltip:{backgroundColor:'#1a1f36',callbacks:{label:ctx=>`${ctx.dataset.label}: $${ctx.raw?.toLocaleString()??'N/A'}`}}},
  scales:{x:{grid:gr()},y:{ticks:{callback:v=>'$'+(v/1000).toFixed(0)+'K'},grid:gr()}}}});

// C10 — Bar: 401 auth events
new Chart(document.getElementById('c10'),{type:'bar',data:{labels:d14.slice(0,10),datasets:[
  {label:'401 Events',data:[14,8,22,6,4,18,5,3,9,4],
    backgroundColor:[14,8,22,6,4,18,5,3,9,4].map(v=>v>15?'rgba(220,38,38,.8)':'rgba(217,119,6,.65)'),borderRadius:4}
]},options:{responsive:true,maintainAspectRatio:false,
  plugins:{legend:{display:false},tooltip:{backgroundColor:'#1a1f36'}},
  scales:{x:{ticks:{maxRotation:0},grid:gr()},y:{grid:gr()}}}});

// C11 — Bar: key age histogram
new Chart(document.getElementById('c11'),{type:'bar',data:{labels:['0–30d','31–60d','61–90d','91–120d','>120d'],datasets:[
  {label:'Keys',data:[4,6,8,3,2],
    backgroundColor:['rgba(5,150,105,.75)','rgba(37,99,235,.75)','rgba(217,119,6,.75)','rgba(220,38,38,.8)','rgba(220,38,38,.95)'],borderRadius:5}
]},options:{responsive:true,maintainAspectRatio:false,
  plugins:{legend:{display:false},tooltip:{callbacks:{label:ctx=>`${ctx.raw} keys — ${ctx.dataIndex>=3?'⚠ Overdue':'Within policy'}`},backgroundColor:'#1a1f36'}},
  scales:{x:{grid:gr()},y:{ticks:{stepSize:2},grid:gr()}}}});

// C12 — Line: weekly active users
new Chart(document.getElementById('c12'),{type:'line',data:{labels:w8,datasets:[
  {label:'Active API Keys',data:[3,3,5,5,8,14,19,24],fill:true,backgroundColor:'rgba(124,58,237,.1)',borderColor:'#7c3aed',borderWidth:2.5,tension:.4,pointRadius:5,pointBackgroundColor:'#7c3aed'}
]},options:{responsive:true,maintainAspectRatio:false,
  plugins:{legend:{display:false},tooltip:{backgroundColor:'#1a1f36'}},
  scales:{x:{grid:gr()},y:{grid:gr(),ticks:{stepSize:4}}}}});

// C13 — Horizontal bar: team requests
new Chart(document.getElementById('c13'),{type:'bar',data:{labels:teams,datasets:[
  {label:'Requests (K)',data:[82,48,31,19,37,12,28,18],backgroundColor:'rgba(37,99,235,.7)',borderRadius:5}
]},options:{indexAxis:'y',responsive:true,maintainAspectRatio:false,
  plugins:{legend:{display:false},tooltip:{callbacks:{label:ctx=>`${ctx.raw}K requests`},backgroundColor:'#1a1f36'}},
  scales:{x:{ticks:{callback:v=>v+'K'},grid:gr()},y:{grid:{display:false}}}}});

// C14 — Grouped bar: quota headroom
new Chart(document.getElementById('c14'),{type:'bar',data:{labels:teams,datasets:[
  {label:'Allocated RPM',data:[1000,200,300,150,400,100,250,200],backgroundColor:'rgba(226,230,239,.9)',borderRadius:4},
  {label:'Peak RPM Used', data:[820,140,210,80,310,45,190,120],backgroundColor:'rgba(37,99,235,.75)',borderRadius:4},
]},options:{responsive:true,maintainAspectRatio:false,
  plugins:{legend:{display:true,position:'bottom',labels:{boxWidth:8,padding:12,font:{size:10}}},tooltip:{backgroundColor:'#1a1f36'}},
  scales:{x:{grid:gr()},y:{grid:gr()}}}});
</script>
</body>
</html>
