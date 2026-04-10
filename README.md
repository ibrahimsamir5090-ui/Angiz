<!DOCTYPE html>
<html lang="en">
<head>
<meta charset="UTF-8" />
<meta name="viewport" content="width=device-width, initial-scale=1.0, maximum-scale=1.0, user-scalable=no" />
<title>MindForm. — Habit Tracker</title>
<link href="https://fonts.googleapis.com/css2?family=Cairo:wght@300;400;500;600;700;900&display=swap" rel="stylesheet" />
<script src="https://cdn.tailwindcss.com"></script>
<script>
  tailwind.config = {
    theme: {
      extend: {
        colors: {
          camel: '#C19A6B',
          beige: '#F5F5DC',
          gold: '#D4AF37',
          dark: '#0a0a0a',
          card: '#111111',
          border: '#1e1e1e',
        },
        fontFamily: {
          cairo: ['Cairo', 'sans-serif'],
        },
      }
    }
  }
</script>
<style>
  * { box-sizing: border-box; margin: 0; padding: 0; }

  :root {
    --camel: #C19A6B;
    --beige: #F5F5DC;
    --gold: #D4AF37;
    --dark: #0a0a0a;
    --card: #111111;
    --border: #1e1e1e;
  }

  html, body {
    background: var(--dark);
    font-family: 'Cairo', sans-serif;
    color: var(--beige);
    min-height: 100vh;
    overflow-x: hidden;
  }

  /* ── ROTATE OVERLAY ── */
  #rotate-overlay {
    display: none;
    position: fixed;
    inset: 0;
    background: #0a0a0a;
    z-index: 9999;
    flex-direction: column;
    align-items: center;
    justify-content: center;
    gap: 24px;
  }

  @media screen and (max-width: 768px) and (orientation: portrait) {
    #rotate-overlay { display: flex; }
    body { overflow: hidden; }
  }

  .rotate-icon {
    width: 72px;
    height: 72px;
    border: 2px solid var(--gold);
    border-radius: 12px;
    display: flex;
    align-items: center;
    justify-content: center;
    animation: rotateHint 2s ease-in-out infinite;
    box-shadow: 0 0 20px rgba(212,175,55,0.3);
  }

  @keyframes rotateHint {
    0%, 100% { transform: rotate(0deg); }
    40% { transform: rotate(-90deg); }
    60% { transform: rotate(-90deg); }
  }

  /* ── GLOW EFFECTS ── */
  .glow-gold {
    box-shadow: 0 0 20px rgba(212,175,55,0.12), 0 0 40px rgba(212,175,55,0.06);
  }
  .glow-camel {
    box-shadow: 0 0 16px rgba(193,154,107,0.15);
  }
  .border-glow {
    border: 1px solid rgba(193,154,107,0.2);
  }

  /* ── CARDS ── */
  .card {
    background: var(--card);
    border: 1px solid var(--border);
    border-radius: 12px;
    transition: border-color 0.3s;
  }
  .card:hover { border-color: rgba(193,154,107,0.25); }

  /* ── SUMMARY CARDS ── */
  .stat-card {
    background: #0f0f0f;
    border: 1px solid #1e1e1e;
    border-radius: 10px;
    padding: 14px 20px;
    text-align: center;
    position: relative;
    overflow: hidden;
    transition: all 0.3s;
  }
  .stat-card::before {
    content: '';
    position: absolute;
    top: 0; left: 0; right: 0;
    height: 2px;
    background: linear-gradient(90deg, transparent, var(--gold), transparent);
    opacity: 0;
    transition: opacity 0.3s;
  }
  .stat-card:hover::before { opacity: 1; }
  .stat-card:hover { border-color: rgba(212,175,55,0.2); }

  .stat-value {
    font-size: clamp(1.6rem, 3vw, 2.4rem);
    font-weight: 900;
    line-height: 1.1;
    letter-spacing: -1px;
  }
  .stat-label {
    font-size: 0.6rem;
    font-weight: 500;
    letter-spacing: 0.18em;
    text-transform: uppercase;
    color: rgba(245,245,220,0.45);
    margin-bottom: 4px;
  }

  /* ── BAR CHART ── */
  .bar-container {
    display: flex;
    align-items: flex-end;
    gap: 3px;
    height: 90px;
    padding: 0 4px;
  }
  .bar-wrap {
    flex: 1;
    display: flex;
    flex-direction: column;
    align-items: center;
    gap: 4px;
    height: 100%;
    justify-content: flex-end;
  }
  .bar {
    width: 100%;
    border-radius: 3px 3px 0 0;
    transition: all 0.6s cubic-bezier(0.34, 1.56, 0.64, 1);
    position: relative;
    overflow: hidden;
    cursor: pointer;
  }
  .bar::after {
    content: '';
    position: absolute;
    top: 0; left: 0; right: 0;
    height: 30%;
    background: linear-gradient(180deg, rgba(255,255,255,0.08) 0%, transparent 100%);
    border-radius: 3px 3px 0 0;
  }
  .bar:hover { filter: brightness(1.3); }
  .bar-label {
    font-size: 0.5rem;
    color: rgba(245,245,220,0.35);
    font-weight: 500;
    letter-spacing: 0.05em;
    white-space: nowrap;
  }
  .bar-camel { background: linear-gradient(180deg, #C19A6B 0%, #8B6A45 100%); }
  .bar-gold  { background: linear-gradient(180deg, #D4AF37 0%, #A88B25 100%); }
  .bar-dim   { background: linear-gradient(180deg, #2a2a2a 0%, #1a1a1a 100%); }

  /* ── DONUT CHART ── */
  .donut-svg { transform: rotate(-90deg); }
  .donut-track { fill: none; stroke: #1e1e1e; }
  .donut-fill {
    fill: none;
    stroke-linecap: round;
    transition: stroke-dashoffset 1.5s cubic-bezier(0.34, 1.56, 0.64, 1);
    filter: drop-shadow(0 0 6px rgba(212,175,55,0.5));
  }

  /* ── PROGRESS BAR ── */
  .prog-bar {
    height: 5px;
    background: #1e1e1e;
    border-radius: 99px;
    overflow: hidden;
    flex: 1;
  }
  .prog-fill {
    height: 100%;
    border-radius: 99px;
    background: linear-gradient(90deg, var(--camel), var(--gold));
    transition: width 1s ease;
    position: relative;
  }
  .prog-fill::after {
    content: '';
    position: absolute;
    top: 0; right: 0;
    width: 6px; height: 100%;
    background: rgba(255,255,255,0.4);
    border-radius: 99px;
    filter: blur(2px);
  }

  /* ── CHECKBOX GRID ── */
  .habit-check {
    width: 20px;
    height: 20px;
    border-radius: 5px;
    border: 1.5px solid rgba(193,154,107,0.25);
    background: transparent;
    cursor: pointer;
    position: relative;
    transition: all 0.2s;
    appearance: none;
    -webkit-appearance: none;
    flex-shrink: 0;
  }
  .habit-check:checked {
    background: linear-gradient(135deg, var(--camel), var(--gold));
    border-color: var(--gold);
    box-shadow: 0 0 8px rgba(212,175,55,0.4);
  }
  .habit-check:checked::after {
    content: '✓';
    position: absolute;
    inset: 0;
    display: flex;
    align-items: center;
    justify-content: center;
    color: #0a0a0a;
    font-size: 11px;
    font-weight: 900;
    line-height: 1;
  }
  .habit-check:hover:not(:checked) {
    border-color: rgba(193,154,107,0.6);
    background: rgba(193,154,107,0.05);
  }

  /* ── WEEK HEADER ── */
  .week-col {
    font-size: 0.55rem;
    font-weight: 700;
    letter-spacing: 0.1em;
    text-transform: uppercase;
    color: var(--camel);
    text-align: center;
  }

  /* ── SECTION TITLE ── */
  .section-title {
    font-size: 0.55rem;
    font-weight: 700;
    letter-spacing: 0.22em;
    text-transform: uppercase;
    color: rgba(193,154,107,0.6);
    margin-bottom: 10px;
  }

  /* ── HABIT ROW ── */
  .habit-row {
    display: flex;
    align-items: center;
    gap: 8px;
    padding: 6px 10px;
    border-radius: 7px;
    transition: background 0.2s;
  }
  .habit-row:hover { background: rgba(193,154,107,0.04); }

  /* ── ANALYSIS TABLE ── */
  .analysis-row td { padding: 3px 8px; font-size: 0.65rem; }
  .analysis-row:hover td { color: var(--gold); }

  /* ── MOOD PILL ── */
  .mood-pill {
    background: #0f0f0f;
    border: 1px solid #1e1e1e;
    border-radius: 99px;
    padding: 4px 14px;
    font-size: 1rem;
    cursor: pointer;
    transition: all 0.2s;
  }
  .mood-pill:hover, .mood-pill.active {
    border-color: var(--gold);
    box-shadow: 0 0 10px rgba(212,175,55,0.2);
    background: rgba(212,175,55,0.05);
  }

  /* ── DIVIDER ── */
  .gold-divider {
    height: 1px;
    background: linear-gradient(90deg, transparent, rgba(212,175,55,0.25), transparent);
  }

  /* ── HEADER GLOW LINE ── */
  .header-glow {
    position: absolute;
    bottom: 0; left: 0; right: 0;
    height: 1px;
    background: linear-gradient(90deg, transparent 0%, rgba(212,175,55,0.3) 30%, rgba(193,154,107,0.5) 50%, rgba(212,175,55,0.3) 70%, transparent 100%);
  }

  /* ── SCROLLBAR ── */
  ::-webkit-scrollbar { width: 4px; height: 4px; }
  ::-webkit-scrollbar-track { background: var(--dark); }
  ::-webkit-scrollbar-thumb { background: #2a2a2a; border-radius: 99px; }
  ::-webkit-scrollbar-thumb:hover { background: var(--camel); }

  /* ── FADE IN ── */
  @keyframes fadeUp {
    from { opacity: 0; transform: translateY(16px); }
    to   { opacity: 1; transform: translateY(0); }
  }
  .fade-up { animation: fadeUp 0.6s ease forwards; }
  .delay-1 { animation-delay: 0.1s; opacity: 0; }
  .delay-2 { animation-delay: 0.2s; opacity: 0; }
  .delay-3 { animation-delay: 0.3s; opacity: 0; }
  .delay-4 { animation-delay: 0.4s; opacity: 0; }
  .delay-5 { animation-delay: 0.5s; opacity: 0; }

  /* ── TOP 10 RANK ── */
  .rank-num {
    font-size: 0.6rem;
    font-weight: 700;
    color: rgba(193,154,107,0.5);
    width: 16px;
    flex-shrink: 0;
  }

  @media (max-width: 900px) and (orientation: landscape) {
    .stat-value { font-size: 1.4rem; }
  }
</style>
</head>
<body>

<!-- ══════════════════════════════════
     ROTATE OVERLAY
══════════════════════════════════ -->
<div id="rotate-overlay">
  <div class="rotate-icon">
    <svg width="36" height="36" viewBox="0 0 36 36" fill="none">
      <rect x="8" y="4" width="20" height="28" rx="3" stroke="#D4AF37" stroke-width="1.5"/>
      <circle cx="18" cy="28" r="2" fill="#D4AF37"/>
    </svg>
  </div>
  <div style="text-align:center">
    <p style="font-family:'Cairo',sans-serif; font-size:0.65rem; letter-spacing:0.2em; text-transform:uppercase; color:rgba(212,175,55,0.7); margin-bottom:6px;">Rotate Your Device</p>
    <p style="font-family:'Cairo',sans-serif; font-size:1.1rem; font-weight:700; color:#F5F5DC;">Best viewed in Landscape</p>
  </div>
  <div style="width:40px; height:1px; background:rgba(212,175,55,0.3);"></div>
  <p style="font-family:'Cairo',sans-serif; font-size:0.6rem; color:rgba(245,245,220,0.3); letter-spacing:0.12em;">MINDFORM. HABIT TRACKER</p>
</div>

<!-- ══════════════════════════════════
     MAIN APP
══════════════════════════════════ -->
<div style="max-width:1400px; margin:0 auto; padding:20px 24px 40px;">

  <!-- HEADER -->
  <div class="fade-up delay-1" style="position:relative; padding-bottom:18px; margin-bottom:20px;">
    <div style="display:flex; align-items:flex-end; justify-content:space-between; flex-wrap:wrap; gap:12px;">
      <div>
        <h1 style="font-size:clamp(2rem,4vw,3.2rem); font-weight:900; letter-spacing:-2px; line-height:1;">
          <span style="color:var(--beige);">MindForm</span><span style="color:var(--gold);">.</span>
        </h1>
        <p style="font-size:0.55rem; letter-spacing:0.28em; color:rgba(193,154,107,0.6); font-weight:600; text-transform:uppercase; margin-top:2px;">Habit Tracker</p>
      </div>
      <div style="display:flex; align-items:center; gap:8px;">
        <div style="width:6px; height:6px; border-radius:50%; background:var(--gold); box-shadow:0 0 8px var(--gold); animation:pulse 2s infinite;"></div>
        <span style="font-size:0.65rem; letter-spacing:0.15em; color:rgba(193,154,107,0.7); font-weight:600; text-transform:uppercase;">— January —</span>
      </div>
    </div>
    <div class="header-glow"></div>
  </div>

  <!-- SUMMARY CARDS -->
  <div class="fade-up delay-2" style="display:grid; grid-template-columns:repeat(3,1fr); gap:12px; margin-bottom:18px;">
    <div class="stat-card glow-gold">
      <div class="stat-label">Goal</div>
      <div class="stat-value" style="color:var(--beige);">372</div>
    </div>
    <div class="stat-card" style="border-color:rgba(212,175,55,0.25);">
      <div class="stat-label">Completed</div>
      <div class="stat-value" style="color:var(--gold);">233</div>
    </div>
    <div class="stat-card">
      <div class="stat-label">Left</div>
      <div class="stat-value" style="color:var(--camel);">139</div>
    </div>
  </div>

  <!-- MAIN GRID -->
  <div style="display:grid; grid-template-columns:1fr 1fr 260px; gap:14px; margin-bottom:14px;">

    <!-- Daily Progress Chart -->
    <div class="card fade-up delay-2" style="padding:14px 16px;">
      <div class="section-title">Daily Progress</div>
      <div class="bar-container" id="daily-chart"></div>
      <div id="daily-labels" style="display:flex; gap:3px; padding:0 4px; margin-top:4px;"></div>
    </div>

    <!-- Weekly Progress Chart -->
    <div class="card fade-up delay-3" style="padding:14px 16px;">
      <div class="section-title">Weekly Progress</div>
      <div class="bar-container" id="weekly-chart"></div>
      <div id="weekly-labels" style="display:flex; gap:3px; padding:0 4px; margin-top:4px; justify-content:space-around;"></div>
    </div>

    <!-- Overall Stats Donut + Analysis -->
    <div style="display:flex; flex-direction:column; gap:12px;">
      <!-- Donut -->
      <div class="card fade-up delay-3" style="padding:14px 16px; display:flex; flex-direction:column; align-items:center;">
        <div class="section-title" style="align-self:flex-start;">Overall Stats</div>
        <div style="position:relative; width:100px; height:100px;">
          <svg width="100" height="100" class="donut-svg" viewBox="0 0 100 100">
            <circle class="donut-track" cx="50" cy="50" r="38" stroke-width="10"/>
            <circle class="donut-fill" id="donut-arc" cx="50" cy="50" r="38" stroke-width="10"
              stroke="url(#donutGrad)"
              stroke-dasharray="238.76"
              stroke-dashoffset="238.76"/>
            <defs>
              <linearGradient id="donutGrad" x1="0%" y1="0%" x2="100%" y2="0%">
                <stop offset="0%" stop-color="#C19A6B"/>
                <stop offset="100%" stop-color="#D4AF37"/>
              </linearGradient>
            </defs>
          </svg>
          <div style="position:absolute;inset:0;display:flex;flex-direction:column;align-items:center;justify-content:center;">
            <span style="font-size:0.6rem;color:rgba(245,245,220,0.4);letter-spacing:0.1em;">DONE</span>
            <span style="font-size:1.1rem;font-weight:900;color:var(--gold);">62.6%</span>
          </div>
        </div>
        <div style="display:flex;gap:14px;margin-top:8px;">
          <div style="display:flex;align-items:center;gap:4px;">
            <div style="width:8px;height:8px;border-radius:50%;background:var(--gold);"></div>
            <span style="font-size:0.55rem;color:rgba(245,245,220,0.5);">Completed 62.6%</span>
          </div>
          <div style="display:flex;align-items:center;gap:4px;">
            <div style="width:8px;height:8px;border-radius:50%;background:#1e1e1e;border:1px solid #333;"></div>
            <span style="font-size:0.55rem;color:rgba(245,245,220,0.5);">Left 37.4%</span>
          </div>
        </div>
      </div>

      <!-- Analysis Table -->
      <div class="card fade-up delay-4" style="padding:14px 16px; flex:1;">
        <div class="section-title">Analysis</div>
        <table style="width:100%;border-collapse:collapse;">
          <thead>
            <tr style="border-bottom:1px solid #1e1e1e;">
              <td style="font-size:0.55rem;letter-spacing:0.12em;color:rgba(193,154,107,0.5);padding-bottom:5px;">WEEK</td>
              <td style="font-size:0.55rem;letter-spacing:0.12em;color:rgba(193,154,107,0.5);padding-bottom:5px;text-align:right;">GOAL</td>
            </tr>
          </thead>
          <tbody id="analysis-body"></tbody>
        </table>
      </div>
    </div>

  </div>

  <!-- BOTTOM GRID: Habits + Top 10 -->
  <div style="display:grid; grid-template-columns:1fr 340px; gap:14px;" class="fade-up delay-4">

    <!-- Habit Checkbox Grid -->
    <div class="card" style="padding:14px 16px; overflow-x:auto;">
      <div class="section-title">My Habits</div>

      <!-- Week headers -->
      <div style="display:flex;align-items:center;gap:8px;padding:0 10px;margin-bottom:6px;">
        <div style="flex:1;min-width:130px;"></div>
        <div style="display:grid;grid-template-columns:repeat(5,1fr);gap:8px;width:500px;flex-shrink:0;">
          <div class="week-col">Week 1</div>
          <div class="week-col">Week 2</div>
          <div class="week-col">Week 3</div>
          <div class="week-col">Week 4</div>
          <div class="week-col">Week 5</div>
        </div>
      </div>

      <div class="gold-divider" style="margin-bottom:8px;"></div>

      <!-- Habit rows -->
      <div id="habit-grid"></div>
    </div>

    <!-- Top 10 Daily Habit -->
    <div class="card" style="padding:14px 16px;">
      <div class="section-title">Top 10 Daily Habit</div>

      <div style="display:flex;gap:6px;margin-bottom:8px;font-size:0.5rem;letter-spacing:0.1em;color:rgba(193,154,107,0.4);text-transform:uppercase;">
        <span style="width:16px;"></span>
        <span style="flex:1;">Habit</span>
        <span style="width:28px;text-align:center;">Goal</span>
        <span style="width:28px;text-align:center;">Done</span>
        <span style="width:80px;text-align:right;">Progress</span>
        <span style="width:28px;text-align:right;">%</span>
      </div>
      <div class="gold-divider" style="margin-bottom:8px;"></div>
      <div id="top10-list"></div>
    </div>

  </div>

  <!-- MOOD + MOTIVATION -->
  <div style="display:grid;grid-template-columns:1fr 1fr;gap:14px;margin-top:14px;" class="fade-up delay-5">
    <div class="card" style="padding:12px 16px;">
      <div class="section-title" style="margin-bottom:8px;">Mood</div>
      <div style="display:flex;gap:8px;flex-wrap:wrap;" id="mood-row"></div>
    </div>
    <div class="card" style="padding:12px 16px;">
      <div class="section-title" style="margin-bottom:8px;">Motivation</div>
      <p style="font-size:0.75rem;color:rgba(193,154,107,0.8);font-style:italic;line-height:1.6;">"The secret of getting ahead is getting started. Small habits compound into extraordinary results."</p>
      <p style="font-size:0.55rem;color:rgba(245,245,220,0.3);margin-top:6px;letter-spacing:0.1em;">Jan 27 — May 21</p>
    </div>
  </div>

</div>

<style>
@keyframes pulse {
  0%,100% { opacity:1; }
  50% { opacity:0.4; }
}
</style>

<script>
// ── DATA ──────────────────────────────────────────

const habits = [
  { emoji:'🌅', name:'Wake up at 08:00' },
  { emoji:'🧘', name:'Meditation' },
  { emoji:'🏋️', name:'GYM' },
  { emoji:'🚿', name:'Cold Shower' },
  { emoji:'💼', name:'Work' },
  { emoji:'📖', name:'Read 10 pages' },
  { emoji:'🧠', name:'Learn a skill' },
  { emoji:'🚫', name:'No sugar' },
  { emoji:'🍷', name:'No alcohol' },
  { emoji:'📵', name:'1H social media' },
  { emoji:'📝', name:'Planning' },
  { emoji:'🛌', name:'Sleep before 11:00' },
];

// Weekly checkbox states (5 weeks, pre-filled with some checks)
const checkStates = [
  [true,true,true,true,false],
  [true,true,true,true,false],
  [true,true,true,true,false],
  [true,true,true,true,false],
  [true,true,true,true,false],
  [true,true,true,true,false],
  [true,true,true,false,false],
  [true,true,true,true,false],
  [true,true,false,true,false],
  [true,true,true,false,false],
  [true,false,true,true,false],
  [false,true,true,false,false],
];

const dailyData = [270,180,290,250,310,180,220,290,260,200,280,240,300,260,190,260,280,310,270,290,230];
const weeklyData = [372,233,139,89,75];
const analysisData = [
  {week:1,goal:372},{week:2,goal:233},{week:3,goal:139},{week:4,goal:89},{week:5,goal:75}
];
const top10 = [
  {name:'Wake up at 08:00', goal:372, actual:233, pct:90},
  {name:'Meditation',        goal:372, actual:233, pct:90},
  {name:'GYM',               goal:372, actual:109, pct:70},
  {name:'Cold Shower',       goal:372, actual:74,  pct:45},
  {name:'Read 10 pages',     goal:373, actual:168, pct:33},
  {name:'No sugar',          goal:372, actual:112, pct:60},
  {name:'No alcohol',        goal:372, actual:98,  pct:40},
  {name:'1H social media',   goal:372, actual:60,  pct:30},
  {name:'Planning',          goal:372, actual:40,  pct:20},
  {name:'Sleep before 11:00',goal:372, actual:10,  pct:10},
];
const moods = ['😁','😊','🙂','😐','😔','😤'];

// ── RENDER DAILY CHART ────────────────────────────

function renderDailyChart() {
  const chart = document.getElementById('daily-chart');
  const labels = document.getElementById('daily-labels');
  const max = Math.max(...dailyData);
  dailyData.forEach((val, i) => {
    const h = Math.round((val / max) * 85);
    const isLast = val === max;
    const wrap = document.createElement('div');
    wrap.className = 'bar-wrap';
    wrap.innerHTML = `<div class="bar ${isLast?'bar-gold':'bar-camel'}" style="height:${h}px;" title="Day ${i+1}: ${val}"></div>`;
    chart.appendChild(wrap);

    const lbl = document.createElement('div');
    lbl.style.flex = '1';
    lbl.style.textAlign = 'center';
    if ((i + 1) % 4 === 0 || i === 0 || i === dailyData.length - 1) {
      lbl.innerHTML = `<span class="bar-label">${i+1}</span>`;
    } else {
      lbl.innerHTML = '<span class="bar-label"></span>';
    }
    labels.appendChild(lbl);
  });
}

// ── RENDER WEEKLY CHART ───────────────────────────

function renderWeeklyChart() {
  const chart = document.getElementById('weekly-chart');
  const labels = document.getElementById('weekly-labels');
  const max = Math.max(...weeklyData);
  weeklyData.forEach((val, i) => {
    const h = Math.round((val / max) * 85);
    const wrap = document.createElement('div');
    wrap.className = 'bar-wrap';
    const cls = i === 0 ? 'bar-gold' : i < 3 ? 'bar-camel' : 'bar-dim';
    wrap.innerHTML = `
      <span style="font-size:0.55rem;color:rgba(245,245,220,0.5);font-weight:600;">${val}</span>
      <div class="bar ${cls}" style="height:${h}px;"></div>`;
    chart.appendChild(wrap);

    const lbl = document.createElement('div');
    lbl.style.flex = '1';
    lbl.style.textAlign = 'center';
    lbl.innerHTML = `<span class="bar-label">W${i+1}</span>`;
    labels.appendChild(lbl);
  });
}

// ── RENDER ANALYSIS TABLE ─────────────────────────

function renderAnalysis() {
  const tbody = document.getElementById('analysis-body');
  analysisData.forEach((row, i) => {
    const tr = document.createElement('tr');
    tr.className = 'analysis-row';
    tr.style.borderBottom = '1px solid #151515';
    tr.style.cursor = 'pointer';
    tr.style.transition = 'color 0.2s';
    const clr = i === 0 ? 'var(--gold)' : i === 1 ? 'var(--camel)' : 'rgba(245,245,220,0.55)';
    tr.innerHTML = `
      <td style="color:rgba(245,245,220,0.45);font-size:0.62rem;">${row.week}</td>
      <td style="color:${clr};font-weight:700;font-size:0.68rem;text-align:right;">${row.goal}</td>`;
    tbody.appendChild(tr);
  });
}

// ── RENDER HABIT GRID ─────────────────────────────

function renderHabitGrid() {
  const grid = document.getElementById('habit-grid');
  habits.forEach((h, hi) => {
    const row = document.createElement('div');
    row.className = 'habit-row';

    let checks = '';
    for (let w = 0; w < 5; w++) {
      const checked = checkStates[hi][w] ? 'checked' : '';
      checks += `<input type="checkbox" class="habit-check" ${checked} title="${h.name} — Week ${w+1}">`;
    }

    row.innerHTML = `
      <div style="flex:1;min-width:130px;display:flex;align-items:center;gap:6px;">
        <span style="font-size:0.85rem;">${h.emoji}</span>
        <span style="font-size:0.65rem;font-weight:500;color:rgba(245,245,220,0.75);white-space:nowrap;overflow:hidden;text-overflow:ellipsis;">${h.name}</span>
      </div>
      <div style="display:grid;grid-template-columns:repeat(5,1fr);gap:8px;width:500px;flex-shrink:0;justify-items:center;">
        ${checks}
      </div>`;
    grid.appendChild(row);
  });
}

// ── RENDER TOP 10 ─────────────────────────────────

function renderTop10() {
  const list = document.getElementById('top10-list');
  top10.forEach((item, i) => {
    const colors = ['var(--gold)','var(--camel)','rgba(193,154,107,0.75)'];
    const clr = colors[Math.min(i, 2)];
    const div = document.createElement('div');
    div.style.cssText = 'display:flex;align-items:center;gap:6px;padding:4px 0;';
    div.innerHTML = `
      <span class="rank-num">${i+1}.</span>
      <span style="flex:1;font-size:0.6rem;color:rgba(245,245,220,0.65);overflow:hidden;text-overflow:ellipsis;white-space:nowrap;">${item.name}</span>
      <span style="width:28px;text-align:center;font-size:0.6rem;color:rgba(245,245,220,0.35);">${item.actual}</span>
      <div class="prog-bar" style="width:70px;flex:none;">
        <div class="prog-fill" style="width:${item.pct}%;background:linear-gradient(90deg,${clr},var(--gold));"></div>
      </div>
      <span style="width:28px;text-align:right;font-size:0.6rem;color:${clr};font-weight:700;">${item.pct}%</span>`;
    list.appendChild(div);

    if (i < top10.length - 1) {
      const hr = document.createElement('div');
      hr.style.cssText = 'height:1px;background:#151515;margin:1px 0;';
      list.appendChild(hr);
    }
  });
}

// ── RENDER MOOD ───────────────────────────────────

function renderMood() {
  const row = document.getElementById('mood-row');
  moods.forEach((m, i) => {
    const btn = document.createElement('button');
    btn.className = 'mood-pill' + (i === 1 ? ' active' : '');
    btn.textContent = m;
    btn.onclick = () => {
      document.querySelectorAll('.mood-pill').forEach(b => b.classList.remove('active'));
      btn.classList.add('active');
    };
    row.appendChild(btn);
  });
}

// ── ANIMATE DONUT ─────────────────────────────────

function animateDonut() {
  const arc = document.getElementById('donut-arc');
  const circumference = 238.76;
  const pct = 0.626;
  const offset = circumference * (1 - pct);
  setTimeout(() => {
    arc.style.strokeDashoffset = offset;
  }, 400);
}

// ── INIT ──────────────────────────────────────────

document.addEventListener('DOMContentLoaded', () => {
  renderDailyChart();
  renderWeeklyChart();
  renderAnalysis();
  renderHabitGrid();
  renderTop10();
  renderMood();
  animateDonut();
});
</script>
</body>
</html>
