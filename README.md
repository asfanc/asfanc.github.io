<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8" />
  <meta name="viewport" content="width=device-width, initial-scale=1.0" />
  <title>Nissan GTR R33: Tokyo Drift</title>
  <style>
    * { box-sizing: border-box; margin: 0; padding: 0; }
    body { background: #000; overflow: hidden; font-family: sans-serif; }
    canvas { display: block; margin: 0 auto; background: black; }
    #menu {
      position: absolute; top: 0; left: 0; width: 100%; height: 100%;
      background: rgba(0,0,0,0.8); color: white; display: flex;
      flex-direction: column; align-items: center; justify-content: center;
    }
    #menu h1 { font-size: 3em; margin-bottom: 20px; color: #ff4c4c; }
    #startBtn {
      background: #ff4c4c; color: white; padding: 15px 30px;
      border: none; font-size: 1.5em; border-radius: 10px; cursor: pointer;
    }
    #controls {
      position: absolute; bottom: 20px; width: 100%; display: flex; justify-content: space-between; padding: 0 40px;
    }
    .btn {
      width: 70px; height: 70px; background: rgba(255,255,255,0.3);
      border-radius: 50%; display: flex; align-items: center; justify-content: center;
      font-size: 2em; color: white; user-select: none;
    }
    #score, #lives {
      position: absolute; top: 10px; font-size: 1.2em; color: white; padding: 10px;
    }
    #score { left: 10px; }
    #lives { right: 10px; }
  </style>
</head>
<body>
  <div id="menu">
    <h1>Tokyo Drift</h1>
    <button id="startBtn">Oyunu Başlat</button>
  </div>

  <div id="score">Puan: 0</div>
  <div id="lives">Can: 3</div>

  <canvas id="game" width="400" height="600"></canvas>
  <div id="controls">
    <div class="btn" id="leftBtn">⬅️</div>
    <div class="btn" id="rightBtn">➡️</div>
  </div>

  <script>
    const canvas = document.getElementById('game');
    const ctx = canvas.getContext('2d');

    let playerX = 170;
    let speed = 4;
    let enemies = [];
    let gameInterval;
    let score = 0;
    let lives = 3;

    const backgroundImg = new Image();
    backgroundImg.src = ![desktop-wallpaper-japan-nightlife-japan-night-street-iphone](https://github.com/user-attachments/assets/cae626ca-9b65-42cd-8d60-215786fa0d60)
; // Japon sokak arka planı

    const playerImg = new Image();
    playerImg.src = <img width="1024" height="1536" alt="file_000000005f64620a8c0e86bc0e170a4b" src="https://github.com/user-attachments/assets/064ccd88-7b85-46fa-ba0d-365b507c4e22" />
; // Nissan GTR R33 görünümü (desenli)

    const enemyImg = new Image();
    enemyImg.src = ![images (2)](https://github.com/user-attachments/assets/9592a99f-f791-43a6-acd5-0eec4feaa0cd)
; // Japon tarzı düşman arabası

    let imagesLoaded = 0;
    const totalImages = 3;

    function imageLoaded() {
      imagesLoaded++;
      if (imagesLoaded === totalImages) {
        startGame();
      }
    }

    backgroundImg.onload = imageLoaded;
    playerImg.onload = imageLoaded;
    enemyImg.onload = imageLoaded;

    function drawBackground() {
      ctx.drawImage(backgroundImg, 0, 0, 400, 600);
    }

    function drawPlayer() {
      ctx.drawImage(playerImg, playerX, 500, 60, 100);
    }

    function drawEnemies() {
      for (let i = 0; i < enemies.length; i++) {
        enemies[i].y += speed;
        ctx.drawImage(enemyImg, enemies[i].x, enemies[i].y, 60, 100);

        if (
          enemies[i].y + 100 > 500 &&
          enemies[i].y < 600 &&
          enemies[i].x < playerX + 60 &&
          enemies[i].x + 60 > playerX
        ) {
          enemies.splice(i, 1);
          i--;
          lives--;
          document.getElementById('lives').innerText = 'Can: ' + lives;
          if (lives <= 0) {
            clearInterval(gameInterval);
            alert('Oyun Bitti! Skor: ' + score);
            location.reload();
          }
        }

        else if (enemies[i] && enemies[i].y > 600) {
          enemies.splice(i, 1);
          i--;
          score++;
          document.getElementById('score').innerText = 'Puan: ' + score;
        }
      }
    }

    function spawnEnemy() {
      const x = Math.floor(Math.random() * 6) * 60;
      enemies.push({ x: x, y: -100 });
    }

    function gameLoop() {
      ctx.clearRect(0, 0, 400, 600);
      drawBackground();
      drawPlayer();
      drawEnemies();
    }

    document.getElementById('leftBtn').addEventListener('touchstart', () => {
      if (playerX > 0) playerX -= 30;
    });
    document.getElementById('rightBtn').addEventListener('touchstart', () => {
      if (playerX < 340) playerX += 30;
    });

    function startGame() {
      document.getElementById('menu').style.display = 'none';
      gameInterval = setInterval(() => {
        gameLoop();
        if (Math.random() < 0.03) spawnEnemy();
      }, 30);
    }

    document.getElementById('startBtn').onclick = () => {
      if (imagesLoaded === totalImages) {
        startGame();
      } else {
        alert('Görseller yükleniyor, lütfen bekle...');
      }
    };
  </script>
</body>
</html>






