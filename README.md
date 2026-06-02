<!DOCTYPE html>
<html lang="tr">
<head>
  <meta charset="UTF-8">
  <meta name="viewport" content="width=device-width, initial-scale=1.0">
  <title>Minimalist Dijital Saat</title>
  <link rel="preconnect" href="https://fonts.googleapis.com">
  <link rel="preconnect" href="https://fonts.gstatic.com" crossorigin>
  <link href="https://fonts.googleapis.com/css2?family=Inter:wght@300;400;600&display=swap" rel="stylesheet">

  <style>
    body, html {
      margin: 0;
      padding: 0;
      overflow: hidden;
      background-color: #0d0d11; /* Koyu, mat bir gece tonu */
      font-family: 'Inter', sans-serif;
    }

    canvas {
      position: absolute;
      top: 0;
      left: 0;
      z-index: 1;
      opacity: 0.6; /* Yıldızları arkada daha soft ve pürüzsüz yapmak için hafif şeffaflık */
    }

    /* Minimalist Saat Alanı */
    .clock-container {
      position: absolute;
      top: 50%;
      left: 50%;
      transform: translate(-50%, -50%);
      z-index: 2;
      text-align: center;
      user-select: none;
    }

    .time {
      font-size: 6rem;
      font-weight: 300; /* İnce, zarif yazı stili */
      color: #ffffff;    /* Saf beyaz */
      letter-spacing: -2px; /* Harfleri birbirine biraz daha yaklaştırarak modern bir hava kattık */
      line-height: 1;
    }

    .date {
      font-size: 1.1rem;
      font-weight: 400;
      color: #8a8a93; /* Gri tonlarında, saati gölgelemeyen zarif bir renk */
      letter-spacing: 2px;
      margin-top: 15px;
      text-transform: uppercase;
    }
  </style>
</head>
<body>

  <canvas id="starfield"></canvas>

  <div class="clock-container">
    <div class="time" id="clock">00:00:00</div>
    <div class="date" id="date">01 OCAK 2026</div>
  </div>

  <script>
    // --- CANVAS ARKA PLAN (SADE BEYAZ YILDIZLAR) ---
    const canvas = document.getElementById('starfield');
    const ctx = canvas.getContext('2d');
    const stars = [];

    function resize() {
      canvas.width = window.innerWidth;
      canvas.height = window.innerHeight;
    }
    window.addEventListener('resize', resize);
    resize();

    for (let i = 0; i < 120; i++) { // Yıldız sayısını biraz azaltarak gözü daha az yoran bir düzen kurduk
      stars.push({
        x: Math.random() * canvas.width,
        y: Math.random() * canvas.height,
        radius: Math.random() * 1.2,
        speed: Math.random() * 0.2 + 0.05 // Akış hızını da biraz yavaşlattık, daha sakin durması için
      });
    }

    function animateStars() {
      ctx.clearRect(0, 0, canvas.width, canvas.height);
      ctx.fillStyle = '#ffffff'; // Yıldızlar artık saf beyaz
      
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

    // --- SAAT FONKSİYONU ---
    function updateClock() {
      const now = new Date();
      
      const hours = String(now.getHours()).padStart(2, '0');
      const minutes = String(now.getMinutes()).padStart(2, '0');
      const seconds = String(now.getSeconds()).padStart(2, '0');
      
      document.getElementById('clock').textContent = `${hours}:${minutes}:${seconds}`;

      const options = { year: 'numeric', month: 'long', day: 'numeric' };
      document.getElementById('date').textContent = now.toLocaleDateString('tr-TR', options);
    }

    setInterval(updateClock, 1000);
    updateClock();
  </script>
</body>
</html>
