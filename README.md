<!doctype html>
<html lang="tr">
<head>
<meta charset="utf-8"/>
<meta name="viewport" content="width=device-width,initial-scale=1"/>
<title>Top + Yerçekimi + Gökkuşağı İzleri</title>
<style>
  :root{
    --bg1:#061028;
    --bg2:#071233;
    --panel: rgba(255,255,255,0.04);
    --accent:#60a5fa;
  }
  html,body{height:100%;margin:0;font-family:Inter,system-ui,Arial;background:
    radial-gradient(1200px 600px at 10% 10%, rgba(96,165,250,0.06), transparent),
    linear-gradient(180deg,var(--bg1), var(--bg2)); color:#e6eef8}
  .wrap{min-height:100vh;display:flex;flex-direction:column;gap:12px;align-items:center;padding:18px;box-sizing:border-box}
  .controls{display:flex;gap:10px;align-items:center}
  .btn{padding:10px 14px;border-radius:10px;border:1px solid rgba(255,255,255,0.08);background:transparent;color:inherit;cursor:pointer;font-weight:600}
  .btn.primary{background:linear-gradient(90deg,var(--accent), #34d399);color:#022; border:0}
  .hint{opacity:0.85;font-size:13px}

  /* canvas alanı */
  .stage{
    width:100%;
    max-width:1100px;
    height:70vh;
    min-height:420px;
    border-radius:14px;
    overflow:hidden;
    border:1px solid rgba(255,255,255,0.04);
    box-shadow:0 12px 40px rgba(2,6,12,0.6);
    position:relative;
    background:
      radial-gradient(circle at 85% 15%, rgba(255,255,255,0.02), transparent 2%),
      linear-gradient(180deg, rgba(255,255,255,0.01), transparent 30%);
  }

  /* info overlay */
  .overlayInfo{
    position:absolute;left:12px;top:12px;padding:8px 10px;border-radius:8px;background:rgba(2,6,12,0.45);backdrop-filter:blur(4px);font-size:13px;color:rgba(255,255,255,0.9)
  }

  @media (max-width:600px){ .stage{height:64vh; min-height:520px} }
</style>
</head>
<body>
  <div class="wrap">
    <div class="controls">
      <button id="startBtn" class="btn primary">Başlat</button>
      <button id="resetBtn" class="btn">Sıfırla</button>
      <div class="hint">Başlat → top üstten bırakılır. Top yuvarlağa çarpınca büyür ve gökkuşağı izleri bırakır.</div>
    </div>

    <div id="stageWrap" class="stage" role="region" aria-label="Etkileşim alanı">
      <canvas id="c"></canvas>
      <div class="overlayInfo">Dokun/tıkla: yeni top bırak (mobilde dokunma).</div>
    </div>
  </div>

<script>
/*
  Topun fiziksel hareketi, yuvarlaklarla çarpışma, 
  çarpışmada hedefin büyümesi ve gökkuşağı parçacıkları üretimi.
*/
(() => {
  const canvas = document.getElementById('c');
  const wrap = document.getElementById('stageWrap');
  const ctx = canvas.getContext('2d', { alpha: true });

  // Kontroller
  const startBtn = document.getElementById('startBtn');
  const resetBtn = document.getElementById('resetBtn');

  // Scene ayarları
  let W=0, H=0, last = 0, rafId=null;
  const gravityBase = 1400; // px/s^2 (masaüstü referansı), responsive olarak ölçeklenecek
  const restitution = 0.66; // zıplama enerjisi korunumu
  const groundFriction = 0.998;

  // top (başlangıçta yukarıda bekleyen obje)
  let ball = null;
  // hedef yuvarlaklar (sabit)
  let targets = [];
  // parçacıklar (gökkuşağı)
  let particles = [];
  // sürekli top izi (fading rainbow dots)
  let trails = [];

  // renk paleti gökkuşağı
  const RAINBOW = [
    '#ff3b30','#ff9500','#ffcc00','#34c759','#5ac8fa','#007aff','#5856d6'
  ];

  // Başlangıç / reset fonksiyonu
  function initScene(){
    cancelAnimationFrame(rafId);
    resizeCanvas();
    // hedefleri temizle
    targets = [];
    particles = [];
    trails = [];

    // oluşturacak hedef yuvarlaklar — farklı boyutlarda ve konumlarda
    const cols = 5;
    const rows = 2;
    const padding = Math.min(W, H) * 0.06;
    const areaW = W - padding*2;
    const areaH = H*0.55;
    for(let r=0;r<rows;r++){
      for(let c=0;c<cols;c++){
        const radius = randInt(Math.round(W*0.045), Math.round(W*0.085));
        const gapX = areaW / cols;
        const x = Math.round(padding + gapX * (c + 0.5) + rand(-gapX*0.12, gapX*0.12));
        const y = Math.round(H*0.24 + r*(areaH/rows) + rand(-areaH*0.06, areaH*0.06));
        const color = RAINBOW[(r*cols + c) % RAINBOW.length];
        targets.push({
          x,y,radius,baseRadius:radius,color,scale:1,scaleAnim:0, lastHit: -99999
        });
      }
    }

    // başlangıçta top yok (kullanıcı başlatacak)
    ball = null;
    drawStatic(); // bir kere çiz
    startLoop();
  }

  // Başlat: topu üst merkezden bırak
  function start(){
    if(ball) return; // zaten varsa ekleme
    createBall(W/2, Math.max(40, H*0.06));
  }

  // Yeni top oluştur (veya tıkla bırakma)
  function createBall(x,y){
    const size = Math.max(18, Math.round(Math.min(W,H)*0.04));
    ball = {
      x, y,
      vx: rand(-60,60),
      vy: 40,
      radius: size,
      color: '#ffffff',
      trailTimer: 0,
      alive: true
    };
  }

  // resize canvas
  function resizeCanvas(){
    W = wrap.clientWidth;
    H = wrap.clientHeight;
    canvas.width = Math.floor(W * devicePixelRatio);
    canvas.height = Math.floor(H * devicePixelRatio);
    canvas.style.width = W + 'px';
    canvas.style.height = H + 'px';
    ctx.setTransform(devicePixelRatio,0,0,devicePixelRatio,0,0);
  }

  // animasyon döngüsü
  function startLoop(){
    last = performance.now();
    function loop(now){
      const dt = Math.min(40, now - last) / 1000; // s
      last = now;
      update(dt);
      render();
      rafId = requestAnimationFrame(loop);
    }
    rafId = requestAnimationFrame(loop);
  }

  // update physics
  function update(dt){
    // gravity ölçeği ekran büyüklüğüne göre ayarlanır
    const gravity = gravityBase * (Math.min(W,H)/800);
    // güncelle hedef animasyonları (scale geri dönmesi)
    targets.forEach(t=>{
      // hit anim: scaleAnim>0 ise effect
      if(t.scaleAnim > 0){
        t.scaleAnim -= dt * 3.4; // düşüş hızı; arttırarak hızlıca küçült
        t.scale = 1 + 0.45 * Math.sin(t.scaleAnim * Math.PI); // yumuşak büyüme
      } else {
        t.scaleAnim = 0;
        t.scale = 1;
      }
    });

    // parçacıkları güncelle
    for(let i = particles.length -1; i>=0; i--){
      const p = particles[i];
      p.vy += gravity * dt * 0.08; // hafif yerçekimi parçacıklara
      p.x += p.vx * dt;
      p.y += p.vy * dt;
      p.life -= dt;
      p.angle += p.spin * dt;
      if(p.life <= 0) particles.splice(i,1);
    }

    // top varsa fizik uygula
    if(ball && ball.alive){
      // yerçekimi
      ball.vy += gravity * dt;
      ball.x += ball.vx * dt;
      ball.y += ball.vy * dt;

      // duvar çarpışmaları (sağ-sol)
      if(ball.x - ball.radius < 0){
        ball.x = ball.radius;
        ball.vx *= -restitution;
      } else if(ball.x + ball.radius > W){
        ball.x = W - ball.radius;
        ball.vx *= -restitution;
      }

      // zemin çatışması
      if(ball.y + ball.radius > H){
        ball.y = H - ball.radius;
        ball.vy *= -restitution;
        ball.vx *= groundFriction;
        // küçük vy ise durdur
        if(Math.abs(ball.vy) < 60) {
          ball.vy = 0;
          // yerden kıpırdanma sonrası sönme
          ball.vx *= 0.98;
          if(Math.abs(ball.vx) < 10) {
            // top durdu → son kalıntı iz bırak
            spawnLandingParticles(ball.x, ball.y, Math.min(18, Math.round(ball.radius/2)));
            // topu pasifleştir (isteğe bağlı)
            //ball.alive = false;
          }
        }
      }

      // top sürekli iz bırakma (her frame bir küçük renkli nokta)
      ball.trailTimer += dt;
      if(ball.trailTimer > 0.02){
        ball.trailTimer = 0;
        trails.push({
          x: ball.x + rand(-4,4),
          y: ball.y + rand(-4,4),
          life: 0.9,
          color: RAINBOW[(Math.floor(Math.random()*RAINBOW.length))],
          r: Math.max(2, ball.radius*0.18 + rand(-1,1))
        });
      }
      // trails güncelle
      for(let i = trails.length -1; i>=0; i--){
        trails[i].life -= dt;
        if(trails[i].life <= 0) trails.splice(i,1);
      }

      // hedeflerle çarpışma (basit daire-daire çarpışma)
      targets.forEach(t=>{
        const dx = ball.x - t.x;
        const dy = ball.y - t.y;
        const dist = Math.hypot(dx,dy);
        const tRadius = t.radius * t.scale;
        if(dist < ball.radius + tRadius){
          // çarpışma anı (cooldown ile tekrar tetiklemeyi engelle)
          const now = performance.now();
          if(now - t.lastHit > 240){
            t.lastHit = now;
            // büyüme animasyonu tetikle
            t.scaleAnim = 1;
            // topa bir tepki ver (basit yansıtma)
            // normal vektör
            const nx = dx / (dist || 1);
            const ny = dy / (dist || 1);
            // yansıtma (topun hızını normal bileşenine göre ters çevir)
            const vDotN = ball.vx*nx + ball.vy*ny;
            ball.vx = ball.vx - (1.9 * vDotN) * nx;
            ball.vy = ball.vy - (1.9 * vDotN) * ny;
            // biraz enerji kaybı uygula
            ball.vx *= 0.98;
            ball.vy *= 0.98;
            // çarpışma noktasında gökkuşağı parçacıkları bırak
            spawnCollisionParticles(ball.x - nx* (ball.radius*0.3), ball.y - ny*(ball.radius*0.3), Math.min(28, Math.round(t.radius*0.8)));
          }
        }
      });
    }
  }

  // render: temizle -> çiz (trails, targets, particles, ball)
  function render(){
    // arkaplan temizle (hafif opak fill ile motion blur / glow hissi)
    ctx.clearRect(0,0,W,H);

    // hafif bir glow efekti için çok sayıda yarı saydam çizim (isteğe göre kaldırılabilir)
    // önce trails (sönük)
    trails.forEach(tr=>{
      ctx.beginPath();
      ctx.fillStyle = hexToRgba(tr.color, 0.2 * Math.max(0, tr.life));
      ctx.arc(tr.x, tr.y, tr.r, 0, Math.PI*2);
      ctx.fill();
    });

    // hedefler (arkaya gölgeli)
    targets.forEach(t=>{
      // gölge halo
      ctx.beginPath();
      const gx = t.x, gy = t.y;
      const gr = t.radius * t.scale * 1.6;
      const ggrad = ctx.createRadialGradient(gx,gy,gr*0.1,gx,gy,gr);
      ggrad.addColorStop(0, hexToRgba(t.color, 0.12));
      ggrad.addColorStop(1, hexToRgba(t.color, 0.0));
      ctx.fillStyle = ggrad;
      ctx.arc(gx,gy,gr,0,Math.PI*2);
      ctx.fill();

      // daire gövdesi
      ctx.beginPath();
      ctx.fillStyle = t.color;
      ctx.arc(t.x, t.y, t.radius * t.scale, 0, Math.PI*2);
      ctx.fill();

      // iç parlaklık
      ctx.beginPath();
      ctx.fillStyle = hexToRgba('#ffffff', 0.06);
      ctx.arc(t.x - t.radius*0.18, t.y - t.radius*0.18, t.radius * t.scale * 0.55, 0, Math.PI*2);
      ctx.fill();
    });

    // parçacıklar (gökkuşağı)
    particles.forEach(p=>{
      ctx.save();
      ctx.translate(p.x, p.y);
      ctx.rotate(p.angle);
      ctx.beginPath();
      ctx.fillStyle = hexToRgba(p.color, Math.max(0, p.life / p.maxLife));
      ctx.arc(0,0,p.r,0,Math.PI*2);
      ctx.fill();
      ctx.restore();
    });

    // top
    if(ball && ball.alive){
      // glow
      ctx.beginPath();
      const glowR = ball.radius * 2.6;
      const ggrad = ctx.createRadialGradient(ball.x, ball.y, ball.radius*0.2, ball.x, ball.y, glowR);
      ggrad.addColorStop(0, 'rgba(255,255,255,0.22)');
      ggrad.addColorStop(1, 'rgba(255,255,255,0.00)');
      ctx.fillStyle = ggrad;
      ctx.arc(ball.x, ball.y, glowR, 0, Math.PI*2);
      ctx.fill();

      // top body
      ctx.beginPath();
      ctx.fillStyle = '#ffffff';
      ctx.arc(ball.x, ball.y, ball.radius, 0, Math.PI*2);
      ctx.fill();

      // küçük parlama
      ctx.beginPath();
      ctx.fillStyle = 'rgba(255,255,255,0.9)';
      ctx.arc(ball.x - ball.radius*0.28, ball.y - ball.radius*0.28, Math.max(2, ball.radius*0.26), 0, Math.PI*2);
      ctx.fill();
    }
  }

  // collision parçacıkları üret (çarpışma sırasında)
  function spawnCollisionParticles(x,y,count){
    for(let i=0;i<count;i++){
      const ang = rand(0, Math.PI*2);
      const speed = rand(80, 420) * (Math.min(W,H)/800);
      const color = RAINBOW[i % RAINBOW.length];
      particles.push({
        x: x + rand(-6,6),
        y: y + rand(-6,6),
        vx: Math.cos(ang) * speed,
        vy: Math.sin(ang) * speed,
        r: rand(2, 6) * (Math.min(W,H)/800 + 0.8),
        color,
        life: rand(0.45, 1.12),
        maxLife: rand(0.45, 1.12),
        angle: rand(0,Math.PI*2),
        spin: rand(-6,6)
      });
    }
  }

  // iniş parçacıkları (zemine çarpınca)
  function spawnLandingParticles(x,y,count){
    for(let i=0;i<count;i++){
      const ang = rand(Math.PI*0.9, Math.PI*0.1 + Math.PI*2);
      const speed = rand(20, 200) * (Math.min(W,H)/800);
      const color = RAINBOW[i % RAINBOW.length];
      particles.push({
        x: x + rand(-10,10),
        y: y + rand(-6,6),
        vx: Math.cos(ang) * speed,
        vy: Math.sin(ang) * speed,
        r: rand(2, 6),
        color,
        life: rand(0.6, 1.5),
        maxLife: rand(0.6, 1.5),
        angle: rand(0,Math.PI*2),
        spin: rand(-4,4)
      });
    }
  }

  // yardımcı: çizimi bir kere statik yap (başlangıç için)
  function drawStatic(){
    ctx.clearRect(0,0,W,H);
    // hafif arkaplan şekli
    ctx.fillStyle = 'rgba(255,255,255,0.02)';
    ctx.fillRect(0,0,W,H);
    // hedefleri hemen çiz
    targets.forEach(t=>{
      ctx.beginPath();
      ctx.fillStyle = t.color;
      ctx.arc(t.x, t.y, t.radius, 0, Math.PI*2);
      ctx.fill();
    });
  }

  // utility helpers
  function rand(min,max){ return Math.random()*(max-min)+min; }
  function randInt(a,b){ return Math.round(rand(a,b)); }
  function hexToRgba(hex, a){
    // hex can be #rrggbb
    const c = hex.replace('#','');
    const r = parseInt(c.substring(0,2),16);
    const g = parseInt(c.substring(2,4),16);
    const b = parseInt(c.substring(4,6),16);
    return `rgba(${r},${g},${b},${a})`;
  }

  // eventler: yeniden boyut, butonlar, tıklama ile yeni top bırakma
  window.addEventListener('resize', () => {
    resizeCanvas();
    // yeniden yerleştir
    // hedefleri yeniden oluşturmak için initScene çağır
    initScene();
  });

  startBtn.addEventListener('click', () => {
    start();
  });
  resetBtn.addEventListener('click', () => {
    initScene();
  });

  // dokun / tık ile top bırakma (kullanıcı tıkladığı yere yakın bırakır)
  canvas.addEventListener('pointerdown', (e) => {
    const rect = canvas.getBoundingClientRect();
    const x = (e.clientX - rect.left);
    const y = (e.clientY - rect.top);
    // eğer top yoksa veya son top sabitlenmişse yeni top oluştur
    if(!ball || (ball && !ball.alive && (ball.vx===0 && ball.vy===0))){
      createBall(x, Math.max(24, y*0.12));
    } else {
      // eğer top varsa tıklanan noktaya hafif kuvvet uygula
      if(ball){
        ball.vx += (x - ball.x) * 0.6;
        ball.vy += (y - ball.y) * 0.35;
      }
    }
  });

  // init
  function boot(){
    resizeCanvas();
    initScene();
  }
  boot();

})();
</script>
</body>
</html>
