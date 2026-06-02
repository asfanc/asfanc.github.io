<!DOCTYPE html>
<html lang="tr">
<head>
  <meta charset="UTF-8">
  <meta name="viewport" content="width=device-width, initial-scale=1.0">
  <title>Antalya - Sana Özel Saat</title>
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
    }

    /* Üst kısma eklenen Antalya alanı */
    .top-info {
      font-size: 1.4rem;
      font-weight: 600;
      color: #ffffff;
      letter-spacing: 6px;
      text-transform: uppercase;
      margin-bottom: 5px;
    }

    /* Doğum gününe kalan süreyi gösteren alan */
    .countdown-note {
      font-size: 0.85rem;
      font-weight: 400;
      color: #ffb703; /* Zarif sarı/altın tonu */
      letter-spacing: 1px;
      margin-bottom: 25px;
    }

    .time {
      font-size: 6rem;
      font-weight: 300;
      color: #ffffff;
      letter-spacing: -2px;
      line-height: 1;
    }

    .date {
      font-size: 1.1rem;
      font-weight: 400;
      color: #8a8a93;
      letter-spacing: 2px;
      margin-top: 15px;
    }
  </style>
</head>
<body>

  <canvas id="starfield"></canvas>

  <div class="clock-container">
    <!-- Antalya Başlığı -->
    <div class="top-info">ANTALYA</div>
    
    <!-- Doğum Günü Canlı Sayaç Alanı -->
    <div class="countdown-note" id="birthday-countdown">Hesaplanıyor...</div>
    
    <!-- Saat ve Tarih -->
    <div class="time" id="clock">00:00:00</div>
    <div class="date" id="date">01 OCAK 2026</div>
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

    // --- SAAT, TARİH VE DOĞUM GÜNÜ GERİ SAYIM SİSTEMİ ---
    function updateClock() {
      const now = new Date();
      
      // 1. Canlı Saat Güncellemesi
      const hours = String(now.getHours()).padStart(2, '0');
      const minutes = String(now.getMinutes()).padStart(2, '0');
      const seconds = String(now.getSeconds()).padStart(2, '0');
      document.getElementById('clock').textContent = `${hours}:${minutes}:${seconds}`;

      // 2. Güncel Tarih Güncellemesi
      const options = { year: 'numeric', month: 'long', day: 'numeric' };
      document.getElementById('date').textContent = now.toLocaleDateString('tr-TR', options);

      // 3. Doğum Günü Canlı Hesaplayıcı (28 Kasım)
      const doğumGünüGünü = 28;
      const doğumGünüAyi = 10; // Kasım ayı (Yazılımda Ocak=0, Kasım=10'dur)
      const doğumSaati = 10;
      const doğumDakikası = 7;

      // Hedef yılı belirliyoruz (Bu yıl geçtiyse sonraki yıla kurar)
      let hedefYıl = now.getFullYear();
      let bdayTarget = new Date(hedefYıl, doğumGünüAyi, doğumGünüGünü, doğumSaati, doğumDakikası, 0);
      
      if (now > bdayTarget) {
        hedefYıl += 1;
        bdayTarget = new Date(hedefYıl, doğumGünüAyi, doğumGünüGünü, doğumSaati, doğumDakikası, 0);
      }

      const countdownEl = document.getElementById('birthday-countdown');
      
      // Tam doğum günü ve saatinde kutlama yap
      if (now.getDate() === doğumGünüGünü && now.getMonth() === doğumGünüAyi) {
        if (now.getHours() === doğumSaati && now.getMinutes() === doğumDakikası) {
          countdownEl.textContent = "🎉 İYİ Kİ DOĞDUN! TAM ŞU AN DOĞDUN! 🎂🎈";
          return;
        } else {
          countdownEl.textContent = "🎂 BUGÜN SENİN DOĞUM GÜNÜN! MUTLU YILLAR! 🎉";
          return;
        }
      }

      // Kalan süreyi hesapla
      const fark = bdayTarget - now;
      
      const kalanGün = Math.floor(fark / (1000 * 60 * 60 * 24));
      const kalanSaat = Math.floor((fark % (1000 * 60 * 60 * 24)) / (1000 * 60 * 60));
      const kalanDakika = Math.floor((fark % (1000 * 60 * 60)) / (1000 * 60));
      const kalanSaniye = Math.floor((fark % (1000 * 60)) / 1000);

      countdownEl.textContent = `⏳ DOĞUM GÜNÜNE: ${kalanGün} GÜN, ${kalanSaat} SAAT, ${kalanDakika} DAKİKA, ${kalanSaniye} SANİYE KALDI`;
    }

    setInterval(updateClock, 1000);
    updateClock();
  </script>
</body>
</html>
