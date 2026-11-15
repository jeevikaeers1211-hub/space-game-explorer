<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8">
  <meta name="viewport" content="width=device-width, initial-scale=1">
  <title>Space Game Explorer</title>
  <style>
    * { box-sizing: border-box; }

    body {
      margin: 0;
      min-height: 100vh;
      overflow: hidden;
      background: #070b1c;
      color: #f5f7ff;
      font-family: system-ui, sans-serif;
    }

    canvas {
      display: block;
      width: 100vw;
      height: 100vh;
    }

    .hud {
      position: fixed;
      top: 16px;
      left: 16px;
      z-index: 2;
      width: min(320px, calc(100vw - 32px));
      padding: 16px;
      border: 1px solid #ffffff2b;
      border-radius: 16px;
      background: #101831d9;
      box-shadow: 0 8px 30px #0005;
      backdrop-filter: blur(8px);
    }

    .hud h1 {
      margin: 0 0 4px;
      font-size: 1.15rem;
    }

    .hud p {
      margin: 8px 0;
      color: #cbd5f5;
      font-size: .9rem;
    }

    #discovery {
      min-height: 64px;
      margin-top: 12px;
      padding: 10px;
      border-radius: 10px;
      background: #ffffff10;
      line-height: 1.4;
    }

    #progress {
      color: #a9bdff;
      font-weight: 700;
    }

    .controls {
      position: fixed;
      right: 20px;
      bottom: 20px;
      z-index: 2;
      display: grid;
      grid-template-columns: repeat(3, 58px);
      grid-template-rows: repeat(2, 58px);
      gap: 8px;
      touch-action: none;
      user-select: none;
    }

    .controls button {
      border: 1px solid #ffffff40;
      border-radius: 14px;
      background: #172343dd;
      color: white;
      font-size: 1.4rem;
      touch-action: none;
    }

    .controls button:active {
      background: #425ba0;
      transform: scale(.96);
    }

    [data-dir="up"] { grid-column: 2; grid-row: 1; }
    [data-dir="left"] { grid-column: 1; grid-row: 2; }
    [data-dir="down"] { grid-column: 2; grid-row: 2; }
    [data-dir="right"] { grid-column: 3; grid-row: 2; }

    @media (min-width: 800px) {
      .controls { opacity: .7; }
    }
  </style>
</head>
<body>
  <canvas id="space" aria-label="Space exploration game"></canvas>

  <section class="hud" aria-live="polite">
    <h1>🚀 Space Game Explorer</h1>
    <p>Explore the galaxy. Fly close to a planet to discover it.</p>
    <div id="progress">Discoveries: 0 / 5</div>
    <div id="discovery">Your mission begins here. What will you find?</div>
  </section>

  <div class="controls" aria-label="Touch movement controls">
    <button data-dir="up" aria-label="Move up">▲</button>
    <button data-dir="left" aria-label="Move left">◀</button>
    <button data-dir="down" aria-label="Move down">▼</button>
    <button data-dir="right" aria-label="Move right">▶</button>
  </div>

  <script>
    const canvas = document.querySelector("#space");
    const ctx = canvas.getContext("2d");
    const progress = document.querySelector("#progress");
    const discovery = document.querySelector("#discovery");

    const player = { x: 0, y: 0, speed: 250 };
    const keys = new Set();
    const found = new Set();

    const planets = [
      { name: "Mercury", x: 500, y: -180, r: 34, color: "#b9a58d",
        fact: "Mercury is the smallest planet and the closest to the Sun." },
      { name: "Venus", x: 1100, y: 220, r: 54, color: "#e5aa62",
        fact: "Venus has a thick atmosphere and is the hottest planet." },
      { name: "Earth", x: -850, y: 400, r: 58, color: "#3989e8",
        fact: "Earth is the only world we know of that supports life." },
      { name: "Mars", x: 200, y: 1050, r: 46, color: "#d95d46",
        fact: "Mars is a rocky planet with giant volcanoes and ancient river valleys." },
      { name: "Jupiter", x: -1200, y: -750, r: 90, color: "#d9ad83",
        fact: "Jupiter is the largest planet and has a giant storm called the Great Red Spot." }
    ];

    const stars = Array.from({ length: 180 }, () => ({
      x: Math.random(),
      y: Math.random(),
      size: Math.random() * 1.8 + .4,
      alpha: Math.random() * .65 + .25
    }));

    function resize() {
      const dpr = Math.min(window.devicePixelRatio || 1, 2);
      canvas.width = Math.floor(innerWidth * dpr);
      canvas.height = Math.floor(innerHeight * dpr);
      ctx.setTransform(dpr, 0, 0, dpr, 0, 0);
    }
    window.addEventListener("resize", resize);
    resize();

    function setDirection(dir, pressed) {
      if (pressed) keys.add(dir);
      else keys.delete(dir);
    }

    const keyDirections = {
      ArrowUp: "up", w: "up", W: "up",
      ArrowDown: "down", s: "down", S: "down",
      ArrowLeft: "left", a: "left", A: "left",
      ArrowRight: "right", d: "right", D: "right"
    };

    window.addEventListener("keydown", event => {
      const dir = keyDirections[event.key];
      if (dir) {
        event.preventDefault();
        setDirection(dir, true);
      }
    });

    window.addEventListener("keyup", event => {
      const dir = keyDirections[event.key];
      if (dir) setDirection(dir, false);
    });

    document.querySelectorAll(".controls button").forEach(button => {
      const dir = button.dataset.dir;
      button.addEventListener("pointerdown", event => {
        event.preventDefault();
        button.setPointerCapture(event.pointerId);
        setDirection(dir, true);
      });
      const stop = () => setDirection(dir, false);
      button.addEventListener("pointerup", stop);
      button.addEventListener("pointercancel", stop);
      button.addEventListener("lostpointercapture", stop);
    });

    function drawBackground(width, height) {
      ctx.fillStyle = "#070b1c";
      ctx.fillRect(0, 0, width, height);

      for (const star of stars) {
        ctx.globalAlpha = star.alpha;
        ctx.fillStyle = "#fff";
        ctx.beginPath();
        ctx.arc(star.x * width, star.y * height, star.size, 0, Math.PI * 2);
        ctx.fill();
      }
      ctx.globalAlpha = 1;
    }

    function drawPlanet(planet, cx, cy) {
      const x = cx + planet.x - player.x;
      const y = cy + planet.y - player.y;

      // Skip planets that are far outside the visible area.
      if (x < -150 || x > innerWidth + 150 || y < -150 || y > innerHeight + 150) return;

      const glow = ctx.createRadialGradient(x, y, planet.r * .5, x, y, planet.r * 1.8);
      glow.addColorStop(0, planet.color + "55");
      glow.addColorStop(1, planet.color + "00");
      ctx.fillStyle = glow;
      ctx.beginPath();
      ctx.arc(x, y, planet.r * 1.8, 0, Math.PI * 2);
      ctx.fill();

      ctx.fillStyle = planet.color;
      ctx.beginPath();
      ctx.arc(x, y, planet.r, 0, Math.PI * 2);
      ctx.fill();

      ctx.fillStyle = "#ffffff30";
      ctx.beginPath();
      ctx.ellipse(x - planet.r * .25, y - planet.r * .3,
        planet.r * .3, planet.r * .12, -.5, 0, Math.PI * 2);
      ctx.fill();

      ctx.fillStyle = "#f5f7ff";
      ctx.font = "14px system-ui, sans-serif";
      ctx.textAlign = "center";
      ctx.fillText(found.has(planet.name) ? `✓ ${planet.name}` : planet.name,
        x, y + planet.r + 22);
    }

    function drawRocket(cx, cy) {
      ctx.save();
      ctx.translate(cx, cy);

      // Small exhaust flame
      if (keys.size > 0) {
        ctx.fillStyle = "#ffb347";
        ctx.beginPath();
        ctx.moveTo(-7, 18);
        ctx.lineTo(0, 34 + Math.random() * 8);
        ctx.lineTo(7, 18);
        ctx.fill();
      }

      ctx.fillStyle = "#f4f6ff";
      ctx.beginPath();
      ctx.moveTo(0, -22);
      ctx.lineTo(14, 17);
      ctx.lineTo(0, 11);
      ctx.lineTo(-14, 17);
      ctx.closePath();
      ctx.fill();

      ctx.fillStyle = "#71c8ff";
      ctx.beginPath();
      ctx.arc(0, 2, 5, 0, Math.PI * 2);
      ctx.fill();

      ctx.restore();
    }

    function update(delta) {
      let dx = Number(keys.has("right")) - Number(keys.has("left"));
      let dy = Number(keys.has("down")) - Number(keys.has("up"));
      const length = Math.hypot(dx, dy) || 1;

      player.x += (dx / length) * player.speed * delta;
      player.y += (dy / length) * player.speed * delta;

      for (const planet of planets) {
        const distance = Math.hypot(planet.x - player.x, planet.y - player.y);
        if (distance < planet.r + 95 && !found.has(planet.name)) {
          found.add(planet.name);
          progress.textContent = `Discoveries: ${found.size} / ${planets.length}`;
          discovery.textContent = `🪐 ${planet.name}: ${planet.fact}`;
          if (found.size === planets.length) {
            discovery.textContent = "🌟 You discovered every planet! Great exploring, astronaut!";
          }
        }
      }
    }

    let previousTime = 0;
    function frame(time) {
      const delta = Math.min((time - previousTime) / 1000 || 0, 0.04);
      previousTime = time;

      update(delta);
      drawBackground(innerWidth, innerHeight);

      const cx = innerWidth / 2;
      const cy = innerHeight / 2;
      planets.forEach(planet => drawPlanet(planet, cx, cy));
      drawRocket(cx, cy);

      requestAnimationFrame(frame);
    }
    requestAnimationFrame(frame);
  </script>
</body>
</html># space-game-explorer
