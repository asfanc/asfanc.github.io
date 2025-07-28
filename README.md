<!DOCTYPE html>
<html lang="tr">
<head>
<meta charset="UTF-8" />
<meta name="viewport" content="width=device-width, initial-scale=1" />
<title>Araba Koşu Oyunu</title>
<style>
  body, html { margin: 0; padding: 0; overflow: hidden; background: #87CEEB; font-family: Arial, sans-serif; }
  #gameCanvas { background: #555; display: block; margin: 20px auto; border-radius: 10px; }
  #menu, #gameOverScreen {
    position: absolute; top: 50%; left: 50%; transform: translate(-50%, -50%);
    background: rgba(0,0,0,0.8); color: white; padding: 20px; border-radius: 10px;
    text-align: center; font-size: 24px; display: none;
  }
  button {
    padding: 10px 20px; font-size: 18px; margin-top: 10px; cursor: pointer;
    border: none; border-radius: 5px; background: #28a745; color: white;
  }
  #score, #lives {
    position: fixed; top: 10px; font-size: 20px; color: white; font-weight: bold;
  }
  #score { left: 20px; }
  #lives { right: 20px; }
</style>
</head>
<body>

<div id="menu">
  <h1>Araba Koşu Oyunu</h1>
  <button id="startBtn">Başlat</button>
</div>

<div id="gameOverScreen">
  <h2>Oyun Bitti!</h2>
  <p>Toplam Puan: <span id="finalScore">0</span></p>
  <button id="restartBtn">Tekrar Oyna</button>
</div>

<canvas id="gameCanvas" width="400" height="600"></canvas>
<div id="score">Puan: 0</div>
<div id="lives">Can: 3</div>

<script>
  const canvas = document.getElementById('gameCanvas');
  const ctx = canvas.getContext('2d');

  const lanes = [70, 170, 270]; // 3 şerit pozisyonları x ekseninde
  let carLane = 1; // ortadaki şerit

  let obstacles = [];
  let score = 0;
  let lives = 3;
  let gameRunning = false;
  let obstacleSpeed = 4;

  const carWidth = 50;
  const carHeight = 90;

  // Basit araba şekli (dikdörtgen + tekerlekler)
  function drawCar(x, y) {
    ctx.fillStyle = 'red';
    ctx.fillRect(x, y, carWidth, carHeight);
    // tekerlekler
    ctx.fillStyle = 'black';
    ctx.fillRect(x + 5, y + carHeight - 15, 15, 10);
    ctx.fillRect(x + carWidth - 20, y + carHeight - 15, 15, 10);
  }

  // Engel (beyaz araba)
  function drawObstacle(x, y) {
    ctx.fillStyle = 'white';
    ctx.fillRect(x, y, carWidth, carHeight);
    ctx.fillStyle = 'gray';
    ctx.fillRect(x + 5, y + 10, carWidth - 10, 20); // cam gibi
  }

  // Menü ve oyun ekranları kontrolü
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

  // Engelleri yarat
  function createObstacle() {
    const laneIndex = Math.floor(Math.random() * lanes.length);
    obstacles.push({
      x: lanes[laneIndex],
      y: -carHeight,
    });
  }

  // Engel oluşturma zamanlayıcı
  let obstacleTimer = 0;
  let obstacleInterval = 100;

  // Oyun döngüsü
  function gameLoop() {
    if (!gameRunning) return;
    ctx.clearRect(0, 0, canvas.width, canvas.height);

    // Yolu çiz
    ctx.fillStyle = '#444';
    ctx.fillRect(50, 0, 300, canvas.height);
    // Şerit çizgileri
    ctx.strokeStyle = 'white';
    ctx.lineWidth = 5;
    ctx.setLineDash([20, 20]);
    ctx.beginPath();
    ctx.moveTo(150, 0);
    ctx.lineTo(150, canvas.height);
    ctx.moveTo(250, 0);
    ctx.lineTo(250, canvas.height);
    ctx.stroke();
    ctx.setLineDash([]);

    // Arabayı çiz
    drawCar(lanes[carLane], canvas.height - carHeight - 10);

    // Engelleri hareket ettir ve çiz
    for (let i = obstacles.length - 1; i >= 0; i--) {
      obstacles[i].y += obstacleSpeed;
      drawObstacle(obstacles[i].x, obstacles[i].y);

      // Çarpışma kontrolü
      if (
        obstacles[i].y + carHeight > canvas.height - carHeight - 10 &&
        obstacles[i].y < canvas.height - 10 &&
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
        // Zamanla hız artsın
        if(score % 50 === 0){
          obstacleSpeed += 0.5;
        }
      }
    }

    // Yeni engel oluştur
    obstacleTimer++;
    if (obstacleTimer > obstacleInterval) {
      createObstacle();
      obstacleTimer = 0;
    }

    requestAnimationFrame(gameLoop);
  }

  // Klavye ile kontrol (sol-sağ)
  window.addEventListener('keydown', (e) => {
    if(!gameRunning) return;
    if (e.key === 'ArrowLeft' && carLane > 0) {
      carLane--;
    } else if (e.key === 'ArrowRight' && carLane < lanes.length -1) {
      carLane++;
    }
  });

  // Dokunmatik kontrol (sağa-sola kaydırma)
  let touchStartX = null;
  window.addEventListener('touchstart', e => {
    touchStartX = e.touches[0].clientX;
  });
  window.addEventListener('touchend', e => {
    if(!gameRunning) return;
    let touchEndX = e.changedTouches[0].clientX;
    if(touchStartX !== null){
      let diff = touchEndX - touchStartX;
      if(diff > 30 && carLane < lanes.length - 1){
        carLane++;
      } else if(diff < -30 && carLane > 0){
        carLane--;
      }
    }
    touchStartX = null;
  });

  startBtn.onclick = startGame;
  restartBtn.onclick = startGame;

  // Başlangıçta menüyü göster
  menu.style.display = 'block';
</script>

</body>
</html>
