/araba-dino
├── index.html
├── style.css
└── script.js<!DOCTYPE html>
<html lang="tr">
<head>
  <meta charset="UTF-8">
  <title>Araba Dino Oyunu</title>
  <link rel="stylesheet" href="style.css">
</head>
<body>
  <div id="game">
    <div id="car"></div>
    <div id="obstacle"></div>
  </div>
  <p>Boşluk tuşuna basarak zıpla</p>
  <script src="script.js"></script>
</body>
</html>body {
  margin: 0;
  padding: 0;
  background: #f0f0f0;
  font-family: sans-serif;
  text-align: center;
}

#game {
  width: 600px;
  height: 200px;
  border: 2px solid black;
  margin: 50px auto;
  position: relative;
  overflow: hidden;
  background: #e3e3e3;
}

#car {
  width: 60px;
  height: 30px;
  background: red;
  position: absolute;
  bottom: 0;
  left: 50px;
  border-radius: 5px;
}

#obstacle {
  width: 60px;
  height: 30px;
  background: blue;
  position: absolute;
  bottom: 0;
  right: -60px;
  border-radius: 5px;
}#car {
  background-image: url('araba.png');
  background-size: cover;
  background-repeat: no-repeat;
  width: 60px;
  height: 30px;
}const car = document.getElementById('car');
const obstacle = document.getElementById('obstacle');

let isJumping = false;
let gravity = 0.9;
let jumpHeight = 150;

function jump() {
  if (isJumping) return;
  isJumping = true;

  let position = 0;
  let upInterval = setInterval(() => {
    if (position >= jumpHeight) {
      clearInterval(upInterval);
      let downInterval = setInterval(() => {
        if (position <= 0) {
          clearInterval(downInterval);
          isJumping = false;
        } else {
          position -= 5;
          car.style.bottom = position + 'px';
        }
      }, 20);
    } else {
      position += 5;
      car.style.bottom = position + 'px';
    }
  }, 20);
}

function checkCollision() {
  let carTop = parseInt(window.getComputedStyle(car).getPropertyValue('bottom'));
  let obstacleLeft = parseInt(window.getComputedStyle(obstacle).getPropertyValue('left'));

  if (obstacleLeft < 110 && obstacleLeft > 40 && carTop < 30) {
    alert("Oyun Bitti!");
    location.reload();
  }
}

function moveObstacle() {
  let obstacleLeft = 600;
  const move = setInterval(() => {
    if (obstacleLeft < -60) {
      obstacleLeft = 600;
    } else {
      obstacleLeft -= 5;
      obstacle.style.left = obstacleLeft + 'px';
    }

    checkCollision();
  }, 20);
}

document.addEventListener('keydown', function (e) {
  if (e.code === 'Space') {
    jump();
  }
});

moveObstacle();
