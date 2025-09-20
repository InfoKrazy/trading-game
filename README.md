<!doctype html>
<html lang="en">
<head>
<meta charset="utf-8" />
<meta name="viewport" content="width=device-width,initial-scale=1" />
<title>InfoKrazy — Trading RNG Game</title>
<link href="https://fonts.googleapis.com/css2?family=Inter:wght@400;600;700&display=swap" rel="stylesheet">
<style>
  :root{
    --bg:#0b1020; --card:#0f1724; --muted:#9aa7bf; --accent:#7c5cff; --accent-2:#00c2ff;
    --glass: rgba(255,255,255,0.03); --radius:14px; --success:#34d399; --danger:#ff7b7b;
    color-scheme: dark;
  }
  html,body{height:100%;margin:0;font-family:Inter,system-ui,-apple-system,Segoe UI,Roboto,'Helvetica Neue',Arial;background:linear-gradient(180deg,#04060b 0%,#071129 100%);color:#eaf0ff}
  .app{max-width:1100px;margin:36px auto;padding:22px}
  header{display:flex;align-items:center;justify-content:space-between;gap:12px;margin-bottom:14px}
  .brand{display:flex;gap:12px;align-items:center}
  .logo{width:52px;height:52px;border-radius:12px;background:linear-gradient(135deg,var(--accent),var(--accent-2));display:flex;align-items:center;justify-content:center;font-weight:800;color:#021027}
  h1{margin:0;font-size:1.25rem}
  .muted{color:var(--muted);font-size:0.9rem}
  .controls{display:flex;gap:8px;align-items:center}
  .btn{background:var(--glass);border:1px solid rgba(255,255,255,0.04);padding:10px 12px;border-radius:10px;color:var(--muted);cursor:pointer;font-weight:600}
  .btn.primary{background:linear-gradient(90deg,var(--accent),var(--accent-2));color:#021027;border:none;box-shadow:0 10px 30px rgba(124,92,255,0.12)}
  .main{display:grid;grid-template-columns:320px 1fr;gap:18px}
  .panel{background:linear-gradient(180deg, rgba(255,255,255,0.02), rgba(255,255,255,0.01));padding:14px;border-radius:12px;border:1px solid rgba(255,255,255,0.03)}
  .sidebar .stat{display:flex;justify-content:space-between;margin-bottom:10px}
  .portfolio-list{display:grid;gap:10px;margin-top:12px}
  .asset-row{display:flex;align-items:center;gap:10px;padding:10px;border-radius:10px;background:rgba(255,255,255,0.015);border:1px solid rgba(255,255,255,0.02);transition:transform .18s ease}
  .asset-row:hover{transform:translateY(-6px)}
  .asset-thumb{width:48px;height:48px;border-radius:9px;display:flex;align-items:center;justify-content:center;background:linear-gradient(135deg,rgba(255,255,255,0.02),transparent);font-weight:700;color:var(--accent)}
  .asset-info{flex:1}
  .price{font-weight:700}
  .small{font-size:0.85rem;color:var(--muted)}
  .trade-panel{display:flex;flex-direction:column;gap:8px;margin-top:12px}
  .input,select{width:100%;padding:10px;border-radius:10px;border:1px solid rgba(255,255,255,0.04);background:transparent;color:inherit}
  .row{display:flex;gap:8px}
  .row .input{flex:1}
  .market{display:grid;grid-template-columns:repeat(auto-fit,minmax(220px,1fr));gap:14px;margin-top:8px}
  .card{padding:12px;border-radius:12px;background:linear-gradient(180deg, rgba(255,255,255,0.015), rgba(255,255,255,0.01));border:1px solid rgba(255,255,255,0.02);transition:transform .18s ease,box-shadow .18s}
  .card:hover{transform:translateY(-6px);box-shadow:0 14px 40px rgba(2,6,23,0.6)}
  .spark{width:100%;height:48px}
  footer{margin-top:14px;text-align:center;color:var(--muted);font-size:0.85rem}
  .goal{padding:8px;border-radius:10px;background:linear-gradient(90deg, rgba(124,92,255,0.12), rgba(0,194,255,0.06));display:inline-block;font-weight:700}
  .theme{cursor:pointer;padding:8px;border-radius:10px}
  /* light theme */
  body.light{color:#0f1724;background:linear-gradient(180deg,#f7fbff 0%,#eef6ff 100%);color-scheme: light}
  body.light .panel{background:linear-gradient(180deg, rgba(2,6,23,0.03), rgba(2,6,23,0.01));}
  body.light .card{background:#ffffff;border:1px solid rgba(2,6,23,0.04);box-shadow:0 8px 18px rgba(2,6,23,0.04)}
  body.light .muted{color:#6b7280}
  .ticker-up{color:var(--success);font-weight:800}
  .ticker-down{color:var(--danger);font-weight:800}
  /* animations */
  .pulse{animation:pulse 1.6s infinite linear}
  @keyframes pulse{0%{transform:scale(1)}50%{transform:scale(0.995)}100%{transform:scale(1)}}
  /* responsive */
  @media (max-width:980px){ .main{grid-template-columns:1fr; } .logo{display:none} .sidebar{order:2} }
</style>
</head>
<body>
  <div class="app">
    <header>
      <div class="brand">
        <div class="logo">IK</div>
        <div>
          <h1>InfoKrazy — Quick Trading</h1>
          <div class="muted">Play the RNG market — try to reach the goal!</div>
        </div>
      </div>
      <div class="controls">
        <div class="muted">Tick: <span id="tick">0</span></div>
        <button class="btn" id="pauseBtn">Pause</button>
        <button class="btn" id="resetBtn">Reset</button>
        <button class="btn primary" id="fastBtn">Fast ×2</button>
        <div class="theme" id="themeToggle" title="Toggle Light/Dark">🌓</div>
      </div>
    </header>

    <main class="main">
      <aside class="panel sidebar">
        <div class="stat"><div>Cash</div><div class="price" id="cash">—</div></div>
        <div class="stat"><div>Net Worth</div><div class="price" id="networth">—</div></div>
        <div class="stat"><div>Portfolio Value</div><div id="portval" class="small">—</div></div>
        <div style="height:8px"></div>
        <div class="goal">Goal: <span id="goalAmt">20000</span></div>
        <div style="height:12px"></div>

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
        </div>

        <div style="height:12px"></div>
        <div class="muted">Portfolio</div>
        <div class="portfolio-list" id="portfolio"></div>
      </aside>

      <section class="market">
        <!-- dynamically populated -->
      </section>
    </main>

    <footer>Tip: Prices update every second. News events randomly change volatility. Progress saved locally.</footer>
  </div>

<script>
(() => {
  // --- CONFIG
  const TICK_MS = 1000;          // base tick (can be sped up)
  const START_CASH = 5000;
  const GOAL = 20000;
  const HISTORY_LEN = 50;

  // Assets - name, ticker, base price, volatility (higher = more swings)
  const ASSETS_DEF = [
    { name: 'NeoCoin', sym: 'NEO', price: 42.5, vol: 0.02 },
    { name: 'Aether', sym: 'ATH', price: 128.0, vol: 0.016 },
    { name: 'ByteRod', sym: 'BYT', price: 9.8, vol: 0.035 },
    { name: 'QuantumX', sym: 'QTX', price: 310.0, vol: 0.01 },
    { name: 'GreenBond', sym: 'GBD', price: 78.2, vol: 0.012 }
  ];

  // --- STATE
  let state = {
    tick: 0,
    cash: START_CASH,
    speedMultiplier: 1,
    paused: false,
    assets: {},
    portfolio: {}, // sym -> qty
    lastEvent: null
  };

  // Utilities
  const $ = id => document.getElementById(id);
  const rand = (min=0,max=1) => Math.random() * (max-min) + min;
  const clamp = (v,min,max) => Math.max(min, Math.min(max, v));
  function format(n){ return (Math.round(n*100)/100).toLocaleString(); }

  // Initialize assets with history
  function initAssets(){
    state.assets = {};
    for(const a of ASSETS_DEF){
      const hist = Array.from({length: HISTORY_LEN}, (_,i) => a.price);
      state.assets[a.sym] = {
        name: a.name, sym: a.sym, price: a.price, base: a.price, vol: a.vol, history: hist
      };
    }
  }

  // Save/load local progress
  function save(){
    try{
      const s = { tick: state.tick, cash: state.cash, portfolio: state.portfolio, assets: {} };
      for(const sbl of Object.keys(state.assets)) s.assets[sbl] = { price: state.assets[sbl].price, history: state.assets[sbl].history };
      localStorage.setItem('ik_trader_v1', JSON.stringify(s));
    }catch(e){}
  }
  function load(){
    try{
      const raw = localStorage.getItem('ik_trader_v1');
      if(!raw) return false;
      const s = JSON.parse(raw);
      state.tick = s.tick || 0;
      state.cash = s.cash ?? START_CASH;
      state.portfolio = s.portfolio || {};
      for(const sym of Object.keys(s.assets || {})){
        if(state.assets[sym]){
          state.assets[sym].price = s.assets[sym].price || state.assets[sym].price;
          state.assets[sym].history = s.assets[sym].history || state.assets[sym].history;
        }
      }
      return true;
    }catch(e){ return false; }
  }

  // Market tick: random walk with drift and volatility; occasional news events affect global volatility
  let globalVolMultiplier = 1;
  function marketTick(){
    state.tick++;
    // small chance for a news event
    if(Math.random() < 0.06){
      triggerEvent();
    } else {
      // slowly decay extra volatility back to normal
      globalVolMultiplier = clamp(globalVolMultiplier * 0.985, 1, 4);
    }

    for(const sym of Object.keys(state.assets)){
      const a = state.assets[sym];
      // random percent change: gaussian-like using central-limit-ish
      let changePct = (rand(-1,1) + rand(-1,1) + rand(-1,1)) / 3; // -1..1 averaging -> mild normal-ish
      changePct *= a.vol * globalVolMultiplier; // scale by volatility
      // slight mean reversion toward base price
      const reversion = 0.0008 * (a.base - a.price);
      changePct += reversion;
      // compute new price, floor at tiny positive value
      a.price = Math.max(0.05, a.price * (1 + changePct));
      // push to history
      a.history.push(a.price);
      if(a.history.length > HISTORY_LEN) a.history.shift();
    }
    updateUI();
    save();
  }

  // Random events that affect market
  function triggerEvent(){
    const roll = Math.random();
    // good or bad global event
    if(roll < 0.4){
      // positive event; mild boom
      globalVolMultiplier = rand(1.4,2.2);
      state.lastEvent = { text: 'Positive news! Market optimism.', impact: 'good' };
    } else if(roll < 0.8){
      // negative event; sharp drop
      globalVolMultiplier = rand(1.8,3.2);
      state.lastEvent = { text: 'Negative shock — market jitters.', impact: 'bad' };
    } else {
      // asset-specific event
      const syms = Object.keys(state.assets);
      const s = syms[Math.floor(Math.random()*syms.length)];
      const a = state.assets[s];
      const sign = Math.random() < 0.5 ? 1 : -1;
      const mag = rand(0.07,0.35) * sign; // up or down 7%-35%
      a.price = Math.max(0.05, a.price * (1 + mag));
      a.history.push(a.price);
      if(a.history.length > HISTORY_LEN) a.history.shift();
      state.lastEvent = { text: `${a.name} (${a.sym}) special news! ${mag>0?'+':''}${Math.round(mag*100)}%`, impact: mag>0?'good':'bad' };
    }
  }

  // Trading
  function buy(sym, qty){
    qty = Math.floor(qty);
    if(qty <= 0) return false;
    const price = state.assets[sym].price;
    const cost = price * qty;
    if(cost > state.cash) return false;
    state.cash -= cost;
    state.portfolio[sym] = (state.portfolio[sym] || 0) + qty;
    return true;
  }
  function sell(sym, qty){
    qty = Math.floor(qty);
    if(qty <= 0) return false;
    if((state.portfolio[sym] || 0) < qty) return false;
    const price = state.assets[sym].price;
    const proceeds = price * qty;
    state.cash += proceeds;
    state.portfolio[sym] -= qty;
    if(state.portfolio[sym] <= 0) delete state.portfolio[sym];
    return true;
  }

  // UI: draw market cards & portfolio
  const marketEl = document.querySelector('.market');
  const symbolSelect = $('symbol');
  const portfolioEl = $('portfolio');
  const cashEl = $('cash');
  const netEl = $('networth');
  const portValEl = $('portval');
  const tickEl = $('tick');
  const goalEl = $('goalAmt');
  const pauseBtn = $('pauseBtn');
  const resetBtn = $('resetBtn');
  const tradeBtn = $('tradeBtn');
  const qtyIn = $('qty');
  const sideIn = $('side');
  const fastBtn = $('fastBtn');
  const themeToggle = $('themeToggle');

  function makeSparklineCanvas(history){
    const c = document.createElement('canvas');
    c.width = 220;
    c.height = 48;
    c.className = 'spark';
    drawSpark(c, history);
    return c;
  }
  function drawSpark(canvas, history){
    const ctx = canvas.getContext('2d');
    const w = canvas.width, h = canvas.height;
    ctx.clearRect(0,0,w,h);
    ctx.lineWidth = 2;
    const arr = history.slice(-Math.min(history.length, 220));
    const min = Math.min(...arr), max = Math.max(...arr);
    const range = Math.max(0.0001, max - min);
    ctx.beginPath();
    for(let i=0;i<arr.length;i++){
      const x = (i / (arr.length-1 || 1)) * (w-4) + 2;
      const y = h - 4 - ((arr[i]-min)/range) * (h-8);
      if(i===0) ctx.moveTo(x,y); else ctx.lineTo(x,y);
    }
    // color: green if last > first else red
    const col = arr[arr.length-1] >= arr[0] ? '#34d399' : '#ff7b7b';
    ctx.strokeStyle = col;
    ctx.stroke();
  }

  // render market and portfolio
  function renderMarket(){
    marketEl.innerHTML = '';
    symbolSelect.innerHTML = '';
    for(const sym of Object.keys(state.assets)){
      const a = state.assets[sym];
      const card = document.createElement('div');
      card.className = 'card';
      const upDown = a.history[a.history.length-1] >= a.history[a.history.length-2] ? 'ticker-up' : 'ticker-down';
      const trend = a.history[a.history.length-1] - a.history[a.history.length-2];
      card.innerHTML = `
        <div style="display:flex;align-items:center;gap:10px">
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
      marketEl.appendChild(card);

      // add to select
      const opt = document.createElement('option');
      opt.value = a.sym;
      opt.textContent = `${a.sym} — ${a.name}`;
      symbolSelect.appendChild(opt);
    }
  }

  function renderPortfolio(){
    portfolioEl.innerHTML = '';
    let portVal = 0;
    for(const sym of Object.keys(state.portfolio)){
      const qty = state.portfolio[sym];
      const a = state.assets[sym];
      const val = qty * a.price;
      portVal += val;
      const row = document.createElement('div');
      row.className = 'asset-row';
      row.innerHTML = `
        <div class="asset-thumb">${sym}</div>
        <div class="asset-info">
          <div style="display:flex;justify-content:space-between">
            <div><strong>${a.name}</strong><div class="small">${sym} • ${qty} units</div></div>
            <div style="text-align:right">
              <div class="small">Value</div>
              <div class="price">${format(val)}</div>
            </div>
          </div>
        </div>
      `;
      portfolioEl.appendChild(row);
    }
    const net = state.cash + portVal;
    cashEl.textContent = format(state.cash);
    netEl.textContent = format(net);
    portValEl.textContent = format(portVal);
    tickEl.textContent = state.tick;
    goalEl.textContent = GOAL.toLocaleString();

    // animate sparklines update
    for(const sym of Object.keys(state.assets)){
      const canvas = document.querySelector(`#spark-${sym} canvas`);
      if(canvas) drawSpark(canvas, state.assets[sym].history);
    }

    // win condition
    if(net >= GOAL){
      // big success - show a toast or celebratory animation
      showVictory();
    }
  }

  // show small confetti-ish effect (simple)
  function showVictory(){
    if(document.querySelector('.victory')) return;
    const el = document.createElement('div');
    el.className = 'victory';
    el.style.position = 'fixed';
    el.style.inset = '0';
    el.style.display = 'flex';
    el.style.alignItems = 'center';
    el.style.justifyContent = 'center';
    el.style.zIndex = 9999;
    el.innerHTML = `<div style="background:linear-gradient(90deg,#34d399,#6c63ff);padding:26px;border-radius:18px;color:#021027;font-weight:800;font-size:1.4rem">Goal reached! 🎉 Net worth: ${format(state.cash + getPortfolioValue())}</div>`;
    document.body.appendChild(el);
    setTimeout(()=>el.remove(),4500);
  }

  function getPortfolioValue(){
    let v = 0;
    for(const s in state.portfolio){
      v += (state.portfolio[s] || 0) * (state.assets[s]?.price || 0);
    }
    return v;
  }

  // trading UI binding
  tradeBtn.addEventListener('click', () => {
    const sym = symbolSelect.value;
    const side = sideIn.value;
    let qty = parseInt(qtyIn.value) || 0;
    if(qty <= 0){ alert('Enter a quantity > 0'); return; }
    if(side === 'buy'){
      if(!buy(sym, qty)) { alert('Not enough cash.'); return; }
    } else {
      if(!sell(sym, qty)) { alert('Not enough holdings.'); return; }
    }
    renderPortfolio();
    save();
  });

  pauseBtn.addEventListener('click', () => {
    state.paused = !state.paused;
    pauseBtn.textContent = state.paused ? 'Resume' : 'Pause';
  });

  resetBtn.addEventListener('click', () => {
    if(!confirm('Reset the game? This clears your progress.')) return;
    localStorage.removeItem('ik_trader_v1');
    initAssets();
    state.cash = START_CASH;
    state.portfolio = {};
    state.tick = 0;
    updateUI();
    save();
  });

  fastBtn.addEventListener('click', () => {
    if(state.speedMultiplier === 1){ state.speedMultiplier = 2; fastBtn.textContent = 'Fast ×2 (On)'; fastBtn.classList.add('pulse'); }
    else { state.speedMultiplier = 1; fastBtn.textContent = 'Fast ×2'; fastBtn.classList.remove('pulse'); }
  });

  themeToggle.addEventListener('click', () => {
    document.body.classList.toggle('light');
  });

  // orchestrator
  let intervalId = null;
  function startLoop(){
    if(intervalId) clearInterval(intervalId);
    intervalId = setInterval(() => {
      if(!state.paused){
        // run marketTick speedMultiplier times per TICK_MS
        for(let i=0;i<state.speedMultiplier;i++) marketTick();
      }
    }, TICK_MS / state.speedMultiplier);
  }

  // initial rendering
  function updateUI(){
    renderMarket();
    renderPortfolio();
  }

  // simple seed and init
  initAssets();
  const loaded = load();
  renderMarket();
  renderPortfolio();
  startLoop();

  // load saved state into inputs etc
  // populate symbol select after initial render
  // keep UI reactive when assets change
  // auto-save periodically
  setInterval(save, 5000);

  // small utilities for restoring persisted history
  // done above in load()

  // Expose minimal console controls for debugging (optional)
  window.ik_state = state;
  window.ik_buy = buy;
  window.ik_sell = sell;
})();
</script>
</body>
</html>
