<!DOCTYPE html>
<html lang="es">
<head>
<meta charset="utf-8" />
<meta name="viewport" content="width=device-width,initial-scale=1" />
<title>Bezier Studio — Proyecto de Cálculo Vectorial</title>
<style>
  :root{
    --bg:#f7f7fb;
    --panel:#ffffff;
    --muted:#9aa3b2;
    --accent:#3b82f6;     /* azul principal */
    --curve:#ff7a7a;      /* coral suave */
    --polygon:#6a8bd6;    /* azul pastel */
    --points:#34b3a0;     /* verde agua */
    --cast1:#ffd86b;      /* amarillo suave */
    --cast2:#f5b981;      /* naranja suave */
    --cast3:#d593b8;      /* rosa suave */
    --velocity:#2ecc71;   /* VERDE para velocidad */
    --accel:#e74c3c;      /* ROJO para aceleración */
    --card-shadow: 0 6px 18px rgba(20,30,60,0.06);
  }
  html,body{height:100%;margin:0;font-family:Inter,system-ui,Arial,sans-serif;background:var(--bg);color:#223;}
  .app{display:flex;height:100vh;gap:18px;padding:18px;box-sizing:border-box;}
  .sidebar{
    width:320px;background:var(--panel);border-radius:12px;padding:18px;box-shadow:var(--card-shadow);display:flex;flex-direction:column;gap:12px;
  }
  .sidebar h1{margin:0;font-size:18px;letter-spacing:-0.2px;}
  .group{background:transparent;padding:8px 0;display:flex;flex-direction:column;gap:8px;}
  label.small{font-size:13px;color:var(--muted);}
  input[type="range"]{width:100%;}
  .controls{display:flex;gap:8px; flex-wrap: wrap;}
  button{
    border:0;background:#f0f3ff;border-radius:8px;padding:8px 10px;color:#1f2b46;font-weight:600;cursor:pointer;box-shadow: 0 2px 6px rgba(16,24,64,0.04);
  }
  button.ghost{background:transparent;border:1px solid #e6e9f2;}
  button.warn{background:#ffece8;color:#7a2e2e;}
  .row{display:flex;gap:8px;align-items:center;}
  .muted{color:var(--muted);font-size:13px;}
  .canvasWrap{flex:1;display:flex;align-items:center;justify-content:center;min-width:0;}
  canvas{background:#fff;border-radius:10px;border:1px solid #e6e9f2;box-shadow:var(--card-shadow);}
  .footerNote{font-size:12px;color:var(--muted);margin-top:6px;}
  .smallBtn{padding:6px 8px;font-size:13px;border-radius:6px}
  .inlineInput{display:flex;gap:8px;align-items:center}
  .labelTag{background:#eef6ff;color:var(--accent);padding:6px 8px;border-radius:999px;font-weight:600;font-size:13px}
  /* Estilos para leyenda de vectores */
  .legend { font-size: 12px; margin-top: 5px; }
  .dot { display:inline-block; width:10px; height:10px; border-radius:50%; margin-right:4px; }
</style>
</head>
<body>
<div class="app">
  <div class="sidebar">
    <h1>Bezier Studio</h1>
    <div class="group">
      <div class="row">
        <div class="labelTag">Cálculo Vectorial</div>
        <div style="flex:1"></div>
        <div class="muted">Validación Empírica</div>
      </div>
    </div>

    <div class="group">
      <label class="small">Parámetro t</label>
      <input id="tSlider" type="range" min="0" max="1" step="0.01" value="0">
      <div class="row">
        <div class="muted">t = <span id="tVal">0.00</span></div>
        <div style="flex:1"></div>
        <button id="playSlider" class="smallBtn">Animar slider</button>
        <button id="playObject" class="smallBtn">Animar objeto</button>
      </div>
    </div>

    <div class="group">
      <label class="small">Vectores (Cinemática)</label>
      <div class="legend">
        <div><span class="dot" style="background:var(--velocity)"></span>Velocidad (1ª Derivada)</div>
        <div style="margin-top:4px"><span class="dot" style="background:var(--accel)"></span>Aceleración (2ª Derivada)</div>
      </div>
      <div class="footerNote" style="margin-top:10px">
        Magnitud V: <span id="vMag">0.00</span> px/s<br>
        Magnitud A: <span id="aMag">0.00</span> px/s²
      </div>
    </div>

    <div class="group">
      <label class="small">Puntos de control</label>
      <div class="controls">
        <button id="addPoint">Añadir punto</button>
        <button id="removePoint" class="ghost">Eliminar</button>
        <button id="reset" class="ghost">Reset</button>
        
        <button id="loadParabola" class="smallBtn" style="background:#eef6ff; color:#3b82f6; width:100%; margin-top:4px;">Ver Ejemplo Parábola</button>
        <button id="loadExp1" class="smallBtn" style="background:#eef6ff; color:#3b82f6; width:100%; margin-top:4px;">▶ Cargar Exp 1: Velocidad</button>
        <button id="loadExp2" class="smallBtn" style="background:#eef6ff; color:#3b82f6; width:100%; margin-top:4px;">▶ Cargar Exp 2: Aceleración Centrípeta</button>
        <button id="loadExp3" class="smallBtn" style="background:#eef6ff; color:#3b82f6; width:100%; margin-top:4px;">▶ Cargar Exp 3: Recta Paramétrica</button>
        
        <button id="exportSVG" class="smallBtn ghost" style="margin-top:4px;">Exportar SVG</button>
      </div>
      <div class="footerNote">Arrastra puntos en el canvas para editar la curva.</div>
    </div>
  </div>

  <div class="canvasWrap">
    <canvas id="canvas" width="1000" height="700"></canvas>
  </div>
</div>

<script>
/* ---------- Setup ---------- */
const canvas = document.getElementById('canvas');
const ctx = canvas.getContext('2d');
const tSlider = document.getElementById('tSlider');
const tVal = document.getElementById('tVal');
const playSliderBtn = document.getElementById('playSlider');
const playObjectBtn = document.getElementById('playObject');
const addPointBtn = document.getElementById('addPoint');
const removePointBtn = document.getElementById('removePoint');
const resetBtn = document.getElementById('reset');
const exportSVGBtn = document.getElementById('exportSVG');
const vMagLabel = document.getElementById('vMag');
const aMagLabel = document.getElementById('aMag');

// Botones de Experimentos
const loadParabolaBtn = document.getElementById('loadParabola');
const loadExp1Btn = document.getElementById('loadExp1');
const loadExp2Btn = document.getElementById('loadExp2');
const loadExp3Btn = document.getElementById('loadExp3');

let points = [
  {x: 150, y: 550},
  {x: 350, y: 150},
  {x: 650, y: 150},
  {x: 850, y: 550}
];

let dragging = null;
let dragOffset = {x:0,y:0};
let sliderAnim = null;
let objectAnim = null;
let objectT = 0;

/* Soft palette */
const COLORS = {
  bg: '#ffffff',
  grid: '#f0f3fa',
  polygon: '#6a8bd6',
  curve: '#ff7a7a',
  points: '#34b3a0',
  cast1: '#ffd86b',
  cast2: '#f5b981',
  cast3: '#d593b8',
  velocity: '#2ecc71', // Verde
  accel: '#e74c3c',    // Rojo
  guide: 'rgba(30,35,60,0.08)'
};

/* ---------- Math helpers ---------- */
function lerp(a,b,t){ return { x: a.x + (b.x - a.x)*t, y: a.y + (b.y - a.y)*t }; }

/* CALCULO DE VECTORES (FÍSICA) */
function getVelocity(t, pts) {
    if (pts.length !== 4) return {x:0, y:0}; // Solo para cúbicas por ahora
    const P0 = pts[0], P1 = pts[1], P2 = pts[2], P3 = pts[3];
    
    const k1 = 3 * Math.pow(1-t, 2);
    const k2 = 6 * (1-t) * t;
    const k3 = 3 * Math.pow(t, 2);

    return {
        x: k1*(P1.x - P0.x) + k2*(P2.x - P1.x) + k3*(P3.x - P2.x),
        y: k1*(P1.y - P0.y) + k2*(P2.y - P1.y) + k3*(P3.y - P2.y)
    };
}

function getAcceleration(t, pts) {
    if (pts.length !== 4) return {x:0, y:0};
    const P0 = pts[0], P1 = pts[1], P2 = pts[2], P3 = pts[3];

    const k1 = 6 * (1-t);
    const k2 = 6 * t;

    return {
        x: k1*(P2.x - 2*P1.x + P0.x) + k2*(P3.x - 2*P2.x + P1.x),
        y: k1*(P2.y - 2*P1.y + P0.y) + k2*(P3.y - 2*P2.y + P1.y)
    };
}

/* De Casteljau iterative */
function deCasteljau(pts, t){
  let layer = pts.map(p => ({x:p.x, y:p.y}));
  while(layer.length > 1){
    const next = [];
    for(let i=0;i<layer.length-1;i++){
      next.push(lerp(layer[i], layer[i+1], t));
    }
    layer = next;
  }
  return layer[0];
}

/* Sample curve */
function sampleCurve(steps=200){
  const arr = [];
  for(let i=0;i<=steps;i++){
    const t = i/steps;
    arr.push(deCasteljau(points, t));
  }
  return arr;
}

/* ---------- Drawing ---------- */
function clear(){
  ctx.clearRect(0,0,canvas.width,canvas.height);
}

function drawGrid(){
  ctx.strokeStyle = COLORS.grid;
  ctx.lineWidth = 1;
  for(let x=0;x<canvas.width;x+=50){
    ctx.beginPath(); ctx.moveTo(x,0); ctx.lineTo(x,canvas.height); ctx.stroke();
  }
  for(let y=0;y<canvas.height;y+=50){
    ctx.beginPath(); ctx.moveTo(0,y); ctx.lineTo(canvas.width,y); ctx.stroke();
  }
}

function drawControlPolygon(){
  if(points.length < 2) return;
  ctx.strokeStyle = COLORS.polygon;
  ctx.lineWidth = 2;
  ctx.beginPath();
  ctx.moveTo(points[0].x, points[0].y);
  for(let i=1;i<points.length;i++) ctx.lineTo(points[i].x, points[i].y);
  ctx.stroke();
  ctx.fillStyle = 'rgba(106,139,214,0.06)';
  for(let i=0;i<points.length-1;i++){
    const a=points[i], b=points[i+1];
    ctx.beginPath(); ctx.arc((a.x+b.x)/2,(a.y+b.y)/2,6,0,Math.PI*2); ctx.fill();
  }
}

function drawBezier(){
  const samples = sampleCurve(400);
  ctx.strokeStyle = COLORS.curve;
  ctx.lineWidth = 3;
  ctx.lineJoin = 'round';
  ctx.lineCap = 'round';
  ctx.beginPath();
  ctx.moveTo(samples[0].x, samples[0].y);
  for(let i=1;i<samples.length;i++) ctx.lineTo(samples[i].x, samples[i].y);
  ctx.stroke();
}

function drawPoints(){
  for(let i=0;i<points.length;i++){
    const p = points[i];
    ctx.fillStyle = 'rgba(52,179,160,0.10)';
    ctx.beginPath(); ctx.arc(p.x,p.y,14,0,Math.PI*2); ctx.fill();
    ctx.fillStyle = COLORS.points;
    ctx.beginPath(); ctx.arc(p.x,p.y,8,0,Math.PI*2); ctx.fill();
    ctx.fillStyle = '#294056';
    ctx.font = '13px Inter,Arial';
    ctx.fillText('P'+i, p.x + 12, p.y - 10);
  }
}

function drawArrow(from, vec, color, label){
    const to = { x: from.x + vec.x, y: from.y + vec.y };
    const headlen = 10; 
    const angle = Math.atan2(to.y - from.y, to.x - from.x);
    
    ctx.strokeStyle = color;
    ctx.lineWidth = 2;
    ctx.beginPath();
    ctx.moveTo(from.x, from.y);
    ctx.lineTo(to.x, to.y);
    ctx.stroke();

    ctx.fillStyle = color;
    ctx.beginPath();
    ctx.moveTo(to.x, to.y);
    ctx.lineTo(to.x - headlen * Math.cos(angle - Math.PI / 6), to.y - headlen * Math.sin(angle - Math.PI / 6));
    ctx.lineTo(to.x - headlen * Math.cos(angle + Math.PI / 6), to.y - headlen * Math.sin(angle + Math.PI / 6));
    ctx.fill();

    if(label){
        ctx.fillStyle = color;
        ctx.font = "bold 12px Arial";
        ctx.fillText(label, to.x + 10, to.y);
    }
}

function drawCasteljau(t){
  if(points.length < 2) return;
  const layers = [];
  layers.push(points.map(p=>({x:p.x,y:p.y})));
  while(layers[layers.length-1].length > 1){
    const prev = layers[layers.length-1];
    const next = [];
    for(let i=0;i<prev.length-1;i++) next.push(lerp(prev[i], prev[i+1], t));
    layers.push(next);
  }
  const colors = [COLORS.cast1, COLORS.cast2, COLORS.cast3];
  for(let level=0; level<layers.length; level++){
    const L = layers[level];
    ctx.strokeStyle = colors[level % colors.length];
    ctx.lineWidth = 1; 
    ctx.beginPath();
    for(let i=0;i<L.length;i++){
      if(i===0) ctx.moveTo(L[i].x,L[i].y);
      else ctx.lineTo(L[i].x,L[i].y);
    }
    ctx.stroke();
    for(const q of L){
      ctx.fillStyle = colors[level % colors.length];
      ctx.beginPath(); ctx.arc(q.x,q.y,4,0,Math.PI*2); ctx.fill();
    }
  }
  const final = layers[layers.length-1][0];
  ctx.fillStyle = '#1f2b46';
  ctx.beginPath(); ctx.arc(final.x, final.y, 6, 0, Math.PI*2); ctx.fill();
  
  return final;
}

function drawVectors(pos, t) {
    if(points.length === 4) {
        const velocity = getVelocity(t, points);
        const acceleration = getAcceleration(t, points);

        const vMag = Math.sqrt(velocity.x**2 + velocity.y**2).toFixed(0);
        const aMag = Math.sqrt(acceleration.x**2 + acceleration.y**2).toFixed(0);
        vMagLabel.textContent = vMag;
        aMagLabel.textContent = aMag;

        const scaleV = 0.3; 
        const scaleA = 0.15; 

        drawArrow(pos, {x: velocity.x * scaleV, y: velocity.y * scaleV}, COLORS.velocity, "v");
        drawArrow(pos, {x: acceleration.x * scaleA, y: acceleration.y * scaleA}, COLORS.accel, "a");
    }
}

function draw(){
  clear();
  drawGrid();
  drawControlPolygon();
  drawBezier();
  drawPoints();
  
  let t = parseFloat(tSlider.value);
  
  if(objectAnim) {
      t = objectT;
  }
  
  tVal.textContent = t.toFixed(2);
  
  const pos = drawCasteljau(t);

  if(pos) {
      drawVectors(pos, t);
  }
}

/* ---------- Interaction ---------- */
canvas.addEventListener('mousedown', (e)=>{
  const r = canvas.getBoundingClientRect();
  const mx = e.clientX - r.left, my = e.clientY - r.top;
  for(let p of points){
    if(Math.hypot(mx-p.x,my-p.y) < 12){ dragging = p; dragOffset.x = mx - p.x; dragOffset.y = my - p.y; break; }
  }
});
canvas.addEventListener('mousemove', (e)=>{
  if(!dragging) return;
  const r = canvas.getBoundingClientRect();
  dragging.x = e.clientX - r.left - dragOffset.x;
  dragging.y = e.clientY - r.top - dragOffset.y;
  dragging.x = Math.max(10, Math.min(canvas.width-10, dragging.x));
  dragging.y = Math.max(10, Math.min(canvas.height-10, dragging.y));
  draw();
});
canvas.addEventListener('mouseup', ()=>{ dragging = null; });
canvas.addEventListener('mouseleave', ()=>{ dragging = null; });

tSlider.addEventListener('input', ()=>{
  if(sliderAnim){ cancelAnimationFrame(sliderAnim); sliderAnim = null; playSliderBtn.textContent='Animar slider'; }
  draw();
});

playSliderBtn.addEventListener('click', ()=>{
  if(sliderAnim){ cancelAnimationFrame(sliderAnim); sliderAnim = null; playSliderBtn.textContent='Animar slider'; return; }
  let start = null;
  const duration = 2500; 
  playSliderBtn.textContent='Detener';
  function step(ts){
    if(!start) start = ts;
    const elapsed = ts - start;
    let tt = Math.min(1, elapsed/duration);
    tSlider.value = tt.toFixed(2);
    draw();
    if(tt < 1) sliderAnim = requestAnimationFrame(step);
    else { sliderAnim = null; playSliderBtn.textContent='Animar slider'; }
  }
  sliderAnim = requestAnimationFrame(step);
});

playObjectBtn.addEventListener('click', ()=>{
  if(objectAnim){ cancelAnimationFrame(objectAnim); objectAnim = null; playObjectBtn.textContent='Animar objeto'; draw(); return; }
  playObjectBtn.textContent='Detener';
  const duration = 3000;
  let start=null;
  function step(ts){
    if(!start) start=ts;
    const elapsed = ts - start;
    objectT = Math.min(1, elapsed/duration);
    tSlider.value = objectT.toFixed(2);
    draw();
    if(objectT < 1) objectAnim = requestAnimationFrame(step);
    else { objectAnim = null; playObjectBtn.textContent='Animar objeto'; }
  }
  objectAnim = requestAnimationFrame(step);
});

addPointBtn.addEventListener('click', ()=>{
  points.push({ x: canvas.width/2 + (Math.random()*80-40), y: canvas.height/2 + (Math.random()*80-40) });
  draw();
});
removePointBtn.addEventListener('click', ()=>{
  if(points.length > 2) points.pop();
  draw();
});
resetBtn.addEventListener('click', ()=>{
  points = [
    {x: 150, y: 550},
    {x: 350, y: 150},
    {x: 650, y: 150},
    {x: 850, y: 550}
  ];
  draw();
});
exportSVGBtn.addEventListener('click', ()=>{
  const samples = sampleCurve(300);
  let path = 'M ' + samples.map(p => `${p.x.toFixed(2)} ${p.y.toFixed(2)}`).join(' L ');
  const svg = `<svg xmlns="http://www.w3.org/2000/svg" width="${canvas.width}" height="${canvas.height}"><path d="${path}" fill="none" stroke="${COLORS.curve}" stroke-width="3" stroke-linecap="round" stroke-linejoin="round"/></svg>`;
  const blob = new Blob([svg], {type:'image/svg+xml'});
  const url = URL.createObjectURL(blob);
  const a = document.createElement('a'); a.href = url; a.download = 'bezier_curve.svg'; a.click();
  URL.revokeObjectURL(url);
});

/* ---------- Cargar Ejemplos Teóricos ---------- */
loadParabolaBtn.addEventListener('click', () => {
  const scale = 300;
  const offsetX = 250; 
  const offsetY = 550; 

  // Parábola teórica: c0=(0,0), c1=(1,0), c2=(1,1)
  points = [
    { x: offsetX,           y: offsetY },           // P0
    { x: offsetX + scale,   y: offsetY },           // P1
    { x: offsetX + scale,   y: offsetY - scale }    // P2
  ];

  tSlider.value = 0.5; 
  if(sliderAnim) cancelAnimationFrame(sliderAnim);
  draw();
  alert("Ejemplo cargado: Parábola (Grado 2). Nota: Los vectores de velocidad/aceleración se calculan para curvas cúbicas (4 puntos), añade un punto más para verlos.");
});

/* ---------- Cargar Experimento 1: Velocidad de Arranque ---------- */
loadExp1Btn.addEventListener('click', () => {
  points = [
    { x: 100, y: 550 },  // P0
    { x: 600, y: 550 },  // P1 (Lejos de P0 -> Gran Velocidad Inicial)
    { x: 700, y: 150 },  // P2
    { x: 900, y: 550 }   // P3
  ];
  tSlider.value = 0; 
  if(sliderAnim) cancelAnimationFrame(sliderAnim);
  draw();
  alert("Experimento 1 Cargado.\n\nObserve la flecha VERDE gigante en el arranque. Esto valida que la velocidad inicial es proporcional a la distancia entre P0 y P1.");
});

/* ---------- Cargar Experimento 2: Aceleración Centrípeta ---------- */
loadExp2Btn.addEventListener('click', () => {
  // Configuración de "Horquilla" (Hairpin) para máxima curvatura
  // Replica la forma de image_99716a.png
  points = [
    { x: 100, y: 250 },  // P0 (Izquierda Arriba)
    { x: 800, y: 100 },  // P1 (Derecha Arriba)
    { x: 800, y: 600 },  // P2 (Derecha Abajo)
    { x: 100, y: 600 }   // P3 (Izquierda Abajo)
  ];
  tSlider.value = 0.5; // Punto de máxima curvatura
  if(sliderAnim) cancelAnimationFrame(sliderAnim);
  draw();
  alert("Experimento 2 Cargado: Curva en Horquilla.\n\nObserve la flecha ROJA (Aceleración) en la punta del giro. Es enorme y apunta hacia la izquierda (adentro de la curva), demostrando visualmente la Fuerza Centrípeta.");
});

/* ---------- Cargar Experimento 3: Recta Paramétrica ---------- */
loadExp3Btn.addEventListener('click', () => {
  // Puntos colineales pero con distribución desigual
  const yFixed = 350;
  points = [
    { x: 100, y: yFixed }, // P0
    { x: 120, y: yFixed }, // P1 (Muy cerca de P0)
    { x: 500, y: yFixed }, // P2
    { x: 900, y: yFixed }  // P3
  ];
  tSlider.value = 0.5; 
  if(sliderAnim) cancelAnimationFrame(sliderAnim);
  draw();
  alert("Experimento 3 Cargado: Trayectoria Recta.\n\nAunque la línea es recta, observe que aparece una flecha ROJA (Aceleración) tangente a la línea. Esto demuestra que el objeto acelera/frena, probando la independencia entre geometría y tiempo.");
});

window.onload = function() {
    const params = new URLSearchParams(window.location.search);
    const exp = params.get('exp');

    if (exp === '1') {
        loadExp1Btn.click(); 
    } else if (exp === '2') {
        loadExp2Btn.click(); 
    } else if (exp === '3') {
        loadExp3Btn.click(); 
    }
};
/* ---------- init ---------- */
draw();
</script>
</body>
</html>

