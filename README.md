<!DOCTYPE html>
<html lang="pt-br">
<head>
  <meta charset="UTF-8" />
  <meta name="viewport" content="width=device-width, initial-scale=1" />
  <title>Neon Racer ULTIMATE</title>
  <style>
    :root {
      --bg: #0a0b10;
      --road: #1a1e29
      --lane: #2a3040;
      --stripe: #dadde6;
      --accent: #6cf;
      --car: #2ee6a6;
      --obstacle: #ff5577;
      --power-nitro: #00d0ff;
      --power-shield: #ffd54a;
      --text: #e8eef9;
      --muted: #9aa6bf;
      --shadow: 0 8px 30px rgba(0,0,0,.35);
      --radius: 18px;
    }

    * { box-sizing: border-box; }
    html, body { height: 100%; }
    body {
      margin: 0;
      font-family: ui-sans-serif, system-ui, -apple-system, Segoe UI, Roboto, Ubuntu, "Helvetica Neue", Arial, "Noto Sans", "Apple Color Emoji", "Segoe UI Emoji";
      color: var(--text);
      background: radial-gradient(1200px 800px at 50% -200px, #111627 0%, #0b0e17 40%, #07080d 100%);
      display: grid;
      place-items: center;
    }

    .game-wrap {
      width: min(92vw, 520px);
      aspect-ratio: 9/16;
      position: relative;
      border-radius: var(--radius);
      overflow: hidden;
      background: linear-gradient(180deg, #0a0b10 0%, #0a0b10 30%, #0a0b10 100%);
      box-shadow: var(--shadow);
      outline: 1px solid rgba(255,255,255,.06);
    }

    /* HUD */
    .hud {
      position: absolute;
      inset: 0;
      pointer-events: none;
      display: grid;
      grid-template-rows: auto 1fr auto;
    }
    .hud-top {
      display: flex; gap: 10px; align-items: center; justify-content: space-between;
      padding: 12px 14px;
      flex-wrap: wrap;
    }
    .pill {
      pointer-events: auto;
      background: rgba(255,255,255,.06);
      border: 1px solid rgba(255,255,255,.08);
      padding: 8px 12px;
      border-radius: 999px;
      backdrop-filter: blur(6px);
      font-weight: 600;
      letter-spacing: .2px;
      display: inline-flex; align-items: center; gap: 8px;
    }
    .muted { color: var(--muted); font-weight: 500; }

    .btn {
      pointer-events: auto;
      border: 1px solid rgba(255,255,255,.14);
      background: linear-gradient(180deg, rgba(255,255,255,.08), rgba(255,255,255,.02));
      color: var(--text);
      padding: 10px 14px;
      border-radius: 12px;
      font-weight: 700;
      cursor: pointer;
      transition: transform .06s ease, filter .2s ease, background .2s ease;
      user-select: none;
    }
    .btn:hover { filter: brightness(1.08); }
    .btn:active { transform: translateY(1px) scale(.98); }

    .btn.primary { border-color: rgba(108, 204, 255, .5); box-shadow: inset 0 0 0 1px rgba(108,204,255,.25); }
    .btn.danger { border-color: rgba(255, 85, 119, .45); box-shadow: inset 0 0 0 1px rgba(255,85,119,.25); }

    .title { font-size: 14px; font-weight: 600; letter-spacing: .3px; opacity: .9; }

    .hud-bottom { display: flex; justify-content: space-between; gap: 10px; padding: 12px; }

    /* Touch controls */
    .controls { display: flex; gap: 10px; }
    .control-btn {
      pointer-events: auto;
      width: 64px; height: 64px; border-radius: 16px;
      display: grid; place-items: center; font-size: 28px; font-weight: 900;
      background: rgba(255,255,255,.06);
      border: 1px solid rgba(255,255,255,.08);
      backdrop-filter: blur(6px);
      user-select: none; cursor: pointer;
      transition: transform .06s ease, background .2s ease;
    }
    .control-btn:active { transform: scale(.96); background: rgba(255,255,255,.1); }

    /* Center overlays */
    .overlay {
      position: absolute; inset: 0; display: grid; place-items: center; text-align: center;
      background: linear-gradient(180deg, rgba(7,8,13,.0), rgba(7,8,13,.5) 60%, rgba(7,8,13,.75));
      padding: 20px;
    }
    .card {
      width: min(92%, 420px);
      background: linear-gradient(180deg, rgba(255,255,255,.06), rgba(255,255,255,.02));
      border: 1px solid rgba(255,255,255,.12);
      border-radius: 20px; box-shadow: var(--shadow);
      padding: 20px; display: grid; gap: 14px;
    }
    .card h1 { margin: 0; font-size: 28px; }
    .card p { margin: 0; color: var(--muted); }
    .row { display: flex; gap: 10px; justify-content: center; flex-wrap: wrap; }

    canvas { display: block; width: 100%; height: 100%; background: var(--bg); }

    .frame { pointer-events: none; position: absolute; inset: 0; border: 6px solid rgba(255,255,255,.06); border-radius: var(--radius); box-shadow: inset 0 0 60px rgba(0,0,0,.35); }

    /* Power-up Badges */
    .badge { display: inline-flex; align-items: center; gap: 6px; padding: 6px 10px; border-radius: 999px; border: 1px solid rgba(255,255,255,.12); background: rgba(255,255,255,.05); }
    .dot { width: 10px; height: 10px; border-radius: 50%; }
    .dot.nitro { background: var(--power-nitro); box-shadow: 0 0 12px var(--power-nitro); }
    .dot.shield { background: var(--power-shield); box-shadow: 0 0 12px var(--power-shield); }
  </style>
</head>
<body>
  <div class="game-wrap" id="game-wrap">
    <canvas id="game"></canvas>

    <audio id="bgm" loop preload="none">
      <!-- Música livre/royalty-free (pode trocar o link se quiser) -->
      <source src="https://cdn.pixabay.com/download/audio/2021/11/09/audio_3a71e0d6b7.mp3?filename=cyberpunk-moonlight-20854.mp3" type="audio/mpeg">
    </audio>

    <div class="hud">
      <div class="hud-top">
        <div class="pill" id="score-pill">🏁 <span class="muted">Pontos</span> <span id="score">0</span></div>
        <div class="pill" id="speed-pill">⚡ <span class="muted">Vel.</span> <span id="speed">0</span></div>
        <div class="pill" id="best-pill" title="Seu melhor">⭐ <span class="muted">Recorde</span> <span id="best">0</span></div>
        <div class="badge" id="nitro-badge" title="Nitro ativo" style="opacity:.5;">
          <span class="dot nitro"></span>
          <span>Nitro</span>
          <span id="nitro-time" class="muted">0.0s</span>
        </div>
        <div class="badge" id="shield-badge" title="Escudo ativo" style="opacity:.5;">
          <span class="dot shield"></span>
          <span>Escudo</span>
          <span id="shield-time" class="muted">—</span>
        </div>
        <div class="pill" id="weather-pill" title="Clima atual">🌤️ <span id="weather">limpo</span></div>
        <div class="pill" id="time-pill" title="Ciclo dia/noite">🕑 <span id="tod">dia</span></div>
        <button class="btn" id="btn-sound" aria-label="Som">🔊 Som</button>
      </div>
      <div></div>
      <div class="hud-bottom">
        <div class="controls">
          <div class="control-btn" id="btn-left" aria-label="Esquerda">◀</div>
          <div class="control-btn" id="btn-right" aria-label="Direita">▶</div>
        </div>
        <div class="row">
          <button class="btn" id="btn-pause" aria-label="Pausar (P)">Pausar</button>
          <button class="btn primary" id="btn-start" aria-label="Iniciar/Retomar (Enter)">Jogar</button>
          <button class="btn danger" id="btn-restart" aria-label="Reiniciar (R)">Reiniciar</button>
        </div>
      </div>
    </div>

    <div class="overlay" id="start-overlay">
      <div class="card">
        <h1>🏎️ Neon Racer — Turbo</h1>
        <p>Desvie dos carros, colete power-ups e marque pontos. A velocidade aumenta com o tempo!</p>
        <p><strong>Controles</strong>: ⬅️ ➡️ ou A/D • Toque nos botões • P = Pausar • R = Reiniciar</p>
        <div class="row">
          <button class="btn primary" id="start-play">Começar</button>
          <button class="btn" id="start-muted">Silenciar</button>
        </div>
      </div>
    </div>

    <div class="overlay" id="gameover-overlay" style="display:none;">
      <div class="card">
        <h1>💥 Fim de Jogo</h1>
        <p>Pontuação: <strong id="final-score">0</strong> • Recorde: <strong id="final-best">0</strong></p>
        <div class="row">
          <button class="btn primary" id="again">Jogar de novo</button>
          <button class="btn" id="share">Compartilhar</button>
        </div>
      </div>
    </div>

    <div class="frame" aria-hidden="true"></div>
  </div>

  <script>
  (function(){
    const canvas = document.getElementById('game');
    const ctx = canvas.getContext('2d');
    const wrap = document.getElementById('game-wrap');

    // HUD elements
    const scoreEl = document.getElementById('score');
    const speedEl = document.getElementById('speed');
    const bestEl  = document.getElementById('best');
    const best = Number(localStorage.getItem('neonRacerBest')||0);
    bestEl.textContent = best;

    const nitroBadge = document.getElementById('nitro-badge');
    const shieldBadge = document.getElementById('shield-badge');
    const nitroTimeEl = document.getElementById('nitro-time');
    const shieldTimeEl = document.getElementById('shield-time');

    const weatherEl = document.getElementById('weather');
    const todEl = document.getElementById('tod');

    const startOverlay = document.getElementById('start-overlay');
    const gameoverOverlay = document.getElementById('gameover-overlay');
    const finalScoreEl = document.getElementById('final-score');
    const finalBestEl = document.getElementById('final-best');

    const btnStartTop = document.getElementById('btn-start');
    const btnPause = document.getElementById('btn-pause');
    const btnRestart = document.getElementById('btn-restart');
    const btnLeft = document.getElementById('btn-left');
    const btnRight = document.getElementById('btn-right');
    const btnSound = document.getElementById('btn-sound');

    const startPlay = document.getElementById('start-play');
    const startMuted = document.getElementById('start-muted');
    const againBtn = document.getElementById('again');
    const shareBtn = document.getElementById('share');

    // Audio: music tag + WebAudio SFX
    const bgm = document.getElementById('bgm');
    const audioCtx = new (window.AudioContext || window.webkitAudioContext)();
    let muted = false;

    function beep(freq = 660, dur = 0.08, type='sine', gain=0.03){
      if(muted) return;
      const o = audioCtx.createOscillator();
      const g = audioCtx.createGain();
      o.type = type; o.frequency.value = freq;
      g.gain.value = 0;
      o.connect(g); g.connect(audioCtx.destination);
      o.start();
      const now = audioCtx.currentTime;
      g.gain.linearRampToValueAtTime(gain, now + 0.01);
      g.gain.exponentialRampToValueAtTime(0.0001, now + dur);
      o.stop(now + dur + 0.02);
    }

    function sfxPower(){ beep(1040, .06, 'triangle', .04); setTimeout(()=>beep(1320,.06,'triangle',.03), 40); }
    function sfxShield(){ beep(520, .1, 'sine', .05); setTimeout(()=>beep(340,.08,'sine',.04), 60); }
    function sfxCrash(){ beep(120, .25, 'square', .06); }

    // Game coordinate system
    const W = 360, H = 640;
    function fitCanvas(){
      const dpr = Math.min(2, window.devicePixelRatio || 1);
      canvas.width = Math.floor(W * dpr);
      canvas.height = Math.floor(H * dpr);
      canvas.style.width = '100%';
      canvas.style.height = '100%';
      ctx.setTransform(dpr, 0, 0, dpr, 0, 0);
    }
    fitCanvas();
    window.addEventListener('resize', fitCanvas);

    // Road config
    const lanes = 3;
    const roadX = 40;      // left margin
    const roadW = W - roadX*2;
    const laneW = roadW / lanes;

    // Player
    const player = { w: 44, h: 88, lane: 1, x: 0, y: 0, color: getCss('--car') };

    // Entities
    const obstacles = [];
    const powerups = []; // {type:'nitro'|'shield', lane, x, y, r}

    // Weather / Day-Night
    const PRECIP_MAX = 120;
    const precip = []; // particles
    let weather = 'limpo'; // 'limpo' | 'chuva' | 'neve' | 'neblina'
    let weatherTimer = 0;
    let fogOffset = 0;

    let skyT = Math.random(); // 0..1 (dia/noite)
    const DAYNIGHT_SPEED = 0.02; // ciclos ~50s

    function rollWeather(){
      const types = ['limpo','chuva','neve','neblina'];
      weather = types[Math.floor(Math.random()*types.length)];
      weatherTimer = 15 + Math.random()*20;
      buildPrecip();
    }

    function buildPrecip(){
      precip.length = 0;
      if(weather==='chuva'){
        const n = 90;
        for(let i=0;i<n;i++){
          precip.push({ x: Math.random()*W, y: Math.random()*H, vy: 380+Math.random()*300, len: 10+Math.random()*12 });
        }
      } else if(weather==='neve'){
        const n = 80;
        for(let i=0;i<n;i++){
          precip.push({ x: Math.random()*W, y: Math.random()*H, vy: 40+Math.random()*60, r: 1+Math.random()*2, drift: -20+Math.random()*40 });
        }
      }
    }

    // State
    let score = 0;
    let speed = 220;          // base px/s
    let alive = false;
    let paused = false;
    let last = 0;
    let spawnTimer = 0;
    let stripeOffset = 0;

    // Power states
    const NITRO_DURATION = 5.0; // seconds
    let nitroUntil = 0;         // timestamp (performance.now()/1000)
    let shieldActive = false;   // true if has shield

    function laneCenter(i){ return roadX + laneW*i + laneW/2; }

    function reset(){
      score = 0; speed = 220; alive = false; paused = false; last = 0; spawnTimer = 0; stripeOffset = 0;
      obstacles.length = 0; powerups.length = 0;
      player.lane = 1;
      player.x = laneCenter(player.lane);
      player.y = H - 120;
      nitroUntil = 0; shieldActive = false;
      rollWeather();
      updateHUD(0);
      drawScene(0);
    }

    function startGame(){
      if(alive) return;
      alive = true; paused = false; last = performance.now();
      startOverlay.style.display = 'none';
      gameoverOverlay.style.display = 'none';
      btnPause.textContent = 'Pausar';
      requestAnimationFrame(loop);
      try { audioCtx.resume(); } catch {}
      if(!muted){ try { bgm.play().catch(()=>{}); } catch {} }
    }

    function gameOver(){
      alive = false; paused = false;
      finalScoreEl.textContent = Math.floor(score);
      const bestPrev = Number(localStorage.getItem('neonRacerBest')||0);
      const newBest = Math.max(bestPrev, Math.floor(score));
      localStorage.setItem('neonRacerBest', String(newBest));
      bestEl.textContent = newBest;
      finalBestEl.textContent = newBest;
      gameoverOverlay.style.display = 'grid';
      sfxCrash();
      try { bgm.pause(); } catch {}
    }

    function updateHUD(nowSec){
      scoreEl.textContent = Math.floor(score);
      const mult = isNitro(nowSec) ? 1.7 : 1.0;
      speedEl.textContent = Math.round(speed * mult);

      // Nitro badge
      const remaining = Math.max(0, nitroUntil - nowSec);
      nitroBadge.style.opacity = remaining > 0 ? 1 : .5;
      nitroTimeEl.textContent = remaining > 0 ? remaining.toFixed(1)+'s' : '0.0s';

      // Shield badge
      shieldBadge.style.opacity = shieldActive ? 1 : .5;
      shieldTimeEl.textContent = shieldActive ? 'ativo' : '—';

      // Weather/Time
      weatherEl.textContent = weather;
      todEl.textContent = skyLight() < 0.5 ? 'noite' : 'dia';
    }

    function spawnObstacle(){
      const used = obstacles.length ? obstacles[obstacles.length-1].lane : -1;
      let l = Math.floor(Math.random()*lanes);
      if(l === used) l = (l+1)%lanes;
      const o = { lane: l, x: laneCenter(l), y: -100, w: 44, h: 88, color: getCss('--obstacle') };
      obstacles.push(o);

      // Chance de power-up logo após obstáculo
      if(Math.random() < 0.33){
        const type = Math.random() < 0.55 ? 'nitro' : 'shield';
        const laneIndex = Math.floor(Math.random()*lanes);
        powerups.push({ type, lane: laneIndex, x: laneCenter(laneIndex), y: -220, r: 16 });
      }
    }

    function drawCar(x, y, w, h, color){
      ctx.save();
      ctx.shadowColor = color.trim();
      ctx.shadowBlur = 18;
      ctx.fillStyle = color;
      roundRect(x - w/2, y - h/2, w, h, 10, true, false);
      ctx.fillStyle = 'rgba(255,255,255,.15)';
      roundRect(x - w*0.32, y - h*0.28, w*0.64, h*0.22, 8, true, false);
      ctx.fillStyle = 'rgba(255,255,255,.85)';
      ctx.fillRect(x - w*0.36, y - h*0.5, w*0.24, 4);
      ctx.fillRect(x + w*0.12, y - h*0.5, w*0.24, 4);
      ctx.restore();
    }

    function drawPowerup(p){
      ctx.save();
      const glow = p.type === 'nitro' ? getCss('--power-nitro') : getCss('--power-shield');
      const grad = ctx.createRadialGradient(p.x, p.y, 2, p.x, p.y, p.r+12);
      grad.addColorStop(0, 'rgba(255,255,255,.9)');
      grad.addColorStop(0.4, p.type==='nitro' ? 'rgba(0,208,255,.9)' : 'rgba(255,213,74,.9)');
      grad.addColorStop(1, 'rgba(0,0,0,0)');
      ctx.fillStyle = grad;
      ctx.shadowColor = glow.trim();
      ctx.shadowBlur = 20;
      ctx.beginPath();
      ctx.arc(p.x, p.y, p.r, 0, Math.PI*2);
      ctx.fill();

      // Ícone
      ctx.fillStyle = 'rgba(0,0,0,.25)';
      ctx.beginPath();
      if(p.type==='nitro'){
        // raio ⚡ simples
        const s = p.r*0.9;
        ctx.moveTo(p.x-4, p.y- s*0.6);
        ctx.lineTo(p.x+2, p.y-2);
        ctx.lineTo(p.x-2, p.y-2);
        ctx.lineTo(p.x+4, p.y+ s*0.6);
        ctx.closePath();
      } else {
        // escudo
        ctx.moveTo(p.x, p.y-8);
        ctx.quadraticCurveTo(p.x+10, p.y-4, p.x+8, p.y+6);
        ctx.lineTo(p.x, p.y+12);
        ctx.lineTo(p.x-8, p.y+6);
        ctx.quadraticCurveTo(p.x-10, p.y-4, p.x, p.y-8);
        ctx.closePath();
      }
      ctx.fill();
      ctx.restore();
    }

    function drawSky(){
      // céu gradiente baseado em skyT (0..1)
      const l = skyLight(); // 0 noite – 1 dia
      const top = lerpColor([8,10,20], [30,60,130], l);   // topo do céu
      const bot = lerpColor([4,5,10], [120,160,220], l);  // horizonte
      const g = ctx.createLinearGradient(0,0,0,H);
      g.addColorStop(0, `rgb(${top[0]},${top[1]},${top[2]})`);
      g.addColorStop(1, `rgb(${bot[0]},${bot[1]},${bot[2]})`);
      ctx.fillStyle = g;
      ctx.fillRect(0,0,W,H);

      // leve escurecimento noturno
      if(l < 0.55){
        ctx.fillStyle = `rgba(0,0,20,${(0.55-l)*0.9})`;
        ctx.fillRect(0,0,W,H);
      }
    }

    function drawScene(dt){
      // Sky & ambience first
      drawSky();

      // road base
      ctx.fillStyle = getCss('--road');
      ctx.fillRect(roadX, 0, roadW, H);

      // lane separators
      ctx.strokeStyle = getCss('--lane');
      ctx.lineWidth = 2;
      for(let i=1;i<lanes;i++){
        const x = roadX + laneW*i;
        ctx.beginPath(); ctx.moveTo(x,0); ctx.lineTo(x,H); ctx.stroke();
      }

      // moving stripes
      const stripeH = 22, gap = 32;
      ctx.fillStyle = getCss('--stripe');
      const stripeAlpha = 0.7 + 0.3*skyLight(); // mais fraco à noite
      ctx.globalAlpha = stripeAlpha;
      for(let i=0;i<lanes;i++){
        const cx = laneCenter(i);
        let y = - (stripeOffset % (stripeH+gap));
        while(y < H){ ctx.fillRect(cx - 3, y, 6, stripeH); y += stripeH + gap; }
      }
      ctx.globalAlpha = 1;

      // powerups
      for(const p of powerups){ drawPowerup(p); }

      // obstacles
      for(const o of obstacles){ drawCar(o.x, o.y, o.w, o.h, o.color); }

      // player (with shield aura)
      if(shieldActive){
        ctx.save();
        ctx.strokeStyle = getCss('--power-shield');
        ctx.globalAlpha = 0.75;
        ctx.lineWidth = 3;
        ctx.beginPath(); ctx.arc(player.x, player.y, 56, 0, Math.PI*2); ctx.stroke();
        ctx.restore();
      }
      drawCar(player.x, player.y, player.w, player.h, player.color);

      // precipitation overlay
      drawWeatherOverlays();

      // headlights at night
      const l = skyLight();
      if(l < 0.5){
        ctx.save();
        const rg = ctx.createRadialGradient(player.x, player.y, 10, player.x, player.y, 140);
        rg.addColorStop(0, 'rgba(255,255,220,0.5)');
        rg.addColorStop(1, 'rgba(255,255,220,0)');
        ctx.fillStyle = rg; ctx.globalCompositeOperation = 'lighter';
        ctx.beginPath(); ctx.arc(player.x, player.y, 140, 0, Math.PI*2); ctx.fill();
        ctx.globalCompositeOperation = 'source-over';
        ctx.restore();
      }
    }

    function drawWeatherOverlays(){
      if(weather==='chuva'){
        ctx.save();
        ctx.strokeStyle = 'rgba(180,200,255,0.6)';
        ctx.lineWidth = 1;
        ctx.beginPath();
        for(const p of precip){
          ctx.moveTo(p.x, p.y);
          ctx.lineTo(p.x - 3, p.y + p.len);
        }
        ctx.stroke();
        ctx.restore();
      } else if(weather==='neve'){
        ctx.save();
        ctx.fillStyle = 'rgba(255,255,255,0.9)';
        for(const f of precip){ ctx.beginPath(); ctx.arc(f.x, f.y, f.r, 0, Math.PI*2); ctx.fill(); }
        ctx.restore();
      }

      if(weather==='neblina'){
        ctx.save();
        fogOffset += 0.002 * H; // leve movimento
        const g = ctx.createLinearGradient(0,0,W,0);
        g.addColorStop(0, 'rgba(255,255,255,0.06)');
        g.addColorStop(0.5, 'rgba(255,255,255,0.16)');
        g.addColorStop(1, 'rgba(255,255,255,0.06)');
        ctx.fillStyle = g;
        ctx.fillRect(-((fogOffset)%W), 0, W*2, H);
        ctx.restore();
      }
    }

    function skyLight(){
      // 0..1 onde 1 = dia claro, 0 = noite
      // usa curva senoidal para amanhecer/entardecer suaves
      const v = 0.5 + 0.5*Math.sin((skyT*2*Math.PI) - Math.PI/2);
      return Math.max(0, Math.min(1, v));
    }

    function isNitro(nowSec){ return nowSec < nitroUntil; }

    function loop(t){
      if(!alive) return;
      const dt = Math.min(32, t - last) / 1000; // seconds
      last = t;
      if(paused){ requestAnimationFrame(loop); return; }

      const nowSec = t / 1000;

      // ambiente
      skyT = (skyT + dt*DAYNIGHT_SPEED) % 1;
      weatherTimer -= dt;
      if(weatherTimer <= 0){ rollWeather(); }

      // update
      const speedMult = isNitro(nowSec) ? 1.7 : 1.0;
      // redução leve de visibilidade/aderência em clima ruim
      const envPenalty = (weather==='chuva'||weather==='neve') ? 0.95 : 1.0;
      const roadSpeed = speed * speedMult * envPenalty;

      // precipitação
      if(weather==='chuva'){
        for(const p of precip){ p.x -= 120*dt; p.y += p.vy*dt; if(p.y>H+20){ p.x = Math.random()*W; p.y = -20; } }
      } else if(weather==='neve'){
        for(const f of precip){ f.x += f.drift*dt; f.y += f.vy*dt; if(f.y>H+10){ f.x = Math.random()*W; f.y = -10; } }
      }

      stripeOffset += roadSpeed * dt * 0.9;
      spawnTimer -= dt;
      if(spawnTimer <= 0){
        spawnObstacle();
        const base = Math.max(0.5, 1.05 - speed/800);
        spawnTimer = base + Math.random()*0.35;
      }

      // move entities
      for(const o of obstacles){ o.y += roadSpeed * dt; }
      for(const p of powerups){ p.y += roadSpeed * dt * 0.9; }

      // cleanup off-screen
      while(obstacles.length && obstacles[0].y - obstacles[0].h/2 > H+40){ obstacles.shift(); score += 10; beep(990, 0.04, 'triangle', 0.02); }
      while(powerups.length && powerups[0].y - powerups[0].r > H+40){ powerups.shift(); }

      // difficulty ramp
      speed += dt * 6;

      // collect powerups
      for(let i=powerups.length-1; i>=0; i--){
        const p = powerups[i];
        if(Math.abs(player.x - p.x) < (player.w/2 + p.r - 8) && Math.abs(player.y - p.y) < (player.h/2 + p.r - 8)){
          if(p.type==='nitro'){ nitroUntil = nowSec + NITRO_DURATION; sfxPower(); }
          else { shieldActive = true; sfxShield(); }
          powerups.splice(i,1);
        }
      }

      // collisions with obstacles
      let collided = false;
      for(let i=0;i<obstacles.length;i++){
        if(rectsOverlap(player, obstacles[i])){ collided = true; break; }
      }
      if(collided){
        if(shieldActive){
          // consome o escudo e limpa os obstáculos próximos
          shieldActive = false;
          sfxShield();
          obstacles.sort((a,b)=>Math.abs(a.y-player.y)-Math.abs(b.y-player.y));
          obstacles.shift();
        } else {
          gameOver();
          return;
        }
      }

      updateHUD(nowSec);
      drawScene(dt);
      requestAnimationFrame(loop);
    }

    function rectsOverlap(a,b){
      return Math.abs(a.x - b.x) < (a.w + b.w)/2 - 6 && Math.abs(a.y - b.y) < (a.h + b.h)/2 - 6;
    }

    function getCss(varName){ return getComputedStyle(document.documentElement).getPropertyValue(varName); }

    function roundRect(x, y, w, h, r, fill, stroke){
      const rr = Math.min(r, w/2, h/2);
      ctx.beginPath();
      ctx.moveTo(x+rr, y);
      ctx.arcTo(x+w, y, x+w, y+h, rr);
      ctx.arcTo(x+w, y+h, x, y+h, rr);
      ctx.arcTo(x, y+h, x, y, rr);
      ctx.arcTo(x, y, x+w, y, rr);
      ctx.closePath();
      if(fill) ctx.fill();
      if(stroke) ctx.stroke();
    }

    // Controls
    function moveLeft(){ if(!alive||paused) return; if(player.lane>0){ player.lane--; player.x = laneCenter(player.lane); beep(330, .05, 'sine', .03); } }
    function moveRight(){ if(!alive||paused) return; if(player.lane<lanes-1){ player.lane++; player.x = laneCenter(player.lane); beep(330, .05, 'sine', .03); } }

    document.addEventListener('keydown', (e)=>{
      if(e.repeat) return;
      const k = e.key.toLowerCase();
      if(k==='arrowleft' || k==='a') moveLeft();
      if(k==='arrowright' || k==='d') moveRight();
      if(k==='p') togglePause();
      if(k==='r') restart();
      if(k==='enter') startGame();
    });

    // Touch / mouse
    btnLeft.addEventListener('pointerdown', (e)=>{ e.preventDefault(); moveLeft(); });
    btnRight.addEventListener('pointerdown', (e)=>{ e.preventDefault(); moveRight(); });

    // Full-area tap: left/right half
    wrap.addEventListener('pointerdown', (e)=>{
      const rect = wrap.getBoundingClientRect();
      const x = e.clientX - rect.left;
      if(x < rect.width/2) moveLeft(); else moveRight();
    });

    // Buttons
    function togglePause(){
      if(!alive){ return; }
      paused = !paused;
      btnPause.textContent = paused ? 'Retomar' : 'Pausar';
      if(!paused){ last = performance.now(); requestAnimationFrame(loop); if(!muted){ try{bgm.play();}catch{} } else { try{bgm.pause();}catch{} } }
    }

    function restart(){ reset(); gameoverOverlay.style.display = 'none'; startOverlay.style.display = 'grid'; try{bgm.pause();}catch{} }

    btnPause.addEventListener('click', togglePause);
    btnRestart.addEventListener('click', restart);
    btnStartTop.addEventListener('click', startGame);

    startPlay.addEventListener('click', ()=>{ muted=false; try{bgm.volume=0.4; bgm.play().catch(()=>{});}catch{} startGame(); });
    startMuted.addEventListener('click', ()=>{ muted=true; try{bgm.pause();}catch{} startGame(); });

    againBtn.addEventListener('click', ()=>{ restart(); startOverlay.style.display='none'; startGame(); });

    shareBtn.addEventListener('click', async ()=>{
      const text = `Meu recorde no Neon Racer: ${Math.floor(score)}! 🏁`;
      try{
        if(navigator.share){ await navigator.share({ text, title: 'Neon Racer' }); }
        else { await navigator.clipboard.writeText(text); alert('Texto copiado! Cole onde quiser.'); }
      }catch{}
    });

    btnSound.addEventListener('click', ()=>{
      muted = !muted;
      btnSound.textContent = muted ? '🔇 Silenciado' : '🔊 Som';
      try { if(muted){ bgm.pause(); } else { bgm.volume = 0.4; bgm.play().catch(()=>{}); } } catch{}
    });

    function lerp(a,b,t){ return a + (b-a)*t; }
    function lerpColor(c1,c2,t){ return [Math.round(lerp(c1[0],c2[0],t)), Math.round(lerp(c1[1],c2[1],t)), Math.round(lerp(c1[2],c2[2],t))]; }

    // Initialize
    reset();
  })();
  </script>
</body>
</html>

