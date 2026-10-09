# sky-hopper
<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8" />
  <meta name="viewport" content="width=device-width, initial-scale=1.0, viewport-fit=cover, user-scalable=no" />
  <title>Sky Hopper</title>
  <style>
    :root {
      --bg1: #0f172a;
      --bg2: #1e293b;
      --panel: rgba(15, 23, 42, 0.72);
      --panel-border: rgba(148, 163, 184, 0.35);
      --text: #e2e8f0;
      --muted: #cbd5e1;
      --accent: #38bdf8;
      --accent2: #f59e0b;
      --danger: #fb7185;
      --good: #34d399;
    }

    * { box-sizing: border-box; }

    html, body {
      margin: 0;
      height: 100%;
      font-family: system-ui, -apple-system, BlinkMacSystemFont, "Segoe UI", sans-serif;
      background: linear-gradient(180deg, var(--bg1), var(--bg2));
      color: var(--text);
      overscroll-behavior: none;
      -webkit-tap-highlight-color: transparent;
    }

    body {
      display: flex;
      align-items: center;
      justify-content: center;
      padding: 0;
    }

    .game-wrap {
      position: relative;
      width: min(100vw, 440px);
      height: min(100vh, 900px);
      max-height: 100vh;
      background: linear-gradient(180deg, #0b1120 0%, #111827 100%);
      overflow: hidden;
      border: 1px solid rgba(148, 163, 184, 0.15);
      box-shadow: 0 30px 60px rgba(0,0,0,0.45);
    }

    canvas {
      display: block;
      width: 100%;
      height: 100%;
      touch-action: none;
      background: linear-gradient(180deg, #08111f 0%, #0f172a 40%, #1d3557 100%);
    }

    #hud {
      position: absolute;
      top: 14px;
      left: 0;
      right: 0;
      display: flex;
      justify-content: space-between;
      padding: 0 16px;
      pointer-events: none;
      z-index: 3;
    }

    .stat {
      background: rgba(15, 23, 42, 0.5);
      border: 1px solid rgba(148, 163, 184, 0.2);
      border-radius: 14px;
      min-width: 94px;
      padding: 8px 10px;
      backdrop-filter: blur(8px);
      -webkit-backdrop-filter: blur(8px);
    }

    .stat span {
      display: block;
      font-size: 10px;
      letter-spacing: 0.12em;
      text-transform: uppercase;
      color: var(--muted);
      opacity: 0.9;
    }

    .stat strong {
      font-size: 24px;
      line-height: 1.2;
      font-weight: 800;
    }

    .panel {
      position: absolute;
      inset: 0;
      display: flex;
      align-items: center;
      justify-content: center;
      z-index: 4;
      background: rgba(2, 6, 23, 0.38);
      backdrop-filter: blur(4px);
      -webkit-backdrop-filter: blur(4px);
      transition: opacity 0.2s ease;
    }

    .panel.hidden {
      opacity: 0;
      pointer-events: none;
    }

    .card {
      width: min(82vw, 320px);
      background: var(--panel);
      border: 1px solid var(--panel-border);
      border-radius: 22px;
      padding: 28px 22px 20px;
      text-align: center;
      box-shadow: 0 20px 50px rgba(15, 23, 42, 0.45);
    }

    h1, h2 {
      margin: 0 0 8px;
      line-height: 1.1;
      letter-spacing: -0.05em;
    }

    h1 {
      font-size: clamp(2.2rem, 7vw, 3.1rem);
    }

    h2 {
      font-size: clamp(1.8rem, 6vw, 2.5rem);
    }

    p {
      margin: 8px 0 18px;
      color: var(--muted);
      line-height: 1.5;
      font-size: 1rem;
    }

    button {
      border: 0;
      border-radius: 999px;
      padding: 14px 22px;
      font-size: 1rem;
      font-weight: 800;
      letter-spacing: 0.04em;
      color: #05131f;
      background: linear-gradient(135deg, var(--accent), #7dd3fc);
      box-shadow: 0 10px 25px rgba(56, 189, 248, 0.45);
      cursor: pointer;
      transition: transform 0.15s ease, box-shadow 0.15s ease;
    }

    button:active {
      transform: scale(0.97);
      box-shadow: 0 6px 18px rgba(56, 189, 248, 0.38);
    }

    .final-score {
      font-size: 1.15rem;
      color: var(--text);
      font-weight: 800;
    }
  </style>
</head>
<body>
  <div class="game-wrap">
    <canvas id="gameCanvas" width="360" height="640" aria-label="Sky Hopper game"></canvas>

    <div id="hud">
      <div class="stat">
        <span>Score</span>
        <strong id="scoreEl">0</strong>
      </div>
      <div class="stat">
        <span>Best</span>
        <strong id="bestEl">0</strong>
      </div>
    </div>

    <div id="startScreen" class="panel">
      <div class="card">
        <h1>Sky Hopper</h1>
        <p>Swipe left and right to dodge the barriers and collect glowing stars.</p>
        <button id="startBtn">Play</button>
      </div>
    </div>

    <div id="gameOverScreen" class="panel hidden">
      <div class="card">
        <h2>Game Over</h2>
        <p>Final score: <span id="finalScore" class="final-score">0</span></p>
        <button id="restartBtn">Retry</button>
      </div>
    </div>
  </div>

  <script>
    const canvas = document.getElementById('gameCanvas');
    const ctx = canvas.getContext('2d');

    const scoreEl = document.getElementById('scoreEl');
    const bestEl = document.getElementById('bestEl');
    const finalScoreEl = document.getElementById('finalScore');

    const startScreen = document.getElementById('startScreen');
    const gameOverScreen = document.getElementById('gameOverScreen');
    const startBtn = document.getElementById('startBtn');
    const restartBtn = document.getElementById('restartBtn');

    const bestKey = 'sky-hop-best-score';
    let bestScore = Number(localStorage.getItem(bestKey) || 0);

    bestEl.textContent = bestScore;

    const state = {
      running: false,
      gameOver: false,
      width: canvas.width,
      height: canvas.height,
      score: 0,
      time: 0,
      obstacleTimer: 0,
      starTimer: 0,
      targetX: canvas.width / 2,
      player: {
        x: canvas.width / 2,
        y: canvas.height - 92,
        radius: 18,
        vx: 0
      },
      obstacles: [],
      stars: [],
      particles: []
    };

    function clamp(value, min, max) {
      return Math.min(Math.max(value, min), max);
    }

    function resetGame() {
      state.running = false;
      state.gameOver = false;
      state.score = 0;
      state.time = 0;
      state.obstacleTimer = 0;
      state.starTimer = 0;
      state.obstacles = [];
      state.stars = [];
      state.particles = [];
      state.player.x = canvas.width / 2;
      state.player.y = canvas.height - 92;
      state.targetX = canvas.width / 2;

      scoreEl.textContent = '0';
      finalScoreEl.textContent = '0';
    }

    function startGame() {
      resetGame();
      state.running = true;
      startScreen.classList.add('hidden');
      gameOverScreen.classList.add('hidden');
    }

    function endGame() {
      if (state.gameOver) return;
      state.running = false;
      state.gameOver = true;

      if (state.score > bestScore) {
        bestScore = Math.floor(state.score);
        localStorage.setItem(bestKey, String(bestScore));
        bestEl.textContent = bestScore;
      }

      finalScoreEl.textContent = Math.floor(state.score);

      gameOverScreen.classList.remove('hidden');
    }

    function spawnObstacle() {
      const width = 48 + Math.random() * 60;
      const height = 28 + Math.random() * 38;
      const x = 18 + Math.random() * (canvas.width - width - 36);
      state.obstacles.push({
        x,
        y: -height,
        width,
        height,
        speed: 220 + Math.random() * 90
      });
    }

    function spawnStar() {
      const radius = 10 + Math.random() * 4;
      const x = 30 + Math.random() * (canvas.width - 60);
      state.stars.push({
        x,
        y: -radius * 2,
        radius,
        speed: 210 + Math.random() * 80,
        drift: (Math.random() - 0.5) * 30
      });
    }

    function addBurst(x, y, color) {
      for (let i = 0; i < 10; i++) {
        state.particles.push({
          x,
          y,
          vx: (Math.random() - 0.5) * 140,
          vy: (Math.random() - 0.5) * 140,
          life: 0.6 + Math.random() * 0.5,
          size: 3 + Math.random() * 4,
          color
        });
      }
    }

    function update(dt) {
      if (!state.running) return;

      state.time += dt;
      state.score += dt * 12;

      scoreEl.textContent = Math.floor(state.score);

      state.player.x += (state.targetX - state.player.x) * Math.min(1, dt * 12);
      state.player.x = clamp(state.player.x, state.player.radius + 8, canvas.width - state.player.radius - 8);

      state.obstacleTimer -= dt;
      state.starTimer -= dt;

      if (state.obstacleTimer <= 0) {
        spawnObstacle();
        state.obstacleTimer = 0.9 + Math.random() * 0.7;
      }

      if (state.starTimer <= 0) {
        spawnStar();
        state.starTimer = 1.8 + Math.random() * 1.2;
      }

      for (let i = state.obstacles.length - 1; i >= 0; i--) {
        const obs = state.obstacles[i];
        obs.y += obs.speed * dt;

        if (
          Math.abs(state.player.x - (obs.x + obs.width / 2)) < (state.player.radius + obs.width / 2 - 4) &&
          Math.abs(state.player.y - (obs.y + obs.height / 2)) < (state.player.radius + obs.height / 2 - 4)
        ) {
          addBurst(state.player.x, state.player.y, '#fb7185');
          endGame();
          return;
        }

        if (obs.y > canvas.height + obs.height) {
          state.obstacles.splice(i, 1);
        }
      }

      for (let i = state.stars.length - 1; i >= 0; i--) {
        const star = state.stars[i];
        star.y += star.speed * dt;
        star.x += star.drift * dt;

        if (
          Math.abs(star.x - state.player.x) < star.radius + state.player.radius &&
          Math.abs(star.y - state.player.y) < star.radius + state.player.radius
        ) {
          state.score += 25;
          addBurst(star.x, star.y, '#fbbf24');
          state.stars.splice(i, 1);
          continue;
        }

        if (star.y > canvas.height + star.radius * 3) {
          state.stars.splice(i, 1);
        }
      }

      for (let i = state.particles.length - 1; i >= 0; i--) {
        const p = state.particles[i];
        p.x += p.vx * dt;
        p.y += p.vy * dt;
        p.life -= dt;
        if (p.life <= 0) {
          state.particles.splice(i, 1);
        }
      }
    }

    function drawStar(x, y, radius, points = 5) {
      ctx.beginPath();
      for (let i = 0; i < points * 2; i++) {
        const angle = (Math.PI / points) * i - Math.PI / 2;
        const r = i % 2 === 0 ? radius : radius * 0.48;
        const px = x + Math.cos(angle) * r;
        const py = y + Math.sin(angle) * r;
        if (i === 0) ctx.moveTo(px, py);
        else ctx.lineTo(px, py);
      }
      ctx.closePath();
      ctx.fill();
    }

    function drawPlayer() {
      ctx.save();
      ctx.translate(state.player.x, state.player.y);

      ctx.beginPath();
      ctx.fillStyle = '#38bdf8';
      ctx.arc(0, 0, state.player.radius, 0, Math.PI * 2);
      ctx.fill();

      ctx.beginPath();
      ctx.fillStyle = '#e0f2fe';
      ctx.arc(-5, -6, 4, 0, Math.PI * 2);
      ctx.arc(5, -6, 4, 0, Math.PI * 2);
      ctx.fill();

      ctx.beginPath();
      ctx.strokeStyle = '#0f172a';
      ctx.lineWidth = 2.5;
      ctx.arc(0, 4, 7, 0.15 * Math.PI, 0.85 * Math.PI);
      ctx.stroke();

      ctx.restore();
    }

    function drawObstacle(obs) {
      ctx.fillStyle = '#f97316';
      ctx.fillRect(obs.x, obs.y, obs.width, obs.height);

      ctx.fillStyle = 'rgba(255,255,255,0.23)';
      ctx.fillRect(obs.x + 6, obs.y + 6, obs.width - 12, 10);
    }

    function drawStarPickup(star) {
      ctx.fillStyle = '#facc15';
      drawStar(star.x, star.y, star.radius);
    }

    function drawParticles() {
      for (const p of state.particles) {
        ctx.fillStyle = p.color;
        ctx.globalAlpha = Math.max(0, p.life);
        ctx.fillRect(p.x, p.y, p.size, p.size);
      }
      ctx.globalAlpha = 1;
    }

    function drawBackground() {
      ctx.clearRect(0, 0, canvas.width, canvas.height);

      const sky = ctx.createLinearGradient(0, 0, 0, canvas.height);
      sky.addColorStop(0, '#08111f');
      sky.addColorStop(0.45, '#0f172a');
      sky.addColorStop(1, '#1d3557');
      ctx.fillStyle = sky;
      ctx.fillRect(0, 0, canvas.width, canvas.height);

      const glow = ctx.createRadialGradient(0, 70, 20, 190, 40, 240);
      glow.addColorStop(0, 'rgba(56,189,248,0.55)');
      glow.addColorStop(1, 'rgba(56,189,248,0)');
      ctx.fillStyle = glow;
      ctx.fillRect(0, 0, canvas.width, canvas.height);

      // distant islands
      ctx.fillStyle = 'rgba(16, 185, 129, 0.10)';
      ctx.beginPath();
      ctx.moveTo(0, canvas.height * 0.72);
      ctx.lineTo(80, canvas.height * 0.66);
      ctx.lineTo(140, canvas.height * 0.75);
      ctx.lineTo(220, canvas.height * 0.64);
      ctx.lineTo(290, canvas.height * 0.72);
      ctx.lineTo(360, canvas.height * 0.68);
      ctx.lineTo(360, canvas.height);
      ctx.lineTo(0, canvas.height);
      ctx.closePath();
      ctx.fill();

      // ground shimmer
      ctx.fillStyle = 'rgba(255,255,255,0.04)';
      for (let i = 0; i < 12; i++) {
        const x = (i * 35 + (state.time * 20) % 40) % (canvas.width + 40);
        ctx.fillRect(x, canvas.height - 24 - (i % 3) * 8, 18, 2);
      }
    }

    function draw() {
      drawBackground();

      for (const star of state.stars) {
        drawStarPickup(star);
      }

      for (const obs of state.obstacles) {
        drawObstacle(obs);
      }

      drawPlayer();
      drawParticles();
    }

    let lastTime = 0;
    function gameLoop(timestamp) {
      const delta = Math.min((timestamp - lastTime) / 1000 || 0.016, 0.033);
      lastTime = timestamp;

      update(delta);
      draw();

      requestAnimationFrame(gameLoop);
    }

    function setPointerX(event) {
      const rect = canvas.getBoundingClientRect();
      const x = (event.clientX - rect.left) * (canvas.width / rect.width);
      state.targetX = clamp(x, 20, canvas.width - 20);
    }

    canvas.addEventListener('pointerdown', (event) => {
      setPointerX(event);
      if (!state.running && !state.gameOver) {
        startGame();
      }
    });

    canvas.addEventListener('pointermove', (event) => {
      if (state.running) setPointerX(event);
    });

    canvas.addEventListener('pointerup', () => {
      // no-op, keeps targetX stable
    });

    startBtn.addEventListener('click', startGame);
    restartBtn.addEventListener('click', () => {
      gameOverScreen.classList.add('hidden');
      startGame();
    });

    resetGame();
    requestAnimationFrame(gameLoop);
  </script>
</body>
</html>
