# Pong-game-
This is the simple pong game 😄
// Simple Pong game
(() => {
  const canvas = document.getElementById('game');
  const ctx = canvas.getContext('2d');

  const width = canvas.width;
  const height = canvas.height;

  // DOM scoreboard
  const leftScoreEl = document.getElementById('left-score');
  const rightScoreEl = document.getElementById('right-score');

  // Game settings
  const paddleWidth = 12;
  const paddleHeight = 100;
  const paddleSpeed = 6; // keyboard speed
  const aiMaxSpeed = 4.3;

  let leftScore = 0;
  let rightScore = 0;
  const winningScore = 10; // optional

  const ballRadius = 8;
  let ballSpeed = 5;

  // Left paddle (player)
  const leftPaddle = {
    x: 20,
    y: (height - paddleHeight) / 2,
    width: paddleWidth,
    height: paddleHeight,
    dy: 0
  };

  // Right paddle (computer)
  const rightPaddle = {
    x: width - 20 - paddleWidth,
    y: (height - paddleHeight) / 2,
    width: paddleWidth,
    height: paddleHeight
  };

  // Ball
  let ball = {
    x: width / 2,
    y: height / 2,
    vx: ballSpeed,
    vy: 0,
    radius: ballRadius
  };

  // Controls
  const keys = { ArrowUp: false, ArrowDown: false };

  // Utility
  function clamp(v, a, b) { return Math.max(a, Math.min(b, v)); }

  // Reset ball to center. 'direction' = 1 => to right, -1 => to left, 0 => random
  function resetBall(direction = 0) {
    ball.x = width / 2;
    ball.y = height / 2;
    ballSpeed = 5; // reset base speed
    const angle = (Math.random() * 0.6 - 0.3); // small random up/down angle in radians
    if (direction === 0) direction = Math.random() < 0.5 ? -1 : 1;
    ball.vx = direction * ballSpeed * Math.cos(angle);
    ball.vy = ballSpeed * Math.sin(angle);
  }

  // Start
  resetBall();

  // Mouse control
  canvas.addEventListener('mousemove', (e) => {
    const rect = canvas.getBoundingClientRect();
    const mouseY = e.clientY - rect.top;
    // center the paddle on the mouse
    leftPaddle.y = clamp(mouseY - leftPaddle.height / 2, 0, height - leftPaddle.height);
  });

  // Keyboard control
  window.addEventListener('keydown', (e) => {
    if (e.key === 'ArrowUp' || e.key === 'ArrowDown') {
      keys[e.key] = true;
      e.preventDefault();
    }
  });
  window.addEventListener('keyup', (e) => {
    if (e.key === 'ArrowUp' || e.key === 'ArrowDown') {
      keys[e.key] = false;
      e.preventDefault();
    }
  });

  // Collision detection between circle and rect
  function circleRectCollision(cx, cy, r, rx, ry, rw, rh) {
    // Find the closest point to the circle within the rectangle
    const closestX = clamp(cx, rx, rx + rw);
    const closestY = clamp(cy, ry, ry + rh);
    // Calculate the distance between circle's center and this closest point
    const dx = cx - closestX;
    const dy = cy - closestY;
    // If the distance is less than the circle's radius, collision!
    return (dx * dx + dy * dy) <= (r * r);
  }

  // Update scoreboard DOM
  function updateScoreboard() {
    leftScoreEl.textContent = leftScore;
    rightScoreEl.textContent = rightScore;
  }

  // Draw everything
  function draw() {
    // clear
    ctx.clearRect(0, 0, width, height);

    // midline dashed
    ctx.strokeStyle = 'rgba(255,255,255,0.06)';
    ctx.lineWidth = 2;
    ctx.beginPath();
    for (let y = 20; y < height; y += 20) {
      ctx.moveTo(width / 2, y);
      ctx.lineTo(width / 2, y + 10);
    }
    ctx.stroke();

    // paddles
    ctx.fillStyle = '#76ecff';
    ctx.fillRect(leftPaddle.x, leftPaddle.y, leftPaddle.width, leftPaddle.height);
    ctx.fillStyle = '#00b1ff';
    ctx.fillRect(rightPaddle.x, rightPaddle.y, rightPaddle.width, rightPaddle.height);

    // ball
    ctx.fillStyle = '#e6f9ff';
    ctx.beginPath();
    ctx.arc(ball.x, ball.y, ball.radius, 0, Math.PI * 2);
    ctx.fill();

    // small glow
    ctx.fillStyle = 'rgba(0,200,255,0.08)';
    ctx.beginPath();
    ctx.arc(ball.x, ball.y, ball.radius * 3, 0, Math.PI * 2);
    ctx.fill();

    // scores (also in DOM)
    // optional in-canvas score if you want
  }

  // Update AI for right paddle: simple tracking with max speed
  function updateAI() {
    // Move paddle center towards ball.y with limited speed
    const targetY = ball.y - rightPaddle.height / 2;
    const delta = targetY - rightPaddle.y;
    const move = clamp(delta, -aiMaxSpeed, aiMaxSpeed);
    rightPaddle.y += move;

    // keep inside bounds
    rightPaddle.y = clamp(rightPaddle.y, 0, height - rightPaddle.height);
  }

  // Main update loop
  let last = 0;
  function update(time) {
    const dt = Math.min((time - last) / 16.6667, 4); // normalize to ~60fps steps, clamp
    last = time;

    // keyboard control apply
    if (keys['ArrowUp']) leftPaddle.y -= paddleSpeed * dt;
    if (keys['ArrowDown']) leftPaddle.y += paddleSpeed * dt;

    // keep in bounds
    leftPaddle.y = clamp(leftPaddle.y, 0, height - leftPaddle.height);

    // AI update
    updateAI();

    // Move ball
    ball.x += ball.vx * dt;
    ball.y += ball.vy * dt;

    // Top/bottom collision
    if (ball.y - ball.radius <= 0) {
      ball.y = ball.radius;
      ball.vy = -ball.vy;
    } else if (ball.y + ball.radius >= height) {
      ball.y = height - ball.radius;
      ball.vy = -ball.vy;
    }

    // Paddle collisions
    // Left paddle
    if (ball.vx < 0 && circleRectCollision(ball.x, ball.y, ball.radius, leftPaddle.x, leftPaddle.y, leftPaddle.width, leftPaddle.height)) {
      // calculate relative intersection to determine bounce angle
      const paddleCenter = leftPaddle.y + leftPaddle.height / 2;
      const relativeY = (ball.y - paddleCenter) / (leftPaddle.height / 2);
      const maxBounce = Math.PI / 3; // 60 deg
      const bounceAngle = relativeY * maxBounce;
      const speed = Math.min(12, Math.hypot(ball.vx, ball.vy) * 1.05 + 0.2); // slightly increase speed
      ball.vx = Math.abs(Math.cos(bounceAngle) * speed);
      ball.vy = Math.sin(bounceAngle) * speed;
      // nudge ball outside paddle to avoid re-collision
      ball.x = leftPaddle.x + leftPaddle.width + ball.radius + 0.5;
    }

    // Right paddle
    if (ball.vx > 0 && circleRectCollision(ball.x, ball.y, ball.radius, rightPaddle.x, rightPaddle.y, rightPaddle.width, rightPaddle.height)) {
      const paddleCenter = rightPaddle.y + rightPaddle.height / 2;
      const relativeY = (ball.y - paddleCenter) / (rightPaddle.height / 2);
      const maxBounce = Math.PI / 3;
      const bounceAngle = relativeY * maxBounce;
      const speed = Math.min(12, Math.hypot(ball.vx, ball.vy) * 1.05 + 0.2);
      ball.vx = -Math.abs(Math.cos(bounceAngle) * speed);
      ball.vy = Math.sin(bounceAngle) * speed;
      ball.x = rightPaddle.x - ball.radius - 0.5;
    }

    // Score detection
    if (ball.x + ball.radius < 0) {
      // right scores
      rightScore++;
      updateScoreboard();
      if (rightScore >= winningScore) {
        // game over behavior (simple)
        setTimeout(() => alert('Computer wins! Refresh to play again.'), 40);
      }
      // reset ball towards the player who lost (left)
      resetBall(1); // send ball to right (so left has to defend)
      return; // short-circuit to avoid double-update in same frame
    } else if (ball.x - ball.radius > width) {
      // left scores
      leftScore++;
      updateScoreboard();
      if (leftScore >= winningScore) {
        setTimeout(() => alert('You win! Refresh to play again.'), 40);
      }
      resetBall(-1); // send ball to left
      return;
    }

    draw();
    requestAnimationFrame(update);
  }

  // initial render
  draw();
  updateScoreboard();
  requestAnimationFrame(update);

})();
