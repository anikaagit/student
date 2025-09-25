---
layout: default
title: Breakout Blocks
author: Anika Marathe
permalink: breakout
---

<style>
canvas { background:#eee; display:block; margin:0 auto; border:1px solid #333; }
h2{margin-top:5px!important;}
p{margin-bottom:5px!important;}
</style>

<canvas id="gameCanvas" width="600" height="400"></canvas>

<div style="max-width:600px;margin:10px auto;font-family:system-ui,Arial;">
<label><strong>🎨 Ball Color:</strong><input type="color" id="ballColorPicker" value="#0095DD"></label>
<br><br>
<label><strong>🧱 Brick Color:</strong><input type="color" id="brickColorPicker" value="#6f42c1"></label>
</div>

<button id="nextLevelBtn" style="display:none;margin:10px auto 0;padding:10px 16px;font-family:system-ui,Arial;font-size:16px;font-weight:600;border:1px solid #222;background:#fff;cursor:pointer;border-radius:8px;display:block;max-width:600px;color:#111 !important;">Next Level ▶</button>

<script>
/* =====================
   Breakout — stable version
   - Robust circle-vs-rect collision
   - Penetration resolution + reflection
   - Per-frame lastHitFrame guard
   - Multi-hit and moving bricks
   - Particle effects
   - Multi-ball powerup (works)
   - Slow motion toggle (Space)
   - Color pickers & gradient bricks
   ===================== */

const canvas = document.getElementById('gameCanvas');
const ctx = canvas.getContext('2d');
const nextLevelBtn = document.getElementById('nextLevelBtn');

// Color pickers
let ballColor = "#0095DD";
let brickColor = "#6f42c1";
document.getElementById('ballColorPicker').addEventListener('input', e => {
  ballColor = e.target.value;
  balls.forEach(b => b.color = ballColor); // update existing balls visually
});
document.getElementById('brickColorPicker').addEventListener('input', e => brickColor = e.target.value);

// Game constants
const powerUpSize = 20;
const powerUpFallSpeed = 1.5;
const speedIncrement = 1.06; // speed up on hits
let frameCount = 0;

// Game state
let level = 1;
const levelSpeedScale = 1.12;
let paused = false;
let score = 0;
let lives = 3;

// Paddle
let paddleHeight = 10;
let basePaddleWidth = 75;
let paddleWidth = basePaddleWidth;
let paddleX = (canvas.width - paddleWidth) / 2;

// Input
let rightPressed = false;
let leftPressed = false;
let slowMotion = false;
document.addEventListener('keydown', (e) => {
  if (e.key === 'Right' || e.key === 'ArrowRight') rightPressed = true;
  else if (e.key === 'Left' || e.key === 'ArrowLeft') leftPressed = true;
  else if (e.code === 'Space') slowMotion = !slowMotion;
});
document.addEventListener('keyup', (e) => {
  if (e.key === 'Right' || e.key === 'ArrowRight') rightPressed = false;
  else if (e.key === 'Left' || e.key === 'ArrowLeft') leftPressed = false;
});
document.addEventListener('mousemove', (e) => {
  const rect = canvas.getBoundingClientRect();
  let relX = e.clientX - rect.left;
  if (relX > 0 && relX < canvas.width) paddleX = relX - paddleWidth / 2;
});

// Bricks settings
let brickRowCount = 4;
const brickColumnCount = 6;
const brickWidth = 75;
const brickHeight = 20;
const brickPadding = 10;
const brickOffsetTop = 30;
const brickOffsetLeft = 50;
const powerUpChance = 0.28;

// Utility
const clamp = (v, a, b) => Math.max(a, Math.min(b, v));

// === Base GameObject ===
class GameObject {
  constructor(x, y, w, h) {
    this.x = x; this.y = y; this.width = w; this.height = h;
  }
  draw() {}
  update() {}
  // generic circle vs rect test helper will be done externally
}

// === Ball ===
class Ball extends GameObject {
  constructor(x, y, dx, dy, radius = 8) {
    super(x, y, radius * 2, radius * 2);
    this.x = x; this.y = y; // center
    this.dx = dx; this.dy = dy;
    this.radius = radius;
    this.color = ballColor;
    this.prevX = x; this.prevY = y;
  }

  draw() {
    ctx.beginPath();
    ctx.arc(this.x, this.y, this.radius, 0, Math.PI * 2);
    ctx.fillStyle = this.color;
    ctx.fill();
    ctx.closePath();
  }

  move() {
    this.prevX = this.x;
    this.prevY = this.y;
    const speedScale = slowMotion ? 0.5 : 1;
    this.x += this.dx * speedScale;
    this.y += this.dy * speedScale;
  }

  bounceWalls() {
    // left / right
    if (this.x - this.radius < 0) {
      this.x = this.radius;
      this.dx = Math.abs(this.dx);
    } else if (this.x + this.radius > canvas.width) {
      this.x = canvas.width - this.radius;
      this.dx = -Math.abs(this.dx);
    }
    // top
    if (this.y - this.radius < 0) {
      this.y = this.radius;
      this.dy = Math.abs(this.dy);
    }
  }

  markForRemoval() {
    ballsToRemove.push(this);
  }
}

// === Brick types ===
class Brick extends GameObject {
  constructor(x, y, w, h) {
    super(x, y, w, h);
    this.status = 1;
    this.powerUp = Math.random() < powerUpChance;
    this._lastHitFrame = -1;
  }
  draw() {
    if (this.status !== 1) return;
    const g = ctx.createLinearGradient(this.x, this.y, this.x + this.width, this.y + this.height);
    g.addColorStop(0, brickColor);
    g.addColorStop(1, '#ffffff');
    ctx.fillStyle = g;
    ctx.fillRect(this.x, this.y, this.width, this.height);
    // little border
    ctx.strokeStyle = 'rgba(0,0,0,0.15)';
    ctx.strokeRect(this.x, this.y, this.width, this.height);
  }
  hit() { this.status = 0; }
}

class MultiHitBrick extends Brick {
  constructor(x, y, w, h, hits = 2) {
    super(x, y, w, h);
    this.hits = hits;
  }
  draw() {
    if (this.status !== 1) return;
    const g = ctx.createLinearGradient(this.x, this.y, this.x + this.width, this.y + this.height);
    g.addColorStop(0, '#ff6347');
    g.addColorStop(1, '#ffa07a');
    ctx.fillStyle = g;
    ctx.fillRect(this.x, this.y, this.width, this.height);
    ctx.fillStyle = 'black';
    ctx.font = '14px Arial';
    ctx.textAlign = 'center';
    ctx.textBaseline = 'middle';
    ctx.fillText(this.hits, this.x + this.width / 2, this.y + this.height / 2);
  }
  hit() {
    this.hits--;
    if (this.hits <= 0) this.status = 0;
  }
}

class MovingBrick extends Brick {
  constructor(x, y, w, h, speed = 1) {
    super(x, y, w, h);
    this.speed = speed;
    this.dir = 1;
  }
  update() {
    if (this.status !== 1) return;
    this.x += this.speed * this.dir;
    if (this.x < 0) { this.x = 0; this.dir *= -1; }
    if (this.x + this.width > canvas.width) { this.x = canvas.width - this.width; this.dir *= -1; }
  }
}

// === PowerUp class ===
class PowerUp extends GameObject {
  constructor(x, y, type) {
    super(x, y, powerUpSize, powerUpSize);
    this.x = x; this.y = y;
    this.type = type;
    this.active = true;
  }
  draw() {
    const grad = ctx.createRadialGradient(this.x, this.y, 5, this.x, this.y, powerUpSize);
    grad.addColorStop(0, 'yellow');
    grad.addColorStop(1, 'red');
    ctx.beginPath();
    ctx.arc(this.x, this.y, powerUpSize / 2, 0, Math.PI * 2);
    ctx.fillStyle = grad; ctx.fill(); ctx.closePath();
    ctx.fillStyle = 'black';
    ctx.font = 'bold 14px Arial';
    ctx.textAlign = 'center';
    ctx.textBaseline = 'middle';
    const symbol = this.type === 'Wide Paddle' ? 'W' : this.type === 'Speed Boost' ? 'S' : 'M';
    ctx.fillText(symbol, this.x, this.y);
  }
  update() {
    this.y += powerUpFallSpeed;
    if (this.y > canvas.height + 50) this.active = false;
    // caught by paddle check happens in main loop
  }
  apply() {
    if (this.type === 'Wide Paddle') {
      paddleWidth = basePaddleWidth + 40;
      setTimeout(() => paddleWidth = basePaddleWidth, 5000);
    } else if (this.type === 'Speed Boost') {
      balls.forEach(b => { b.dx *= 1.5; b.dy *= 1.5; });
      setTimeout(() => balls.forEach(b => { b.dx /= 1.5; b.dy /= 1.5; }), 5000);
    } else if (this.type === 'Multi-Ball') {
      // spawn one extra ball above paddle with a safe initial position/direction
      const angle = (Math.random() * 1.2 - 0.6); // -0.6..0.6 radians
      const speed = 4;
      const spawnX = clamp(paddleX + paddleWidth / 2 + (Math.random() - 0.5) * 20, 20, canvas.width - 20);
      const spawnY = canvas.height - paddleHeight - 10 - 12; // above paddle
      balls.push(new Ball(spawnX, spawnY, speed * Math.cos(angle), -Math.abs(speed * Math.sin(angle))));
    }
    this.active = false;
  }
}

// === Particle (visual effect) ===
class Particle {
  constructor(x, y, dx, dy, color, life = 40) {
    this.x = x; this.y = y; this.dx = dx; this.dy = dy; this.color = color; this.life = life;
  }
  draw() {
    ctx.beginPath();
    ctx.arc(this.x, this.y, 2, 0, Math.PI * 2);
    ctx.fillStyle = this.color; ctx.fill(); ctx.closePath();
  }
  update() {
    this.x += this.dx; this.y += this.dy; this.dy += 0.05; this.life--;
  }
}

// === Game arrays ===
let bricks = [];
let powerUps = [];
let particles = [];
let balls = [];
let ballsToRemove = [];

// initialize
function resetBalls() {
  balls = [ new Ball(canvas.width / 2, canvas.height - 30, 3, -3) ];
  balls.forEach(b => b.color = ballColor);
}
resetBalls();

function initBricks() {
  bricks = [];
  for (let c = 0; c < brickColumnCount; c++) {
    for (let r = 0; r < brickRowCount; r++) {
      const x = c * (brickWidth + brickPadding) + brickOffsetLeft;
      const y = r * (brickHeight + brickPadding) + brickOffsetTop;
      const rand = Math.random();
      if (rand < 0.18) bricks.push(new MultiHitBrick(x, y, brickWidth, brickHeight, 2));
      else if (rand < 0.36) bricks.push(new MovingBrick(x, y, brickWidth, brickHeight, 0.8 + Math.random() * 0.8));
      else bricks.push(new Brick(x, y, brickWidth, brickHeight));
    }
  }
}
initBricks();

// helper: circle-vs-rect precise collision and resolution
function handleBallBrickCollision(ball, brick) {
  // find closest point from circle center to rect
  const closestX = clamp(ball.x, brick.x, brick.x + brick.width);
  const closestY = clamp(ball.y, brick.y, brick.y + brick.height);
  const dx = ball.x - closestX;
  const dy = ball.y - closestY;
  const dist2 = dx * dx + dy * dy;
  if (dist2 >= ball.radius * ball.radius) return false; // no collision

  // don't allow multiple hits same frame
  if (brick._lastHitFrame === frameCount) return false;
  brick._lastHitFrame = frameCount;

  const dist = Math.sqrt(dist2) || 0.0001;
  // penetration depth (how much overlap)
  const penetration = ball.radius - dist;

  // collision normal (from rect toward ball)
  const nx = dx / dist;
  const ny = dy / dist;

  // push ball out along normal by penetration + a tiny epsilon
  ball.x += nx * (penetration + 0.5);
  ball.y += ny * (penetration + 0.5);

  // reflect ball velocity across normal: v' = v - 2*(v·n)*n
  const vDotN = ball.dx * nx + ball.dy * ny;
  ball.dx = ball.dx - 2 * vDotN * nx;
  ball.dy = ball.dy - 2 * vDotN * ny;

  // small speed increase
  const speed = Math.hypot(ball.dx, ball.dy) * speedIncrement;
  const ang = Math.atan2(ball.dy, ball.dx);
  ball.dx = speed * Math.cos(ang);
  ball.dy = speed * Math.sin(ang);

  // mark brick hit, spawn powerup/particles
  brick.hit();
  if (brick.powerUp && Math.random() < 0.98) { // keep some randomness on spawn
    const types = ["Wide Paddle", "Speed Boost", "Multi-Ball"];
    const type = types[Math.floor(Math.random() * types.length)];
    powerUps.push(new PowerUp(brick.x + brick.width / 2, brick.y + 8, type));
  }
  // particles
  for (let i = 0; i < 10; i++) {
    const a = Math.random() * Math.PI * 2;
    const s = Math.random() * 2;
    particles.push(new Particle(ball.x, ball.y, Math.cos(a) * s, Math.sin(a) * s, 'orange', 30));
  }

  score++;
  return true;
}

// helper: paddle collision resolution (circle vs rect simplified)
function handleBallPaddle(ball) {
  // check if overlapping paddle rectangle
  const closestX = clamp(ball.x, paddleX, paddleX + paddleWidth);
  const closestY = clamp(ball.y, canvas.height - paddleHeight, canvas.height);
  const dx = ball.x - closestX;
  const dy = ball.y - closestY;
  const dist2 = dx * dx + dy * dy;
  if (dist2 >= ball.radius * ball.radius) return false;

  // reposition ball above paddle
  ball.y = canvas.height - paddleHeight - ball.radius - 0.5;
  // reflect vertically
  ball.dy = -Math.abs(ball.dy) * levelSpeedScale;

  // nudge horizontal velocity based on where it hit the paddle (gives player control)
  const relative = (ball.x - (paddleX + paddleWidth / 2)) / (paddleWidth / 2); // -1..1
  ball.dx += relative * 2;

  // boost speed slightly
  const speed = Math.hypot(ball.dx, ball.dy) * speedIncrement;
  const ang = Math.atan2(ball.dy, ball.dx);
  ball.dx = speed * Math.cos(ang);
  ball.dy = speed * Math.sin(ang);

  // ensure ball not flagged for removal
  return true;
}

// remove fallen balls and manage lives
function processFallenBalls() {
  balls = balls.filter(b => !ballsToRemove.includes(b));
  ballsToRemove = [];
  if (balls.length === 0) {
    lives--;
    if (lives <= 0) {
      alert('GAME OVER');
      document.location.reload();
    } else {
      resetBalls();
    }
  }
}

// === Draw helpers ===
function drawPaddle() {
  ctx.beginPath();
  ctx.rect(paddleX, canvas.height - paddleHeight, paddleWidth, paddleHeight);
  ctx.fillStyle = '#28a745';
  ctx.fill();
  ctx.closePath();
}
function drawScore() {
  ctx.font = '16px Arial';
  ctx.fillStyle = '#0095DD';
  ctx.fillText('Score: ' + score, 8, 20);
}
function drawLives() {
  ctx.font = '16px Arial';
  ctx.fillStyle = '#0095DD';
  ctx.fillText('Lives: ' + lives, canvas.width - 65, 20);
}

// === Main loop ===
function draw() {
  frameCount++;
  ctx.clearRect(0, 0, canvas.width, canvas.height);

  // update bricks (moving bricks)
  bricks.forEach(b => { if (typeof b.update === 'function') b.update(); });

  // draw bricks
  bricks.forEach(b => b.draw());

  // draw paddle & UI
  drawPaddle();
  drawScore();
  drawLives();

  // update/draw powerups, apply if caught
  for (let i = powerUps.length - 1; i >= 0; i--) {
    const p = powerUps[i];
    if (!p.active) { powerUps.splice(i, 1); continue; }
    p.update();
    // check paddle catch
    if (p.y + powerUpSize / 2 >= canvas.height - paddleHeight &&
        p.x > paddleX && p.x < paddleX + paddleWidth) {
      p.apply();
      powerUps.splice(i, 1);
      continue;
    }
    p.draw();
  }

  // update/draw particles
  for (let i = particles.length - 1; i >= 0; i--) {
    const p = particles[i];
    p.update();
    p.draw();
    if (p.life <= 0) particles.splice(i, 1);
  }

  // update & handle balls
  ballsToRemove = [];
  for (let bi = 0; bi < balls.length; bi++) {
    const ball = balls[bi];

    // move
    ball.move();
    // wall bounce
    ball.bounceWalls();

    // bricks collisions — iterate bricks and resolve
    for (let j = 0; j < bricks.length; j++) {
      const brick = bricks[j];
      if (brick.status !== 1) continue;
      handleBallBrickCollision(ball, brick);
    }

    // paddle collision
    if (ball.y - ball.radius > canvas.height + 50) {
      // clearly fallen off bottom
      ballsToRemove.push(ball);
      continue;
    } else {
      // handle paddle if overlapping
      if (handleBallPaddle(ball)) {
        // no-op (handled)
      }
    }

    // draw ball
    ball.draw();
  }

  // remove fallen balls & life handling
  processFallenBalls();

  // paddle movement
  if (rightPressed && paddleX < canvas.width - paddleWidth) paddleX += 7;
  else if (leftPressed && paddleX > 0) paddleX -= 7;

  // next level
  if (!paused && bricks.filter(b => b.status === 1).length === 0) {
    paused = true;
    nextLevelBtn.style.display = 'block';
  }

  requestAnimationFrame(draw);
}

// next level button
nextLevelBtn.addEventListener('click', () => {
  level++;
  if (brickRowCount < 8) brickRowCount++;
  initBricks();
  resetBalls();
  paused = false;
  nextLevelBtn.style.display = 'none';
});

// init game
resetBalls();
initBricks();
draw();
</script>
