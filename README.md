<!-- car-game.html -->
<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8" />
  <meta name="viewport" content="width=device-width, initial-scale=1.0"/>
  <title>Araba Kaçış Oyunu</title>
  <style>
    body {
      margin: 0;
      overflow: hidden;
      font-family: sans-serif;
      background: #333;
    }
    #game {
      width: 400px;
      height: 600px;
      margin: 20px auto;
      position: relative;
      background: #111;
      border: 4px solid #fff;
      overflow: hidden;
    }
    .car {
      width: 50px;
      height: 100px;
      background: red;
      position: absolute;
      bottom: 10px;
      left: 175px;
      border-radius: 10px;
    }
    .obstacle {
      width: 50px;
      height: 100px;
      background: gray;
      position: absolute;
      top: -120px;
      left: 0;
      border-radius: 10px;
    }
    #score {
      color: white;
      text-align: center;
      font-size: 24px;
      margin-top: 10px;
    }
  </style>
</head>
<body>
  <div id="score">Skor: 0</div>
  <div id="game">
    <div class="car" id="car"></div>
  </div>

  <script>
    const game = document.getElementById("game");
    const car = document.getElementById("car");
    let score = 0;
    let left = 175;

    // Araba kontrolü
    document.addEventListener("keydown", (e) => {
      if (e.key === "ArrowLeft" && left > 0) left -= 25;
      if (e.key === "ArrowRight" && left < 350) left += 25;
      car.style.left = left + "px";
    });

    // Mobil dokunma desteği (sağ ve sol ekran dokunuşu)
    document.addEventListener("touchstart", (e) => {
      const x = e.touches[0].clientX;
      if (x < window.innerWidth / 2 && left > 0) left -= 25;
      if (x >= window.innerWidth / 2 && left < 350) left += 25;
      car.style.left = left + "px";
    });

    // Engel oluştur ve hareket ettir
    function createObstacle() {
      const obs = document.createElement("div");
      obs.classList.add("obstacle");
      obs.style.left = Math.floor(Math.random() * 8) * 50 + "px";
      game.appendChild(obs);

      let top = -100;
      const move = setInterval(() => {
        top += 5;
        obs.style.top = top + "px";

        // Çarpışma kontrol
        if (
          top > 480 &&
          top < 600 &&
          Math.abs(parseInt(obs.style.left) - left) < 50
        ) {
          clearInterval(move);
          alert("Oyun Bitti! Skor: " + score);
          window.location.reload();
        }

        if (top > 600) {
          clearInterval(move);
          obs.remove();
        }
      }, 20);
    }

    // Engel üretme ve skor artırma döngüsü
    setInterval(() => {
      createObstacle();
    }, 1000);

    setInterval(() => {
      score++;
      document.getElementById("score").textContent = "Skor: " + score;
    }, 1000);
  </script>
</body>
</html><!-- car-game.html -->
<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8" />
  <meta name="viewport" content="width=device-width, initial-scale=1.0"/>
  <title>Araba Kaçış Oyunu</title>
  <style>
    body {
      margin: 0;
      overflow: hidden;
      font-family: sans-serif;
      background: #333;
    }
    #game {
      width: 400px;
      height: 600px;
      margin: 20px auto;
      position: relative;
      background: #111;
      border: 4px solid #fff;
      overflow: hidden;
    }
    .car {
      width: 50px;
      height: 100px;
      background: red;
      position: absolute;
      bottom: 10px;
      left: 175px;
      border-radius: 10px;
    }
    .obstacle {
      width: 50px;
      height: 100px;
      background: gray;
      position: absolute;
      top: -120px;
      left: 0;
      border-radius: 10px;
    }
    #score {
      color: white;
      text-align: center;
      font-size: 24px;
      margin-top: 10px;
    }
  </style>
</head>
<body>
  <div id="score">Skor: 0</div>
  <div id="game">
    <div class="car" id="car"></div>
  </div>

  <script>
    const game = document.getElementById("game");
    const car = document.getElementById("car");
    let score = 0;
    let left = 175;

    // Araba kontrolü
    document.addEventListener("keydown", (e) => {
      if (e.key === "ArrowLeft" && left > 0) left -= 25;
      if (e.key === "ArrowRight" && left < 350) left += 25;
      car.style.left = left + "px";
    });

    // Mobil dokunma desteği (sağ ve sol ekran dokunuşu)
    document.addEventListener("touchstart", (e) => {
      const x = e.touches[0].clientX;
      if (x < window.innerWidth / 2 && left > 0) left -= 25;
      if (x >= window.innerWidth / 2 && left < 350) left += 25;
      car.style.left = left + "px";
    });

    // Engel oluştur ve hareket ettir
    function createObstacle() {
      const obs = document.createElement("div");
      obs.classList.add("obstacle");
      obs.style.left = Math.floor(Math.random() * 8) * 50 + "px";
      game.appendChild(obs);

      let top = -100;
      const move = setInterval(() => {
        top += 5;
        obs.style.top = top + "px";

        // Çarpışma kontrol
        if (
          top > 480 &&
          top < 600 &&
          Math.abs(parseInt(obs.style.left) - left) < 50
        ) {
          clearInterval(move);
          alert("Oyun Bitti! Skor: " + score);
          window.location.reload();
        }

        if (top > 600) {
          clearInterval(move);
          obs.remove();
        }
      }, 20);
    }

    // Engel üretme ve skor artırma döngüsü
    setInterval(() => {
      createObstacle();
    }, 1000);

    setInterval(() => {
      score++;
      <!-- car-game.html -->
<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8" />
  <meta name="viewport" content="width=device-width, initial-scale=1.0"/>
  <title>Araba Kaçış Oyunu</title>
  <style>
    body {
      margin: 0;
      overflow: hidden;
      font-family: sans-serif;
      background: #333;
    }
    #game {
      width: 400px;
      height: 600px;
      margin: 20px auto;
      position: relative;
      background: #111;
      border: 4px solid #fff;
      overflow: hidden;
    }
    .car {
      width: 50px;
      height: 100px;
      background: red;
      position: absolute;
      bottom: 10px;
      left: 175px;
      border-radius: 10px;
    }
    .obstacle {
      width: 50px;
      height: 100px;
      background: gray;
      position: absolute;
      top: -120px;
      left: 0;
      border-radius: 10px;
    }
    #score {
      color: white;
      text-align: center;
      font-size: 24px;
      margin-top: 10px;
    }
  </style>
</head>
<body>
  <div id="score">Skor: 0</div>
  <div id="game">
    <div class="car" id="car"></div>
  </div>

  <script>
    const game = document.getElementById("game");
    const car = document.getElementById("car");
    let score = 0;
    let left = 175;

    // Araba kontrolü
    document.addEventListener("keydown", (e) => {
      if (e.key === "ArrowLeft" && left > 0) left -= 25;
      if (e.key === "ArrowRight" && left < 350) left += 25;
      car.style.left = left + "px";
    });

    // Mobil dokunma desteği (sağ ve sol ekran dokunuşu)
    document.addEventListener("touchstart", (e) => {
      const x = e.touches[0].clientX;
      if (x < window.innerWidth / 2 && left > 0) left -= 25;
      if (x >= window.innerWidth / 2 && left < 350) left += 25;
      car.style.left = left + "px";
    });

    // Engel oluştur ve hareket ettir
    function createObstacle() {
      const obs = document.createElement("div");
      obs.classList.add("obstacle");
      obs.style.left = Math.floor(Math.random() * 8) * 50 + "px";
      game.appendChild(obs);

      let top = -100;
      const move = setInterval(() => {
        top += 5;
        obs.style.top = top + "px";

        // Çarpışma kontrol
        if (
          top > 480 &&
          top < 600 &&
          Math.abs(parseInt(obs.style.left) - left) < 50
        ) {
          clearInterval(move);
          alert("Oyun Bitti! Skor: " + score);
          window.location.reload();
        }

        if (top > 600) {
          clearInterval(move);
          obs.remove();
        }
      }, 20);
    }

    // Engel üretme ve skor artırma döngüsü
    setInterval(() => {
      createObstacle();
    }, 1000);

    setInterval(() => {
      score++;
      document.getElementById("score").textContent = "Skor: " + score;
    }, 1000);
  </script>
</body>
</html>document.getElementById("score").textContent = "Skor: " + score;
    }, 1000);
  </script>
</body>
</html>)

