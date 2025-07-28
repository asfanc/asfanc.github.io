<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8" />
  <meta name="viewport" content="width=device-width, initial-scale=1.0"/>
  <title>Araba Kaçış Oyunu</title>
  <link rel="stylesheet" href="style.css" />
</head>
<body>
  <div id="gameContainer">
    <img id="background" src="https://i.imgur.com/wx2FsAE.jpg" alt="road" />
    <img id="playerCar" src="https://i.imgur.com/Jj1iZ5B.png" alt="car" />
    <img id="enemyCar" src="https://i.imgur.com/6xkHkRs.png" alt="enemy" />
    <img id="crashEffect" src="https://i.imgur.com/KhrFXfd.png" alt="crash" />

    <div id="controls">
      <button id="leftBtn">◀</button>
      <button id="rightBtn">▶</button>
    </div>

    <div id="info">
      <span id="score">Puan: 0</span>
      <span id="lives">Can: 3</span>
    </div>
  </div>

  <audio id="crashSound" src="https://www.soundjay.com/mechanical/sounds/crash-1.mp3"></audio>

  <script src="game.js"></script>
</body>
</html>
body {
  margin: 0;
  overflow: hidden;
  font-family: sans-serif;
  background: #000;
}

#gameContainer {
  position: relative;
  width: 100%;
  height: 100vh;
  overflow: hidden;
}

#background {
  position: absolute;
  width: 100%;
  height: 100%;
  object-fit: cover;
  z-index: 0;
}

#playerCar, #enemyCar, #crashEffect {
  position: absolute;
  width: 60px;
  z-index: 2;
}

#playerCar {
  bottom: 100px;
  left: 50%;
  transform: translateX(-50%);
}

#enemyCar {
  top: -100px;
  left: 50%;
  transform: translateX(-50%);
}

#crashEffect {
  display: none;
  width: 100px;
  top: 50%;
  left: 50%;
  transform: translate(-50%, -50%);
  z-index: 3;
}

#controls {
  position: absolute;
  bottom: 10px;
  width: 100%;
  text-align: center;
  z-index: 4;
}

#controls button {
  font-size: 40px;
  padding: 10px 30px;
  margin: 0 20px;
}

#info {
  position: absolute;
  top: 10px;
  left: 10px;
  color: white;
  font-size: 18px;
  z-index: 5;
}const playerCar = document.getElementById('playerCar');
const enemyCar = document.getElementById('enemyCar');
const crashEffect = document.getElementById('crashEffect');
const crashSound = document.getElementById('crashSound');
const leftBtn = document.getElementById('leftBtn');
const rightBtn = document.getElementById('rightBtn');
const scoreDisplay = document.getElementById('score');
const livesDisplay = document.getElementById('lives');

let playerX = window.innerWidth / 2 - 30;
let enemyY = -100;
let enemyX = Math.random() * (window.innerWidth - 60);
let lives = 3;
let score = 0;
let crashed = false;

function movePlayer(dir) {
  if (dir === 'left') playerX -= 30;
  if (dir === 'right') playerX += 30;
  playerX = Math.max(0, Math.min(window.innerWidth - 60, playerX));
  playerCar.style.left = playerX + 'px';
}

leftBtn.addEventListener('click', () => movePlayer('left'));
rightBtn.addEventListener('click', () => movePlayer('right'));

function gameLoop() {
  if (crashed) return;

  enemyY += 5;
  if (enemyY > window.innerHeight) {
    enemyY = -100;
    enemyX = Math.random() * (window.innerWidth - 60);
    score += 10;
    scoreDisplay.textContent = 'Puan: ' + score;
  }

  enemyCar.style.top = enemyY + 'px';
  enemyCar.style.left = enemyX + 'px';

  const px = playerCar.getBoundingClientRect();
  const ex = enemyCar.getBoundingClientRect();

  if (
    px.top < ex.bottom &&
    px.bottom > ex.top &&
    px.left < ex.right &&
    px.right > ex.left
  ) {
    crash();
  }

  requestAnimationFrame(gameLoop);
}

function crash() {
  crashed = true;
  crashSound.play();
  crashEffect.style.display = 'block';

  setTimeout(() => {
    crashEffect.style.display = 'none';
    lives--;
    livesDisplay.textContent = 'Can: ' + lives;
    if (lives <= 0) {
      alert('Oyun Bitti! Puan: ' + score);
      location.reload();
    } else {
      crashed = false;
      enemyY = -100;
      gameLoop();
    }
  }, 1000);
}

window.onload = () => {
  movePlayer(); // ortala
  gameLoop();
};
