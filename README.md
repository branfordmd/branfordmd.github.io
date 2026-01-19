<!doctype html>
<html lang="en">
<head>
  <meta charset="utf-8" />
  <meta name="viewport" content="width=device-width,initial-scale=1" />
  <title>Projectile Motion Simulator</title>
  <style>
    :root { --pad: 14px; }
    body {
      margin: 0;
      font-family: system-ui, -apple-system, Segoe UI, Roboto, Arial, sans-serif;
      background: #0b0c10;
      color: #eaf0ff;
    }
    .wrap {
      max-width: 1100px;
      margin: 0 auto;
      padding: var(--pad);
      display: grid;
      gap: var(--pad);
      grid-template-columns: 360px 1fr;
    }
    @media (max-width: 920px) {
      .wrap { grid-template-columns: 1fr; }
    }
    .card {
      background: #131623;
      border: 1px solid #222a44;
      border-radius: 16px;
      padding: var(--pad);
      box-shadow: 0 8px 24px rgba(0,0,0,.25);
    }
    h1 { font-size: 18px; margin: 0 0 10px; }
    h2 { font-size: 14px; margin: 16px 0 10px; opacity: .9; }
    .row { display: grid; grid-template-columns: 1fr 92px; gap: 10px; align-items: center; margin: 10px 0; }
    label { font-size: 13px; opacity: .9; }
    input[type="number"]{
      width: 100%;
      padding: 8px 10px;
      border-radius: 10px;
      border: 1px solid #2a3355;
      background: #0f1220;
      color: #eaf0ff;
      outline: none;
    }
    input[type="range"]{ width: 100%; }
    .btns { display: flex; gap: 10px; margin-top: 12px; flex-wrap: wrap; }
    button{
      border: 1px solid #2a3355;
      background: #1a2040;
      color: #eaf0ff;
      padding: 10px 12px;
      border-radius: 12px;
      cursor: pointer;
      font-weight: 600;
    }
    button:hover{ filter: brightness(1.08); }
    button:active{ transform: translateY(1px); }
    .kpi {
      display: grid;
      grid-template-columns: 1fr 1fr;
      gap: 10px;
      margin-top: 10px;
    }
    .kpi .box{
      background: #0f1220;
      border: 1px solid #2a3355;
      border-radius: 12px;
      padding: 10px;
    }
    .kpi .big{ font-size: 18px; font-weight: 750; margin-top: 6px; }
    .muted{ opacity: .75; font-size: 12px; }
    .canvasWrap { position: relative; }
    canvas{
      width: 100%;
      height: 520px;
      display: block;
      background: linear-gradient(#0c1022, #090a12);
      border-radius: 16px;
      border: 1px solid #222a44;
    }
    .legend {
      display:flex;
      gap:14px;
      align-items:center;
      flex-wrap:wrap;
      margin-top: 10px;
      font-size: 12px;
      opacity:.85;
    }
    .dot { width:10px; height:10px; border-radius:999px; display:inline-block; margin-right:6px; vertical-align:middle; }
    .footerNote{ margin-top: 10px; font-size: 12px; opacity: .7; line-height: 1.35; }
    .topbar { display:flex; justify-content: space-between; align-items:center; gap: 10px; flex-wrap: wrap; }
    .pill {
      font-size: 12px;
      padding: 6px 10px;
      border-radius: 999px;
      background:#0f1220;
      border:1px solid #2a3355;
      opacity:.9;
    }
    .smallGrid {
      display:grid;
      grid-template-columns: 1fr 1fr;
      gap: 10px;
      margin-top: 10px;
    }
  </style>
</head>
<body>
  <div class="wrap">
    <div class="card">
      <div class="topbar">
        <h1>Projectile Motion Simulator</h1>
        <div class="pill">No air resistance • g = 9.81 m/s²</div>
      </div>

      <h2>Inputs</h2>

      <div class="row">
        <label for="angle">Angle (°)</label>
        <input id="angleN" type="number" min="0" max="90" step="1" value="45">
      </div>
      <input id="angle" type="range" min="0" max="90" step="1" value="45" />

      <div class="row">
        <label for="v0">Initial velocity (m/s)</label>
        <input id="v0N" type="number" min="0" max="200" step="0.1" value="20">
      </div>
      <input id="v0" type="range" min="0" max="200" step="0.1" value="20" />

      <div class="row">
        <label for="h0">Initial height (m)</label>
        <input id="h0N" type="number" min="0" max="200" step="0.1" value="1.5">
      </div>
      <input id="h0" type="range" min="0" max="200" step="0.1" value="1.5" />

      <div class="smallGrid">
        <div>
          <label class="muted" for="gN">Gravity (m/s²)</label>
          <input id="gN" type="number" min="0.1" max="30" step="0.01" value="9.81">
        </div>
        <div>
          <label class="muted" for="dtN">Time step (s)</label>
          <input id="dtN" type="number" min="0.001" max="0.2" step="0.001" value="0.01">
        </div>
      </div>

      <div class="btns">
        <button id="runBtn">Run</button>
        <button id="pauseBtn">Pause</button>
        <button id="resetBtn">Reset</button>
      </div>

      <h2>Results</h2>
      <div class="kpi">
        <div class="box">
          <div class="muted">Time of flight</div>
          <div class="big"><span id="tof">—</span> s</div>
        </div>
        <div class="box">
          <div class="muted">Horizontal distance</div>
          <div class="big"><span id="range">—</span> m</div>
        </div>
        <div class="box">
          <div class="muted">Max height</div>
          <div class="big"><span id="hmax">—</span> m</div>
        </div>
        <div class="box">
          <div class="muted">Impact speed</div>
          <div class="big"><span id="vimpact">—</span> m/s</div>
        </div>
      </div>

      <div class="footerNote">
        Equations: <span class="muted">x(t)=v₀cosθ·t, y(t)=h₀+v₀sinθ·t−½gt²</span><br/>
        Time of flight is the positive root when <span class="muted">y(t)=0</span>.
      </div>
    </div>

    <div class="card">
      <div class="canvasWrap">
        <canvas id="c"></canvas>
      </div>
      <div class="legend">
        <span><span class="dot" style="background:#7aa2ff"></span>Trajectory</span>
        <span><span class="dot" style="background:#ffd36a"></span>Projectile</span>
        <span><span class="dot" style="background:#a8ffb0"></span>Ground</span>
      </div>
    </div>
  </div>

<script>
(() => {
  const $ = (id) => document.getElementById(id);

  // Inputs
  const angleR = $("angle"), angleN = $("angleN");
  const v0R = $("v0"), v0N = $("v0N");
  const h0R = $("h0"), h0N = $("h0N");
  const gN = $("gN");
  const dtN = $("dtN");

  // Outputs
  const tofEl = $("tof");
  const rangeEl = $("range");
  const hmaxEl = $("hmax");
  const vimpactEl = $("vimpact");

  // Buttons
  const runBtn = $("runBtn");
  const pauseBtn = $("pauseBtn");
  const resetBtn = $("resetBtn");

  // Canvas
  const canvas = $("c");
  const ctx = canvas.getContext("2d");

  function resizeCanvas() {
    const cssW = canvas.clientWidth;
    const cssH = canvas.clientHeight;
    const dpr = Math.max(1, window.devicePixelRatio || 1);
    canvas.width = Math.floor(cssW * dpr);
    canvas.height = Math.floor(cssH * dpr);
    ctx.setTransform(dpr, 0, 0, dpr, 0, 0);
  }
  window.addEventListener("resize", () => { resizeCanvas(); drawStatic(); if(trajectory.length) drawAll(); });
  resizeCanvas();

  // Sync sliders <-> number inputs
  function bindPair(rangeEl, numEl, onChange) {
    const syncFromRange = () => { numEl.value = rangeEl.value; onChange(); };
    const syncFromNum = () => {
      let v = Number(numEl.value);
      if (!Number.isFinite(v)) v = Number(rangeEl.min);
      v = Math.min(Number(rangeEl.max), Math.max(Number(rangeEl.min), v));
      numEl.value = v;
      rangeEl.value = v;
      onChange();
    };
    rangeEl.addEventListener("input", syncFromRange);
    numEl.addEventListener("input", syncFromNum);
  }

  let trajectory = [];      // {x,y}
  let animId = null;
  let running = false;

  // World scale / view
  let world = { xmin: 0, xmax: 50, ymin: 0, ymax: 20 };

  function deg2rad(deg) { return deg * Math.PI / 180; }

  function solveTimeOfFlight(v0, theta, h0, g) {
    // y(t)=h0 + v0*sinθ t - 0.5 g t^2 = 0  =>  0.5 g t^2 - v0*sinθ t - h0 = 0
    const a = 0.5 * g;
    const b = -v0 * Math.sin(theta);
    const c = -h0;
    const disc = b*b - 4*a*c;
    if (disc < 0) return 0;
    const t1 = (-b + Math.sqrt(disc)) / (2*a);
    const t2 = (-b - Math.sqrt(disc)) / (2*a);
    return Math.max(t1, t2, 0);
  }

  function computeAll() {
    const angle = Number(angleN.value);
    const v0 = Number(v0N.value);
    const h0 = Number(h0N.value);
    const g = Number(gN.value);

    const theta = deg2rad(angle);

    const tof = solveTimeOfFlight(v0, theta, h0, g);
    const vx = v0 * Math.cos(theta);
    const vy = v0 * Math.sin(theta);

    const range = vx * tof;

    // Max height: occurs when vy - g t = 0 => t = vy/g (if vy>0)
    const tTop = (g > 0) ? Math.max(0, vy / g) : 0;
    const yTop = h0 + vy * tTop - 0.5 * g * tTop * tTop;

    // Impact speed
    const vyImpact = vy - g * tof;
    const vImpact = Math.sqrt(vx*vx + vyImpact*vyImpact);

    tofEl.textContent = tof.toFixed(3);
    rangeEl.textContent = range.toFixed(3);
    hmaxEl.textContent = Math.max(h0, yTop).toFixed(3);
    vimpactEl.textContent = vImpact.toFixed(3);

    // Set view bounds with padding
    const ymax = Math.max(h0, yTop) * 1.15 + 1;
    const xmax = Math.max(5, range * 1.10 + 1);
    world = { xmin: 0, xmax, ymin: 0, ymax: Math.max(5, ymax) };

    return { angle, v0, h0, g, theta, tof, vx, vy, range, yTop, vImpact };
  }

  function worldToCanvas(x, y) {
    const w = canvas.clientWidth;
    const h = canvas.clientHeight;

    // leave margins
    const m = 36;
    const plotW = w - 2*m;
    const plotH = h - 2*m;

    const sx = plotW / (world.xmax - world.xmin);
    const sy = plotH / (world.ymax - world.ymin);

    const cx = m + (x - world.xmin) * sx;
    const cy = h - m - (y - world.ymin) * sy;
    return { cx, cy };
  }

  function clear() {
    ctx.clearRect(0, 0, canvas.clientWidth, canvas.clientHeight);
  }

  function drawAxes() {
    const w = canvas.clientWidth;
    const h = canvas.clientHeight;
    const m = 36;

    // border
    ctx.strokeStyle = "rgba(255,255,255,0.12)";
    ctx.lineWidth = 1;
    ctx.strokeRect(m, m, w - 2*m, h - 2*m);

    // grid + labels (simple)
    ctx.fillStyle = "rgba(255,255,255,0.75)";
    ctx.font = "12px system-ui";

    const xTicks = 5;
    const yTicks = 5;

    ctx.strokeStyle = "rgba(255,255,255,0.08)";
    for (let i = 0; i <= xTicks; i++) {
      const x = world.xmin + (i / xTicks) * (world.xmax - world.xmin);
      const p = worldToCanvas(x, 0);
      ctx.beginPath();
      ctx.moveTo(p.cx, m);
      ctx.lineTo(p.cx, h - m);
      ctx.stroke();
      ctx.fillText(x.toFixed(0) + " m", p.cx - 12, h - 12);
    }

    for (let j = 0; j <= yTicks; j++) {
      const y = world.ymin + (j / yTicks) * (world.ymax - world.ymin);
      const p = worldToCanvas(0, y);
      ctx.beginPath();
      ctx.moveTo(m, p.cy);
      ctx.lineTo(w - m, p.cy);
      ctx.stroke();
      ctx.fillText(y.toFixed(0) + " m", 8, p.cy + 4);
    }

    // axis labels
    ctx.fillStyle = "rgba(255,255,255,0.8)";
    ctx.fillText("Horizontal distance (m)", m + 8, m - 10);
    ctx.save();
    ctx.translate(14, h/2);
    ctx.rotate(-Math.PI/2);
    ctx.fillText("Height (m)", 0, 0);
    ctx.restore();
  }

  function drawGround() {
    const p0 = worldToCanvas(world.xmin, 0);
    const p1 = worldToCanvas(world.xmax, 0);
    ctx.strokeStyle = "rgba(168,255,176,0.85)";
    ctx.lineWidth = 2;
    ctx.beginPath();
    ctx.moveTo(p0.cx, p0.cy);
    ctx.lineTo(p1.cx, p1.cy);
    ctx.stroke();
  }

  function drawTrajectory() {
    if (!trajectory.length) return;
    ctx.strokeStyle = "rgba(122,162,255,0.95)";
    ctx.lineWidth = 2;

    ctx.beginPath();
    for (let i = 0; i < trajectory.length; i++) {
      const pt = trajectory[i];
      const p = worldToCanvas(pt.x, pt.y);
      if (i === 0) ctx.moveTo(p.cx, p.cy);
      else ctx.lineTo(p.cx, p.cy);
    }
    ctx.stroke();
  }

  function drawProjectile(pos) {
    const p = worldToCanvas(pos.x, pos.y);
    ctx.fillStyle = "rgba(255,211,106,0.95)";
    ctx.beginPath();
    ctx.arc(p.cx, p.cy, 6, 0, Math.PI*2);
    ctx.fill();
  }

  function drawStatic() {
    clear();
    drawAxes();
    drawGround();
  }

  function drawAll(currentPos=null) {
    drawStatic();
    drawTrajectory();
    if (currentPos) drawProjectile(currentPos);
  }

  // Simulation state
  let sim = { t: 0, tof: 0, vx: 0, vy: 0, h0: 0, g: 9.81 };
  function resetSim() {
    running = false;
    if (animId) cancelAnimationFrame(animId);
    animId = null;
    trajectory = [];

    const info = computeAll();
    sim = { t: 0, tof: info.tof, vx: info.vx, vy: info.vy, h0: info.h0, g: info.g };

    // start point
    trajectory.push({ x: 0, y: info.h0 });
    drawAll({ x: 0, y: info.h0 });
  }

  function step() {
    if (!running) return;

    const dt = Math.max(0.001, Math.min(0.2, Number(dtN.value) || 0.01));
    sim.t += dt;

    // clamp at impact
    if (sim.t > sim.tof) sim.t = sim.tof;

    const x = sim.vx * sim.t;
    const y = sim.h0 + sim.vy * sim.t - 0.5 * sim.g * sim.t * sim.t;

    trajectory.push({ x, y: Math.max(0, y) });

    drawAll({ x, y: Math.max(0, y) });

    if (sim.t >= sim.tof) {
      running = false;
      animId = null;
      return;
    }
    animId = requestAnimationFrame(step);
  }

  function run() {
    if (running) return;
    running = true;
    if (!trajectory.length) resetSim();
    if (!animId) animId = requestAnimationFrame(step);
  }

  function pause() {
    running = false;
    if (animId) cancelAnimationFrame(animId);
    animId = null;
  }

  // When inputs change, recompute + redraw (and stop animation)
  function onInputChange() {
    pause();
    resetSim();
  }

  bindPair(angleR, angleN, onInputChange);
  bindPair(v0R, v0N, onInputChange);
  bindPair(h0R, h0N, onInputChange);

  gN.addEventListener("input", onInputChange);
  dtN.addEventListener("input", () => { /* dt can change mid-run */ });

  runBtn.addEventListener("click", run);
  pauseBtn.addEventListener("click", pause);
  resetBtn.addEventListener("click", resetSim);

  // init
  resetSim();
})();
</script>
</body>
</html>
