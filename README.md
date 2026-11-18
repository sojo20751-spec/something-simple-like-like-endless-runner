<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8" />
  <meta name="viewport" content="width=device-width, initial-scale=1.0, user-scalable=no" />
  <title>Mobile Endless Runner</title>
  <style>
    * {
      box-sizing: border-box;
      touch-action: manipulation;
    }
    body {
      margin: 0;
      padding: 0;
      background: #111;
      color: #fff;
      font-family: system-ui, -apple-system, sans-serif;
      display: flex;
      flex-direction: column;
      align-items: center;
      justify-content: center;
      min-height: 100vh;
      overflow: hidden;
    }
    canvas {
      border: 2px solid #444;
      background: #222;
      max-width: 100%;
      max-height: 80vh;
      box-shadow: 0 10px 25px rgba(0,0,0,0.5);
    }
    p {
      margin-top: 12px;
      color: #aaa;
      font-size: 14px;
      text-align: center;
    }
  </style>
</head>
<body>

  <canvas id="gameCanvas" width="400" height="600"></canvas>
  <p>Swipe left/right on touchscreens or use <strong>A / D</strong> keys.</p>

<script>
const canvas = document.getElementById("gameCanvas");
const ctx = canvas.getContext("2d");

// Game Constants & Lanes
const LANES = [80, 200, 320];
let currentLane = 1;
let score = 0;
let gameOver = false;
let gameSpeed = 5;

// Player Setup
const player = {
  x: LANES[currentLane],
  y: 500,
  width: 40,
  height: 60,
  targetX: LANES[currentLane],
  color: "#00FFCC"
};

const obstacles = [];
let frameCount = 0;

// Swipe Controls (Mobile)
let touchStartX = 0;
let touchStartY = 0;

window.addEventListener("touchstart", (e) => {
  touchStartX = e.touches[0].clientX;
  touchStartY = e.touches[0].clientY;
}, { passive: true });

window.addEventListener("touchend", (e) => {
  if (gameOver) {
    resetGame();
    return;
  }
  const deltaX = e.changedTouches[0].clientX - touchStartX;
  const deltaY = e.changedTouches[0].clientY - touchStartY;

  // Ensure horizontal swipe is larger than vertical drift
  if (Math.abs(deltaX) > Math.abs(deltaY) && Math.abs(deltaX) > 30) {
    if (deltaX < 0 && currentLane > 0) {
      currentLane--;
    } else if (deltaX > 0 && currentLane < LANES.length - 1) {
      currentLane++;
    }
    player.targetX = LANES[currentLane];
  }
});

// Keyboard Controls (Desktop)
window.addEventListener("keydown", (e) => {
  if (gameOver) {
    resetGame();
    return;
  }
  if ((e.key === "ArrowLeft" || e.key === "a") && currentLane > 0) {
    currentLane--;
  } else if ((e.key === "ArrowRight" || e.key === "d") && currentLane < LANES.length - 1) {
    currentLane++;
  }
  player.targetX = LANES[currentLane];
});

function spawnObstacle() {
  const randomLane = Math.floor(Math.random() * LANES.length);
  obstacles.push({
    x: LANES[randomLane],
    y: -60,
    width: 50,
    height: 50,
    color: "#FF3366"
  });
}

function resetGame() {
  currentLane = 1;
  player.x = LANES[currentLane];
  player.targetX = LANES[currentLane];
  obstacles.length = 0;
  score = 0;
  gameSpeed = 5;
  frameCount = 0;
  gameOver = false;
}

function update() {
  if (gameOver) return;

  // Smooth lerp movement between lanes
  player.x += (player.targetX - player.x) * 0.25;

  frameCount++;
  if (frameCount % 60 === 0) {
    spawnObstacle();
    gameSpeed += 0.05;
  }

  for (let i = obstacles.length - 1; i >= 0; i--) {
    let obs = obstacles[i];
    obs.y += gameSpeed;

    // AABB Collision Detection
    if (
      player.x - player.width / 2 < obs.x + obs.width / 2 &&
      player.x + player.width / 2 > obs.x - obs.width / 2 &&
      player.y - player.height / 2 < obs.y + obs.height / 2 &&
      player.y + player.height / 2 > obs.y - obs.height / 2
    ) {
      gameOver = true;
    }

    if (obs.y > canvas.height + 60) {
      obstacles.splice(i, 1);
      score += 10;
    }
  }
}

function draw() {
  ctx.clearRect(0, 0, canvas.width, canvas.height);

  // Draw Track Lines
  ctx.strokeStyle = "#333";
  ctx.lineWidth = 2;
  ctx.beginPath();
  ctx.moveTo(140, 0); ctx.lineTo(140, canvas.height);
  ctx.moveTo(260, 0); ctx.lineTo(260, canvas.height);
  ctx.stroke();

  // Draw Player
  ctx.fillStyle = player.color;
  ctx.fillRect(player.x - player.width / 2, player.y - player.height / 2, player.width, player.height);

  // Draw Obstacles
  obstacles.forEach((obs) => {
    ctx.fillStyle = obs.color;
    ctx.fillRect(obs.x - obs.width / 2, obs.y - obs.height / 2, obs.width, obs.height);
  });

  // Draw UI
  ctx.fillStyle = "#FFF";
  ctx.font = "bold 20px sans-serif";
  ctx.fillText(`Score: ${score}`, 20, 35);

  if (gameOver) {
    ctx.fillStyle = "rgba(0, 0, 0, 0.8)";
    ctx.fillRect(0, 0, canvas.width, canvas.height);

    ctx.fillStyle = "#FF3366";
    ctx.font = "bold 32px sans-serif";
    ctx.textAlign = "center";
    ctx.fillText("GAME OVER", canvas.width / 2, canvas.height / 2 - 20);

    ctx.fillStyle = "#FFF";
    ctx.font = "16px sans-serif";
    ctx.fillText("Tap or press any key to restart", canvas.width / 2, canvas.height / 2 + 20);
    ctx.textAlign = "start";
  }
}

function gameLoop() {
  update();
  draw();
  requestAnimationFrame(gameLoop);
}

gameLoop();
</script>
</body>
</html>
# something-simple-like-like-endless-runner