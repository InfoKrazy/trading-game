<!doctype html>
<html lang="en">
<head>
<meta charset="utf-8" />
<meta name="viewport" content="width=device-width,initial-scale=1" />
<title>InfoKrazy — Trading RNG Game (Enhanced)</title>
<link href="https://fonts.googleapis.com/css2?family=Inter:wght@400;600;700&display=swap" rel="stylesheet">
<!-- Chart.js CDN for detailed charts -->
<script src="https://cdn.jsdelivr.net/npm/chart.js@4.4.0/dist/chart.umd.min.js"></script>
<style>
  :root{
    --bg:#0b1020; --panel:#0f1724; --muted:#9aa7bf; --accent:#7c5cff; --accent-2:#00c2ff;
    --glass: rgba(255,255,255,0.03); --radius:12px; --success:#34d399; --danger:#ff7b7b;
    color-scheme: dark;
  }
  html,body{height:100%;margin:0;font-family:Inter,system-ui,-apple-system,Segoe UI,Roboto,'Helvetica Neue',Arial;background:linear-gradient(180deg,#04060b 0%,#071129 100%);color:#eaf0ff}
  .app{max-width:1200px;margin:28px auto;padding:18px}
  header{display:flex;align-items:center;justify-content:space-between;gap:12px;margin-bottom:12px}
  .brand{display:flex;gap:12px;align-items:center}
  .logo{width:52px;height:52px;border-radius:12px;background:linear-gradient(135deg,var(--accent),var(--accent-2));display:flex;align-items:center;justify-content:center;font-weight:800;color:#021027}
  h1{margin:0;font-size:1.15rem}
  .muted{color:var(--muted);font-size:0.9rem}
  .controls{display:flex;gap:8px;align-items:center}
  .btn{background:var(--glass);border:1px solid rgba(255,255,255,0.04);padding:8px 10px;border-radius:10px;color:var(--muted);cursor:pointer;font-weight:600}
  .btn.primary{background:linear-gradient(90deg,var(--accent),var(--accent-2));color:#021027;border:none;box-shadow:0 10px 30px rgba(124,92,255,0.12)}
  .main{display:grid;grid-template-columns:320px 1fr;gap:18px}
  .panel{background:linear-gradient(180deg, rgba(255,255,255,0.02), rgba(255,255,255,0.01));padding:14px;border-radius:12px;border:1px solid rgba(255,255,255,0.03)}
  .sidebar .stat{display:flex;justify-content:space-between;margin-bottom:8px}
  .portfolio-list{display:grid;gap:8px;margin-top:10px}
  .asset-row{display:flex;align-items:center;gap:8px;padding:8px;border-radius:10px;background:rgba(255,255,255,0.012);border:1px solid rgba(255,255,255,0.02);transition:transform .18s}
  .asset-row:hover{transform:translateY(-6px)}
  .asset-thumb{width:44px;height:44px;border-radius:8px;display:flex;align-items:center;justify-content:center;background:linear-gradient(135deg,rgba(255,255,255,0.02),transparent);font-weight:700;color:var(--accent)}
  .asset-info{flex:1}
  .price{font-weight:800}
  .small{font-size:0.85rem;color:var(--muted)}
  .trade-panel{display:flex;flex-direction:column;gap:8px;margin-top:10px}
  .input,select{width:100%;padding:8px;border-radius:10px;border:1px solid rgba(255,255,255,0.04);background:transparent;color:inherit}
  .row{display:flex;gap:8px}
  .row .input{flex:1}
  .market{display:grid;grid-template-columns:repeat(auto-fit,minmax(220px,1fr));gap:12px;margin-top:8px}
  .card{padding:12px;border-radius:12px;background:linear-gradient(180deg, rgba(255,255,255,0.015), rgba(255,255,255,0.01));border:1px solid rgba(255,255,255,0.02);transition:transform .18s,box-shadow .18s;cursor:pointer}
  .card:hover{transform:translateY(-6px);box-shadow:0 14px 40px rgba(2,6,23,0.6)}
  .spark{width:100%;height:48px}
  footer{margin-top:14px;text-align:center;color:var(--muted);font-size:0.85rem}
  .goal{padding:6px;border-radius:10px;background:linear-gradient(90deg, rgba(124,92,255,0.12), rgba(0,194,255,0.06));display:inline-block;font-weight:700}
  .theme{cursor:pointer;padding:8px;border-radius:10px}
  body.light{color:#0f1724;background:linear-gradient(180deg,#f7fbff 0%,#eef6ff 100%);color-scheme: light}
  body.light .panel{background:linear-gradient(180deg, rgba(2,6,23,0.03), rgba(2,6,23,0.01));}
  body.light .card{background:#ffffff;border:1px solid rgba(2,6,23,0.04);box-shadow:0 8px 18px rgba(2,6,23,0.04)}
  .ticker-up{color:var(--success);font-weight:800}
  .ticker-down{color:var(--danger);font-weight:800}
  .pulse{animation:pulse 1.6s infinite linear}
  @keyframes pulse{0%{transform:scale(1)}50%{transform:scale(0.995)}100%{transform:scale(1)}}
  /* modal */
  .modal-backdrop{position:fixed;inset:0;background:rgba(2,6,23,0.6);display:none;align-items:center;justify-content:center;padding:18px;z-index:300}
  .modal{width:100%;max-width:920px;background:linear-gradient(180deg,#07102a,#071124);border-radius:12px;padding:12px;border:1px solid rgba(255,255,255,0.04);color:inherit}
  .modal header{display:flex;justify-content:space-between;align-items:center;margin-bottom:8px}
  .chart-canvas{width:100%;height:320px}
  /* toast/achievements */
  .toast{position:fixed;right:18px;bottom:18px;background:linear-gradient(90deg,#6c63ff,#00c2ff);color:#021027;padding:12px 16px;border-radius:12px;font-weight:800;box-shadow:0 10px 30px rgba(2,6,23,0.5);z-index:500}
  .achievement{position:fixed;left:18px;bottom:18px;background:#fff;border-radius:12px;padding:10px 12px;color:#021027;font-weight:800;z-index:500}
  /* leaderboard */
  .leaderboard-list{max-height:240px;overflow:auto;display:grid;gap:6px;padding-top:8px}
  .leader-row{display:flex;justify-content:space-between;padding:8px;border-radius:8px;background:rgba(255,255,255,0.015);border:1px solid rgba(255,255,255,0.02)}
  /* confetti canvas */
  #confetti{position:fixed;inset:0;pointer-events:none;z-index:1000;display:none}
  @media (max-width:980px){ .main{grid-template-columns:1fr} .logo{display:none} .sidebar{order:2} .modal{max-width:98%} }
</style>
</head>
<body>
  <canvas id="confetti"></canvas>
  <div class="app">
    <header>
      <div class="brand">
        <div class="logo">IK</div>
        <div>
          <h1>InfoKrazy — RNG Trading (Enhanced)</h1>
          <div class="muted">RNG-driven market • Leaderboards • Charts • Achievements • Sound</div>
        </div>
      </div>
      <div class="controls">
        <div class="muted">Tick: <span id="tick">0</span></div>
        <button class="btn" id="pauseBtn">Pause</button>
        <button class="btn" id="resetBtn">Reset</button>
        <button class="btn primary" id="fastBtn">Fast ×2</button>
        <div class="theme" id="themeToggle" title="Toggle Light/Dark">🌓</div>
        <button class="btn" id="soundToggle">🔊 On</button>
      </div>
    </header>

    <main class="main">
      <aside class="panel sidebar">
        <div class="stat"><div>Cash</div><div class="price" id="cash">—</div></div>
        <div class="stat"><div>Net Worth</div><div class="price" id="networth">—</div></div>
        <div class="stat"><div>Portfolio Value</div><div id="portval" class="small">—</div></div>
        <div style="height:6px"></div>
        <div class="goal">Goal: <span id="goalAmt">20000</span></div>
        <div style="height:10px"></div>

        <div class="muted">Trade</div>
        <div class="trade-panel">
          <select id="symbol" class="input"></select>
          <div class="row">
            <input id="qty" class="input" type="number" value="1" min="1" />
            <select id="side" class="input" style="max-width:120px">
              <option value="buy">Buy</option>
              <option value="sell">Sell</option>
            </select>
          </div>
          <button class="btn primary" id="tradeBtn">Execute Trade</button>
          <div style="display:flex;gap:8px;margin-top:6px">
            <button class="btn" id="chartBtn">Show Asset Chart</button>
            <button class="btn" id="submitScoreBtn">Submit Score</button>
          </div>
        </div>

        <div style="height:10px"></div>
        <div class="muted">Achievements</div>
        <div id="achievementsList" style="display:grid;gap:8px;margin-top:8px"></div>

        <div style="height:10px"></div>
        <div class="muted">Leaderboard</div>
        <div style="display:flex;gap:8px;margin-top:8px">
          <input id="playerName" class="input" placeholder="Your name" />
          <button class="btn" id="saveScoreBtn">Save</button>
        </div>
        <div class="leaderboard-list" id="leaderboard"></div>
      </aside>

      <section class="market panel">
        <div style="display:flex;justify-content:space-between;align-items:center">
          <div class="muted">Market</div>
          <div id="lastEvent" class="small muted"></div>
        </div>
        <div class="market" id="marketGrid"></div>
      </section>
    </main>

    <footer>Tip: Prices update every second. Open an asset chart to view detailed history. Progress saved locally.</footer>
  </div>

  <!-- Modal: Charts -->
  <div class="modal-backdrop" id="modal">
    <div class="modal" role="dialog" aria-modal="true">
      <header>
        <div id="modalTitle"><strong>Asset Chart</strong></div>
        <div><button class="btn" id="closeModal">Close</button></div>
      </header>
      <canvas id="assetChart" class="chart-canvas"></canvas>
    </div>
  </div>

  <!-- Toast area -->
  <div id="toastArea"></div>

<script>
/* ------------------------
   CONFIG + INITIAL STATE
   ------------------------ */
const TICK_MS = 1000;
const START_CASH = 5000;
const GOAL = 20000;
const HISTORY_LEN = 120;
const ASSETS_DEF = [
  { name:'NeoCoin', sym:'NEO', price:42.5, vol:0.02 },
  { name:'Aether', sym:'ATH', price:128.0, vol:0.016 },
  { name:'ByteRod', sym:'BYT', price:9.8, vol:0.035 },
  { name:'QuantumX', sym:'QTX', price:310.0, vol:0.01 },
  { name:'GreenBond', sym:'GBD', price:78.2, vol:0.012 }
];

let state = {
  tick: 0, cash: START_CASH, paused: false, speedMultiplier: 1,
  assets: {}, portfolio: {}, lastEvent: null, soundOn: true
};

/* ------------------------
   UTILS
   ------------------------ */
const $ = id => document.getElementById(id);
const rand = (a=0,b=1) => Math.random()*(b-a)+a;
const clamp = (v,m,M) => Math.max(m,Math.min(M,v));
const format = n => (Math.round(n*100)/100).toLocaleString();
function notify(text, time=3500){ const t=document.createElement('div'); t.className='toast'; t.textContent=text; document.body.appendChild(t); setTimeout(()=>t.remove(),time); }
function achievementToast(text){ const a=document.createElement('div'); a.className='achievement'; a.textContent=text; document.body.appendChild(a); setTimeout(()=>a.remove(),4000); }

/* ------------------------
   SOUND (WebAudio)
   ------------------------ */
const audioCtx = new (window.AudioContext || window.webkitAudioContext)();
function playTone(freq=440,dur=0.08,type='sine',gain=0.06){
  if(!state.soundOn) return;
  const o = audioCtx.createOscillator();
  const g = audioCtx.createGain();
  o.type = type; o.frequency.value = freq;
  g.gain.value = 0;
  o.connect(g); g.connect(audioCtx.destination);
  const now = audioCtx.currentTime;
  g.gain.linearRampToValueAtTime(gain, now + 0.01);
  o.start(now);
  g.gain.exponentialRampToValueAtTime(0.0001, now + dur);
  o.stop(now + dur + 0.02);
}
function playClick(){ playTone(720,0.06,'sawtooth',0.04); }
function playBuy(){ playTone(880,0.12,'sine',0.08); }
function playSell(){ playTone(340,0.12,'sine',0.08); }
function playEventGood(){ playTone(1200,0.18,'triangle',0.08); }
function playEventBad(){ playTone(220,0.18,'triangle',0.08); }

/* ------------------------
   ASSET + MARKET LOGIC
   ------------------------ */
function initAssets(){
  state.assets = {};
  ASSETS_DEF.forEach(a => {
    const hist = Array.from({length:HISTORY_LEN},()=>a.price);
    state.assets[a.sym] = { name:a.name, sym:a.sym, price:a.price, base:a.price, vol:a.vol, history:hist };
  });
}
let globalVolMultiplier = 1;
function triggerEvent(){
  const roll = Math.random();
  if(roll < 0.42){
    globalVolMultiplier = rand(1.4,2.2);
    state.lastEvent = 'Positive news — market uplift';
    playEventGood();
  } else if(roll < 0.84){
    globalVolMultiplier = rand(1.8,3.2);
    state.lastEvent = 'Negative shock — market jitter';
    playEventBad();
  } else {
    const syms = Object.keys(state.assets);
    const sym = syms[Math.floor(Math.random()*syms.length)];
    const a = state.assets[sym];
    const sign = Math.random()<0.5?1:-1;
    const mag = rand(0.07,0.35)*sign;
    a.price = Math.max(0.05, a.price*(1+mag));
    a.history.push(a.price); if(a.history.length>HISTORY_LEN) a.history.shift();
    state.lastEvent = `${a.sym} special news ${mag>0?'+':''}${Math.round(mag*100)}%`;
    if(mag>0) playEventGood(); else playEventBad();
  }
}
function marketTick(){
  state.tick++;
  if(Math.random()<0.06) triggerEvent();
  else globalVolMultiplier = clamp(globalVolMultiplier * 0.985,1,4);
  Object.keys(state.assets).forEach(sym=>{
    const a = state.assets[sym];
    let changePct = (rand(-1,1)+rand(-1,1)+rand(-1,1))/3;
    changePct *= a.vol * globalVolMultiplier;
    const reversion = 0.0008 * (a.base - a.price);
    changePct += reversion;
    a.price = Math.max(0.05, a.price * (1 + changePct));
    a.history.push(a.price); if(a.history.length>HISTORY_LEN) a.history.shift();
  });
  if(state.lastEvent) { $('lastEvent').textContent = state.lastEvent; setTimeout(()=>{$('lastEvent').textContent='';},4000); }
  updateUI();
  save();
}

/* ------------------------
   TRADING
   ------------------------ */
function buy(sym,qty){
  qty = Math.floor(qty);
  if(qty<=0) return false;
  const cost = state.assets[sym].price * qty;
  if(cost > state.cash) return false;
  state.cash -= cost;
  state.portfolio[sym] = (state.portfolio[sym]||0)+qty;
  playBuy();
  maybeAward('first_trade');
  return true;
}
function sell(sym,qty){
  qty = Math.floor(qty);
  if(qty<=0) return false;
  if((state.portfolio[sym]||0) < qty) return false;
  const proceeds = state.assets[sym].price * qty;
  state.cash += proceeds;
  state.portfolio[sym] -= qty; if(state.portfolio[sym]<=0) delete state.portfolio[sym];
  playSell();
  return true;
}

/* ------------------------
   PERSISTENCE: save/load state+leaderboard+achievements
   ------------------------ */
const STORAGE_KEY = 'ik_trader_v2';
const LB_KEY = 'ik_trader_lb_v1';
const ACH_KEY = 'ik_trader_ach_v1';

function save(){
  try{
    const s = { tick:state.tick, cash:state.cash, portfolio:state.portfolio, assets:{} };
    Object.keys(state.assets).forEach(sym=>{
      s.assets[sym] = { price: state.assets[sym].price, history: state.assets[sym].history };
    });
    localStorage.setItem(STORAGE_KEY, JSON.stringify(s));
  }catch(e){}
}
function load(){
  try{
    const raw = localStorage.getItem(STORAGE_KEY); if(!raw) return false;
    const s = JSON.parse(raw);
    state.tick = s.tick||0; state.cash = s.cash ?? START_CASH; state.portfolio = s.portfolio||{};
    Object.keys(s.assets||{}).forEach(sym=>{
      if(state.assets[sym]) {
        state.assets[sym].price = s.assets[sym].price || state.assets[sym].price;
        state.assets[sym].history = s.assets[sym].history || state.assets[sym].history;
      }
    });
    return true;
  }catch(e){ return false; }
}

/* ------------------------
   UI: Market + Portfolio + Sparklines
   ------------------------ */
const marketGrid = $('marketGrid'), symbolSelect = $('symbol'), portfolioEl = $('portfolio'),
      cashEl = $('cash'), netEl = $('networth'), portValEl = $('portval'), tickEl = $('tick'),
      goalEl = $('goalAmt'), pauseBtn = $('pauseBtn'), resetBtn = $('resetBtn'),
      tradeBtn = $('tradeBtn'), qtyIn = $('qty'), sideIn = $('side'), fastBtn = $('fastBtn'),
      themeToggle = $('themeToggle'), soundToggle = $('soundToggle'),
      chartBtn = $('chartBtn'), modal = $('modal'), closeModal = $('closeModal'), assetChartCanvas = $('assetChart'),
      leaderboardEl = $('leaderboard'), playerNameEl = $('playerName'), saveScoreBtn = $('saveScoreBtn'), submitScoreBtn = $('submitScoreBtn'),
      achievementsListEl = $('achievementsList');

function makeSparklineCanvas(history){ const c=document.createElement('canvas'); c.width=220; c.height=48; c.className='spark'; drawSpark(c,history); return c; }
function drawSpark(canvas, history){ const ctx=canvas.getContext('2d'), w=canvas.width, h=canvas.height; ctx.clearRect(0,0,w,h); ctx.lineWidth=2; const arr=history.slice(-Math.min(history.length,220)); const min=Math.min(...arr), max=Math.max(...arr); const range=Math.max(0.0001, max-min); ctx.beginPath(); for(let i=0;i<arr.length;i++){ const x=(i/(arr.length-1||1))*(w-4)+2; const y=h-4-((arr[i]-min)/range)*(h-8); if(i===0) ctx.moveTo(x,y); else ctx.lineTo(x,y); } const col = arr[arr.length-1] >= arr[0] ? '#34d399' : '#ff7b7b'; ctx.strokeStyle = col; ctx.stroke(); }

function renderMarket(){
  marketGrid.innerHTML=''; symbolSelect.innerHTML='';
  Object.keys(state.assets).forEach(sym=>{
    const a = state.assets[sym];
    const card = document.createElement('div'); card.className='card';
    const trend = a.history[a.history.length-1] - a.history[a.history.length-2];
    const upDown = trend>=0 ? 'ticker-up' : 'ticker-down';
    card.innerHTML = `
      <div style="display:flex;gap:10px;align-items:center">
        <div class="asset-thumb">${a.sym}</div>
        <div style="flex:1">
          <div style="display:flex;justify-content:space-between;align-items:center">
            <div><strong>${a.name}</strong><div class="small">${a.sym}</div></div>
            <div style="text-align:right">
              <div class="${upDown}">${format(a.price)}</div>
              <div class="small">${trend>=0?'+':''}${format(trend)}</div>
            </div>
          </div>
          <div style="height:6px"></div>
          <div id="spark-${a.sym}"></div>
        </div>
      </div>
    `;
    const sparkWrapper = card.querySelector(`#spark-${a.sym}`);
    sparkWrapper.appendChild(makeSparklineCanvas(a.history));
    // click to quick-select and show chart
    card.addEventListener('click', ()=>{ symbolSelect.value = a.sym; showAssetChart(a.sym); });
    marketGrid.appendChild(card);

    const opt = document.createElement('option'); opt.value = a.sym; opt.textContent = `${a.sym} — ${a.name}`; symbolSelect.appendChild(opt);
  });
}

function renderPortfolio(){
  portfolioEl.innerHTML=''; let portVal = 0;
  Object.keys(state.portfolio).forEach(sym=>{
    const qty = state.portfolio[sym], a = state.assets[sym]; const val = qty * a.price; portVal += val;
    const row = document.createElement('div'); row.className='asset-row';
    row.innerHTML = `
      <div class="asset-thumb">${sym}</div>
      <div class="asset-info">
        <div style="display:flex;justify-content:space-between">
          <div><strong>${a.name}</strong><div class="small">${sym} • ${qty} units</div></div>
          <div style="text-align:right"><div class="small">Value</div><div class="price">${format(val)}</div></div>
        </div>
      </div>
    `;
    portfolioEl.appendChild(row);
  });
  const net = state.cash + portVal;
  cashEl.textContent = format(state.cash); netEl.textContent = format(net); portValEl.textContent = format(portVal); tickEl.textContent = state.tick; goalEl.textContent = GOAL.toLocaleString();
  // update sparklines
  Object.keys(state.assets).forEach(sym=>{
    const canvas = document.querySelector(`#spark-${sym} canvas`);
    if(canvas) drawSpark(canvas,state.assets[sym].history);
  });
  if(net >= GOAL) onGoalReached();
}

/* ------------------------
   CHARTS (Chart.js)
   ------------------------ */
let chartInstance = null;
function showAssetChart(sym){
  const asset = state.assets[sym];
  if(!asset) return;
  modal.style.display = 'flex';
  const ctx = assetChartCanvas.getContext('2d');
  if(chartInstance) chartInstance.destroy();
  const labels = asset.history.map((_,i)=>i - asset.history.length + 1);
  const ma = movingAverage(asset.history, 8);
  chartInstance = new Chart(ctx, {
    type: 'line',
    data: {
      labels,
      datasets: [
        { label: `${asset.sym} price`, data: asset.history, borderWidth:2, tension:0.25, pointRadius:0 },
        { label: 'MA(8)', data: ma, borderWidth:1.5, borderDash:[6,4], tension:0.25, pointRadius:0 }
      ]
    },
    options: {
      scales: { x:{ display:false }, y:{ ticks:{callback: v => format(v) } } },
      plugins: { legend:{ position:'top' } }
    }
  });
}

function movingAverage(arr, n=5){
  const res = [];
  for(let i=0;i<arr.length;i++){
    const slice = arr.slice(Math.max(0,i-n+1), i+1);
    const avg = slice.reduce((s,x)=>s+x,0)/slice.length;
    res.push(avg);
  }
  return res;
}

/* ------------------------
   LEADERBOARD
   ------------------------ */
function loadLeaderboard(){
  try{
    const raw = localStorage.getItem(LB_KEY);
    return raw ? JSON.parse(raw) : [];
  }catch(e){ return []; }
}
function saveLeaderboard(list){
  try{ localStorage.setItem(LB_KEY, JSON.stringify(list)); }catch(e){}
}
function addScore(name, score){
  const list = loadLeaderboard();
  list.push({ name, score, t: Date.now() });
  list.sort((a,b)=>b.score-a.score);
  saveLeaderboard(list.slice(0,30));
  renderLeaderboard();
}
function renderLeaderboard(){
  const list = loadLeaderboard();
  leaderboardEl.innerHTML = '';
  list.forEach((r, i) => {
    const row = document.createElement('div'); row.className='leader-row';
    row.innerHTML = `<div>${i+1}. ${r.name}</div><div><strong>${format(r.score)}</strong></div>`;
    leaderboardEl.appendChild(row);
  });
}

/* ------------------------
   ACHIEVEMENTS
   ------------------------ */
const ACHIEVEMENTS_DEF = [
  { id:'first_trade', title:'First Trade', desc:'Execute your first trade.' },
  { id:'big_profit', title:'Big Profit', desc:'Reach net worth >= 2× starting cash.' },
  { id:'diversify', title:'Diversify', desc:'Hold at least 3 different assets simultaneously.' },
  { id:'hold_long', title:'Patient Holder', desc:'Keep one position for 120 ticks.' }
];
function loadAchievements(){ try{ const raw = localStorage.getItem(ACH_KEY); return raw?JSON.parse(raw):{}; }catch(e){ return {}; } }
function saveAchievements(obj){ try{ localStorage.setItem(ACH_KEY, JSON.stringify(obj)); }catch(e){} }
let achievements = loadAchievements();
function maybeAward(id){
  if(achievements[id]) return;
  // conditions:
  if(id === 'first_trade') { achievements[id] = true; saveAchievements(achievements); achievementToast('Achievement: First Trade 🎉'); renderAchievements(); }
}
function evaluateAchievements(){
  const net = state.cash + getPortfolioValue();
  if(!achievements['big_profit'] && net >= START_CASH*2){ achievements['big_profit']=true; saveAchievements(achievements); achievementToast('Achievement: Big Profit 🤑'); }
  if(!achievements['diversify']){
    if(Object.keys(state.portfolio).length >= 3){ achievements['diversify']=true; saveAchievements(achievements); achievementToast('Achievement: Diversified Portfolio 🌈'); }
  }
  // hold_long is more sophisticated: we track the age of holdings
  if(!achievements['hold_long']){
    // store ages in achievements.trackHold (not user-visible)
    achievements._holdAges = achievements._holdAges || {};
    Object.keys(state.portfolio).forEach(sym => {
      achievements._holdAges[sym] = (achievements._holdAges[sym]||0) + 1;
      if(achievements._holdAges[sym] >= 120){ achievements['hold_long']=true; saveAchievements(achievements); achievementToast('Achievement: Patient Holder ⏳'); }
    });
    // reset ages for removed positions
    Object.keys(achievements._holdAges||{}).forEach(sym=>{ if(!state.portfolio[sym]) delete achievements._holdAges[sym]; });
    saveAchievements(achievements);
  }
  renderAchievements();
}
function renderAchievements(){
  achievementsListEl.innerHTML='';
  ACHIEVEMENTS_DEF.forEach(a => {
    const got = achievements[a.id];
    const el = document.createElement('div');
    el.className='asset-row';
    el.style.justifyContent='space-between';
    el.innerHTML = `<div><strong>${a.title}</strong><div class="small">${a.desc}</div></div><div style="align-self:center">${got?'<span class="small" style="color:var(--success)">✓</span>':'—'}</div>`;
    achievementsListEl.appendChild(el);
  });
}

/* ------------------------
   CONFETTI (simple)
   ------------------------ */
const confettiCanvas = $('confetti'); const confettiCtx = confettiCanvas.getContext('2d'); let confettiPieces=[];
function resizeConfetti(){ confettiCanvas.width = window.innerWidth; confettiCanvas.height = window.innerHeight; }
function spawnConfetti(){
  confettiCanvas.style.display='block'; confettiPieces = [];
  for(let i=0;i<120;i++){
    confettiPieces.push({
      x: Math.random()*confettiCanvas.width, y: Math.random()*-confettiCanvas.height,
      vx: rand(-1,1), vy: rand(2,6), r: rand(4,9), rot: rand(0,Math.PI*2), color: `hsl(${rand(0,360)},80%,60%)`
    });
  }
  requestAnimationFrame(confettiFrame); setTimeout(()=>{ confettiCanvas.style.display='none'; confettiPieces=[]; }, 4500);
}
function confettiFrame(){
  confettiCtx.clearRect(0,0,confettiCanvas.width,confettiCanvas.height);
  confettiPieces.forEach(p=>{
    p.x += p.vx; p.y += p.vy; p.vy += 0.06;
    confettiCtx.save(); confettiCtx.translate(p.x,p.y); confettiCtx.rotate(p.rot); confettiCtx.fillStyle = p.color;
    confettiCtx.fillRect(-p.r/2, -p.r/2, p.r, p.r*2);
    confettiCtx.restore();
  });
  if(confettiPieces.length) requestAnimationFrame(confettiFrame);
}

/* ------------------------
   GAME LOOP CONTROLS
   ------------------------ */
let intervalId = null;
function startLoop(){
  if(intervalId) clearInterval(intervalId);
  intervalId = setInterval(()=>{
    if(!state.paused){
      for(let i=0;i<state.speedMultiplier;i++) marketTick();
      evaluateAchievements();
    }
  }, TICK_MS / state.speedMultiplier);
}

/* ------------------------
   UI HOOKS / BINDINGS
   ------------------------ */
tradeBtn.addEventListener('click', ()=>{
  const sym = symbolSelect.value; const side = sideIn.value; const qty = parseInt(qtyIn.value)||0;
  if(qty<=0){ alert('Quantity must be > 0'); return; }
  if(side==='buy'){ if(!buy(sym,qty)){ alert('Not enough cash'); return; } } else { if(!sell(sym,qty)){ alert('Not enough holdings'); return; } }
  renderPortfolio(); save();
});
pauseBtn.addEventListener('click', ()=>{ state.paused = !state.paused; pauseBtn.textContent = state.paused ? 'Resume' : 'Pause'; });
resetBtn.addEventListener('click', ()=>{ if(!confirm('Reset game and clear progress?')) return; localStorage.removeItem(STORAGE_KEY); localStorage.removeItem(ACH_KEY); initAssets(); state.cash = START_CASH; state.portfolio={}; state.tick=0; achievements={}; renderAll(); save(); });
fastBtn.addEventListener('click', ()=>{ state.speedMultiplier = state.speedMultiplier===1?2:1; fastBtn.textContent = state.speedMultiplier===1? 'Fast ×2' : 'Fast ×2 (On)'; fastBtn.classList.toggle('pulse'); startLoop(); });
themeToggle.addEventListener('click', ()=>{ document.body.classList.toggle('light'); });
soundToggle.addEventListener('click', ()=>{ state.soundOn = !state.soundOn; soundToggle.textContent = state.soundOn? '🔊 On' : '🔇 Off'; });

$('chartBtn').addEventListener('click', ()=> showAssetChart(symbolSelect.value));
$('closeModal').addEventListener('click', ()=> { modal.style.display='none'; if(chartInstance) chartInstance.destroy(); });

saveScoreBtn.addEventListener('click', ()=> {
  const name = (playerNameEl.value || 'Anon').trim();
  const score = Math.round(state.cash + getPortfolioValue());
  addScore(name, score);
  notify('Score saved to leaderboard');
});
submitScoreBtn.addEventListener('click', ()=> {
  const name = (playerNameEl.value || 'Anon').trim();
  const score = Math.round(state.cash + getPortfolioValue());
  addScore(name, score);
  notify('Submitted score!');
});

/* ------------------------
   WIN CONDITION
   ------------------------ */
let goalReached = false;
function onGoalReached(){
  if(goalReached) return;
  goalReached = true;
  notify('Goal reached! 🎉 Net worth achieved');
  achievementToast('Goal reached — Congrats!');
  addScore('Auto', Math.round(state.cash + getPortfolioValue()));
  resizeConfetti(); spawnConfetti();
  playTone(880,0.4,'sine',0.12);
}

/* ------------------------
   HELPERS
   ------------------------ */
function getPortfolioValue(){ return Object.keys(state.portfolio).reduce((s,sym)=> s + (state.portfolio[sym] * (state.assets[sym]?.price||0)), 0); }

/* ------------------------
   RENDER + INIT
   ------------------------ */
function updateUI(){ renderMarket(); renderPortfolio(); renderLeaderboard(); renderAchievements(); }
function renderAll(){ renderMarket(); renderPortfolio(); renderLeaderboard(); renderAchievements(); }

initAssets();
load(); renderAll();
startLoop();
window.addEventListener('resize', resizeConfetti);

/* expose for debugging */
window.ik_state = state;
window.ik_addScore = addScore;

</script>
</body>
</html>
