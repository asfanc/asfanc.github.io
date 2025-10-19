<!doctype html>
<html lang="tr">
<head>
<meta charset="utf-8"/>
<meta name="viewport" content="width=device-width,initial-scale=1"/>
<title>Beyaz İnce Çember — Ortadan Bırakılan Top + Gökkuşağı İzi</title>
<style>
  :root{
    --bg:#02030a;       /* arka plan (siyah benzeri) */
    --ring-color: #ffffff;
    --ring-stroke: 4px; /* çember kalınlığı */
  }
  html,body{height:100%;margin:0;background:var(--bg);font-family:Inter,system-ui,Arial;color:#e6eef8}
  .wrap{min-height:100vh;display:flex;flex-direction:column;align-items:center;gap:12px;padding:18px;box-sizing:border-box}
  .controls{display:flex;gap:8px;align-items:center}
  .btn{padding:9px 12px;border-radius:10px;border:1px solid rgba(255,255,255,0.08);background:transparent;color:#eaf4ff;cursor:pointer;font-weight:700}
  .btn.primary{background:linear-gradient(90deg,#89f7fe,#66a6ff);color:#022;border:0}
  .hint{opacity:0.82;font-size:13px}

  .stage{ width:100%; max-width:980px; height:72vh; min-height:520px; border-radius:12px; position:relative; overflow:hidden; box-shadow: 0 10px 30px rgba(0,0,0,0.6) }
  canvas{ display:block; width:100%; height:100%; touch-action:none; }

  /* küçük açıklama */
  .info{position:absolute;left:12px;top:12px;padding:8px 10px;border-radius:8px;background:rgba(255,255,255,0.03);backdrop-filter:blur(4px);font-size:13px;color:rgba(255,255,255,0.9);z-index:20}
  @media (max-width:600px){
    .stage{height:68vh; min-height:520px}
  }
</style>
</head>
<body>
  <div class="wrap">
    <div class="controls">
      <button id="startBtn" class="btn primary">Başlat</button>
      <button id="resetBtn" class="btn">Sıfırla</button>
      <div class="hint">Başlat: ortadaki top bırakılır. Ekrana dokunursan da yeni top bırakabilirsin.</div>
    </div>

    <div class="stage" id="stage">
      <div class="info">Tek ince beyaz çember — içten bırakılan top çarpınca çember pop ve gökkuşağı izi.</div>
      <canvas id="c"></canvas>
    </div>
  </div>

<script>
/* Tek dosya — tek çember, ortadan bırakılan top, çarpmada pop ve gökkuşağı izi */
(() => {
  const canvas = document.getElementById('c');
  const stage = document.getElementById('stage');
  const ctx = canvas.getContext('2d', { alpha: true });
  const startBtn = document.getElementById('startBtn');
  const resetBtn = document.getElementById('resetBtn');

  let W=0,H=0,DPR = devicePixelRatio || 1;
  let raf = null, last = 0;

  // Fizik
  const GRAVITY_BASE = 1400; // px/s^2
  const REST = 0.68;
  const FRICTION = 0.998;

  // Ring (tek çember)
  let ring = {
    x: 0, y: 0, r: 160, stroke: 4, scale: 1, anim: 0, lastHit: -9999
  };

  // Bekleyen inner top (başlangıçta merkezde gösterilir)
  let innerPresent = true;
  const innerRFrac = 0.12; // ring radius ratio for inner top size

  // Aktif fizik toplar
  let balls = [];

  // Parçacıklar (gökkuşağı izi)
  let particles = [];

  // trail (küçük renk noktaları topun geçtiği yolda)
  let trails = [];

  const RAINBOW = ['#ff3b30','#ff9500','#ffcc00','#34c759','#5ac8fa','#007aff','#5856d6'];

  function resize(){
    W = stage.clientWidth;
    H = stage.clientHeight;
    DPR = devicePixelRatio || 1;
    canvas.width = Math.floor(W * DPR);
    canvas.height = Math.floor(H * DPR);
    canvas.style.width = W + 'px';
    canvas.style.height = H + 'px';
    ctx.setTransform(DPR,0,0,DPR,0,0);

    // ring orta nokta ve yarıçap ayarla (ekran merkezli, responsive)
    ring.x = W/2;
    ring.y = H/2;
    ring.r = Math.round(Math.min(W,H) * 0.31); // çember büyüklüğü
    ring.stroke = Math.max(2, Math.round(Math.min(W,H)*0.006));
  }

  // Başlangıç / reset
  function init(){
    cancelAnimationFrame(raf);
    resize();
    ring.scale = 1; ring.anim = 0; ring.lastHit = -9999;
    innerPresent = true;
    balls = [];
    particles = [];
    trails = [];
    last = performance.now();
    loop(last);
  }

  // top ortadan bırak (ilk topu oluştur)
  function releaseCenter(){
    if(!innerPresent) return;
    const r = Math.max(8, Math.round(ring.r * innerRFrac));
    const ball = {
      x: ring.x,
      y: ring.y,
      vx: rand(-40,40),
      vy: rand(10,40),
      r,
      alive: true,
      trailTimer: 0
    };
    balls.push(ball);
    innerPresent = false;
  }

  // kullanıcı tıklaması ile merkezden bırakma
  canvas.addEventListener('pointerdown', (e) => {
    // eğer merkezde bekleyen top varsa bırak
    if(innerPresent){
      releaseCenter();
      return;
    }
    // değilse tıklama pozisyonuna yakın bir top bırak (alternatif)
    const rect = canvas.getBoundingClientRect();
    const x = (e.clientX - rect.left);
    const y = (e.clientY - rect.top);
    const r = Math.max(8, Math.round(ring.r * innerRFrac));
    balls.push({
      x, y: Math.max(y, ring.y - ring.r + 8), // güvenlik
      vx: rand(-40,40), vy: rand(-30,20),
      r, alive:true, trailTimer:0
    });
  });

  // update physics
  function update(dt){
    const gravity = GRAVITY_BASE * (Math.min(W,H)/800);

    // ring animation (pop)
    if(ring.anim > 0){
      ring.anim -= dt * 3.8;
      ring.scale = 1 + 0.18 * Math.sin(ring.anim * Math.PI);
    } else {
      ring.anim = 0;
      ring.scale = 1;
    }

    // particles update
    for(let i = particles.length-1;i>=0;i--){
      const p = particles[i];
      p.vy += gravity * 0.03 * dt;
      p.x += p.vx * dt;
      p.y += p.vy * dt;
      p.life -= dt;
      p.angle += p.spin * dt;
      if(p.life <= 0) particles.splice(i,1);
    }

    // trails update
    for(let i = trails.length-1;i>=0;i--){
      trails[i].life -= dt;
      if(trails[i].life <= 0) trails.splice(i,1);
    }

    // balls physics
    for(let i = balls.length-1;i>=0;i--){
      const b = balls[i];
      b.vy += gravity * dt;
      b.x += b.vx * dt;
      b.y += b.vy * dt;

      // wall left/right
      if(b.x - b.r < 0){
        b.x = b.r; b.vx *= -REST;
      } else if(b.x + b.r > W){
        b.x = W - b.r; b.vx *= -REST;
      }

      // floor / ceiling
      if(b.y + b.r > H){
        b.y = H - b.r;
        b.vy *= -REST;
        b.vx *= FRICTION;
        if(Math.abs(b.vy) < 40) b.vy = 0;
      } else if(b.y - b.r < 0){
        b.y = b.r;
        b.vy *= -REST;
      }

      // ring collision (circle boundary): check distance from ring center
      const dx = b.x - ring.x;
      const dy = b.y - ring.y;
      const dist = Math.hypot(dx, dy);
      const ringRadiusNow = ring.r * ring.scale;
      // collision when ball center is at distance >= ringRadiusNow - b.r  (içten kenara çarpma)
      if(dist + b.r > ringRadiusNow - 0.3){
        // sunucu zaman kontrolu ile sürekli tetiklemeyi engelle
        const now = performance.now();
        if(now - ring.lastHit > 160){
          ring.lastHit = now;
          ring.anim = 1; // tetikle pop
          // yansıtma hesapla (normal)
          const nx = dx / (dist || 1);
          const ny = dy / (dist || 1);
          const vDotN = b.vx*nx + b.vy*ny;
          b.vx = b.vx - (1.9 * vDotN) * nx;
          b.vy = b.vy - (1.9 * vDotN) * ny;
          b.vx *= 0.98; b.vy *= 0.98;
          // spawn gökkuşağı partiküller (çarpma noktasında)
          spawnCollisionParticles(b.x - nx*(b.r*0.6), b.y - ny*(b.r*0.6), Math.min(26, Math.round(b.r*2.6)));
        }
      }

      // top trail (küçük renkli noktalar)
      b.trailTimer += dt;
      if(b.trailTimer > 0.02){
        b.trailTimer = 0;
        trails.push({
          x: b.x + rand(-3,3),
          y: b.y + rand(-3,3),
          r: Math.max(1.6, b.r*0.14),
          color: RAINBOW[Math.floor(Math.random()*RAINBOW.length)],
          life: 0.9
        });
      }
    }
  }

  function spawnCollisionParticles(x,y,count){
    for(let i=0;i<count;i++){
      const ang = rand(0, Math.PI*2);
      const speed = rand(80, 420) * (Math.min(W,H)/800);
      particles.push({
        x: x + rand(-6,6),
        y: y + rand(-6,6),
        vx: Math.cos(ang) * speed,
        vy: Math.sin(ang) * speed,
        r: rand(2,5) * (Math.min(W,H)/800 + 0.7),
        color: RAINBOW[i % RAINBOW.length],
        life: rand(0.45, 1.08),
        maxLife: rand(0.45, 1.08),
        angle: rand(0,Math.PI*2),
        spin: rand(-6,6)
      });
    }
  }

  // render
  function render(){
    ctx.clearRect(0,0,W,H);

    // background (solid same as body)
    ctx.fillStyle = getComputedStyle(document.documentElement).getPropertyValue('--bg') || '#02030a';
    ctx.fillRect(0,0,W,H);

    // draw trails (fading colored dots)
    for(const t of trails){
      ctx.beginPath();
      ctx.fillStyle = rgba(t.color, Math.max(0, t.life));
      ctx.arc(t.x, t.y, t.r, 0, Math.PI*2);
      ctx.fill();
    }

    // draw particles
    for(const p of particles){
      ctx.save();
      ctx.translate(p.x, p.y);
      ctx.rotate(p.angle);
      ctx.beginPath();
      ctx.fillStyle = rgba(p.color, Math.max(0, p.life / p.maxLife));
      ctx.arc(0,0,p.r,0,Math.PI*2);
      ctx.fill();
      ctx.restore();
    }

    // draw ring (only stroke, ortası arka planla aynı görünsün)
    ctx.save();
    ctx.beginPath();
    ctx.lineWidth = ring.stroke;
    ctx.strokeStyle = rgba('#ffffff', 0.98);
    ctx.arc(ring.x, ring.y, ring.r * ring.scale, 0, Math.PI*2);
    ctx.stroke();
    ctx.restore();

    // draw inner small top if present (gösterim amacı)
    if(innerPresent){
      const innerR = Math.max(6, Math.round(ring.r * innerRFrac));
      ctx.beginPath();
      // inner görünüşü: hafif mat beyaz
      ctx.fillStyle = '#f1f5f9';
      ctx.arc(ring.x, ring.y, innerR, 0, Math.PI*2);
      ctx.fill();
    }

    // draw balls
    for(const b of balls){
      // glow
      ctx.beginPath();
      const glowR = b.r * 2.2;
      const ggrad = ctx.createRadialGradient(b.x, b.y, b.r*0.2, b.x, b.y, glowR);
      ggrad.addColorStop(0, 'rgba(255,255,255,0.22)');
      ggrad.addColorStop(1, 'rgba(255,255,255,0.00)');
      ctx.fillStyle = ggrad;
      ctx.arc(b.x, b.y, glowR, 0, Math.PI*2);
      ctx.fill();
      // body
      ctx.beginPath();
      ctx.fillStyle = '#ffffff';
      ctx.arc(b.x, b.y, b.r, 0, Math.PI*2);
      ctx.fill();
      // highlight
      ctx.beginPath();
      ctx.fillStyle = 'rgba(255,255,255,0.9)';
      ctx.arc(b.x - b.r*0.26, b.y - b.r*0.26, Math.max(1.2, b.r*0.26), 0, Math.PI*2);
      ctx.fill();
    }
  }

  // animation loop
  function loop(now){
    const dt = Math.min(40, now - last) / 1000;
    last = now;
    update(dt);
    render();
    raf = requestAnimationFrame(loop);
  }

  // helpers
  function rand(a,b){ return Math.random()*(b-a)+a; }
  function rgba(hex, a){
    // hex like '#rrggbb'
    const c = hex.replace('#','');
    const r = parseInt(c.substring(0,2),16);
    const g = parseInt(c.substring(2,4),16);
    const b = parseInt(c.substring(4,6),16);
    return `rgba(${r},${g},${b},${a})`;
  }

  // buttons
  startBtn.addEventListener('click', () => {
    releaseCenter();
  });
  resetBtn.addEventListener('click', () => {
    init();
  });

  // handle resize
  let resizeTimeout;
  window.addEventListener('resize', ()=> {
    clearTimeout(resizeTimeout);
    resizeTimeout = setTimeout(()=> {
      resize();
    }, 120);
  });

  // init
  init();

})();
</script>
</body>
</html>
