<!doctype html>
<html lang="tr">
<head>
<meta charset="utf-8"/>
<meta name="viewport" content="width=device-width,initial-scale=1"/>
<title>Frikik Prototip — Otomatik Baraj & Mobil</title>
<style>
:root{ --bg:#04121a; --accent:#ffd166; --panel:rgba(0,0,0,0.44); }
*{ box-sizing:border-box; }
body{ margin:0; font-family:Inter,Segoe UI, Arial, sans-serif; background:linear-gradient(#031018,#042431); color:#fff; display:flex; align-items:center; justify-content:center; min-height:100vh; padding:14px;}
#app{ width:1000px; max-width:98vw; display:flex; gap:14px; align-items:flex-start; }
#menu{ width:320px; background:linear-gradient(180deg,#072b2a,#032018); padding:16px; border-radius:12px; box-shadow:0 8px 28px rgba(0,0,0,0.6); }
h1{ margin:0 0 8px 0; font-size:20px; }
.stadium{ height:140px; border-radius:8px; background-image: radial-gradient(ellipse at 50% 20%, rgba(255,255,255,0.04), transparent 20%), linear-gradient(180deg,#0a4d2d,#06321b); margin-bottom:12px; position:relative; overflow:hidden; }
.btn{ display:block; width:100%; padding:12px; border-radius:10px; background:linear-gradient(180deg,#ffd166,#ff9a00); border:none; color:#222; font-weight:800; cursor:pointer; margin-top:8px; }
.ghost{ background:transparent; border:1px solid rgba(255,255,255,0.06); padding:8px 10px; border-radius:8px; cursor:pointer; color:#fff; }
#gameWrap{ flex:1; display:flex; flex-direction:column; gap:10px; align-items:center; }
canvas{ width:100%; max-width:840px; height:auto; border-radius:10px; background:linear-gradient(#1b6b2f,#0b4e23); box-shadow:0 8px 24px rgba(0,0,0,0.6); display:block; }
.controls{ display:flex; gap:12px; width:100%; max-width:840px; justify-content:space-between; align-items:flex-start; }
.panel{ width:320px; background:var(--panel); padding:12px; border-radius:10px; }
.small{ font-size:13px; color:rgba(255,255,255,0.92); }
.players{ display:flex; gap:8px; margin-top:8px; }
.playerBtn{ flex:1; padding:8px; border-radius:8px; background:rgba(255,255,255,0.06); text-align:center; cursor:pointer; }
.active{ outline:3px solid rgba(255,255,255,0.08); background:rgba(255,255,255,0.12); }
.mobileControls{ display:flex; gap:8px; flex-direction:column; align-items:stretch; }
.bigBtn{ padding:14px; border-radius:12px; font-weight:800; font-size:16px; border:none; cursor:pointer; }
.bigGhost{ background:rgba(255,255,255,0.06); color:#fff; border-radius:12px; padding:12px; font-weight:700; }
.row{ display:flex; gap:8px; }
.iconBtn{ flex:1; background:linear-gradient(180deg,#66d17a,#2bb34f); color:#062; border-radius:10px; padding:12px; text-align:center; font-weight:800; cursor:pointer; }
#msg{ text-align:center; font-weight:700; color:#ffe; margin-top:8px; min-height:20px; }

/* Responsive: mobilde kontroller altta */
@media (max-width:920px){
  #app{ flex-direction:column; align-items:stretch; }
  #menu{ width:100%; order:2; }
  #gameWrap{ order:1; }
  .controls{ flex-direction:column; align-items:stretch; }
  .panel{ width:100%; }
  canvas{ max-width:100%; height:60vh; }
}
</style>
</head>
<body>
  <div id="app">
    <!-- MENU -->
    <div id="menu">
      <h1>Frikik Maestro</h1>
      <div class="stadium"><div style="position:absolute;inset:0;background:radial-gradient(circle at 50% 10%, rgba(255,255,255,0.03), transparent 20%);"></div></div>
      <div class="small">Menü: Futbol temalı. Baraj otomatik zıplar. Mobil uyumlu.</div>

      <div style="margin-top:12px;">
        <div class="small">Oyuncu Seçimi</div>
        <div class="players" id="menuPlayers">
          <div class="playerBtn active" data-player="ronaldo">Ronaldo</div>
          <div class="playerBtn" data-player="talisca">Talisca</div>
          <div class="playerBtn" data-player="rice">Declan Rice</div>
        </div>
      </div>

      <div style="margin-top:12px;">
        <div class="small">Zorluk: Kolay (Kaleci A)</div>
        <div class="small" style="margin-top:6px;">Baraj: Dinamik (uzak=2, yakın=5) ve otomatik zıplar.</div>
      </div>

      <button id="startBtn" class="btn">Oyunu Başlat</button>
    </div>

    <!-- GAME -->
    <div id="gameWrap">
      <canvas id="canvas" width="840" height="520"></canvas>

      <div class="controls">
        <div class="panel">
          <div class="small"><strong>Seçili: </strong><span id="selName">Ronaldo</span></div>
          <div style="margin-top:8px;">
            <label class="small">Falso</label>
            <input id="spin" type="range" min="-100" max="100" value="0" style="width:100%"/>
            <div class="small">Negatif = sola, Pozitif = sağa</div>
          </div>

          <div style="margin-top:8px;">
            <label class="small">Yükseklik (Z)</label>
            <input id="height" type="range" min="10" max="120" value="60" style="width:100%"/>
            <div class="small">Yükseklik baraj ve kaleci karşısında önemli</div>
          </div>

          <div style="margin-top:10px;">
            <label class="small">Güç</label>
            <div style="height:12px;background:#111;border-radius:8px; overflow:hidden;margin-top:6px;">
              <div id="pbar" style="height:100%; width:30%; background:linear-gradient(#ffd166,#ff8a00)"></div>
            </div>
            <div style="display:flex; gap:8px; margin-top:8px;">
              <button id="lock" class="ghost">Güç Kilitle</button>
              <button id="shoot" class="btn">Şut Çek</button>
            </div>
          </div>

          <div style="margin-top:10px;">
            <div class="row">
              <button id="reset" class="bigGhost">Yeni Nokta</button>
              <button id="toggleHelp" class="bigGhost">Yardım</button>
            </div>
          </div>
          <div id="msg"></div>
        </div>

        <!-- Mobil controls / info -->
        <div class="panel">
          <div class="small"><strong>Mobil Kontroller</strong></div>
          <div style="margin-top:8px;" class="mobileControls">
            <div class="row">
              <button id="upBtn" class="bigBtn bigGhost">↑ Yükseklik+</button>
              <button id="downBtn" class="bigBtn bigGhost">↓ Yükseklik−</button>
            </div>
            <div class="row" style="margin-top:8px;">
              <button id="leftBtn" class="bigBtn bigGhost">← Falso−</button>
              <button id="rightBtn" class="bigBtn bigGhost">Falso+</button>
            </div>
            <div class="row" style="margin-top:8px;">
              <button id="shootBtn" class="iconBtn">⚽ Şut</button>
            </div>
            <div style="margin-top:8px;" class="small">Klavye: 1/2/3 oyuncu seç, Space=baraj zıplat (otomatik de olur), Enter=şut</div>
          </div>
        </div>
      </div>
    </div>
  </div>

<script>
/* Oyunun ana JS'i
   - Dinamik baraj sayısı (uzak=>2, yakın=>5)
   - Otomatik baraj zıplama, zamanlanmış
   - Mobil büyük butonlar
   - 3D top (z ekseni ve gölge)
*/

// Canvas
const canvas = document.getElementById('canvas');
const ctx = canvas.getContext('2d');
const W = canvas.width, H = canvas.height;

// UI
const startBtn = document.getElementById('startBtn');
const menuPlayers = document.getElementById('menuPlayers');
const spinInput = document.getElementById('spin');
const heightInput = document.getElementById('height');
const pbar = document.getElementById('pbar');
const lockBtn = document.getElementById('lock');
const shootBtn = document.getElementById('shoot');
const resetBtn = document.getElementById('reset');
const selName = document.getElementById('selName');
const msgEl = document.getElementById('msg');
const upBtn = document.getElementById('upBtn');
const downBtn = document.getElementById('downBtn');
const leftBtn = document.getElementById('leftBtn');
const rightBtn = document.getElementById('rightBtn');
const shootMobileBtn = document.getElementById('shootBtn');

let inMenu = true;
let selectedPlayer='ronaldo';
let freeKickPos = null;
let aimPos = { x: W/2, y: 60 };
let ball = null;
let power = 30;
let powerIncreasing = true;
let powerLocked = false;
let spin = 0;
let heightSetting = 60;
let messageTimer = 0;
let scheduledWallTimeout = null;

// players
const players = {
  ronaldo:{ name:'Ronaldo', power:95, curve:78, skin:'#ffe0cc', hair:'#222' },
  talisca:{ name:'Talisca', power:90, curve:88, skin:'#2b2b2b', hair:'#fff' },
  rice:{ name:'Declan Rice', power:82, curve:76, skin:'#ffe8d6', hair:'#2e2e2e' },
};

// keeper (kolay A)
const keeper = { x: W/2, y: 80, w:44, h:62, reaction:0.42, diveStartedAt:0, diving:false, targetX:W/2, diveDur:0.55 };

// wall config (distance from ball)
const wallCfg = { count:4, distance:130, widthEach:20, gap:6, baseYoffset:-8, isJumping:false, jumpStart:0, jumpDur:0.5 };
let wallPieces = [];

// menu interactions
menuPlayers.querySelectorAll('.playerBtn').forEach(btn=>{
  btn.addEventListener('click', ()=>{
    menuPlayers.querySelectorAll('.playerBtn').forEach(b=>b.classList.remove('active'));
    btn.classList.add('active');
    selectedPlayer = btn.dataset.player;
    selName.textContent = players[selectedPlayer].name;
  });
});

startBtn.addEventListener('click', ()=>{
  inMenu = false;
  document.getElementById('menu').style.display = 'none';
  showMsg('Saha üzerine tıkla frikik noktası seçmek için.');
});

// inputs
spinInput.addEventListener('input', ()=> spin = parseInt(spinInput.value));
heightInput.addEventListener('input', ()=> heightSetting = parseInt(heightInput.value));
lockBtn.addEventListener('click', ()=> { powerLocked = !powerLocked; lockBtn.textContent = powerLocked ? 'Güç Kilitlendi' : 'Güç Kilitle';});
shootBtn.addEventListener('click', performShoot);
resetBtn.addEventListener('click', ()=> { freeKickPos=null; ball=null; wallPieces=[]; showMsg('Yeni nokta bekleniyor...'); });

// mobile controls
upBtn.addEventListener('touchstart', ()=> adjustHeight(6)); upBtn.addEventListener('click', ()=> adjustHeight(6));
downBtn.addEventListener('touchstart', ()=> adjustHeight(-6)); downBtn.addEventListener('click', ()=> adjustHeight(-6));
leftBtn.addEventListener('touchstart', ()=> adjustSpin(-6)); leftBtn.addEventListener('click', ()=> adjustSpin(-6));
rightBtn.addEventListener('touchstart', ()=> adjustSpin(6)); rightBtn.addEventListener('click', ()=> adjustSpin(6));
shootMobileBtn.addEventListener('touchstart', ()=> performShoot()); shootMobileBtn.addEventListener('click', ()=> performShoot());

function adjustHeight(d){ heightSetting = clamp(heightSetting + d, 10, 120); heightInput.value = heightSetting; showMsg('Yükseklik: '+heightSetting); }
function adjustSpin(d){ spin = clamp(spin + d, -100, 100); spinInput.value = spin; showMsg('Falso: '+spin); }

// canvas click
canvas.addEventListener('click', (e)=>{
  if (inMenu) return;
  const r = canvas.getBoundingClientRect();
  const x = e.clientX - r.left, y = e.clientY - r.top;
  if (y < 140) { aimPos.x = clamp(x, 120, W-120); aimPos.y = 60; showMsg('Hedef ayarlandı'); }
  else { freeKickPos = { x,y }; computeWallBase(); showMsg('Frikik noktası ayarlandı'); }
});

// keyboard
window.addEventListener('keydown', (e)=>{
  if (e.key === ' ') { e.preventDefault(); triggerWallJumpImmediate(); }
  if (e.key === 'Enter') { e.preventDefault(); performShoot(); }
  if (e.key === '1') selectPlayerKey('ronaldo');
  if (e.key === '2') selectPlayerKey('talisca');
  if (e.key === '3') selectPlayerKey('rice');
  if (e.key === 'ArrowUp') { powerLocked = !powerLocked; lockBtn.textContent = powerLocked ? 'Güç Kilitlendi' : 'Güç Kilitle'; }
});

function selectPlayerKey(k){ selectedPlayer=k; selName.textContent = players[k].name; menuPlayers.querySelectorAll('.playerBtn').forEach(b=>b.classList.remove('active')); const el=document.querySelector(`[data-player="${k}"]`); if(el) el.classList.add('active'); }

// compute wall base - DİNAMİK sayi
function computeWallBase(){
  wallPieces = [];
  if (!freeKickPos) return;
  const goal = { x: W/2, y: 40 };
  const dx = goal.x - freeKickPos.x, dy = goal.y - freeKickPos.y;
  const dist = Math.hypot(dx,dy);
  // dynamic count based on distance (thresholds tuned)
  let count;
  if (dist > 420) count = 2;        // uzak
  else if (dist > 300) count = 3;   // orta-uzak
  else if (dist > 180) count = 4;   // orta
  else count = 5;                   // yakın
  wallCfg.count = count;
  // compute center point for wall
  const len = Math.max(1, Math.hypot(dx,dy));
  const nx = dx/len, ny = dy/len;
  const center = { x: freeKickPos.x + nx * wallCfg.distance, y: freeKickPos.y + ny * wallCfg.distance };
  const perp = { x: -ny, y: nx };
  const totalWidth = wallCfg.count * wallCfg.widthEach + (wallCfg.count-1)*wallCfg.gap;
  let startOffset = - totalWidth/2 + wallCfg.widthEach/2;
  for (let i=0;i<wallCfg.count;i++){
    const offset = startOffset + i*(wallCfg.widthEach + wallCfg.gap);
    const px = center.x + perp.x * offset;
    const py = center.y + perp.y * offset + wallCfg.baseYoffset;
    wallPieces.push({ x: px, y: py, baseY: py, h: 40, w: wallCfg.widthEach });
  }
}

// schedule automatic wall jump so it occurs when ball arrives
function scheduleWallJump(distanceToWall, ballSpeed){
  if (scheduledWallTimeout) { clearTimeout(scheduledWallTimeout); scheduledWallTimeout = null; }
  // time = distance / speed (speed in px/frame but our vx/vy are per frame-ish; better to estimate by speed magnitude)
  // ballSpeed here in px/frame; convert to px/sec approx: *60 (frames)
  const sec = Math.max(0.06, distanceToWall / (Math.max(0.1, ballSpeed) * 60));
  // trigger jump slightly before (150ms)
  const delay = Math.max(20, (sec*1000) - 160);
  scheduledWallTimeout = setTimeout(()=> { triggerWallJump(); scheduledWallTimeout = null; }, delay);
}

// trigger wall jump (animated)
function triggerWallJump(){
  if (!freeKickPos || wallPieces.length===0) return;
  wallCfg.isJumping = true;
  wallCfg.jumpStart = performance.now()/1000;
}

// allow immediate manual trigger (keyboard)
function triggerWallJumpImmediate(){ triggerWallJump(); showMsg('Baraj zıpladı (manuel)'); }

// perform shoot
function performShoot(){
  if (!freeKickPos) { showMsg('Önce sahada frikik noktası seç.'); return; }
  if (ball) return;
  const pl = players[selectedPlayer];
  const finalPower = (pl.power/100) * (power/100) * 140;
  const dir = { x: aimPos.x - freeKickPos.x, y: aimPos.y - freeKickPos.y };
  const distToGoal = Math.hypot(dir.x, dir.y);
  const norm = { x: dir.x / distToGoal, y: dir.y / distToGoal };
  const speed = Math.max(4, finalPower/6);
  const vx = norm.x * speed;
  const vy = norm.y * speed;
  const vz = (heightSetting/30);
  const spinStrength = (spin/100) * (pl.curve/100) * 2.6;

  ball = { x: freeKickPos.x, y: freeKickPos.y, z:6, vx, vy, vz, spin:spinStrength, perp:{x:-norm.y,y:norm.x}, time:0, owner:selectedPlayer, goalY:40 };
  // recompute wall base & schedule wall jump timing
  computeWallBase();
  if (wallPieces.length>0){
    // approximate distance from ball to wall center (use middle piece)
    const mid = wallPieces[Math.floor(wallPieces.length/2)];
    const dx = mid.x - ball.x, dy = mid.y - ball.y;
    const distToWall = Math.hypot(dx,dy);
    scheduleWallJump(distToWall, Math.hypot(vx,vy));
  }

  // keeper predict
  const timeEst = Math.hypot(aimPos.x - freeKickPos.x, aimPos.y-freeKickPos.y) / Math.max(0.1, Math.hypot(vx,vy));
  keeper.targetX = clamp(freeKickPos.x + (aimPos.x - freeKickPos.x) + ball.spin * timeEst * 30, 120, W-120);
  keeper.diveStartedAt = performance.now()/1000 + keeper.reaction + (Math.random()*0.08 - 0.04);
  keeper.diving = false;

  showMsg('Şut atıldı!');
}

// update loop
let last = performance.now();
function loop(now){
  const dt = (now - last)/1000; last = now;
  // power oscillation
  if (!powerLocked){
    const speed = 80;
    power += (powerIncreasing?1:-1) * speed * dt;
    if (power >= 100){ power=100; powerIncreasing=false; }
    if (power <= 6){ power=6; powerIncreasing=true; }
    pbar.style.width = power + '%';
  }
  // update ball
  updateBall(dt);
  // keeper
  updateKeeper(dt);
  // wall update
  updateWall(dt);
  // draw
  draw();
  if (messageTimer && performance.now() > messageTimer) { msgEl.textContent=''; messageTimer=0; }
  requestAnimationFrame(loop);
}
requestAnimationFrame(loop);

// update ball physics
function updateBall(dt){
  if (!ball) return;
  ball.x += ball.vx;
  ball.y += ball.vy;
  ball.z += ball.vz;
  ball.vz -= 180 * dt / 60; // gravity-ish
  ball.time += dt;
  const tfrac = clamp(ball.time / 1.8, 0, 1);
  const lateral = ball.spin * Math.sin(tfrac * Math.PI) * 18;
  ball.x += ball.perp.x * lateral * dt;
  ball.y += ball.perp.y * lateral * dt;

  // check wall collisions
  if (wallPieces.length>0){
    for (let wp of wallPieces){
      let wallTopZ = wp.h;
      if (wallCfg.isJumping){
        const jt = clamp((performance.now()/1000 - wallCfg.jumpStart)/wallCfg.jumpDur, 0, 1);
        const jumpHeight = clamp(Math.sin(jt*Math.PI)*60, 0, 60);
        wallTopZ += jumpHeight;
      }
      const dx = ball.x - wp.x, dy = ball.y - wp.y;
      if (Math.abs(dx) < wp.w/1.6 && Math.abs(dy) < 30){
        if (ball.z <= wallTopZ && ball.vz < 0){
          showMsg('Baraj blokladı!');
          ball = null;
          return;
        }
      }
    }
  }

  // goal check
  if (ball && ball.y <= ball.goalY + 10){
    const goalLeft = 120, goalRight = W-120;
    const keeperWillCatch = keeper.diving && Math.abs(ball.x - keeper.x) < 38 && ball.z < 90;
    if (keeperWillCatch){
      showMsg('Kaleci kurtardı!');
      ball = null; return;
    } else {
      if (ball.x >= goalLeft+10 && ball.x <= goalRight-10){
        showMsg('GOL!!!');
      } else {
        showMsg('Kaçtı!');
      }
      ball = null; return;
    }
  }

  // out safety
  if (ball && (ball.x < -80 || ball.x > W+80 || ball.y < -200)) ball = null;
}

// keeper update (kolay)
function updateKeeper(dt){
  const now = performance.now()/1000;
  if (ball && !keeper.diving && now >= keeper.diveStartedAt){
    keeper.diving = true; keeper.diveStartedAt = now; keeper.diveDur = 0.45 + Math.random()*0.4;
  }
  if (keeper.diving){
    const t = clamp((now - keeper.diveStartedAt)/keeper.diveDur, 0, 1);
    keeper.x += (keeper.targetX - keeper.x) * (0.04 + t*0.4);
    keeper.y = 60 - Math.sin(t*Math.PI)*28;
    if (t >= 1) { keeper.diving = false; keeper.y = 80; }
  } else {
    keeper.x += (W/2 - keeper.x) * 0.04;
    keeper.y += (80 - keeper.y) * 0.06;
  }
}

// wall update: reset jump after duration
function updateWall(dt){
  if (wallCfg.isJumping){
    const elapsed = performance.now()/1000 - wallCfg.jumpStart;
    if (elapsed > wallCfg.jumpDur + 0.18) wallCfg.isJumping = false;
  }
}

// draw
function draw(){
  ctx.clearRect(0,0,W,H);
  drawField();

  if (freeKickPos){
    ctx.fillStyle = '#fff';
    ctx.beginPath(); ctx.arc(freeKickPos.x, freeKickPos.y, 7,0,Math.PI*2); ctx.fill();
    drawShooter(freeKickPos.x-24, freeKickPos.y+18, players[selectedPlayer]);
  }

  // draw wall
  drawWall();

  // aim
  ctx.fillStyle = 'rgba(255,255,0,0.95)';
  ctx.beginPath(); ctx.arc(aimPos.x, aimPos.y, 6,0,Math.PI*2); ctx.fill();

  // keeper
  drawKeeper();

  // ball 3D
  if (ball){
    const shadowScale = clamp((100 - ball.z)/140, 0.28, 1.0);
    ctx.fillStyle = 'rgba(0,0,0,0.28)';
    ctx.beginPath(); ctx.ellipse(ball.x, ball.y+Math.max(6, ball.z/4), 12*shadowScale, 6*shadowScale, 0,0,Math.PI*2); ctx.fill();
    ctx.fillStyle = '#fff'; ctx.beginPath(); ctx.arc(ball.x, ball.y - ball.z/10, 8,0,Math.PI*2); ctx.fill();
  }

  pbar.style.width = power + '%';
}

function drawField(){
  const g = ctx.createLinearGradient(0,0,0,H); g.addColorStop(0,'#1c7a3f'); g.addColorStop(1,'#0f5b2b');
  ctx.fillStyle = g; ctx.fillRect(0,0,W,H);
  ctx.strokeStyle = 'rgba(255,255,255,0.9)'; ctx.lineWidth = 2;
  ctx.beginPath(); ctx.moveTo(0,H/2); ctx.lineTo(W,H/2); ctx.stroke();
  ctx.beginPath(); ctx.arc(W/2, H/2, 48, 0, Math.PI*2); ctx.stroke();
  ctx.strokeRect(120, 0, W-240, 140); ctx.strokeRect(W/2 - 70, 0, 140, 60);
  ctx.fillStyle = '#fff'; ctx.fillRect(120, 30, W-240, 6);
}

function drawKeeper(){
  ctx.save(); ctx.translate(keeper.x, ke
