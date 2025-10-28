<!DOCTYPE html><html lang="tr">
<head>
<meta charset="UTF-8" />
<meta name="viewport" content="width=device-width, initial-scale=1.0" />
<title>Doğum Sayacı</title>
<style>
  body {
    font-family: 'Arial', sans-serif;
    background-color: #0a0a0a;
    color: #00ffcc;
    text-align: center;
    margin-top: 100px;
  }
  h1 {
    font-size: 2.5rem;
  }
  .time {
    font-size: 1.8rem;
    margin-top: 20px;
  }
</style>
</head>
<body>
  <h1>Benim Doğumumdan Beri Geçen Süre</h1>
  <div class="time" id="counter"></div>  <script>
    const birthDate = new Date('2011-11-28T10:07:00'); // 28 Kasım 2011 saat 10:07

    function updateCounter() {
      const now = new Date();
      const diff = now - birthDate;

      const days = Math.floor(diff / (1000 * 60 * 60 * 24));
      const hours = Math.floor((diff / (1000 * 60 * 60)) % 24);
      const minutes = Math.floor((diff / (1000 * 60)) % 60);
      const seconds = Math.floor((diff / 1000) % 60);

      document.getElementById('counter').innerText =
        `${days} gün, ${hours} saat, ${minutes} dakika, ${seconds} saniye`;
    }

    setInterval(updateCounter, 1000);
    updateCounter();
  </script></body>
</html>
