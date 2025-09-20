<!DOCTYPE html>
<html lang="en">
<head>
<meta charset="UTF-8">
<meta name="viewport" content="width=device-width,initial-scale=1.0">
<title>InfoKrazy Trading Game</title>
<style>
body {margin:0; font-family:sans-serif; background:#0b1020; color:#eaf0ff;}
header {text-align:center; padding:20px; font-size:1.5rem; background:#0f1724; font-weight:bold;}
main {display:flex; gap:20px; padding:20px; max-width:1000px; margin:auto;}
aside {flex:1; background:#0f1724; padding:10px; border-radius:10px;}
section {flex:2; display:grid; grid-template-columns:repeat(auto-fit,minmax(150px,1fr)); gap:10px;}
.stock-card {background:rgba(255,255,255,0.05); padding:10px; border-radius:10px; cursor:pointer;}
button {padding:5px 10px; margin-top:5px; cursor:pointer; border:none; border-radius:5px; background:linear-gradient(90deg,#7c5cff,#00c2ff); color:#021027;}
input {width:60px; border-radius:5px; border:none; padding:3px; margin-top:5px;}
</style>
</head>
<body>
<header>InfoKrazy Trading Game</header>
<main>
<aside>
  <div>Cash: $<span id="cash">0</span></div>
  <div>Portfolio Value: $<span id="portval">0</span></div>
  <div>Net Worth: $<span id="networth">0</span></div>
  <hr>
  <div><strong>Portfolio:</strong></div>
  <div id="portfolio"></div>
</aside>
<section id="market"></section>
</main>

<script>
const START_CASH = 5000;
let cash = START_CASH;
let portfolio = {};
const ASSETS = [
  {name:'NeoCoin', sym:'NEO', price:42.5, vol:0.02},
  {name:'Aether', sym:'ATH', price:128, vol:0.016},
  {name:'ByteRod', sym:'BYT', price:9.8, vol:0.035},
  {name:'QuantumX', sym:'QTX', price:310, vol:0.01},
  {name:'GreenBond', sym:'GBD', price:78.2, vol:0.012}
];

// Helper functions
function format(n){return Math.round(n*100)/100;}
function portfolioValue(){return Object.keys(portfolio).reduce((sum,sym)=>sum + portfolio[sym]*ASSETS.find(a=>a.sym===sym).price,0);}

// Update UI
function updateUI(){
  document.getElementById('cash').textContent = format(cash);
  const portVal = portfolioValue();
  document.getElementById('portval').textContent = format(portVal);
  document.getElementById('networth').textContent = format(cash+portVal);
  const portDiv = document.getElementById('portfolio');
  portDiv.innerHTML = '';
  for(const sym in portfolio){
    const qty = portfolio[sym];
    const price = ASSETS.find(a=>a.sym===sym).price;
    const div = document.createElement('div');
    div.textContent = `${sym} x${qty} = $${format(qty*price)}`;
    portDiv.appendChild(div);
  }
}

// Render market
function renderMarket(){
  const market = document.getElementById('market');
  market.innerHTML = '';
  ASSETS.forEach(a=>{
    const card = document.createElement('div'); card.className='stock-card';
    card.innerHTML = `<div><strong>${a.name} (${a.sym})</strong></div>
      <div>Price: $<span class="price">${format(a.price)}</span></div>
      <input type="number" value="1" min="1" class="qty">
      <button class="buy">Buy</button>
      <button class="sell">Sell</button>`;
    // Buy
    card.querySelector('.buy').addEventListener('click', ()=>{
      const qty = parseInt(card.querySelector('.qty').value) || 0;
      const cost = qty*a.price;
      if(cost>cash){alert('Not enough cash'); return;}
      cash -= cost;
      portfolio[a.sym] = (portfolio[a.sym]||0)+qty;
      updateUI();
    });
    // Sell
    card.querySelector('.sell').addEventListener('click', ()=>{
      const qty = parseInt(card.querySelector('.qty').value) || 0;
      if((portfolio[a.sym]||0)<qty){alert('Not enough holdings'); return;}
      cash += qty*a.price;
      portfolio[a.sym]-=qty;
      if(portfolio[a.sym]<=0) delete portfolio[a.sym];
      updateUI();
    });
    market.appendChild(card);
  });
}

// Update stock prices
function tickMarket(){
  ASSETS.forEach(a=>{
    const change = (Math.random()-0.5)*2*a.vol*a.price;
    a.price = Math.max(0.1, a.price+change);
  });
  // Update prices in UI
  document.querySelectorAll('.stock-card').forEach((card,i)=>{
    card.querySelector('.price').textContent = format(ASSETS[i].price);
  });
  updateUI();
}

init();
function init(){cash = START_CASH; portfolio={}; renderMarket(); updateUI();}
setInterval(tickMarket,10000);
</script>
</body>
</html>
