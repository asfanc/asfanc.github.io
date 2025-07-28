<!DOCTYPE html>
<html lang="tr">
<head>
<meta charset="UTF-8" />
<meta name="viewport" content="width=device-width, initial-scale=1" />
<title>Gerçekçi Araba Koşu Oyunu</title>
<style>
  body, html { margin: 0; padding: 0; overflow: hidden; background: linear-gradient(to top, #3a6186, #89253e); font-family: 'Segoe UI', Tahoma, Geneva, Verdana, sans-serif; color: #eee;}
  #gameCanvas { background: #2c3e50; display: block; margin: 20px auto; border-radius: 15px; box-shadow: 0 0 20px rgba(0,0,0,0.7);}
  
  #menu, #gameOverScreen {
    position: absolute; top: 50%; left: 50%; transform: translate(-50%, -50%);
    background: rgba(20, 20, 20, 0.95);
    border-radius: 15px;
    width: 320px;
    padding: 30px 20px;
    box-shadow: 0 8px 20px rgba(255, 255, 255, 0.1);
    text-align: center;
    display: none;
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

  #score, #lives {
    position: fixed; top: 15px; font-size: 22px; font-weight: 700; color: #f7b733;
    text-shadow: 1px 1px 3px #000;
    user-select: none;
  }
  #score { left: 25px; }
  #lives { right: 25px; }
</style>
</head>
<body>

<div id="menu">
  <h1>Araba Koşu Oyunu</h1>
  <p>Engellerden kaç, mümkün olduğunca uzun git ve en yüksek puanı yap!</p>
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

  const carWidth = 50;
  const carHeight = 90;

  // Araç görselleri (base64 PNG)
  const playerCarImg = new Image();
  playerCarImg.src = 'data:image/png;base64,iVBORw0KGgoAAAANSUhEUgAAADIAAAAYCAYAAAAb14UQAAAACXBIWXMAAAsTAAALEwEAmpwYAAABOklEQVRoge3XvW6DQBSA4c+SknDL0OLHoMg0EQSDIVnoGDr9gjY2CbODLgLZmk8Nl+S+mFSkshGjI+4k0N78+ab3p5C0IlVKYj37qSzYv2zN1+XcNj8YkY7P6lIfV1CDkB7pLGh0bMn/jc5QPEsFE+YAX5nUD8Xy4EEu5CyAn7CxcJ3ieT8FKDJvTw+Rh5PgOP+mkLrrk0d09zp+PhhCgOEWlYk9R4AyqCjiMwTWz6d4uKRR0DhH8FLpT7j2Q0Awl27eGqRMC2OcDw+HwHvEBP3ih0yJAaHo+/nGz9XuCukbFCJZ+3SCaZEM1ZtCqaLPGlDnsmOVY5vRcH0bELImH84iN7ylmh9trYsCiG13wxn/s67AR+W9Mq+G6A2HhcxDBrgPwDL8B3BzD4FEtpkcx6l4iOVgHeG8/FOV6k8izp8NMgAAAABJRU5ErkJggg==';

  const obstacleCarImg = new Image();
  obstacleCarImg.src = 'data:image/png;base64,iVBORw0KGgoAAAANSUhEUgAAADIAAAAYCAYAAAAb14UQAAAACXBIWXMAAAsTAAALEwEAmpwYAAABNklEQVRoge3XwUoDQRBF0W8ROgfzGEziAHPom4CVP0AWVq1LLgARHqW6GvHsghFrLX5J88e+1ka3h5Vj7sPyxQyLUq7VVCPT4elEY1nPPufqOSqHq+eQDYf2jS9zMXvE/v6GZjGb/9kO7pF7hTQjNqNLzBuFA0bLZcSnIznvvDnQNHx8i8ii6Ax8Dvo7c2ThDcQn6Jw2Swg25GLDlnCbSQtwz8vUQscZzRQtfwh/GoFEKYdyH+AsnE1+DiPwbNPcV+CzYLg4d4d+NADki3IQfGx7ZHYpBTEbMfPw7B6p6OcBdV6pzj+DP8zSG6rO9HeZJso0nFZ8LS7gO7GOzNXOwM42i7W5K+B7TfD7P80hZprYA6HRhH0kW0tAAAAAElFTkSuQmCC';

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

    // Yol
    const roadX = 50;
    const roadWidth = 300;
    ctx.fillStyle = '#2c3e50';
    ctx.fillRect(roadX, 0, roadWidth, canvas.height);

    // Şerit çizgileri
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

  window.addEventListener('keydown', (e) => {
    if (!gameRunning) return;
    if (e.key === 'ArrowLeft' && carLane > 0) {
      carLane--;
    } else if (e.key === 'ArrowRight' && carLane < lanes.length - 1) {
      carLane++;
    }
  });

  let touchStartX = null;
  window.addEventListener('touchstart', e => {
    touchStartX = e.touches[0].clientX;
  });
  window.addEventListener('touchend', e => {
    if (!gameRunning) return;
    let touchEndX = e.changedTouches[0].clientX;
    if (touchStartX !== null) {
      let diff = touchEndX - touchStartX;
      if (diff > 30 && carLane < lanes.length - 1) {
        carLane++;
      } else if (diff < -30 && carLane > 0) {
        carLane--;
      }
    }
    touchStartX = null;
  });

  startBtn.onclick = startGame;
  restartBtn.onclick = startGame;

  menu.style.display = 'block';
</script>

</body>
</html><!DOCTYPE html>
<html lang="tr">
<head>
<meta charset="UTF-8" />
<meta name="viewport" content="width=device-width, initial-scale=1" />
<title>Gerçekçi Araba Koşu Oyunu</title>
<style>
  body, html { margin: 0; padding: 0; overflow: hidden; background: linear-gradient(to top, #3a6186, #89253e); font-family: 'Segoe UI', Tahoma, Geneva, Verdana, sans-serif; color: #eee;}
  #gameCanvas { background: #2c3e50; display: block; margin: 20px auto; border-radius: 15px; box-shadow: 0 0 20px rgba(0,0,0,0.7);}
  
  #menu, #gameOverScreen {
    position: absolute; top: 50%; left: 50%; transform: translate(-50%, -50%);
    background: rgba(20, 20, 20, 0.95);
    border-radius: 15px;
    width: 320px;
    padding: 30px 20px;
    box-shadow: 0 8px 20px rgba(255, 255, 255, 0.1);
    text-align: center;
    display: none;
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

  #score, #lives {
    position: fixed; top: 15px; font-size: 22px; font-weight: 700; color: #f7b733;
    text-shadow: 1px 1px 3px #000;
    user-select: none;
  }
  #score { left: 25px; }
  #lives { right: 25px; }
</style>
</head>
<body>

<div id="menu">
  <h1>Araba Koşu Oyunu</h1>
  <p>Engellerden kaç, mümkün olduğunca uzun git ve en yüksek puanı yap!</p>
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

  const carWidth = 50;
  const carHeight = 90;

  // Araç görselleri (base64 PNG)
  const playerCarImg = new Image();
  playerCarImg.src = 'data:image/png;base64,iVBORw0KGgoAAAANSUhEUgAAADIAAAAYCAYAAAAb14UQAAAACXBIWXMAAAsTAAALEwEAmpwYAAABOklEQVRoge3XvW6DQBSA4c+SknDL0OLHoMg0EQSDIVnoGDr9gjY2CbODLgLZmk8Nl+S+mFSkshGjI+4k0N78+ab3p5C0IlVKYj37qSzYv2zN1+XcNj8YkY7P6lIfV1CDkB7pLGh0bMn/jc5QPEsFE+YAX5nUD8Xy4EEu5CyAn7CxcJ3ieT8FKDJvTw+Rh5PgOP+mkLrrk0d09zp+PhhCgOEWlYk9R4AyqCjiMwTWz6d4uKRR0DhH8FLpT7j2Q0Awl27eGqRMC2OcDw+HwHvEBP3ih0yJAaHo+/nGz9XuCukbFCJZ+3SCaZEM1ZtCqaLPGlDnsmOVY5vRcH0bELImH84iN7ylmh9trYsCiG13wxn/s67AR+W9Mq+G6A2HhcxDBrgPwDL8B3BzD4FEtpkcx6l4iOVgHeG8/FOV6k8izp8NMgAAAABJRU5ErkJggg==';

  const obstacleCarImg = new Image();
  obstacleCarImg.src = 'data:image/png;base64,iVBORw0KGgoAAAANSUhEUgAAADIAAAAYCAYAAAAb14UQAAAACXBIWXMAAAsTAAALEwEAmpwYAAABNklEQVRoge3XwUoDQRBF0W8ROgfzGEziAHPom4CVP0AWVq1LLgARHqW6GvHsghFrLX5J88e+1ka3h5Vj7sPyxQyLUq7VVCPT4elEY1nPPufqOSqHq+eQDYf2jS9zMXvE/v6GZjGb/9kO7pF7hTQjNqNLzBuFA0bLZcSnIznvvDnQNHx8i8ii6Ax8Dvo7c2ThDcQn6Jw2Swg25GLDlnCbSQtwz8vUQscZzRQtfwh/GoFEKYdyH+AsnE1+DiPwbNPcV+CzYLg4d4d+NADki3IQfGx7ZHYpBTEbMfPw7B6p6OcBdV6pzj+DP8zSG6rO9HeZJso0nFZ8LS7gO7GOzNXOwM42i7W5K+B7TfD7P80hZprYA6HRhH0kW0tAAAAAElFTkSuQmCC';

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

    // Yol
    const roadX = 50;
    const roadWidth = 300;
    ctx.fillStyle = '#2c3e50';
    ctx.fillRect(roadX, 0, roadWidth, canvas.height);

    // Şerit çizgileri
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

  window.addEventListener('keydown', (e) => {
    if (!gameRunning) return;
    if (e.key === 'ArrowLeft' && carLane > 0) {
      carLane--;
    } else if (e.key === 'ArrowRight' && carLane < lanes.length - 1) {
      carLane++;
    }
  });

  let touchStartX = null;
  window.addEventListener('touchstart', e => {
    touchStartX = e.touches[0].clientX;
  });
  window.addEventListener('touchend', e => {
    if (!gameRunning) return;
    let touchEndX = e.changedTouches[0].clientX;
    if (touchStartX !== null) {
      let diff = touchEndX - touchStartX;
      if (diff > 30 && carLane < lanes.length - 1) {
        carLane++;
      } else if (diff < -30 && carLane > 0) {
        carLane--;
      }
    }
    touchStartX = null;
  });

  startBtn.onclick = startGame;
  restartBtn.onclick = startGame;

  menu.style.display = 'block';
</script>

</body>
</html>
