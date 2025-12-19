<!DOCTYPE html>
<html lang="tr">
<head>
<meta charset="UTF-8" />
<meta name="viewport" content="width=device-width, initial-scale=1.0" />
<title>28 Kasım 2011 - 10:07</title>
<style>
  body {
    font-family: 'Poppins', sans-serif;
    background: radial-gradient(circle at bottom, #000010, #000);
    color: #00ffe0;
    text-align: center;
    display: flex;
    flex-direction: column;
    align-items: center;
    justify-content: center;
    min-height: 100vh;
    margin: 0;
    overflow: hidden;
  }
  canvas#stars {
    position: fixed;
    top: 0;
    left: 0;
    width: 100%;
    height: 100%;
    z-index: -1;
  }
  .title {
    font-size: 3rem;
    font-weight: 700;
    letter-spacing: 4px;
    margin-bottom: 1rem;
  }
  .title span {
    text-shadow: 0 0 8px currentColor, 0 0 16px currentColor;
    animation: glowColor 2s ease-in-out infinite alternate;
  }
  h1 {
    font-size: 2.4rem;
    margin-bottom: 0.5rem;
    letter-spacing: 1px;
    color: #00ffe0;
    text-shadow: 0 0 10px #00ffe0, 0 0 20px #00ffff, 0 0 30px #00ffe0;
    animation: glow 2s ease-in-out infinite alternate;
  }
  h2 {
    font-size: 1.2rem;
    color: #ccc;
    margin-top: 0;
    margin-bottom: 2rem;
  }
  .time {
    font-size: 2rem;
    line-height: 1.6;
    letter-spacing: 0.5px;
    white-space: pre-line;
    color: #b8fff5;
    text-shadow: 0 0 5px #00ffe0, 0 0 10px #00ffff;
    animation: glowText 2s ease-in-out infinite alternate;
  }
  @keyframes glow {
    from { text-shadow: 0 0 10px #00ffe0, 0 0 20px #00ffff; }
    to { text-shadow: 0 0 25px #00ffff, 0 0 40px #00ffe0; }
  }
  @keyframes glowText {
    from { opacity: 0.8; }
    to { opacity: 1; }
  }
  @keyframes glowColor {
    from { filter: brightness(0.8); }
    to { filter: brightness(1.3); }
  }
</style>
</head>
<body>
  <canvas id="stars"></canvas>
  <div class="title">
    <span style="color:#ff0000">A</span>
    <span style="color:#ffffff">N</span>
    <span style="color:#ff0000">T</span>
    <span style="color:#ffffff">A</span>
    <span style="color:#ff0000">L</span>
    <span style="color:#ffffff">Y</span>
    <span style="color:#ff0000">A</span>
  </div>
  <h1>Benim Doğumumdan Beri Geçen Süre</h1>
  <h2>Doğum Tarihim: 28 Kasım 2011 - Saat 10:07</h2>
  <div class="time" id="counter"></div>

  <script>
    const birthDate = new Date('2011-11-28T10:07:00'); // 28 Kasım 2011 10:07

    function updateCounter() {
      const now = new Date();
      const diff = now - birthDate;

      const days = Math.floor(diff / (1000 * 60 * 60 * 24));
      const hours = Math.floor((diff / (1000 * 60 * 60)) % 24);
      const minutes = Math.floor((diff / (1000 * 60)) % 60);
      const seconds = Math.floor((diff / 1000) % 60);

      document.getElementById('counter').innerText =
        `${days} gün\n${hours} saat\n${minutes} dakika\n${seconds} saniye`;
    }

    setInterval(updateCounter, 1000);
    updateCounter();

    // Yıldız animasyonu
    const canvas = document.getElementById('stars');
    const ctx = canvas.getContext('2d');
    let stars = [];

    function resize() {
      canvas.width = window.innerWidth;
      canvas.height = window.innerHeight;
    }

    window.addEventListener('resize', resize);
    resize();

    for (let i = 0; i < 200; i++) {
      stars.push({
        x: Math.random() * canvas.width,
        y: Math.random() * canvas.height,
        radius: Math.random() * 1.5,
        speed: Math.random() * 0.3 + 0.05
      });
    }

    function animateStars() {
      ctx.clearRect(0, 0, canvas.width, canvas.height);
      ctx.fillStyle = '#00ffe0';
      for (let s of stars) {
        ctx.beginPath();
        ctx.arc(s.x, s.y, s.radius, 0, Math.PI * 2);
        ctx.fill();
        s.y += s.speed;
        if (s.y > canvas.height) s.y = 0;
      }
      requestAnimationFrame(animateStars);
    }

    animateStars();
  </script>
</body>
</html>


Tamam! 🌟 Artık sayfanın en üstünde ANTALYA yazısı, bir kırmızı bir beyaz renkte harflerle parlayan şekilde görünüyor.
İstersen bu yazıya devirme (dönme) animasyonu veya yanıp sönme efekti de ekleyebilirim. Hangisini istersin?
