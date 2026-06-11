<!DOCTYPE html>
<html lang="pt-BR">
<head>
<meta charset="UTF-8">
<meta name="viewport" content="width=device-width, initial-scale=1.0">
<title>LogWatch — Dashboard de Entregas</title>
<script src="https://cdnjs.cloudflare.com/ajax/libs/Chart.js/4.4.1/chart.umd.min.js"></script>
<style>
  @import url('https://fonts.googleapis.com/css2?family=Inter:wght@300;400;500;600;700&family=Space+Grotesk:wght@400;500;700&display=swap');

  :root {
    --bg: #0b0f1a;
    --surface: #111827;
    --surface2: #1a2234;
    --border: #1f2d45;
    --text: #e8edf5;
    --muted: #6b7a99;
    --accent: #3b82f6;
    --accent2: #6366f1;
    --ok: #10b981;
    --warn: #f59e0b;
    --danger: #ef4444;
    --danger-soft: rgba(239,68,68,0.12);
    --ok-soft: rgba(16,185,129,0.12);
    --warn-soft: rgba(245,158,11,0.12);
  }

  * { box-sizing: border-box; margin: 0; padding: 0; }

  body {
    background: var(--bg);
    color: var(--text);
    font-family: 'Inter', sans-serif;
    min-height: 100vh;
  }

  /* ─── HEADER ─── */
  header {
    background: var(--surface);
    border-bottom: 1px solid var(--border);
    padding: 0 28px;
    display: flex;
    align-items: center;
    justify-content: space-between;
    height: 60px;
    position: sticky;
    top: 0;
    z-index: 100;
  }
  .logo {
    font-family: 'Space Grotesk', sans-serif;
    font-size: 1.2rem;
    font-weight: 700;
    letter-spacing: -0.03em;
    display: flex;
    align-items: center;
    gap: 10px;
  }
  .logo-icon {
    width: 32px; height: 32px;
    background: linear-gradient(135deg, var(--accent), var(--accent2));
    border-radius: 8px;
    display: flex; align-items: center; justify-content: center;
    font-size: 16px;
  }
  .header-right {
    display: flex; align-items: center; gap: 12px;
    font-size: 0.78rem; color: var(--muted);
  }
  .live-badge {
    display: flex; align-items: center; gap: 5px;
    background: var(--ok-soft);
    color: var(--ok);
    border: 1px solid var(--ok);
    border-radius: 20px;
    padding: 3px 10px;
    font-size: 0.72rem;
    font-weight: 600;
  }
  .live-dot {
    width: 6px; height: 6px;
    background: var(--ok);
    border-radius: 50%;
    animation: pulse 1.8s ease-in-out infinite;
  }
  @keyframes pulse { 0%,100%{opacity:1} 50%{opacity:.3} }

  /* ─── LAYOUT ─── */
  main { padding: 24px 28px; max-width: 1400px; margin: 0 auto; }

  /* ─── FILTER BAR ─── */
  .filter-bar {
    display: flex; gap: 12px; flex-wrap: wrap;
    margin-bottom: 24px;
    background: var(--surface);
    border: 1px solid var(--border);
    border-radius: 12px;
    padding: 14px 18px;
    align-items: center;
  }
  .filter-bar label { font-size: 0.75rem; color: var(--muted); font-weight: 500; }
  .filter-group { display: flex; align-items: center; gap: 8px; }
  select {
    background: var(--surface2);
    border: 1px solid var(--border);
    color: var(--text);
    border-radius: 8px;
    padding: 6px 10px;
    font-size: 0.8rem;
    cursor: pointer;
    font-family: 'Inter', sans-serif;
  }
  select:focus { outline: none; border-color: var(--accent); }
  .filter-divider { width: 1px; height: 24px; background: var(--border); }
  .filter-title { font-size: 0.78rem; font-weight: 600; color: var(--muted); margin-right: 4px; }

  /* ─── KPI ROW ─── */
  .kpi-row {
    display: grid;
    grid-template-columns: repeat(4, 1fr);
    gap: 14px;
    margin-bottom: 24px;
  }
  .kpi {
    background: var(--surface);
    border: 1px solid var(--border);
    border-radius: 12px;
    padding: 18px 20px;
    position: relative;
    overflow: hidden;
    transition: border-color .2s;
  }
  .kpi:hover { border-color: var(--accent); }
  .kpi-label { font-size: 0.72rem; font-weight: 500; color: var(--muted); text-transform: uppercase; letter-spacing: .07em; margin-bottom: 8px; }
  .kpi-value { font-family: 'Space Grotesk', sans-serif; font-size: 2rem; font-weight: 700; line-height: 1; }
  .kpi-sub { font-size: 0.72rem; color: var(--muted); margin-top: 6px; }
  .kpi-bar { position: absolute; bottom: 0; left: 0; height: 3px; border-radius: 0 3px 0 0; }
  .kpi.danger .kpi-value { color: var(--danger); }
  .kpi.danger .kpi-bar { background: var(--danger); width: 70%; }
  .kpi.ok .kpi-value { color: var(--ok); }
  .kpi.ok .kpi-bar { background: var(--ok); width: 40%; }
  .kpi.warn .kpi-value { color: var(--warn); }
  .kpi.warn .kpi-bar { background: var(--warn); }
  .kpi.accent .kpi-value { color: var(--accent); }
  .kpi.accent .kpi-bar { background: var(--accent); width: 100%; }

  /* ─── GRID ─── */
  .grid-2 { display: grid; grid-template-columns: 1fr 1fr; gap: 14px; margin-bottom: 14px; }
  .grid-3 { display: grid; grid-template-columns: 1.2fr 1fr 0.8fr; gap: 14px; margin-bottom: 14px; }

  /* ─── CARD ─── */
  .card {
    background: var(--surface);
    border: 1px solid var(--border);
    border-radius: 12px;
    padding: 18px 20px;
  }
  .card-title {
    font-size: 0.75rem; font-weight: 600; color: var(--muted);
    text-transform: uppercase; letter-spacing: .07em;
    margin-bottom: 16px;
    display: flex; align-items: center; justify-content: space-between;
  }
  .card-title span { font-size: 0.7rem; font-weight: 400; }

  canvas { max-height: 220px; }

  /* ─── TABLE ─── */
  table { width: 100%; border-collapse: collapse; font-size: 0.8rem; }
  th {
    text-align: left; padding: 8px 10px;
    font-size: 0.68rem; font-weight: 600; color: var(--muted);
    text-transform: uppercase; letter-spacing: .06em;
    border-bottom: 1px solid var(--border);
  }
  td { padding: 9px 10px; border-bottom: 1px solid var(--border); }
  tr:last-child td { border-bottom: none; }
  tr:hover td { background: var(--surface2); }

  .badge {
    display: inline-flex; align-items: center; gap: 4px;
    border-radius: 20px; padding: 2px 9px;
    font-size: 0.68rem; font-weight: 600;
  }
  .badge.atrasado { background: var(--danger-soft); color: var(--danger); border: 1px solid var(--danger); }
  .badge.ok { background: var(--ok-soft); color: var(--ok); border: 1px solid var(--ok); }
  .badge.critico { background: rgba(239,68,68,.25); color: #fca5a5; border: 1px solid var(--danger); }

  .delay-bar-wrap { display: flex; align-items: center; gap: 8px; }
  .delay-bar-bg { flex: 1; height: 6px; background: var(--surface2); border-radius: 3px; overflow: hidden; }
  .delay-bar-fill { height: 100%; border-radius: 3px; }
  .delay-val { font-size: 0.72rem; font-weight: 600; min-width: 28px; text-align: right; }

  /* ─── RANKING ─── */
  .rank-item {
    display: flex; align-items: center; gap: 12px;
    padding: 10px 0;
    border-bottom: 1px solid var(--border);
  }
  .rank-item:last-child { border-bottom: none; }
  .rank-num {
    font-family: 'Space Grotesk', sans-serif;
    font-size: 1rem; font-weight: 700;
    width: 24px; text-align: center; color: var(--muted);
  }
  .rank-num.top { color: var(--danger); }
  .rank-info { flex: 1; }
  .rank-name { font-size: 0.83rem; font-weight: 600; }
  .rank-detail { font-size: 0.7rem; color: var(--muted); margin-top: 2px; }
  .rank-score {
    font-family: 'Space Grotesk', sans-serif;
    font-size: 1rem; font-weight: 700; color: var(--danger);
  }

  /* ─── ALERT STRIP ─── */
  .alert-strip {
    background: rgba(239,68,68,.08);
    border: 1px solid rgba(239,68,68,.3);
    border-radius: 10px;
    padding: 12px 16px;
    font-size: 0.78rem;
    color: #fca5a5;
    display: flex; align-items: center; gap: 10px;
    margin-bottom: 24px;
  }
  .alert-icon { font-size: 1rem; }

  @media (max-width: 900px) {
    .kpi-row { grid-template-columns: 1fr 1fr; }
    .grid-2, .grid-3 { grid-template-columns: 1fr; }
  }
</style>
</head>
<body>

<header>
  <div class="logo">
    <div class="logo-icon">📦</div>
    LogWatch
  </div>
  <div class="header-right">
    <span id="clock"></span>
    <div class="live-badge"><div class="live-dot"></div> Tempo real</div>
  </div>
</header>

<main>

  <!-- FILTER BAR -->
  <div class="filter-bar">
    <span class="filter-title">Filtros</span>
    <div class="filter-divider"></div>
    <div class="filter-group">
      <label>Transportadora</label>
      <select id="fTransp" onchange="applyFilters()">
        <option value="">Todas</option>
        <option value="RotaMax">RotaMax</option>
        <option value="ViaCargo">ViaCargo</option>
        <option value="FlashLog">FlashLog</option>
      </select>
    </div>
    <div class="filter-group">
      <label>Região</label>
      <select id="fRegiao" onchange="applyFilters()">
        <option value="">Todas</option>
        <option value="Sudeste">Sudeste</option>
        <option value="Sul">Sul</option>
        <option value="Nordeste">Nordeste</option>
        <option value="Norte">Norte</option>
        <option value="Centro-Oeste">Centro-Oeste</option>
      </select>
    </div>
    <div class="filter-group">
      <label>Status</label>
      <select id="fStatus" onchange="applyFilters()">
        <option value="">Todos</option>
        <option value="atrasado">Atrasados</option>
        <option value="no_prazo">No prazo</option>
      </select>
    </div>
  </div>

  <!-- ALERT -->
  <div class="alert-strip" id="alertStrip">
    <span class="alert-icon">🚨</span>
    <span id="alertText"></span>
  </div>

  <!-- KPI ROW -->
  <div class="kpi-row">
    <div class="kpi accent">
      <div class="kpi-label">Total de Entregas</div>
      <div class="kpi-value" id="kpiTotal">—</div>
      <div class="kpi-sub">Registros carregados</div>
      <div class="kpi-bar" style="width:100%"></div>
    </div>
    <div class="kpi danger">
      <div class="kpi-label">Atrasadas</div>
      <div class="kpi-value" id="kpiAtrasadas">—</div>
      <div class="kpi-sub" id="kpiAtrasadasSub">dias_reais > prazo_dias</div>
      <div class="kpi-bar"></div>
    </div>
    <div class="kpi ok">
      <div class="kpi-label">No Prazo</div>
      <div class="kpi-value" id="kpiOk">—</div>
      <div class="kpi-sub" id="kpiOkSub">dias_reais ≤ prazo_dias</div>
      <div class="kpi-bar"></div>
    </div>
    <div class="kpi warn">
      <div class="kpi-label">Atraso Máximo</div>
      <div class="kpi-value" id="kpiMaxAtraso">—</div>
      <div class="kpi-sub" id="kpiMaxAtrasoDet">dias além do prazo</div>
      <div class="kpi-bar" id="kpiWarnBar" style="width:80%"></div>
    </div>
  </div>

  <!-- ROW 1: BAR + PIE -->
  <div class="grid-2">
    <div class="card">
      <div class="card-title">Índice de Atraso por Transportadora <span>atrasos / total</span></div>
      <canvas id="chartTransp"></canvas>
    </div>
    <div class="card">
      <div class="card-title">Distribuição por Região <span>% do total filtrado</span></div>
      <canvas id="chartRegiao"></canvas>
    </div>
  </div>

  <!-- ROW 2: scatter + ranking + region heatmap -->
  <div class="grid-3">
    <div class="card">
      <div class="card-title">Prazo vs. Dias Reais <span>linha = prazo ideal</span></div>
      <canvas id="chartScatter"></canvas>
    </div>
    <div class="card">
      <div class="card-title">🔴 Ranking de Atraso <span>maior desvio</span></div>
      <div id="rankingList"></div>
    </div>
    <div class="card">
      <div class="card-title">Regiões Críticas <span>atrasos + desvio médio</span></div>
      <div id="heatmapList"></div>
    </div>
  </div>

  <!-- TABLE -->
  <div class="card">
    <div class="card-title">Detalhamento de Entregas <span id="tableCount"></span></div>
    <div style="overflow-x:auto">
      <table>
        <thead>
          <tr>
            <th>ID</th>
            <th>Transportadora</th>
            <th>Região</th>
            <th>Prazo (dias)</th>
            <th>Real (dias)</th>
            <th>Desvio</th>
            <th>Status</th>
          </tr>
        </thead>
        <tbody id="tableBody"></tbody>
      </table>
    </div>
  </div>

</main>

<script>
// ── DATA ──────────────────────────────────────────────────────────────
const RAW = [
  {id:301, transp:'RotaMax',  regiao:'Sudeste',      prazo:3, real:7},
  {id:302, transp:'ViaCargo', regiao:'Sul',           prazo:5, real:5},
  {id:303, transp:'FlashLog', regiao:'Nordeste',      prazo:4, real:9},
  {id:304, transp:'RotaMax',  regiao:'Norte',         prazo:6, real:4},
  {id:305, transp:'ViaCargo', regiao:'Centro-Oeste',  prazo:2, real:6},
  {id:306, transp:'FlashLog', regiao:'Sul',           prazo:5, real:12},
  {id:307, transp:'RotaMax',  regiao:'Sul',           prazo:6, real:9},
  {id:308, transp:'ViaCargo', regiao:'Sudeste',       prazo:3, real:4},
  {id:309, transp:'FlashLog', regiao:'Norte',         prazo:5, real:5},
  {id:310, transp:'ViaCargo', regiao:'Nordeste',      prazo:4, real:8},
].map(d => ({...d, desvio: d.real - d.prazo, atrasado: d.real > d.prazo}));

// ── CHART INSTANCES ────────────────────────────────────────────────────
let charts = {};
function destroyChart(id) { if(charts[id]){ charts[id].destroy(); delete charts[id]; } }

const COLORS = {
  RotaMax:  '#6366f1',
  ViaCargo: '#3b82f6',
  FlashLog: '#8b5cf6',
};
const REGION_COLORS = ['#ef4444','#f59e0b','#3b82f6','#10b981','#6366f1'];

function hexToRgba(hex, a) {
  const r=parseInt(hex.slice(1,3),16), g=parseInt(hex.slice(3,5),16), b=parseInt(hex.slice(5,7),16);
  return `rgba(${r},${g},${b},${a})`;
}

// ── FILTER + RENDER ────────────────────────────────────────────────────
function applyFilters() {
  const t = document.getElementById('fTransp').value;
  const r = document.getElementById('fRegiao').value;
  const s = document.getElementById('fStatus').value;
  let data = RAW.filter(d =>
    (!t || d.transp === t) &&
    (!r || d.regiao === r) &&
    (!s || (s==='atrasado' ? d.atrasado : !d.atrasado))
  );
  render(data);
}

function render(data) {
  // KPIs
  const atrasadas = data.filter(d=>d.atrasado);
  const okList    = data.filter(d=>!d.atrasado);
  document.getElementById('kpiTotal').textContent     = data.length;
  document.getElementById('kpiAtrasadas').textContent = atrasadas.length;
  document.getElementById('kpiOk').textContent        = okList.length;

  const maxDesvio = atrasadas.length ? Math.max(...atrasadas.map(d=>d.desvio)) : 0;
  const worstId   = atrasadas.length ? atrasadas.find(d=>d.desvio===maxDesvio)?.id : '—';
  document.getElementById('kpiMaxAtraso').textContent  = maxDesvio ? '+'+maxDesvio : '0';
  document.getElementById('kpiMaxAtrasoDet').textContent = worstId !== '—' ? `Entrega #${worstId}` : 'Sem atrasos';

  const pct = data.length ? Math.round(atrasadas.length/data.length*100) : 0;
  document.getElementById('kpiAtrasadasSub').textContent = `${pct}% do total`;
  document.getElementById('kpiOkSub').textContent = data.length ? `${100-pct}% do total` : '—';

  // Alert
  const alertStrip = document.getElementById('alertStrip');
  const alertText  = document.getElementById('alertText');
  if(atrasadas.length) {
    alertStrip.style.display='flex';
    const worst = [...atrasadas].sort((a,b)=>b.desvio-a.desvio).slice(0,3).map(d=>`#${d.id} (+${d.desvio}d)`).join(', ');
    alertText.textContent = `${atrasadas.length} entrega(s) com atraso detectada(s). Mais críticas: ${worst}`;
  } else {
    alertStrip.style.display='none';
  }

  renderTranspChart(data);
  renderRegiaoChart(data);
  renderScatter(data);
  renderRanking(atrasadas);
  renderHeatmap(data);
  renderTable(data);
}

// ── CHART: TRANSPORTADORA ──────────────────────────────────────────────
function renderTranspChart(data) {
  destroyChart('chartTransp');
  const transpNames = ['RotaMax','ViaCargo','FlashLog'];
  const totals    = transpNames.map(t=>data.filter(d=>d.transp===t).length);
  const atrasados = transpNames.map(t=>data.filter(d=>d.transp===t&&d.atrasado).length);

  const ctx = document.getElementById('chartTransp').getContext('2d');
  charts['chartTransp'] = new Chart(ctx, {
    type:'bar',
    data: {
      labels: transpNames,
      datasets: [
        {
          label:'Atrasadas',
          data: atrasados,
          backgroundColor: transpNames.map(t=>hexToRgba(COLORS[t],.85)),
          borderColor: transpNames.map(t=>COLORS[t]),
          borderWidth: 1.5,
          borderRadius: 6,
        },
        {
          label:'No Prazo',
          data: transpNames.map((t,i)=>totals[i]-atrasados[i]),
          backgroundColor: 'rgba(16,185,129,.2)',
          borderColor: 'rgba(16,185,129,.6)',
          borderWidth: 1.5,
          borderRadius: 6,
        }
      ]
    },
    options: {
      responsive:true, maintainAspectRatio:false,
      plugins:{ legend:{ labels:{ color:'#6b7a99', font:{size:11} } } },
      scales:{
        x:{ ticks:{color:'#6b7a99'}, grid:{color:'#1f2d45'} },
        y:{ ticks:{color:'#6b7a99', stepSize:1}, grid:{color:'#1f2d45'}, beginAtZero:true }
      }
    }
  });
}

// ── CHART: REGIAO ──────────────────────────────────────────────────────
function renderRegiaoChart(data) {
  destroyChart('chartRegiao');
  const regioes = [...new Set(data.map(d=>d.regiao))];
  const counts  = regioes.map(r=>data.filter(d=>d.regiao===r).length);
  const atrasadosByR = regioes.map(r=>data.filter(d=>d.regiao===r&&d.atrasado).length);

  const ctx = document.getElementById('chartRegiao').getContext('2d');
  charts['chartRegiao'] = new Chart(ctx, {
    type:'doughnut',
    data:{
      labels: regioes,
      datasets:[{
        data: counts,
        backgroundColor: REGION_COLORS.slice(0,regioes.length).map(c=>hexToRgba(c,.75)),
        borderColor:      REGION_COLORS.slice(0,regioes.length),
        borderWidth: 1.5,
        hoverOffset: 6,
      }]
    },
    options:{
      responsive:true, maintainAspectRatio:false,
      plugins:{
        legend:{ position:'right', labels:{ color:'#6b7a99', font:{size:11}, padding:12 } },
        tooltip:{
          callbacks:{
            afterLabel: (ctx2) => {
              const r = regioes[ctx2.dataIndex];
              const a = atrasadosByR[ctx2.dataIndex];
              return `Atrasos: ${a}`;
            }
          }
        }
      }
    }
  });
}

// ── CHART: SCATTER ─────────────────────────────────────────────────────
function renderScatter(data) {
  destroyChart('chartScatter');
  const maxVal = Math.max(...data.map(d=>Math.max(d.prazo,d.real)), 1) + 1;

  const transpNames = ['RotaMax','ViaCargo','FlashLog'];
  const datasets = transpNames
    .filter(t=>data.some(d=>d.transp===t))
    .map(t=>({
      label: t,
      data: data.filter(d=>d.transp===t).map(d=>({x:d.prazo, y:d.real, id:d.id})),
      backgroundColor: data.filter(d=>d.transp===t).map(d=>d.atrasado?'rgba(239,68,68,.8)':hexToRgba(COLORS[t],.7)),
      pointRadius: 7,
      pointHoverRadius: 9,
    }));

  // ideal line
  datasets.push({
    label:'Prazo ideal',
    data:[{x:0,y:0},{x:maxVal,y:maxVal}],
    type:'line',
    borderColor:'rgba(16,185,129,.5)',
    borderDash:[5,4],
    borderWidth:1.5,
    pointRadius:0,
    fill:false,
  });

  const ctx = document.getElementById('chartScatter').getContext('2d');
  charts['chartScatter'] = new Chart(ctx, {
    type:'scatter',
    data:{datasets},
    options:{
      responsive:true, maintainAspectRatio:false,
      plugins:{
        legend:{ labels:{ color:'#6b7a99', font:{size:11} } },
        tooltip:{
          callbacks:{
            label: item => {
              const pt = item.raw;
              if(pt.id) return `#${pt.id} — Prazo: ${pt.x}d, Real: ${pt.y}d`;
              return 'Prazo ideal';
            }
          }
        }
      },
      scales:{
        x:{ title:{display:true,text:'Prazo (dias)',color:'#6b7a99',font:{size:10}}, ticks:{color:'#6b7a99'}, grid:{color:'#1f2d45'}, min:0, max:maxVal },
        y:{ title:{display:true,text:'Dias reais',color:'#6b7a99',font:{size:10}}, ticks:{color:'#6b7a99'}, grid:{color:'#1f2d45'}, min:0, max:maxVal }
      }
    }
  });
}

// ── RANKING ────────────────────────────────────────────────────────────
function renderRanking(atrasadas) {
  const sorted = [...atrasadas].sort((a,b)=>b.desvio-a.desvio).slice(0,5);
  const el = document.getElementById('rankingList');
  if(!sorted.length){el.innerHTML='<div style="color:var(--muted);font-size:.8rem;padding:12px 0">Sem atrasos no filtro atual.</div>';return;}
  el.innerHTML = sorted.map((d,i)=>`
    <div class="rank-item">
      <div class="rank-num ${i===0?'top':''}">${i+1}</div>
      <div class="rank-info">
        <div class="rank-name">#${d.id} — ${d.transp}</div>
        <div class="rank-detail">${d.regiao} · Real: ${d.real}d · Prazo: ${d.prazo}d</div>
        <div class="delay-bar-wrap" style="margin-top:5px">
          <div class="delay-bar-bg">
            <div class="delay-bar-fill" style="width:${Math.min(d.desvio/12*100,100)}%;background:${i===0?'#ef4444':'#f59e0b'}"></div>
          </div>
          <div class="delay-val" style="color:${i===0?'#ef4444':'#f59e0b'}">+${d.desvio}d</div>
        </div>
      </div>
    </div>
  `).join('');
}

// ── HEATMAP (region summary) ───────────────────────────────────────────
function renderHeatmap(data) {
  const regioes = [...new Set(data.map(d=>d.regiao))];
  const summary = regioes.map(r=>{
    const entries = data.filter(d=>d.regiao===r);
    const atrasadas = entries.filter(d=>d.atrasado);
    const avgDesvio = atrasadas.length ? (atrasadas.reduce((a,b)=>a+b.desvio,0)/atrasadas.length).toFixed(1) : 0;
    const score = atrasadas.length * parseFloat(avgDesvio);
    return {r, total:entries.length, atrasadas:atrasadas.length, avgDesvio, score};
  }).sort((a,b)=>b.score-a.score);

  const maxScore = summary[0]?.score || 1;
  const el = document.getElementById('heatmapList');
  el.innerHTML = summary.map((s,i)=>{
    const pct = Math.round(s.score/maxScore*100);
    const color = pct>60?'#ef4444':pct>30?'#f59e0b':'#10b981';
    return `
      <div class="rank-item" style="align-items:flex-start">
        <div class="rank-num ${i===0?'top':''}">${i+1}</div>
        <div class="rank-info">
          <div class="rank-name">${s.r}</div>
          <div class="rank-detail">${s.atrasadas} atrasadas de ${s.total} · desvio médio +${s.avgDesvio}d</div>
          <div class="delay-bar-wrap" style="margin-top:5px">
            <div class="delay-bar-bg">
              <div class="delay-bar-fill" style="width:${pct}%;background:${color}"></div>
            </div>
            <div class="delay-val" style="color:${color};font-size:.65rem">${pct}%</div>
          </div>
        </div>
      </div>`;
  }).join('');
}

// ── TABLE ──────────────────────────────────────────────────────────────
function renderTable(data) {
  document.getElementById('tableCount').textContent = `${data.length} registros`;
  const sorted = [...data].sort((a,b)=>b.desvio-a.desvio);
  document.getElementById('tableBody').innerHTML = sorted.map(d=>{
    const desvioTxt = d.desvio > 0 ? `<span style="color:var(--danger);font-weight:600">+${d.desvio}</span>` :
                      d.desvio < 0 ? `<span style="color:var(--ok);font-weight:600">${d.desvio}</span>` :
                      `<span style="color:var(--muted)">0</span>`;
    const badge = d.desvio >= 5 ? `<span class="badge critico">🔴 Crítico</span>` :
                  d.atrasado    ? `<span class="badge atrasado">⚠ Atrasado</span>` :
                                  `<span class="badge ok">✔ No prazo</span>`;
    return `<tr>
      <td style="font-weight:600;color:var(--accent)">#${d.id}</td>
      <td>${d.transp}</td>
      <td>${d.regiao}</td>
      <td>${d.prazo}</td>
      <td>${d.real}</td>
      <td>${desvioTxt}</td>
      <td>${badge}</td>
    </tr>`;
  }).join('');
}

// ── CLOCK ─────────────────────────────────────────────────────────────
function updateClock(){
  document.getElementById('clock').textContent = new Date().toLocaleString('pt-BR',{hour:'2-digit',minute:'2-digit',second:'2-digit'});
}
setInterval(updateClock, 1000);
updateClock();

// ── INIT ───────────────────────────────────────────────────────────────
render(RAW);
</script>
</body>
</html>
