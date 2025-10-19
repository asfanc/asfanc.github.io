<!doctype html>
<html lang="tr">
<head>
<meta charset="utf-8" />
<meta name="viewport" content="width=device-width,initial-scale=1"/>
<title>Yuvarlak İz Efekti — Başlat</title>
<style>
  :root{
    --bg:#0f1724;
    --circle:#60a5fa;
    --accent:#34d399;
    --trail-alpha:0.22;
  }
  html,body{
    height:100%;
    margin:0;
    font-family:Inter,system-ui,Arial;
    background:linear-gradient(180deg,var(--bg), #071029 120%);
    color:#e6eef8;
  }

  .wrap{
    display:flex;
    flex-direction:column;
    gap:12px;
    align-items:center;
    padding:18px;
    min-height:100vh;
    box-sizing:border-box;
  }

  .controls{
    display:flex;
    gap:10px;
    align-items:center;
  }
  .btn{
    background:transparent;
    border:2px solid rgba(255,255,255,0.12);
    color:inherit;
    padding:10px 14px;
    border-radius:10px;
    cursor:pointer;
    font-weight:600;
    backdrop-filter: blur(4px);
  }
  .btn.primary{
    background:linear-gradient(90deg,var(--accent), #60a5fa);
    color:#06202a;
    border:0;
    box-shadow:0 6px 18px rgba(3,7,18,0.5);
  }
  .hint{opacity:0.85;font-size:14px}

  /* oyun alanı */
  .stage{
    position:relative;
    width:100%;
    max-width:1100px;
    height:70vh;
    min-height:360px;
    border-radius:14px;
    overflow:hidden;
    border:1px solid rgba(255,255,255,0.04);
    background:
      radial-gradient(circle at 20% 20%, rgba(255,255,255,0.02), transparent 10%),
      linear-gradient(180deg, rgba(255,255,255,0.01), transparent 40%);
    box-shadow:0 8px 40px rgba(2,6,12,0.6);
  }

  /* yuvarlaklar (interactive) */
  .circle{
    position:absolute;
    width:64px;
    height:64px;
    border-radius:50%;
    display:flex;
    align-items:center;
    justify-content:center;
    transform-origin:center;
    transition: transform 280ms cubic-bezier(.2,.9,.3,1), box-shadow 200ms;
    user-select:none;
    -webkit-user-select:none;
    touch-action:manipulation;
    cursor:pointer;
  }
  .circle .label{
    font-size:12px;
    pointer-events:none;
    opacity:0.95;
  }

  /* iz / trail */
  .trail{
    position:absolute;
    border-radius:50%;
    pointer-events:none;
    transform-origin:center;
    animation:trailFade 900ms forwards;
  }
  @keyframes trailFade{
    0% { opacity: 1; transform: scale(1); }
    100% { opacity: 0; transform: scale(1.85); }
  }

  /* aktif değil ikonu */
  .inactiveOverlay{
    position:absolute;
    inset:0;
    display:flex;
    align-items:center;
    justify-content:center;
    background:linear-gradient(180deg, rgba(2,6,12,0.35), rgba(2,6,12,0.45));
    color: rgba(255,255,255,0.6);
    font-weight:600;
    gap:10px;
    pointer-events:none;
    transition:opacity .25s;
  }

  /* küçük ekran düzenlemeleri */
  @media (max-width:600px){
    .circle{ width:56px; height:56px; }
    .stage{ height:62vh; min-height:420px; }
  }
</style>
</head>
<body>
  <div class="wrap">
    <div class="controls">
      <button id="startBtn" class="btn primary">Başlat</button>
      <button id="resetBtn" class="btn">Sıfırla</button>
      <div class="hint">Buton aktifse yuvarlaklara dokun → büyücek ve iz bırakacak.</div>
    </div>

    <div id="stage" class="stage" aria-label="Etkileşim alanı">
      <div id="overlay" class="inactiveOverlay">Etkileşim devre dışı — Başlat'a bas</div>
      <!-- yuvarlaklar JS ile oluşturulacak -->
    </div>
  </div>

<script>
(() => {
  const stage = document.getElementById('stage');
  const startBtn = document.getElementById('startBtn');
  const resetBtn = document.getElementById('resetBtn');
  const overlay = document.getElementById('overlay');

  let active = false;
  // ayarlar:
  const CIRCLE_COUNT = 10;
  const CIRCLE_MIN = 48; // px
  const CIRCLE_MAX = 88; // px
  const GROW_SCALE = 1.6; // büyüme faktörü
  const TRAIL_LIFETIME = 900; // ms, CSS anim ile eşleşmeli

  // rastgele ama ekran dışına taşmayacak pozisyon
  function rand(min, max){ return Math.random()*(max-min)+min; }

  // rastgele renk hattı
  const palette = [
    'rgba(96,165,250,0.98)', // mavi
    'rgba(99,102,241,0.98)', // mor
    'rgba(52,211,153,0.98)', // yeşil
    'rgba(249,115,22,0.98)', // turuncu
    'rgba(236,72,153,0.98)'  // pembe
  ];

  function makeCircles(){
    // temizle
    stage.querySelectorAll('.circle, .trail').forEach(n=>n.remove());

    const rect = stage.getBoundingClientRect();
    for(let i=0;i<CIRCLE_COUNT;i++){
      const size = Math.round(rand(CIRCLE_MIN, CIRCLE_MAX));
      const el = document.createElement('button');
      el.className = 'circle';
      el.type = 'button';
      // renk, hafif gölge
      const color = palette[i % palette.length];
      el.style.background = color;
      el.style.boxShadow = `0 8px 24px rgba(2,6,23,0.45)`;
      el.style.width = size+'px';
      el.style.height = size+'px';

      // label - isteğe bağlı
      const lab = document.createElement('div');
      lab.className = 'label';
      lab.textContent = (i+1);
      el.appendChild(lab);

      // pozisyon (içeride kalacak şekilde)
      const padding = 8;
      const x = rand(padding, Math.max(1, rect.width - size - padding));
      const y = rand(padding, Math.max(1, rect.height - size - padding));
      el.style.left = Math.round(x)+'px';
      el.style.top = Math.round(y)+'px';
      // benzersiz id
      el.dataset.idx = i;

      // event listeners
      // tıklama/dokunma için aynı handler
      const handler = (ev) => {
        if(!active) return;
        ev.preventDefault();
        pokeCircle(el, ev);
      };

      el.addEventListener('click', handler);
      el.addEventListener('touchstart', handler, {passive:false});

      stage.appendChild(el);
    }
  }

  function pokeCircle(circleEl, ev){
    // görsel büyütme
    circleEl.style.transition = 'transform 250ms cubic-bezier(.2,.9,.3,1), box-shadow 180ms';
    circleEl.style.transform = `scale(${GROW_SCALE})`;
    circleEl.style.boxShadow = '0 10px 30px rgba(2,6,23,0.7)';

    // kısa bir süre sonra eski haline dön
    setTimeout(() => {
      circleEl.style.transform = '';
      circleEl.style.boxShadow = '';
    }, 260);

    // iz bırak
    createTrail(circleEl);
  }

  function createTrail(circleEl){
    const rect = stage.getBoundingClientRect();
    const cRect = circleEl.getBoundingClientRect();
    // trail el
    const t = document.createElement('div');
    t.className = 'trail';
    // aynı boyutla başla
    const w = cRect.width, h = cRect.height;
    t.style.width = w + 'px';
    t.style.height = h + 'px';
    // absolute pos: konum stage içindeki koordinata çevir
    const left = cRect.left - rect.left;
    const top  = cRect.top  - rect.top;
    t.style.left = left + 'px';
    t.style.top  = top + 'px';

    // renk ve transparanlık
    // trail rengini circle renginin rgba'sından oluştur (varsayım: circle inline background rgba)
    const bg = window.getComputedStyle(circleEl).backgroundColor || 'rgba(96,165,250,1)';
    // hafifletilmiş (alpha düşür)
    let trailColor = bg.replace(/rgba?\(([^)]+)\)/, (m,p)=>{
      // p = "r, g, b, [a]"  -> ekley alpha
      const parts = p.split(',').map(s=>s.trim());
      const r = parts[0], g=parts[1], b=parts[2];
      return `rgba(${r}, ${g}, ${b}, ${0.22})`;
    });
    t.style.background = trailColor;
    t.style.filter = 'blur(0.6px)';
    t.style.zIndex = 10;

    stage.appendChild(t);

    // CSS anim ile fade & scale (keyframes defined). Temizle sonrası.
    setTimeout(()=>{
      // trail süresi bittiğinde kaldır
      setTimeout(()=> t.remove(), TRAIL_LIFETIME + 60);
    }, 10);
  }

  // Başlat / Durdur
  function setActive(state){
    active = !!state;
    overlay.style.opacity = active ? '0' : '1';
    overlay.style.pointerEvents = active ? 'none' : 'auto';
    startBtn.textContent = active ? 'Durdur' : 'Başlat';
    startBtn.classList.toggle('primary', !active);
  }

  startBtn.addEventListener('click', () => {
    setActive(!active);
  });

  resetBtn.addEventListener('click', ()=>{
    // yeniden oluştur
    makeCircles();
    setActive(false);
  });

  // ilk oluştur
  window.addEventListener('load', () => {
    makeCircles();
    setActive(false);
  });

  // pencere boyutu değiştiğinde konumları yeniden oluştur (düzensizleşme olmasın)
  let resizeTimer;
  window.addEventListener('resize', () => {
    clearTimeout(resizeTimer);
    resizeTimer = setTimeout(() => {
      makeCircles();
      setActive(false);
    }, 220);
  });

  // erişilebilirlik: klavye ile tıklama
  stage.addEventListener('keydown', (e) => {
    if(e.key === 'Enter' || e.key === ' ') {
      const el = document.activeElement;
      if(el && el.classList.contains('circle')) {
        pokeCircle(el, e);
      }
    }
  });
})();
</script>
</body>
</html>
