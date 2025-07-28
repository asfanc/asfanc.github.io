<!DOCTYPE html>
<html lang="tr">
<head>
<meta charset="UTF-8" />
<meta name="viewport" content="width=device-width, initial-scale=1" />
<title>Nissan GTR R33 Tarzı Araba Koşu Oyunu</title>
<style>
  body, html { margin: 0; padding: 0; overflow: hidden; background: linear-gradient(to top, #0f2027, #203a43, #2c5364); font-family: 'Segoe UI', Tahoma, Geneva, Verdana, sans-serif; color: #eee; }
  #gameCanvas { background: #1c1c1c; display: block; margin: 20px auto; border-radius: 15px; box-shadow: 0 0 25px rgba(0,0,0,0.9); }

  #score, #lives {
    position: fixed; top: 15px; font-size: 22px; font-weight: 700; color: #f7b733;
    text-shadow: 1px 1px 3px #000;
    user-select: none;
    z-index: 10;
  }
  #score { left: 25px; }
  #lives { right: 25px; }

  #menu, #gameOverScreen {
    position: absolute; top: 50%; left: 50%; transform: translate(-50%, -50%);
    background: rgba(20, 20, 20, 0.95);
    border-radius: 15px;
    width: 320px;
    padding: 30px 20px;
    box-shadow: 0 8px 20px rgba(255, 255, 255, 0.1);
    text-align: center;
    display: none;
    z-index: 20;
  }
  #menu h1, #gameOverScreen h2 {
    font-weight: 700;
    margin-bottom: 15px;
    font-size: 32px;
    text-shadow: 1px 1px 6px #ff4c60;
  }
  #menu p {
    font-size: 16px;
    margin-bottom: 25px;
    color: #ccc;
  }
  button {
    padding: 15px 30px;
    font-size: 20px;
    font-weight: 600;
    color: white;
    background: linear-gradient(45deg, #ff4c60, #ff784e);
    border: none;
    border-radius: 10px;
    cursor: pointer;
    transition: all 0.3s ease;
    box-shadow: 0 5px 15px rgba(255, 76, 96, 0.5);
  }
  button:hover {
    background: linear-gradient(45deg, #ff784e, #ff4c60);
    box-shadow: 0 8px 25px rgba(255, 120, 78, 0.8);
  }

  /* Sağ ve Sol butonları ekran altına yerleştir */
  #controls {
    position: fixed;
    bottom: 20px;
    left: 50%;
    transform: translateX(-50%);
    width: 300px;
    display: flex;
    justify-content: space-between;
    z-index: 15;
  }
  #controls button {
    width: 140px;
    background: linear-gradient(45deg, #3498db, #2980b9);
    box-shadow: 0 5px 15px rgba(41, 128, 185, 0.7);
  }
  #controls button:active {
    background: linear-gradient(45deg, #2980b9, #3498db);
  }
</style>
</head>
<body>

<div id="menu">
  <h1>Araba Koşu Oyunu</h1>
  <p>Nissan GTR R33 tarzı arabanla engellerden kaç!</p>
  <button id="startBtn">Başlat</button>
</div>

<div id="gameOverScreen">
  <h2>Oyun Bitti!</h2>
  <p>Toplam Puanın:</p>
  <p style="font-size: 28px; font-weight: 800; color: #ff4c60;" id="finalScore">0</p>
  <button id="restartBtn">Tekrar Oyna</button>
</div>

<canvas id="gameCanvas" width="400" height="600"></canvas>
<div id="score">Puan: 0</div>
<div id="lives">Can: 3</div>

<div id="controls">
  <button id="leftBtn">Sol</button>
  <button id="rightBtn">Sağ</button>
</div>

<script>
  const canvas = document.getElementById('gameCanvas');
  const ctx = canvas.getContext('2d');

  const lanes = [70, 170, 270];
  let carLane = 1;

  let obstacles = [];
  let score = 0;
  let lives = 3;
  let gameRunning = false;
  let obstacleSpeed = 4;

  const carWidth = 70;
  const carHeight = 120;

  // Nissan GTR R33 benzeri araba çizimi (basit stilize)
  // Player car base64 PNG (stilize GTR33 kırmızı)
  const playerCarImg = new Image();
  playerCarImg.src = 'data:image/png;base64,iVBORw0KGgoAAAANSUhEUgAAAEwAAABJCAYAAACxD9VEAAAACXBIWXMAAAsTAAALEwEAmpwYAAAAg0lEQVR4nO3RsQ2AMAwEwPHy/09C1j3FYv9DsAS2Gb5cybMXc0gC0bRq9P2AgAAAAAAAAAAAACAzp/AxOW+Gk5/AqL8L16ocX0HR+jH8zKj1P0eDd17yAgAAAAAAAAAAgEPGqNY2o9LQBAAAAAAAAAAAL8pDAAAAAAAAAAAANdDqCgEvl5Uq6/9vgIYB7VkBQAAAAAAAACASwMT7DaQzvSAAAAAElFTkSuQmCC';

  // Engel arabası (gri stilize GTR33)
  const obstacleCarImg = new Image();
  obstacleCarImg.src = 'data:image/png;base64,iVBORw0KGgoAAAANSUhEUgAAAEwAAABJCAYAAACxD9VEAAAACXBIWXMAAAsTAAALEwEAmpwYAAAAgklEQVR4nO3RoQkAMAwE0N3/6Ua+xTjHLMN8iX0HdNlCQAAAAAAAAAA4Hn8M1r+OQvwoOPwsOmjwCo/C4ehxfQdH6Mf7MoPU/R4N3XvICAgAAAAAAAAAA8AcVKQaq/TTKHNfwIBAAAAAAAAAL/8AwAAAAAAAAAAAC0cBtkAAAAAAAAAAAAAgM/X9AIEgBW9BgYofvZ+eDAAAAABJRU5ErkJggg==';

  function drawCar(x, y) {
    ctx.drawImage(playerCarImg, x, y, carWidth, carHeight);
  }

  function drawObstacle(x, y) {
    ctx.drawImage(obstacleCarImg, x, y, carWidth, carHeight);
  }

  const menu = document.getElementById('menu');
  const gameOverScreen = document.getElementById('gameOverScreen');
  const startBtn = document.getElementById('startBtn');
  const restartBtn = document.getElementById('restartBtn');
  const finalScore = document.getElementById('finalScore');
  const scoreDisplay = document.getElementById('score');
  const livesDisplay = document.getElementById('lives');

  function resetGame() {
    carLane = 1;
    obstacles = [];
    score = 0;
    lives = 3;
    obstacleSpeed = 4;
    scoreDisplay.textContent = 'Puan: 0';
    livesDisplay.textContent = 'Can: 3';
  }

  function startGame() {
    resetGame();
    menu.style.display = 'none';
    gameOverScreen.style.display = 'none';
    gameRunning = true;
    requestAnimationFrame(gameLoop);
  }

  function endGame() {
    gameRunning = false;
    finalScore.textContent = score;
    gameOverScreen.style.display = 'block';
  }

  function createObstacle() {
    const laneIndex = Math.floor(Math.random() * lanes.length);
    obstacles.push({
      x: lanes[laneIndex],
      y: -carHeight,
    });
  }

  let obstacleTimer = 0;
  let obstacleInterval = 90;

  function gameLoop() {
    if (!gameRunning) return;
    ctx.clearRect(0, 0, canvas.width, canvas.height);

    // Yol ve şeritler
    const roadX = 50;
    const roadWidth = 300;
    ctx.fillStyle = '#1c1c1c';
    ctx.fillRect(roadX, 0, roadWidth, canvas.height);

    ctx.strokeStyle = 'white';
    ctx.lineWidth = 6;
    ctx.setLineDash([30, 25]);
    ctx.beginPath();
    ctx.moveTo(roadX + 100, 0);
    ctx.lineTo(roadX + 100, canvas.height);
    ctx.moveTo(roadX + 200, 0);
    ctx.lineTo(roadX + 200, canvas.height);
    ctx.stroke();
    ctx.setLineDash([]);

    drawCar(lanes[carLane], canvas.height - carHeight - 15);

    for (let i = obstacles.length - 1; i >= 0; i--) {
      obstacles[i].y += obstacleSpeed;
      drawObstacle(obstacles[i].x, obstacles[i].y);

      if (
        obstacles[i].y + carHeight > canvas.height - carHeight - 15 &&
        obstacles[i].y < canvas.height - 15 &&
        obstacles[i].x === lanes[carLane]
      ) {
        lives--;
        livesDisplay.textContent = 'Can: ' + lives;
        obstacles.splice(i, 1);
        if (lives <= 0) {
          endGame();
          return;
        }
      } else if (obstacles[i].y > canvas.height) {
        obstacles.splice(i, 1);
        score += 10;
        scoreDisplay.textContent = 'Puan: ' + score;
        if (score % 50 === 0) {
          obstacleSpeed += 0.5;
        }
      }
    }

    obstacleTimer++;
    if (obstacleTimer > obstacleInterval) {
      createObstacle();
      obstacleTimer = 0;
    }

    requestAnimationFrame(gameLoop);
  }

  // Sağ ve sol butonları ile hareket
  const leftBtn = document.getElementById('leftBtn');
  const rightBtn = document.getElementById('rightBtn');

  leftBtn.addEventListener('click', () => {
    if (!gameRunning) return;
    if (carLane > 0) carLane--;
  });

  rightBtn.addEventListener('click', () => {
    if (!gameRunning) return;
    if (carLane < lanes.length - 1) carLane++;
  });

  // Klavye ok tuşları ile kontrol
  window.addEventListener('keydown', (e) => {
    if (!gameRunning) return;
    if (e.key === 'ArrowLeft' && carLane > 0) {
      carLane--;
    } else if (e.key === 'ArrowRight' && carLane < lanes.length - 1) {
      carLane++;
    }
  });

  startBtn.onclick = startGame;
  restartBtn.onclick = startGame;

  // Başlangıçta menüyü göster
  menu.style.display = 'block';
</script>

</body>
</html>
