<!DOCTYPE html>
<html lang="tr">
<head>
  <meta charset="UTF-8">
  <meta name="viewport" content="width=device-width, initial-scale=1.0">
  <title>Antalya - Özel Alt Alta Ömür Sayacı</title>
  <link rel="preconnect" href="https://fonts.googleapis.com">
  <link rel="preconnect" href="https://fonts.gstatic.com" crossorigin>
  <link href="https://fonts.googleapis.com/css2?family=Inter:wght@300;400;600&display=swap" rel="stylesheet">

  <style>
    body, html {
      margin: 0;
      padding: 0;
      overflow: hidden;
      background-color: #0d0d11;
      font-family: 'Inter', sans-serif;
    }

    canvas {
      position: absolute;
      top: 0;
      left: 0;
      z-index: 1;
      opacity: 0.6;
    }

    .clock-container {
      position: absolute;
      top: 50%;
      left: 50%;
      transform: translate(-50%, -50%);
      z-index: 2;
      text-align: center;
      user-select: none;
      width: 100%;
    }

    .top-info {
      font-size: 1.4rem;
      font-weight: 600;
      color: #ffffff;
      letter-spacing: 6px;
      text-transform: uppercase;
      margin-bottom: 5px;
    }

    .countdown-note {
      font-size: 0.85rem;
      font-weight: 400;
      color: #ffb703;
      letter-spacing: 1px;
      margin-bottom: 20px;
    }

    /* Kırmızıyla işaretlediğin yeni alt alta liste yapısı */
    .time-list {
      display: flex;
      flex-direction: column;
      align-items: center;
      gap: 4px; /* Satırlar arası temiz boşluk */
      margin-bottom: 25px;
    }

    .time-row {
      font-size: 1.8rem;
      font-weight: 400;
      color: #ffffff;
      line-height: 1.2;
    }

    .time-row span {
      font-weight: 600;
      color: #00ffe0; /* Sayılar senin istediğin o neon mavi renginde parlar */
    }

    /* Salisenin fontunu azıcık küçülttüm ki o çılgın hız gözü yormasın */
    .salise-row {
      font-size: 1.5rem;
      opacity: 0.9;
    }

    .date {
      font-size: 1.1rem;
      font-weight: 400;
      color: #8a8a93;
      letter-spacing: 1px;
    }
  </style>
</head>
<body>

  <canvas id="starfield"></canvas>

  <div class="clock-container">
    <div class="top-info">ANTALYA</div>
    
    <div class="countdown-note" id="birthday-countdown">Hesaplanıyor...</div>
    
    <div class="time-list">
      <div class="time-row"><span id="val-yil">00</span> Yıl</div>
      <div class="time-row"><span id="val-ay">00</span> Ay</div>
      <div class="time-row"><span id="val-gun">00</span> Gün</div>
      <div class="time-row"><span id="val-saat">00</span> Saat</div>
      <div class="time-row"><span id="val-dakika">00</span> Dakika</div>
      <div class="time-row"><span id="val-saniye">00</span> Saniye</div>
      <div class="time-row salise-row"><span id="val-salise">00</span> Salise</div>
    </div>
    
    <div class="date">28 Kasım 2011 - 10:07</div>
  </div>

  <script>
    // --- CANVAS ARKA PLAN (BEYAZ YILDIZLAR) ---
    const canvas = document.getElementById('starfield');
    const ctx = canvas.getContext('2d');
    const stars = [];

    function resize() {
      canvas.width = window.innerWidth;
      canvas.height = window.innerHeight;
    }
    window.addEventListener('resize', resize);
    resize();

    for (let i = 0; i < 120; i++) {
      stars.push({
        x: Math.random() * canvas.width,
        y: Math.random() * canvas.height,
        radius: Math.random() * 1.2,
        speed: Math.random() * 0.2 + 0.05
      });
    }

    function animateStars() {
      ctx.clearRect(0, 0, canvas.width, canvas.height);
      ctx.fillStyle = '#ffffff';
      
      for (let s of stars) {
        ctx.beginPath();
        ctx.arc(s.x, s.y, s.radius, 0, Math.PI * 2);
        ctx.fill();
        s.y += s.speed;
        if (s.y > canvas.height) {
          s.y = 0;
          s.x = Math.random() * canvas.width;
        }
      }
      requestAnimationFrame(animateStars);
    }
    animateStars();

    // --- ÖMÜR VE SALİSE MOTORU ---
    const dogumTarihi = new Date(2011, 10, 28, 10, 7, 0, 0); // 28 Kasım 2011 10:07:00:000

    function updateCounter() {
      const now = new Date();
      
      // --- 1. ÜST ALAN: Geri Sayım (Bu fonksiyon milisaniyede bir dönmesin diye performansı korur) ---
      if (now.getMilliseconds() < 20) { 
        const dogumGunuGunu = 28;
        const dogumGunuAyi = 10;
        let hedefYil = now.getFullYear();
        let sonrakiBday = new Date(hedefYil, dogumGunuAyi, dogumGunuGunu, 10, 7, 0);
        
        if (now > sonrakiBday) {
          hedefYil += 1;
          sonrakiBday = new Date(hedefYil, dogumGunuAyi, dogumGunuGunu, 10, 7, 0);
        }

        const countdownEl = document.getElementById('birthday-countdown');
        if (now.getDate() === dogumGunuGunu && now.getMonth() === dogumGunuAyi) {
          countdownEl.textContent = "🎂 BUGÜN SENİN DOĞUM GÜNÜN! MUTLU YILLAR! 🎉";
        } else {
          const diff = sonrakiBday - now;
          const kGun = Math.floor(diff / (1000 * 60 * 60 * 24));
          const kSaat = Math.floor((diff % (1000 * 60 * 60 * 24)) / (1000 * 60 * 60));
          const kDakika = Math.floor((diff % (1000 * 60 * 60)) / (1000 * 60));
          const kSaniye = Math.floor((diff % (1000 * 60)) / 1000);
          countdownEl.textContent = `⏳ DOĞUM GÜNÜNE: ${kGun} GÜN, ${kSaat} SAAT, ${kDakika} DAKİKA, ${kSaniye} SANİYE KALDI`;
        }
      }

      // --- 2. KIRMIZI ALAN: Doğumdan Bugüne Geçen Detaylı Süre + Salise Hesabı ---
      let gecenYil = now.getFullYear() - dogumTarihi.getFullYear();
      let gecenAy = now.getMonth() - dogumTarihi.getMonth();
      let gecenGun = now.getDate() - dogumTarihi.getDate();
      let gecenSaat = now.getHours() - dogumTarihi.getHours();
      let gecenDakika = now.getMinutes() - dogumTarihi.getMinutes();
      let gecenSaniye = now.getSeconds() - dogumTarihi.getSeconds();
      
      // JavaScript milisaniyeyi (0-999) alır, biz bunu çift haneli saliseye (0-99) çeviriyoruz
      let gecenSalise = Math.floor(now.getMilliseconds() / 10);

      // Zaman taşmalarını düzelten hassas algoritma
      if (gecenSaniye < 0) { gecenSaniye += 60; gecenDakika--; }
      if (gecenDakika < 0) { gecenDakika += 60; gecenSaat--; }
      if (gecenSaat < 0) { gecenSaat += 24; gecenGun--; }
      if (gecenGun < 0) {
        const oncekiAy = new Date(now.getFullYear(), now.getMonth(), 0).getDate();
        gecenGun += oncekiAy;
        gecenAy--;
      }
      if (gecenAy < 0) { gecenAy += 12; gecenYil--; }

      // HTML elementlerini bozmadan sadece içindeki sayıları saniyenin yüzde biri hızında yeniliyoruz
      document.getElementById('val-yil').textContent = gecenYil;
      document.getElementById('val-ay').textContent = gecenAy;
      document.getElementById('val-gun').textContent = gecenGun;
      document.getElementById('val-saat').textContent = String(gecenSaat).padStart(2, '0');
      document.getElementById('val-dakika').textContent = String(gecenDakika).padStart(2, '0');
      document.getElementById('val-saniye').textContent = String(gecenSaniye).padStart(2, '0');
      document.getElementById('val-salise').textContent = String(gecenSalise).padStart(2, '0');
    }

    // Çılgın akan saliseler için sayacı her 10 milisaniyede bir tetikliyoruz!
    setInterval(updateCounter, 10);
    updateCounter();
  </script>
</body>
</html>
