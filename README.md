<!DOCTYPE html>
<html lang="en">
<head>
<meta charset="UTF-8">
<meta name="viewport" content="width=device-width,initial-scale=1.0">
<title>InfoKrazy Trading Game</title>
<link href="https://fonts.googleapis.com/css2?family=Inter:wght@400;600;700&display=swap" rel="stylesheet">
<script src="https://cdn.jsdelivr.net/npm/chart.js"></script>
<script src="https://cdn.jsdelivr.net/npm/chartjs-chart-financial@0.1.0/dist/chartjs-chart-financial.min.js"></script>
<style>
:root{
  --bg:#0b1020; --panel:#0f1724; --muted:#9aa7bf; --accent:#7c5cff; --accent2:#00c2ff;
  --success:#34d399; --danger:#ff7b7b; --glass: rgba(255,255,255,0.03); --radius:12px;
}
body{margin:0;font-family:Inter,sans-serif;background:var(--bg);color:#eaf0ff;}
header{display:flex;align-items:center;justify-content:center;padding:16px 0;font-size:1.3rem;font-weight:700;letter-spacing:1px;}
main{display:grid;grid-template-columns:320px 1fr;gap:18px;padding:16px;max-width:1200px;margin:auto;}
.panel{background:var(--panel);padding:14px;border-radius:var(--radius);}
.market{display:grid;grid-template-columns:repeat(auto-fit,minmax(220px,1fr));gap:12px;}
.card{background:var(--glass);border-radius:var(--radius);padding:12px;transition:transform .18s,box-shadow .18s;cursor:pointer;}
.card:hover{transform:translateY(-6px);box-shadow:0 14px 40px rgba(0,0,0,0.6);}
.asset-thumb{width:44px;height:44px;border-radius:8px;background:linear-gradient(135deg,var(--accent),var(--accent2));display:flex;align-items:center;justify-content:center;font-weight:700;color:#021027;}
.asset-info{margin-left:8px;flex:1;}
.trade-menu{display:none;margin-top:8px;padding:8px;background:rgba(255,255,255,0.02);border-radius:10px;transition:all .3s;}
.trade-menu.active{display:flex;flex-direction:column;gap:8px;}
.input,select{padding:6px;border-radius:8px;border:1px solid rgba(255,255,255,0.04);background:transparent;color:inherit;width:100%;}
button{padding:6px 8px;border:none;border-radius:8px;background:linear-gradient(90deg,var(--accent),var(--accent2));color:#021027;font-weight:600;cursor:pointer;transition:opacity .18s;}
button:hover{opacity:.8;}
.sidebar{display:flex;flex-direction:column;gap:12px;}
.stat{display:flex;justify-content:space-between;font-weight:600;}
.portfolio-list{display:grid;gap:8px;}
.asset-row{display:flex;justify-content:space-between;padding:8px;background:rgba(255,255,255,0.015);border-radius:8px;}
.chart-container{margin-top:12px;}
.modal-backdrop{position:fixed;inset:0;background:rgba(2,6,23,0.6);display:none;align-items:center;justify-content:center;padding:18px;z-index:300;}
.modal{width:100%;max-width:800px;background:var(--panel);border-radius:12px;padding:12px;border:1px solid rgba(255,255,255,0.04);}
.close-btn{align-self:flex-end;cursor:pointer;color:var(--accent);}
.estimate{font-size:0.85rem;color:var(--muted);}
footer{margin-top:16px;text-align:center;color:var(--muted);}
@media(max-width:980px){main{grid-template-columns:1fr;}}
</style>
</head>
<body>
<header>InfoKrazy — RNG Trading Game</header>
<main>
<aside class="panel sidebar">
  <div class="stat"><div>Cash</div><div id="cash">0</div></div>
  <div class="stat"><div>Net Worth</div><div id="networth">0</div></div>
  <div class="stat"><div>Portfolio Value</div><div id="portval">0</div></div>
  <div style="margin-top:12px;"><strong>Portfolio</strong></div>
  <div class="portfolio-list" id="portfolio"></div>
</aside>
<section class="market panel" id="marketGrid"></section>
</main>
<footer>Prices update every 10 seconds. Click a stock to trade and view charts.</footer>

<div class="modal-backdrop" id="modal">
  <div class="modal">
    <div style="display:flex;justify-content:space-between;align-items:center;">
      <div id="modalTitle"></div>
      <div class="close-btn" id="closeModal">✖</div>
    </div>
    <canvas id="assetChart"></canvas>
    <div style="margin-top:8px;">
      <label for="chartType">Chart type:</label>
      <select id="chartType">
        <option value="line">Line</option>
        <option value="candlestick">Candlestick</option>
      </select>
    </div>
  </div>
</div>

<script>
Chart.register(Chart.FinancialController, Chart.CandlestickElement, Chart.LinearScale, Chart.TimeScale, Chart.CategoryScale, Chart.LineController, Chart.LineElement, Chart.PointElement);

/*----------------- CONFIG -----------------*/
const START_CASH=5000,HISTORY_LEN=60;
const ASSETS_DEF=[
{name:'NeoCoin',sym:'NEO',price:42.5,vol:0.02},
{name:'Aether',sym:'ATH',price:128,vol:0.016},
{name:'ByteRod',sym:'BYT',price:9.8,vol:0.035},
{name:'QuantumX',sym:'QTX',price:310,vol:0.01},
{name:'GreenBond',sym:'GBD',price:78.2,vol:0.012}
];
let state={cash:START_CASH,assets:{},portfolio:{}};

/*----------------- INIT -----------------*/
function initAssets(){ASSETS_DEF.forEach(a=>state.assets[a.sym]={...a,history:Array.from({length:HISTORY_LEN},()=>a.price)});}
function rand(a,b){return Math.random()*(b-a)+a;}
function format(n){return Math.round(n*100)/100;}
function getPortfolioValue(){return Object.keys(state.portfolio).reduce((s,sym)=>s+state.portfolio[sym]*state.assets[sym].price,0);}
function updateUI(){document.getElementById('cash').textContent=format(state.cash);
const portVal=getPortfolioValue();
document.getElementById('portval').textContent=format(portVal);
document.getElementById('networth').textContent=format(state.cash+portVal);
renderPortfolio(); renderMarket();}

/*----------------- MARKET -----------------*/
const marketGrid=document.getElementById('marketGrid');
function renderMarket(){
  marketGrid.innerHTML='';
  Object.values(state.assets).forEach(a=>{
    const card=document.createElement('div'); card.className='card';
    card.innerHTML=`
      <div style="display:flex;align-items:center;gap:8px;">
        <div class="asset-thumb">${a.sym}</div>
        <div class="asset-info">
          <div><strong>${a.name}</strong></div>
          <div class="estimate">$${format(a.price)}</div>
        </div>
      </div>
      <div class="trade-menu">
        <input type="number" min="1" value="1" class="input qty" placeholder="Quantity">
        <select class="input side">
          <option value="buy">Buy</option>
          <option value="sell">Sell</option>
        </select>
        <div class="estimate">Estimated: $<span class="estimated">${format(a.price)}</span></div>
        <button class="tradeBtn">Execute Trade</button>
        <button class="chartBtn">Show Chart</button>
      </div>
    `;
    const tradeMenu=card.querySelector('.trade-menu');
    // prevent toggle when interacting with inputs/buttons
    card.addEventListener('click',(e)=>{
      if(e.target.closest('.trade-menu')) return;
      tradeMenu.classList.toggle('active');
    });
    const qtyInput=card.querySelector('.qty');
    const sideSelect=card.querySelector('.side');
    const estSpan=card.querySelector('.estimated');
    function updateEstimate(){estSpan.textContent=format(qtyInput.value*a.price);}
    qtyInput.addEventListener('input',updateEstimate);
    sideSelect.addEventListener('change',updateEstimate);
    updateEstimate();

    card.querySelector('.tradeBtn').addEventListener('click',e=>{
      e.stopPropagation();
      const qty=parseInt(qtyInput.value)||0;
      const side=sideSelect.value;
      if(qty<=0){alert('Qty>0');return;}
      if(side==='buy'){const cost=a.price*qty;if(cost>state.cash){alert('Not enough cash');return;}
        state.cash-=cost; state.portfolio[a.sym]=(state.portfolio[a.sym]||0)+qty;
      }else{if((state.portfolio[a.sym]||0)<qty){alert('Not enough holdings');return;}
        state.cash+=a.price*qty; state.portfolio[a.sym]-=qty;if(state.portfolio[a.sym]<=0) delete state.portfolio[a.sym];}
      updateUI();
    });

    card.querySelector('.chartBtn').addEventListener('click',e=>{e.stopPropagation(); showAssetChart(a.sym);});
    marketGrid.appendChild(card);
  });
}

/*----------------- PORTFOLIO -----------------*/
const portfolioEl=document.getElementById('portfolio');
function renderPortfolio(){portfolioEl.innerHTML='';Object.keys(state.portfolio).forEach(sym=>{
  const qty=state.portfolio[sym]; const price=state.assets[sym].price;
  const row=document.createElement('div'); row.className='asset-row';
  row.innerHTML=`<div>${sym} x${qty}</div><div>$${format(qty*price)}</div>`; portfolioEl.appendChild(row);
});}

/*----------------- PRICE UPDATE -----------------*/
function tickMarket(){Object.values(state.assets).forEach(a=>{
  const change=(Math.random()-0.5)*2*a.vol*a.price;
  a.price=Math.max(0.1,a.price+change);
  a.history.push(a.price); if(a.history.length>HISTORY_LEN) a.history.shift();
}); updateUI();}
setInterval(tickMarket,10000);

/*----------------- CHART MODAL -----------------*/
const modal=document.getElementById('modal');
const closeModal=document.getElementById('closeModal');
const chartTypeSelect=document.getElementById('chartType');
const modalTitle=document.getElementById('modalTitle');
let assetChart=null,curAsset=null;
closeModal.addEventListener('click',()=>modal.style.display='none');
chartTypeSelect.addEventListener('change',()=>drawChart(curAsset));

function showAssetChart(sym){modal.style.display='flex'; curAsset=sym;
modalTitle.textContent=`${state.assets[sym].name} (${sym}) Chart`; drawChart(sym);}

function drawChart(sym){const a=state.assets[sym]; const ctx=document.getElementById('assetChart').getContext('2d'); const type=chartTypeSelect.value;
const labels=a.history.map((_,i)=>i); if(assetChart) assetChart.destroy();
if(type==='line'){assetChart=new Chart(ctx,{type:'line',data:{labels,datasets:[{label:sym,data:a.history,borderColor:'#7c5cff',backgroundColor:'rgba(124,92,255,0.2)',tension:0.3}]},options:{responsive:true,plugins:{legend:{display:false}}}});}
else{const candleData=a.history.map((p,i)=>({x:i,o:p*rand(0.98,1.02),h:p*rand(1,1.03),l:p*rand(0.97,1),c:p*rand(0.98,1.02)}));
assetChart=new Chart(ctx,{type:'candlestick',data:{datasets:[{label:sym,data:candleData}]},options:{responsive:true}});}
}
setInterval(()=>{if(curAsset)drawChart(curAsset)},10000);

/*----------------- INIT -----------------*/
initAssets(); updateUI();
</script>
</body>
</html>
