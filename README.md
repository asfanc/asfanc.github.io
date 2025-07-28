<!DOCTYPE html><html lang="en">
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
  </div>  <div id="score">Puan: 0</div>
  <div id="lives">Can: 3</div><canvas id="game" width="400" height="600"></canvas>

  <div id="controls">
    <div class="btn" id="leftBtn">⬅️</div>
    <div class="btn" id="rightBtn">➡️</div>
  </div>  <script>
    const canvas = document.getElementById('game');
    const ctx = canvas.getContext('2d');

    let playerX = 170;
    let speed = 4;
    let enemies = [];
    let gameInterval;
    let score = 0;
    let lives = 3;

    function drawBackground() {
      ctx.fillStyle = '#111';
      ctx.fillRect(0, 0, canvas.width, canvas.height);
    }

    function drawPlayer() {
      ctx.fillStyle = '#ff3333';
      ctx.fillRect(playerX, 500, 60, 100);
    }

    function drawEnemies() {
      ctx.fillStyle = '#00ccff';
      for (let i = 0; i < enemies.length; i++) {
        enemies[i].y += speed;
        ctx.fillRect(enemies[i].x, enemies[i].y, 60, 100);

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
        } else if (enemies[i] && enemies[i].y > 600) {
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
      ctx.clearRect(0, 0, canvas.width, canvas.height);
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

    document.getElementById('startBtn').onclick = () => {
      document.getElementById('menu').style.display = 'none';
      gameInterval = setInterval(() => {
        gameLoop();
        if (Math.random() < 0.03) spawnEnemy();
      }, 30);
    };
  </script></body>
</html>
    
