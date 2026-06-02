<!DOCTYPE html>
<html lang="tr">
<head>
  <meta charset="UTF-8">
  <meta name="viewport" content="width=device-width, initial-scale=1.0">
  <title>Antalya - Geçen Ömür Sayacı</title>
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
      width: 90%; /* Mobil ekranlarda taşma yapmaması için genişlik */
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

    /* Bir sonraki doğum gününe kalan süre (Üstteki küçük yazı) */
    .countdown-note {
      font-size: 0.85rem;
      font-weight: 400;
      color: #ffb703;
      letter-spacing: 1px;
      margin-bottom: 25px;
    }

    /* Yeşille Çizdiğin Büyük Alan: Doğumdan Bugüne Geçen Canlı Süre */
    .time {
      font-size: 2.2rem; /* Çok fazla metin sığacağı için yazı boyutunu mobilde sığacak şekilde optimize ettim */
      font-weight: 400;
      color: #ffffff;
      letter-spacing: 0px;
      line-height: 1.4;
      margin-bottom: 20px;
    }

    .time span {
      font-weight: 600;
      color: #00ffe0; /* Sayıları daha rahat okumak için hafif neon mavi tonu */
    }

    /* Sarı ile Çizdiğin Alt Kısım: Doğum Tarihin */
    .date {
      font-size: 1.2rem;
      font-weight: 400;
      color: #8a8a93;
      letter-spacing: 1px;
    }
  </style>
</head>
<body>

  <canvas id="starfield"></canvas>

  <div class="clock-container">
    <!-- Antalya Başlığı -->
    <div class="top-info">ANTALYA</div>
    
    <!-- Gelecek Doğum Gününe Kalan Süre -->
    <div class="countdown-note" id="birthday-countdown">Hesaplanıyor...</div>
    
    <!-- YEŞİL ALAN: Doğum Gününden Beri Geçen Toplam Zaman (Canlı) -->
    <div class="time" id="passed-time">Hesaplanıyor...</div>
    
    <!-- SARI ALAN: Doğum Tarihin ve Saatin (Sabit) -->
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

    // --- DOĞUM GÜNÜ HESAPLAMA MOTORU ---
    const dogumTarihi = new Date(2011, 10, 28, 10, 7, 0); // 28 Kasım 2011 10:07 (Yazılımda Kasım = 10. ay)

    function updateCounter() {
      const now = new Date();
      
      // --- 1. ÜST ALAN: Gelecek Doğum Gününe Kalan Süre Kontrolü ---
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
        const kalanFark = airspaceCount(now, sonrakiBday);
        countdownEl.textContent = `⏳ DOĞUM GÜNÜNE: ${kalanFark.gun} GÜN, ${kalanFark.saat} SAAT, ${kalanFark.dakika} DAKİKA, ${kalanFark.saniye} SANİYE KALDI`;
      }

      // --- 2. YEŞİL ALAN: Doğumdan Bugüne Geçen Detaylı Süre Hesabı ---
      let gecenYil = now.getFullYear() - dogumTarihi.getFullYear();
      let gecenAy = now.getMonth() - dogumTarihi.getMonth();
      let gecenGun = now.getDate() - dogumTarihi.getDate();
      let gecenSaat = now.getHours() - dogumTarihi.getHours();
      let gecenDakika = now.getMinutes() - dogumTarihi.getMinutes();
      let gecenSaniye = now.getSeconds() - dogumTarihi.getSeconds();

      // Negatif değerleri düzeltme (Zaman kaymaları için matematiksel dengeleme)
      if (gecenSaniye < 0) { gecenSaniye += 60; gecenDakika--; }
      if (gecenDakika < 0) { gecenDakika += 60; gecenSaat--; }
      if (gecenSaat < 0) { gecenSaat += 24; gecenGun--; }
      if (gecenGun < 0) {
        const oncekiAy = new Date(now.getFullYear(), now.getMonth(), 0).getDate();
        gecenGun += oncekiAy;
        gecenAy--;
      }
      if (gecenAy < 0) { gecenAy += 12; gecenYil--; }

      // Ekrana formatlı şekilde yazdırma
      document.getElementById('passed-time').innerHTML = 
        `<span>${gecenYil}</span> Yıl, <span>${gecenAy}</span> Ay, <span>${gecenGun}</span> Gün<br>` +
        `<span>${gecenSaat}</span> Saat, <span>${gecenDakika}</span> Dakika, <span>${gecenSaniye}</span> Saniye`;
    }

    // Basit fark hesaplama yardımcı fonksiyonu
    function airspaceCount(start, end) {
      const diff = end - start;
      return {
        gun: Math.floor(diff / (1000 * 60 * 60 * 24)),
        saat: Math.floor((diff % (1000 * 60 * 60 * 24)) / (1000 * 60 * 60)),
        dakika: Math.floor((diff % (1000 * 60 * 60)) / (1000 * 60)),
        saniye: Math.floor((diff % (1000 * 60)) / 1000)
      };
    }

    setInterval(updateCounter, 1000);
    updateCounter();
  </script>
</body>
</html>
