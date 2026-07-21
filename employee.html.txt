<!DOCTYPE html>
<html lang="en">
<head>
<meta charset="UTF-8">
<meta name="viewport" content="width=device-width, initial-scale=1.0">
<title>Employee Attrition Analysis — Dashboard</title>
<link rel="preconnect" href="https://fonts.googleapis.com">
<link rel="preconnect" href="https://fonts.gstatic.com" crossorigin>
<link href="https://fonts.googleapis.com/css2?family=Space+Grotesk:wght@400;500;600;700&family=Inter:wght@400;500;600&family=IBM+Plex+Mono:wght@400;500;600&display=swap" rel="stylesheet">
<style>
  :root{
    --bg:#0f1419;
    --panel:#161d24;
    --panel2:#1c262f;
    --border:#2a3640;
    --border-strong:#3a4854;
    --text:#e9edf1;
    --text-dim:#93a1ab;
    --text-faint:#5f6c76;
    --risk:#e2694a;
    --risk-dim:#5a3226;
    --safe:#4fb8a6;
    --safe-dim:#1f3d38;
    --gold:#e8b34c;
    --gold-dim:#4a3c1e;
  }
  *{box-sizing:border-box; margin:0; padding:0;}
  html{scroll-behavior:smooth;}
  body{
    background:var(--bg);
    color:var(--text);
    font-family:'Inter', sans-serif;
    line-height:1.55;
    -webkit-font-smoothing:antialiased;
  }
  h1,h2,h3,.display{ font-family:'Space Grotesk', sans-serif; letter-spacing:-0.01em;}
  .mono{ font-family:'IBM Plex Mono', monospace; }
  a{ color:inherit; }

  /* ---------- layout shells ---------- */
  .wrap{ max-width:1080px; margin:0 auto; padding:0 32px; }
  section{ padding:88px 0; border-top:1px solid var(--border); }
  section:first-of-type{ border-top:none; }
  .eyebrow{
    font-family:'IBM Plex Mono', monospace;
    font-size:12px;
    letter-spacing:0.14em;
    text-transform:uppercase;
    color:var(--text-faint);
    margin-bottom:14px;
  }
  .section-title{ font-size:30px; font-weight:600; margin-bottom:12px; }
  .section-lede{ font-size:16px; color:var(--text-dim); max-width:640px; margin-bottom:48px; }

  /* ---------- nav ---------- */
  nav{
    position:sticky; top:0; z-index:50;
    background:rgba(15,20,25,0.88);
    backdrop-filter:blur(10px);
    border-bottom:1px solid var(--border);
  }
  nav .wrap{ display:flex; align-items:center; justify-content:space-between; padding:16px 32px; }
  .nav-mark{ display:flex; align-items:center; gap:10px; font-weight:600; font-size:14px; }
  .nav-mark .dot{ width:8px; height:8px; border-radius:50%; background:var(--risk); box-shadow:0 0 0 3px var(--risk-dim); }
  .nav-links{ display:flex; gap:28px; }
  .nav-links a{ font-size:13px; color:var(--text-dim); text-decoration:none; transition:color .15s; }
  .nav-links a:hover{ color:var(--text); }

  /* ---------- hero ---------- */
  .hero{ padding:72px 0 60px; border-top:none; }
  .hero-tag{
    display:inline-flex; align-items:center; gap:8px;
    font-family:'IBM Plex Mono', monospace; font-size:12px; color:var(--risk);
    background:var(--risk-dim); border:1px solid #6b3a2a;
    padding:5px 12px; border-radius:100px; margin-bottom:24px;
  }
  .hero h1{ font-size:44px; font-weight:700; line-height:1.08; max-width:760px; margin-bottom:20px; }
  .hero p.lede{ font-size:17px; color:var(--text-dim); max-width:600px; margin-bottom:40px; }
  .meta-row{ display:flex; gap:36px; flex-wrap:wrap; margin-bottom:56px; }
  .meta-item .label{ font-size:11px; text-transform:uppercase; letter-spacing:0.1em; color:var(--text-faint); margin-bottom:6px; }
  .meta-item .val{ font-family:'IBM Plex Mono', monospace; font-size:15px; color:var(--text); }

  /* ---------- signal gauge (signature element) ---------- */
  .gauge-card{
    background:var(--panel); border:1px solid var(--border); border-radius:14px;
    padding:32px 36px;
  }
  .gauge-head{ display:flex; justify-content:space-between; align-items:baseline; margin-bottom:6px; flex-wrap:wrap; gap:8px;}
  .gauge-head h3{ font-size:16px; font-weight:600; }
  .gauge-head .sub{ font-size:13px; color:var(--text-faint); font-family:'IBM Plex Mono', monospace;}
  .gauge-track{
    position:relative; height:64px; margin:28px 0 10px;
    background:linear-gradient(90deg, var(--risk-dim) 0%, var(--risk-dim) 22%, #23303a 22%, #23303a 100%);
    border-radius:10px; border:1px solid var(--border-strong);
  }
  .noise-zone{
    position:absolute; left:0; top:0; bottom:0; width:22%;
    border-right:1px dashed #6b3a2a;
  }
  .noise-label{
    position:absolute; left:6px; top:6px; font-size:10px; font-family:'IBM Plex Mono', monospace;
    color:var(--risk); text-transform:uppercase; letter-spacing:.08em;
  }
  .marker{
    position:absolute; top:50%; transform:translate(-50%,-50%);
    display:flex; flex-direction:column; align-items:center; gap:4px;
  }
  .marker .pin{ width:14px; height:14px; border-radius:50%; border:3px solid var(--bg); box-shadow:0 0 0 1px var(--border-strong); }
  .marker.dt .pin{ background:var(--gold); }
  .marker.lr .pin{ background:var(--safe); }
  .marker .tag{
    position:absolute; top:26px; white-space:nowrap; font-size:11px;
    font-family:'IBM Plex Mono', monospace; color:var(--text-dim);
  }
  .gauge-scale{ display:flex; justify-content:space-between; font-size:11px; color:var(--text-faint); font-family:'IBM Plex Mono', monospace; margin-top:34px; }
  .gauge-foot{ margin-top:22px; font-size:13.5px; color:var(--text-dim); border-top:1px solid var(--border); padding-top:18px; }

  /* ---------- KPI grid ---------- */
  .kpi-grid{ display:grid; grid-template-columns:repeat(4,1fr); gap:14px; margin-bottom:8px; }
  .kpi{ background:var(--panel); border:1px solid var(--border); border-radius:12px; padding:20px 22px; }
  .kpi .label{ font-size:12px; color:var(--text-faint); text-transform:uppercase; letter-spacing:.08em; margin-bottom:10px; }
  .kpi .num{ font-family:'IBM Plex Mono', monospace; font-size:26px; font-weight:600; }
  .kpi .num.risk{ color:var(--risk); }
  .kpi .num.safe{ color:var(--safe); }
  .kpi .sub{ font-size:12.5px; color:var(--text-dim); margin-top:6px; }

  /* ---------- chart cards ---------- */
  .grid-2{ display:grid; grid-template-columns:1fr 1fr; gap:20px; }
  .card{ background:var(--panel); border:1px solid var(--border); border-radius:14px; padding:24px 26px; }
  .card h3{ font-size:15px; font-weight:600; margin-bottom:2px; }
  .card .card-sub{ font-size:12.5px; color:var(--text-faint); margin-bottom:18px; }
  .chart-box{ position:relative; width:100%; height:220px; }
  .chart-box.tall{ height:280px; }
  .full{ grid-column:1 / -1; }

  /* ---------- correlation heatmap ---------- */
  .heat-wrap{ overflow-x:auto; }
  table.heat{ border-collapse:collapse; width:100%; font-family:'IBM Plex Mono', monospace; font-size:11.5px; }
  table.heat th{ font-weight:500; color:var(--text-faint); padding:6px 8px; text-align:center; font-size:10.5px; }
  table.heat th.rowlabel, table.heat td.rowlabel{ text-align:left; color:var(--text-dim); padding-right:12px; white-space:nowrap; }
  table.heat td{ text-align:center; padding:9px 6px; color:#0f1419; border-radius:4px; }

  /* ---------- confusion matrices ---------- */
  .cm-grid{ display:grid; grid-template-columns:repeat(2,1fr); gap:20px; }
  .cm-table{ width:100%; border-collapse:separate; border-spacing:6px; margin-top:8px;}
  .cm-table td, .cm-table th{ text-align:center; font-family:'IBM Plex Mono', monospace; font-size:13px; padding:14px 8px; border-radius:8px; }
  .cm-table th{ background:transparent; color:var(--text-faint); font-size:10.5px; text-transform:uppercase; letter-spacing:.06em; font-weight:500; }
  .cm-tn, .cm-tp{ background:var(--safe-dim); color:var(--safe); }
  .cm-fp, .cm-fn{ background:var(--risk-dim); color:var(--risk); }

  /* ---------- metrics table ---------- */
  table.metrics{ width:100%; border-collapse:collapse; font-size:14px; }
  table.metrics th{ text-align:left; font-size:11px; text-transform:uppercase; letter-spacing:.07em; color:var(--text-faint); font-weight:500; padding:10px 12px; border-bottom:1px solid var(--border-strong); }
  table.metrics td{ padding:14px 12px; border-bottom:1px solid var(--border); font-family:'IBM Plex Mono', monospace; }
  table.metrics td.rowname{ font-family:'Inter', sans-serif; font-weight:500; }
  table.metrics tr:last-child td{ border-bottom:none; }

  /* ---------- findings / recs ---------- */
  .findings-list{ list-style:none; display:flex; flex-direction:column; gap:16px; }
  .findings-list li{
    display:flex; gap:14px; background:var(--panel); border:1px solid var(--border);
    border-radius:12px; padding:18px 20px; font-size:14.5px; color:var(--text-dim);
  }
  .findings-list li .bullet{ color:var(--risk); font-family:'IBM Plex Mono', monospace; flex-shrink:0; }
  .rec-grid{ display:grid; grid-template-columns:repeat(2,1fr); gap:16px; }
  .rec-card{ background:var(--panel2); border:1px solid var(--border); border-radius:12px; padding:20px 22px; }
  .rec-card .num{ font-family:'IBM Plex Mono', monospace; color:var(--gold); font-size:13px; margin-bottom:10px; }
  .rec-card p{ font-size:14px; color:var(--text-dim); }
  .rec-card strong{ color:var(--text); font-weight:500;}

  footer{ padding:48px 0 64px; text-align:center; color:var(--text-faint); font-size:12.5px; font-family:'IBM Plex Mono', monospace; }

  @media (max-width:760px){
    .kpi-grid{ grid-template-columns:repeat(2,1fr); }
    .grid-2, .cm-grid, .rec-grid{ grid-template-columns:1fr; }
    .hero h1{ font-size:32px; }
    .nav-links{ display:none; }
  }
</style>
</head>
<body>

<nav>
  <div class="wrap">
    <div class="nav-mark"><span class="dot"></span>Attrition Analysis</div>
    <div class="nav-links">
      <a href="#overview">Overview</a>
      <a href="#explore">Explore</a>
      <a href="#model">Modeling</a>
      <a href="#findings">Findings</a>
    </div>
  </div>
</nav>

<div class="wrap hero" id="overview">
  <div class="hero-tag">Synthetic dataset · illustrative, not real-workforce findings</div>
  <h1>Two models, both close to a coin flip.</h1>
  <p class="lede">A statistical and machine-learning read on 1,470 employee records — where attrition risk actually concentrates, and why a Decision Tree and Logistic Regression still can't reliably tell you who's about to leave.</p>

  <div class="meta-row">
    <div class="meta-item"><div class="label">Records</div><div class="val">1,470</div></div>
    <div class="meta-item"><div class="label">Features</div><div class="val">11</div></div>
    <div class="meta-item"><div class="label">Models</div><div class="val">Decision Tree · Logistic Regression</div></div>
    <div class="meta-item"><div class="label">Report date</div><div class="val">Jul 16, 2026</div></div>
  </div>

  <div class="gauge-card">
    <div class="gauge-head">
      <h3>Model signal vs. random chance — ROC AUC</h3>
      <span class="sub">0.500 = coin flip · 1.000 = perfect separation</span>
    </div>
    <div class="gauge-track">
      <div class="noise-zone"><span class="noise-label">near-random zone</span></div>
      <div class="marker dt" style="left:15.6%;"><div class="pin"></div><span class="tag">Decision Tree · 0.539</span></div>
      <div class="marker lr" style="left:2.8%;"><div class="pin"></div><span class="tag" style="top:-30px;">Logistic Reg. · 0.507</span></div>
    </div>
    <div class="gauge-scale"><span>0.50</span><span>0.60</span><span>0.70</span><span>0.80</span><span>0.90</span><span>1.00</span></div>
    <div class="gauge-foot">Both models land inside the near-random zone. Neither one, as configured, separates likely leavers from likely stayers meaningfully better than guessing.</div>
  </div>
</div>

<div class="wrap">
  <div class="kpi-grid">
    <div class="kpi"><div class="label">Overall attrition</div><div class="num risk">6.1%</div><div class="sub">90 of 1,470 employees left</div></div>
    <div class="kpi"><div class="label">Retained</div><div class="num safe">93.9%</div><div class="sub">1,380 employees</div></div>
    <div class="kpi"><div class="label">Strongest gradient</div><div class="num">Tenure</div><div class="sub">13.3% → 2.2% across bands</div></div>
    <div class="kpi"><div class="label">Weakest signal</div><div class="num">Overtime</div><div class="sub">6.2% vs 6.1% — near identical</div></div>
  </div>
</div>

<section id="explore">
  <div class="wrap">
    <div class="eyebrow">04 · Exploratory data analysis</div>
    <h2 class="section-title">Where the differences actually show up</h2>
    <p class="section-lede">Department and overtime barely move the needle. Tenure and job satisfaction do — modestly, but consistently.</p>

    <div class="grid-2" style="margin-bottom:20px;">
      <div class="card">
        <h3>Attrition by department</h3>
        <div class="card-sub">Range: 5.3% – 6.9%, tight around the 6.1% average</div>
        <div class="chart-box"><canvas id="deptChart" role="img" aria-label="Bar chart of attrition rate by department: IT 6.9%, Sales 6.5%, R&D 6.1%, HR 5.4%, Finance 5.3%">IT 6.9%, Sales 6.5%, R&D 6.1%, HR 5.4%, Finance 5.3%</canvas></div>
      </div>
      <div class="card">
        <h3>Attrition by tenure band</h3>
        <div class="card-sub">Clearest gradient in the dataset</div>
        <div class="chart-box"><canvas id="tenureChart" role="img" aria-label="Bar chart of attrition rate by years at company: 0-2 years 13.3%, 3-5 years 10.1%, 6-9 years 9.5%, 10-14 years 9.1%, 15-24 years 6.1%, 25+ years 2.2%">0-2yr 13.3%, 3-5yr 10.1%, 6-9yr 9.5%, 10-14yr 9.1%, 15-24yr 6.1%, 25+yr 2.2%</canvas></div>
      </div>
    </div>

    <div class="grid-2" style="margin-bottom:20px;">
      <div class="card">
        <h3>Overtime status</h3>
        <div class="card-sub">6.2% (overtime) vs 6.1% (no overtime) — essentially flat</div>
        <div class="chart-box"><canvas id="otChart" role="img" aria-label="Bar chart comparing attrition rate for employees with overtime (6.2%) versus without (6.1%)">No overtime 6.1%, overtime 6.2%</canvas></div>
      </div>
      <div class="card">
        <h3>Job satisfaction</h3>
        <div class="card-sub">Lowest score (1) nearly doubles the rate at score 3</div>
        <div class="chart-box"><canvas id="satChart" role="img" aria-label="Bar chart of attrition rate by job satisfaction score: level 1 9.6%, level 2 7.8%, level 3 4.9%, level 4 5.4%">Score 1: 9.6%, Score 2: 7.8%, Score 3: 4.9%, Score 4: 5.4%</canvas></div>
      </div>
    </div>

    <div class="card full">
      <h3>Correlation with attrition — numeric features</h3>
      <div class="card-sub">Every individual correlation is modest. Tenure (YearsAtCompany, −0.15) is the strongest single number in the matrix.</div>
      <div class="heat-wrap" id="heatmapWrap"></div>
    </div>
  </div>
</section>

<section id="model">
  <div class="wrap">
    <div class="eyebrow">05 · Predictive modeling</div>
    <h2 class="section-title">What the models actually got right</h2>
    <p class="section-lede">Both were trained on the same 80/20 stratified split, with class-balanced weighting to offset the ~6% base rate.</p>

    <div class="card full" style="margin-bottom:20px;">
      <h3>Performance metrics</h3>
      <div class="card-sub">Test set, 294 employees</div>
      <table class="metrics">
        <thead><tr><th>Metric</th><th>Decision Tree</th><th>Logistic Regression</th></tr></thead>
        <tbody>
          <tr><td class="rowname">Accuracy</td><td>67.3%</td><td>63.9%</td></tr>
          <tr><td class="rowname">Precision</td><td>5.7%</td><td>6.0%</td></tr>
          <tr><td class="rowname">Recall</td><td>27.8%</td><td>33.3%</td></tr>
          <tr><td class="rowname">F1 score</td><td>9.4%</td><td>10.2%</td></tr>
          <tr><td class="rowname">ROC AUC</td><td>0.539</td><td>0.507</td></tr>
        </tbody>
      </table>
    </div>

    <div class="cm-grid" style="margin-bottom:20px;">
      <div class="card">
        <h3>Confusion matrix — Decision Tree</h3>
        <div class="card-sub">Predicted vs actual, 294 test cases</div>
        <table class="cm-table">
          <tr><th></th><th>Pred. retained</th><th>Pred. attrition</th></tr>
          <tr><th>Actual retained</th><td class="cm-tn">193</td><td class="cm-fp">83</td></tr>
          <tr><th>Actual attrition</th><td class="cm-fn">13</td><td class="cm-tp">5</td></tr>
        </table>
      </div>
      <div class="card">
        <h3>Confusion matrix — Logistic Regression</h3>
        <div class="card-sub">Predicted vs actual, 294 test cases</div>
        <table class="cm-table">
          <tr><th></th><th>Pred. retained</th><th>Pred. attrition</th></tr>
          <tr><th>Actual retained</th><td class="cm-tn">182</td><td class="cm-fp">94</td></tr>
          <tr><th>Actual attrition</th><td class="cm-fn">12</td><td class="cm-tp">6</td></tr>
        </table>
      </div>
    </div>

    <div class="card full">
      <h3>Decision Tree feature importance</h3>
      <div class="card-sub">YearsAtCompany dominates; JobSatisfaction, OverTime and Department contribute almost nothing to this particular tree</div>
      <div class="chart-box tall"><canvas id="featChart" role="img" aria-label="Horizontal bar chart of decision tree feature importance, YearsAtCompany highest at roughly 0.46, followed by Age, MonthlyIncome, DistanceFromHome, with EnvironmentSatisfaction, YearsInCurrentRole, Gender near zero, and JobSatisfaction, YearsSinceLastPromotion, OverTime, Department at zero">YearsAtCompany ~0.46, Age ~0.17, MonthlyIncome ~0.16, DistanceFromHome ~0.15, EnvironmentSatisfaction ~0.03, YearsInCurrentRole ~0.02, Gender ~0.01, JobSatisfaction ~0, YearsSinceLastPromotion ~0, OverTime ~0, Department ~0</canvas></div>
    </div>
  </div>
</section>

<section id="findings">
  <div class="wrap">
    <div class="eyebrow">06 · Key findings</div>
    <h2 class="section-title">What held up, and what didn't</h2>
    <ul class="findings-list" style="margin-bottom:56px;">
      <li><span class="bullet">01</span>Tenure is the strongest observed signal — attrition risk is markedly higher among newer employees and declines steadily with years of service.</li>
      <li><span class="bullet">02</span>Satisfaction matters, but modestly — lower job satisfaction correlates with higher attrition, though the effect is smaller than tenure's.</li>
      <li><span class="bullet">03</span>Department and overtime show limited standalone predictive value — differences across categories are small relative to overall variability.</li>
      <li><span class="bullet">04</span>Both models perform close to chance on ROC AUC (0.50–0.54) — as configured, neither reliably separates likely leavers from likely stayers.</li>
      <li><span class="bullet">05</span>Class imbalance drives the low precision scores — with only ~6% of employees leaving, false positives naturally outnumber true positives at most thresholds.</li>
    </ul>

    <div class="eyebrow">08 · Recommendations</div>
    <h2 class="section-title" style="margin-bottom:32px;">Where to take this next</h2>
    <div class="rec-grid">
      <div class="rec-card"><div class="num">R1</div><p><strong>Prioritize early-tenure retention.</strong> Onboarding quality, mentorship, and first-year check-ins are reasonable investments given the tenure gradient.</p></div>
      <div class="rec-card"><div class="num">R2</div><p><strong>Track satisfaction trends, not snapshots.</strong> Monitoring change over time may be more informative than a single survey point.</p></div>
      <div class="rec-card"><div class="num">R3</div><p><strong>Expand the feature set before production use.</strong> Engagement survey results, manager-relationship indicators, and performance trajectory would likely help.</p></div>
      <div class="rec-card"><div class="num">R4</div><p><strong>Address class imbalance explicitly.</strong> Resampling, cost-sensitive learning, or threshold calibration tied to real business cost.</p></div>
      <div class="rec-card"><div class="num">R5</div><p><strong>Re-run on real HR data.</strong> Validate whether the tenure and satisfaction patterns observed here hold in an actual workforce.</p></div>
      <div class="rec-card"><div class="num">R6</div><p><strong>Treat this as a pipeline, not a verdict.</strong> The workflow is sound; the current signal strength is the thing to improve.</p></div>
    </div>
  </div>
</section>

<footer>Synthetic dataset, generated with a fixed random seed · figures illustrate methodology rather than a real organization</footer>

<script src="https://cdnjs.cloudflare.com/ajax/libs/Chart.js/4.4.1/chart.umd.js"></script>
<script>
  const RISK = '#e2694a';
  const SAFE = '#4fb8a6';
  const GOLD = '#e8b34c';
  const GRID = 'rgba(255,255,255,0.06)';
  const MUTED = '#93a1ab';

  Chart.defaults.font.family = "'IBM Plex Mono', monospace";
  Chart.defaults.font.size = 11;
  Chart.defaults.color = MUTED;

  const baseBar = {
    responsive:true, maintainAspectRatio:false,
    plugins:{ legend:{ display:false }, tooltip:{ backgroundColor:'#1c262f', borderColor:'#2a3640', borderWidth:1, titleColor:'#e9edf1', bodyColor:'#e9edf1', padding:10, displayColors:false } },
    scales:{
      x:{ grid:{ display:false }, ticks:{ color:MUTED } },
      y:{ grid:{ color:GRID }, ticks:{ color:MUTED, callback:v=>v+'%' }, beginAtZero:true }
    }
  };

  new Chart(document.getElementById('deptChart'), {
    type:'bar',
    data:{ labels:['IT','Sales','R&D','HR','Finance'],
      datasets:[{ data:[6.9,6.5,6.1,5.4,5.3], backgroundColor:RISK, borderRadius:4, maxBarThickness:34 }] },
    options: baseBar
  });

  new Chart(document.getElementById('tenureChart'), {
    type:'bar',
    data:{ labels:['0-2','3-5','6-9','10-14','15-24','25+'],
      datasets:[{ data:[13.3,10.1,9.5,9.1,6.1,2.2], backgroundColor:RISK, borderRadius:4, maxBarThickness:30 }] },
    options: baseBar
  });

  new Chart(document.getElementById('otChart'), {
    type:'bar',
    data:{ labels:['No overtime','Overtime'],
      datasets:[{ data:[6.1,6.2], backgroundColor:[SAFE,RISK], borderRadius:4, maxBarThickness:60 }] },
    options: baseBar
  });

  new Chart(document.getElementById('satChart'), {
    type:'bar',
    data:{ labels:['Score 1','Score 2','Score 3','Score 4'],
      datasets:[{ data:[9.6,7.8,4.9,5.4], backgroundColor:RISK, borderRadius:4, maxBarThickness:44 }] },
    options: baseBar
  });

  new Chart(document.getElementById('featChart'), {
    type:'bar',
    data:{
      labels:['YearsAtCompany','Age','MonthlyIncome','DistanceFromHome','EnvironmentSatisfaction','YearsInCurrentRole','Gender','JobSatisfaction','YearsSinceLastPromotion','OverTime','Department'],
      datasets:[{ data:[0.46,0.17,0.16,0.15,0.03,0.02,0.01,0,0,0,0], backgroundColor:GOLD, borderRadius:4, maxBarThickness:18 }]
    },
    options:{
      indexAxis:'y', responsive:true, maintainAspectRatio:false,
      plugins:{ legend:{ display:false }, tooltip:{ backgroundColor:'#1c262f', borderColor:'#2a3640', borderWidth:1, titleColor:'#e9edf1', bodyColor:'#e9edf1', padding:10, displayColors:false } },
      scales:{
        x:{ grid:{ color:GRID }, ticks:{ color:MUTED }, beginAtZero:true, max:0.5 },
        y:{ grid:{ display:false }, ticks:{ color:MUTED } }
      }
    }
  });

  // correlation heatmap
  const corrLabels = ['Age','MonthlyIncome','YearsAtCompany','YearsInCurrentRole','YearsSinceLastPromotion','JobSatisfaction','EnvironmentSatisfaction','DistanceFromHome','Attrition'];
  const corrData = [
    [1.00, 0.01,-0.02,-0.01, 0.00,-0.01, 0.02, 0.03, 0.04],
    [0.01, 1.00,-0.01, 0.04, 0.01, 0.01,-0.02, 0.00,-0.05],
    [-0.02,-0.01,1.00, 0.66, 0.64, 0.02, 0.03, 0.02,-0.15],
    [-0.01, 0.04, 0.66,1.00, 0.43, 0.00, 0.03, 0.00,-0.09],
    [0.00, 0.01, 0.64, 0.43,1.00,-0.01, 0.04, 0.03,-0.08],
    [-0.01, 0.01, 0.02, 0.00,-0.01,1.00,-0.03, 0.04,-0.06],
    [0.02,-0.02, 0.03, 0.03, 0.04,-0.03,1.00,-0.04,-0.05],
    [0.03, 0.00, 0.02, 0.00, 0.03, 0.04,-0.04,1.00, 0.02],
    [0.04,-0.05,-0.15,-0.09,-0.08,-0.06,-0.05, 0.02,1.00]
  ];

  function corrColor(v){
    // diverging: coral for negative, teal for positive, gray near zero
    const a = Math.min(Math.abs(v)/1,1);
    if (Math.abs(v) < 0.005) return '#2a3640';
    if (v > 0) {
      const l = 78 - a*46;
      return `hsl(170, 45%, ${l}%)`;
    } else {
      const l = 78 - a*46;
      return `hsl(14, 60%, ${l}%)`;
    }
  }

  let heatHtml = '<table class="heat"><thead><tr><th></th>';
  corrLabels.forEach(l => heatHtml += `<th>${l.replace(/([A-Z])/g,' $1').trim()}</th>`);
  heatHtml += '</tr></thead><tbody>';
  corrLabels.forEach((rowLabel, i) => {
    heatHtml += `<tr><td class="rowlabel">${rowLabel.replace(/([A-Z])/g,' $1').trim()}</td>`;
    corrData[i].forEach(v => {
      heatHtml += `<td style="background:${corrColor(v)}">${v.toFixed(2)}</td>`;
    });
    heatHtml += '</tr>';
  });
  heatHtml += '</tbody></table>';
  document.getElementById('heatmapWrap').innerHTML = heatHtml;
</script>

</body>
</html>