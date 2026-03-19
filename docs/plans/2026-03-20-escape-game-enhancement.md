# EXEM 방탈출 게임 고도화 Implementation Plan

> **For Claude:** REQUIRED SUB-SKILL: Use superpowers:executing-plans to implement this plan task-by-task.

**Goal:** Enhance the EXEM escape room game with immersive 1st-person UI/UX, hint system, new puzzle types, and stronger narrative.

**Architecture:** Single `index.html` with inline CSS and JS. All enhancements are additive — new CSS rules, new JS functions, expanded game data. No build tools, no external libraries.

**Tech Stack:** HTML5, CSS3 (3D transforms, animations, glassmorphism), Vanilla JS (drag & drop API, touch events), SVG (circular timer)

**Target file:** `exem-escape/index.html` (~1,471 lines currently)

---

## Task 1: CSS Design System — Glassmorphism + Enhanced Cards

**Files:**
- Modify: `index.html` — `<style>` section (lines 22–776)

**Step 1: Add new CSS variables and glassmorphism base**

Add to `:root` (after line 37):

```css
--glass-bg: rgba(26, 26, 46, 0.65);
--glass-border: rgba(255, 255, 255, 0.08);
--glass-shadow: 0 8px 32px rgba(0, 0, 0, 0.4);
--glass-blur: blur(16px);
--glow-blue: 0 0 20px rgba(64, 226, 255, 0.3);
--glow-green: 0 0 20px rgba(16, 185, 129, 0.3);
--glow-gold: 0 0 20px rgba(245, 158, 11, 0.3);
--transition-fast: 0.2s ease;
--transition-med: 0.4s ease;
--transition-slow: 0.6s ease;
```

Update `.room-card` and `.quiz-card` CSS:

```css
.room-card, .quiz-card {
  background: var(--glass-bg);
  backdrop-filter: var(--glass-blur);
  -webkit-backdrop-filter: var(--glass-blur);
  border: 1px solid var(--glass-border);
  box-shadow: var(--glass-shadow);
  border-radius: 16px;
  padding: 1.5rem;
  animation: slideIn 0.3s ease;
}
```

Update `.stage-clear`:

```css
.stage-clear {
  display: none;
  background: var(--glass-bg);
  backdrop-filter: var(--glass-blur);
  -webkit-backdrop-filter: var(--glass-blur);
  border: 1px solid rgba(16, 185, 129, 0.3);
  border-radius: 16px;
  padding: 1.5rem;
  text-align: center;
  box-shadow: var(--glow-green);
}
```

Update `.option-btn`:

```css
.option-btn {
  display: block;
  width: 100%;
  padding: 0.85rem 1rem;
  border-radius: 10px;
  border: 1px solid var(--glass-border);
  background: rgba(34, 34, 59, 0.6);
  backdrop-filter: blur(8px);
  color: var(--text);
  font-size: 0.9rem;
  text-align: left;
  cursor: pointer;
  transition: all var(--transition-fast);
  line-height: 1.5;
}

.option-btn:hover {
  background: rgba(64, 226, 255, 0.12);
  border-color: var(--blue);
  transform: translateX(4px) scale(1.01);
  box-shadow: var(--glow-blue);
}
```

**Step 2: Verify in browser**

Open `index.html` in browser. Check:
- Cards have frosted glass effect
- Option buttons glow on hover
- All existing elements remain readable

**Step 3: Commit**

```bash
git add index.html
git commit -m "style: add glassmorphism design system with CSS variables"
```

---

## Task 2: Circular SVG Timer + Split Progress Bars

**Files:**
- Modify: `index.html` — CSS section + HTML game header + JS timer functions

**Step 1: Add circular timer CSS**

Add new CSS (before `/* Responsive */` comment):

```css
/* Circular Timer */
.timer-circle {
  position: relative;
  width: 56px;
  height: 56px;
  flex-shrink: 0;
}

.timer-circle svg {
  transform: rotate(-90deg);
  width: 56px;
  height: 56px;
}

.timer-circle .track {
  fill: none;
  stroke: var(--surface2);
  stroke-width: 4;
}

.timer-circle .progress {
  fill: none;
  stroke: var(--blue);
  stroke-width: 4;
  stroke-linecap: round;
  transition: stroke-dashoffset 1s linear, stroke 0.5s ease;
}

.timer-circle .progress.warning { stroke: var(--gold); }
.timer-circle .progress.danger { stroke: var(--red); }

.timer-circle .timer-text {
  position: absolute;
  inset: 0;
  display: flex;
  align-items: center;
  justify-content: center;
  font-family: 'Unica 77 LL', monospace;
  font-size: 0.72rem;
  font-weight: 700;
  color: var(--text);
}

/* Overall progress (5 stages) */
.overall-progress {
  display: flex;
  align-items: center;
  gap: 4px;
}

.progress-dot {
  width: 10px;
  height: 10px;
  border-radius: 50%;
  background: var(--surface2);
  border: 1px solid var(--border);
  transition: all var(--transition-med);
}

.progress-dot.completed {
  background: var(--green);
  border-color: var(--green);
  box-shadow: var(--glow-green);
}

.progress-dot.current {
  background: var(--blue);
  border-color: var(--blue);
  box-shadow: var(--glow-blue);
  transform: scale(1.3);
}
```

**Step 2: Replace timer HTML**

Replace the game header (line ~809–813) with:

```html
<div class="game-header">
  <div class="timer-circle" id="timerCircle">
    <svg viewBox="0 0 56 56">
      <circle class="track" cx="28" cy="28" r="24"/>
      <circle class="progress" id="timerProgress" cx="28" cy="28" r="24"
              stroke-dasharray="150.8" stroke-dashoffset="0"/>
    </svg>
    <div class="timer-text" id="timerDisplay">10:00</div>
  </div>
  <div class="overall-progress" id="overallProgress"></div>
  <div class="office-map" id="officeMap"></div>
  <div class="score-display">점수 <span id="scoreDisplay">0</span></div>
</div>
```

**Step 3: Update JS timer functions**

Replace `updateTimer()` (line ~1193):

```javascript
function updateTimer() {
  const m = Math.floor(secondsLeft / 60);
  const s = secondsLeft % 60;
  const display = document.getElementById('timerDisplay');
  display.textContent = `${m}:${s.toString().padStart(2, '0')}`;

  // Update circular progress
  const circumference = 2 * Math.PI * 24; // ~150.8
  const pct = secondsLeft / 600;
  const offset = circumference * (1 - pct);
  const progress = document.getElementById('timerProgress');
  progress.style.strokeDashoffset = offset;

  progress.classList.remove('warning', 'danger');
  if (secondsLeft <= 120) progress.classList.add('danger');
  else if (secondsLeft <= 180) progress.classList.add('warning');
}
```

Add overall progress builder to `startGame()` after minimap build:

```javascript
// Build overall progress dots
const overallEl = document.getElementById('overallProgress');
overallEl.innerHTML = stages.map((_, i) =>
  `<div class="progress-dot" id="progressDot${i}"></div>`
).join('');
```

Add to `updateOfficeMap()`:

```javascript
// Update overall progress dots
stages.forEach((_, i) => {
  const dot = document.getElementById(`progressDot${i}`);
  if (!dot) return;
  dot.className = 'progress-dot';
  if (i < stageIdx) dot.classList.add('completed');
  else if (i === stageIdx) dot.classList.add('current');
});
```

**Step 4: Verify in browser**

- Circular timer counts down with arc animation
- Dot progress shows current stage
- Timer turns gold at 3min, red at 2min

**Step 5: Commit**

```bash
git add index.html
git commit -m "feat: circular SVG timer and stage progress dots"
```

---

## Task 3: 1st-Person Stage Transitions + Parallax

**Files:**
- Modify: `index.html` — CSS animations + JS transition logic

**Step 1: Add 3D transition and parallax CSS**

Add new CSS:

```css
/* 3D Stage Transition */
#stageContent {
  perspective: 800px;
  transform-style: preserve-3d;
}

@keyframes doorOpen {
  0%   { clip-path: inset(0 50% 0 50%); opacity: 0; }
  40%  { clip-path: inset(0 20% 0 20%); opacity: 0.6; }
  100% { clip-path: inset(0 0 0 0); opacity: 1; }
}

@keyframes roomZoomIn {
  0%   { transform: perspective(800px) translateZ(0); opacity: 1; }
  100% { transform: perspective(800px) translateZ(200px); opacity: 0; }
}

@keyframes roomZoomOut {
  0%   { transform: perspective(800px) translateZ(-200px); opacity: 0; }
  100% { transform: perspective(800px) translateZ(0); opacity: 1; }
}

.stage-door-open { animation: doorOpen 0.8s ease-out forwards; }
.stage-zoom-in   { animation: roomZoomIn 0.4s ease-in forwards; pointer-events: none; }
.stage-zoom-out  { animation: roomZoomOut 0.5s ease-out forwards; }

/* Parallax background */
#bg-layer {
  transition: background-position 0.1s ease-out, opacity 0.5s ease;
}

/* Cutscene overlay */
.cutscene-overlay {
  position: fixed;
  inset: 0;
  z-index: 100;
  background: rgba(0, 0, 0, 0.92);
  display: flex;
  flex-direction: column;
  align-items: center;
  justify-content: center;
  padding: 2rem;
  opacity: 0;
  animation: fadeIn 0.5s ease forwards;
}

.cutscene-text {
  max-width: 500px;
  font-size: 1.1rem;
  line-height: 2;
  color: var(--muted);
  text-align: center;
  font-style: italic;
}

.cutscene-text .highlight {
  color: var(--blue-light);
  font-style: normal;
  font-weight: 600;
}

.cutscene-skip {
  margin-top: 2rem;
  padding: 0.6rem 1.5rem;
  border-radius: 8px;
  border: 1px solid var(--border);
  background: transparent;
  color: var(--muted);
  font-size: 0.85rem;
  cursor: pointer;
  transition: all var(--transition-fast);
}

.cutscene-skip:hover {
  border-color: var(--blue);
  color: var(--blue);
}

@keyframes fadeIn {
  from { opacity: 0; }
  to   { opacity: 1; }
}

@keyframes fadeOut {
  from { opacity: 1; }
  to   { opacity: 0; }
}
```

**Step 2: Add parallax JS**

Add after visual effects section:

```javascript
// ── PARALLAX ──
function initParallax() {
  const bg = document.getElementById('bg-layer');

  // Mouse parallax (PC)
  document.addEventListener('mousemove', (e) => {
    if (!gameStarted) return;
    const x = (e.clientX / window.innerWidth - 0.5) * 20;
    const y = (e.clientY / window.innerHeight - 0.5) * 10;
    bg.style.backgroundPosition = `calc(50% + ${x}px) calc(50% + ${y}px)`;
  });

  // Device orientation parallax (Mobile)
  if (window.DeviceOrientationEvent) {
    window.addEventListener('deviceorientation', (e) => {
      if (!gameStarted) return;
      const x = (e.gamma || 0) * 0.5; // left-right tilt
      const y = (e.beta || 0) * 0.3;  // front-back tilt
      bg.style.backgroundPosition = `calc(50% + ${x}px) calc(50% + ${y}px)`;
    });
  }
}
```

**Step 3: Add cutscene data to stages**

Add `cutscene` property to each stage object (except stage 0):

```javascript
// Add to stages[1]:
cutscene: '서랍에서 찾은 단서 카드를 손에 쥐고 일어섰습니다.\n\n사무실 구석, 파일 캐비닛의 금속 표면이 희미하게 빛납니다.\n\n발걸음을 옮기자 복도의 형광등이 한 번 깜빡입니다...',

// Add to stages[2]:
cutscene: '캐비닛에서 찾은 카드 3장을 주머니에 넣었습니다.\n\n책상 위 컴퓨터 화면이 대기 모드로 깜빡이고 있습니다.\n\n비밀번호 입력창이 당신을 기다리고 있습니다...',

// Add to stages[3]:
cutscene: '컴퓨터 화면에 나타난 지도를 따라 책장 뒤로 향합니다.\n\n무거운 책장을 밀자 — 먼지가 날리며 금고가 드러났습니다.\n\n세 개의 버튼이 차가운 금속 위에서 빛나고 있습니다...',

// Add to stages[4]:
cutscene: '금고에서 꺼낸 열쇠를 꽉 쥐고 출구로 달려갑니다.\n\n복도 끝, 마지막 문 앞에 도착했습니다.\n\n열쇠를 넣으려는 순간 — 마지막 잠금장치가 보입니다...',
```

**Step 4: Update stage transition with 3D + cutscene**

Replace `loadStage()` and `nextStage()`:

```javascript
function loadStage(stageIdx, animate) {
  const sc = document.getElementById('stageContent');

  const doLoad = () => {
    currentStage = stageIdx;
    currentQuiz = 0;
    const stage = stages[stageIdx];

    updateOfficeMap(stageIdx);

    document.getElementById('bg-layer').style.backgroundImage = `url('${stage.bg}')`;

    document.getElementById('roomIcon').textContent = stage.icon;
    document.getElementById('roomTitle').textContent = stage.title;
    document.getElementById('roomLocation').textContent = stage.location;
    typeText(document.getElementById('roomStory'), stage.story, 14);
    document.getElementById('roomHint').textContent = stage.hint;

    document.getElementById('stageClear').classList.remove('visible');
    document.getElementById('quizCard').style.display = 'block';

    loadQuiz(stageIdx, 0);

    sc.classList.remove('stage-zoom-in', 'stage-leaving');
    sc.classList.add('stage-zoom-out');
    setTimeout(() => sc.classList.remove('stage-zoom-out'), 550);
  };

  if (animate) {
    sc.classList.add('stage-zoom-in');
    setTimeout(doLoad, 420);
  } else {
    doLoad();
  }
}

function showCutscene(text, callback) {
  const overlay = document.createElement('div');
  overlay.className = 'cutscene-overlay';

  const textEl = document.createElement('div');
  textEl.className = 'cutscene-text';
  overlay.appendChild(textEl);

  const skipBtn = document.createElement('button');
  skipBtn.className = 'cutscene-skip';
  skipBtn.textContent = '건너뛰기 →';
  overlay.appendChild(skipBtn);

  document.body.appendChild(overlay);

  typeText(textEl, text, 30);

  function closeCutscene() {
    overlay.style.animation = 'fadeOut 0.4s ease forwards';
    setTimeout(() => {
      overlay.remove();
      callback();
    }, 400);
  }

  skipBtn.onclick = closeCutscene;
  // Also close on text click after typing finishes
  setTimeout(() => { textEl.onclick = closeCutscene; }, text.length * 30 + 500);
  // Auto-close after reading time
  setTimeout(closeCutscene, Math.max(text.length * 35 + 2000, 5000));
}

function nextStage() {
  if (currentStage >= stages.length - 1) {
    showEnding(false);
    return;
  }

  spawnConfetti(60);
  const nextIdx = currentStage + 1;
  const nextCutscene = stages[nextIdx].cutscene;

  if (nextCutscene) {
    showCutscene(nextCutscene, () => {
      addFlicker();
      setTimeout(() => {
        loadStage(nextIdx, true);
        window.scrollTo({ top: 0, behavior: 'smooth' });
      }, 150);
    });
  } else {
    addFlicker();
    setTimeout(() => {
      loadStage(nextIdx, true);
      window.scrollTo({ top: 0, behavior: 'smooth' });
    }, 150);
  }
}
```

Call `initParallax()` inside `startGame()`:

```javascript
gameStarted = true;
initParallax();
```

**Step 5: Verify in browser**

- Stage transitions have 3D zoom effect
- Cutscene shows between stages with typing
- Background responds to mouse movement
- Skip button works

**Step 6: Commit**

```bash
git add index.html
git commit -m "feat: 1st-person 3D transitions, cutscenes, and parallax"
```

---

## Task 4: Cinematic Intro Screen

**Files:**
- Modify: `index.html` — intro CSS + HTML

**Step 1: Add intro animation CSS**

```css
/* Cinematic Intro */
@keyframes glowPulse {
  0%, 100% { filter: drop-shadow(0 0 8px rgba(64, 226, 255, 0.4)); }
  50%      { filter: drop-shadow(0 0 24px rgba(64, 226, 255, 0.8)); }
}

@keyframes btnPulse {
  0%, 100% { box-shadow: 0 0 0 0 rgba(64, 226, 255, 0.4); }
  50%      { box-shadow: 0 0 0 12px rgba(64, 226, 255, 0); }
}

@keyframes introLineIn {
  from { opacity: 0; transform: translateY(12px); }
  to   { opacity: 1; transform: translateY(0); }
}

.intro-logo-img {
  height: 60px;
  margin-bottom: 0.5rem;
  animation: glowPulse 3s ease-in-out infinite;
}

.btn-start {
  animation: btnPulse 2s ease-in-out infinite;
}

.intro-story-line {
  opacity: 0;
  animation: introLineIn 0.6s ease forwards;
}

/* Stagger intro lines */
.intro-story-line:nth-child(1) { animation-delay: 0.3s; }
.intro-story-line:nth-child(2) { animation-delay: 0.9s; }
.intro-story-line:nth-child(3) { animation-delay: 1.5s; }
.intro-story-line:nth-child(4) { animation-delay: 2.1s; }
.intro-story-line:nth-child(5) { animation-delay: 2.7s; }
.intro-story-line:nth-child(6) { animation-delay: 3.3s; }

/* Minimap-style stage badges */
.stage-list {
  display: flex;
  gap: 0.75rem;
  margin: 1.5rem 0 2rem;
  flex-wrap: wrap;
  justify-content: center;
}

.stage-badge {
  display: flex;
  flex-direction: column;
  align-items: center;
  gap: 0.2rem;
  padding: 0.5rem;
  border-radius: 10px;
  border: 1px solid var(--border);
  background: var(--glass-bg);
  backdrop-filter: blur(8px);
  font-size: 1.4rem;
  cursor: default;
  transition: all var(--transition-fast);
  position: relative;
  min-width: 48px;
}

.stage-badge::after {
  content: attr(data-name);
  font-size: 0.65rem;
  color: var(--muted);
  opacity: 0;
  transform: translateY(4px);
  transition: all var(--transition-fast);
  position: absolute;
  bottom: -20px;
  white-space: nowrap;
}

.stage-badge:hover::after {
  opacity: 1;
  transform: translateY(0);
}

.stage-badge:hover {
  border-color: var(--blue);
  box-shadow: var(--glow-blue);
  transform: translateY(-2px);
}
```

**Step 2: Update intro HTML**

Replace intro section (lines ~782–803):

```html
<div id="intro">
  <img src="logo.png" alt="EXEM" class="intro-logo-img">
  <div class="intro-title">🔐 사무실을 탈출하라!</div>
  <div class="intro-story">
    <div class="intro-story-line"><em>2026년 3월, 입사 첫날.</em></div>
    <div class="intro-story-line">동료들은 모두 퇴근하고 당신만 남겨졌는데…</div>
    <div class="intro-story-line"><em>딸깍— 사무실 문이 잠겼습니다!</em></div>
    <div class="intro-story-line">사방을 둘러보니 책상 위에 메모가 하나 있습니다.</div>
    <div class="intro-story-line"><em>"엑셈의 핵심가치를 증명하면 탈출할 수 있다."</em></div>
  </div>
  <div class="stage-list">
    <div class="stage-badge" data-name="책상 서랍">📁</div>
    <div class="stage-badge" data-name="파일 캐비닛">🗄️</div>
    <div class="stage-badge" data-name="컴퓨터">💻</div>
    <div class="stage-badge" data-name="금고">🔒</div>
    <div class="stage-badge" data-name="탈출 출구">🚪</div>
  </div>
  <button class="btn-start" onclick="startGame()">탈출 시작하기 →</button>
</div>
```

**Step 3: Verify in browser**

- Logo has glow pulse
- Story lines fade in one by one with stagger
- Stage badges show names on hover
- Start button pulses

**Step 4: Commit**

```bash
git add index.html
git commit -m "feat: cinematic intro with staggered animations and minimap badges"
```

---

## Task 5: Enhanced Quiz Interactions

**Files:**
- Modify: `index.html` — CSS + JS feedback functions

**Step 1: Add enhanced interaction CSS**

```css
/* Enhanced quiz feedback */
@keyframes unlockPulse {
  0%   { transform: scale(1); }
  30%  { transform: scale(1.15); }
  60%  { transform: scale(0.95); }
  100% { transform: scale(1); }
}

@keyframes greenFlash {
  0%   { background: transparent; }
  30%  { background: rgba(16, 185, 129, 0.15); }
  100% { background: transparent; }
}

@keyframes redFlash {
  0%   { background: transparent; }
  30%  { background: rgba(239, 68, 68, 0.12); }
  100% { background: transparent; }
}

.unlock-anim { animation: unlockPulse 0.5s ease; }
.screen-green-flash { animation: greenFlash 0.6s ease; }
.screen-red-flash   { animation: redFlash 0.5s ease; }

/* Typewriter cursor for fill input */
.fill-input {
  caret-color: var(--blue);
}

.fill-input::placeholder {
  animation: blink 1s step-end infinite;
}

@keyframes blink {
  50% { opacity: 0; }
}

/* Discovery popup */
.discovery-popup {
  position: fixed;
  top: 50%;
  left: 50%;
  transform: translate(-50%, -50%) scale(0);
  z-index: 200;
  background: var(--glass-bg);
  backdrop-filter: var(--glass-blur);
  border: 1px solid var(--gold);
  border-radius: 16px;
  padding: 1.5rem 2.5rem;
  text-align: center;
  animation: popIn 0.4s cubic-bezier(0.34, 1.56, 0.64, 1) forwards;
  box-shadow: var(--glow-gold);
}

@keyframes popIn {
  to { transform: translate(-50%, -50%) scale(1); }
}

@keyframes popOut {
  from { transform: translate(-50%, -50%) scale(1); opacity: 1; }
  to   { transform: translate(-50%, -50%) scale(0.8); opacity: 0; }
}

.discovery-popup .discovery-icon { font-size: 2rem; margin-bottom: 0.5rem; }
.discovery-popup .discovery-text { color: var(--gold); font-weight: 700; font-size: 1rem; }

/* Sparkle particles */
.sparkle {
  position: fixed;
  width: 6px;
  height: 6px;
  background: var(--gold);
  border-radius: 50%;
  pointer-events: none;
  z-index: 201;
  animation: sparkleFade 0.8s ease-out forwards;
}

@keyframes sparkleFade {
  0%   { transform: scale(1) translate(0, 0); opacity: 1; }
  100% { transform: scale(0) translate(var(--tx), var(--ty)); opacity: 0; }
}
```

**Step 2: Add enhanced JS feedback functions**

```javascript
// ── ENHANCED FEEDBACK ──
function screenFlash(color) {
  const game = document.getElementById('game');
  const cls = color === 'green' ? 'screen-green-flash' : 'screen-red-flash';
  game.classList.remove(cls);
  void game.offsetWidth;
  game.classList.add(cls);
  game.addEventListener('animationend', () => game.classList.remove(cls), { once: true });

  // Vibrate on mobile
  if (color === 'red' && navigator.vibrate) {
    navigator.vibrate([50, 30, 50]);
  } else if (color === 'green' && navigator.vibrate) {
    navigator.vibrate(30);
  }
}

function showDiscovery(text) {
  const popup = document.createElement('div');
  popup.className = 'discovery-popup';
  popup.innerHTML = `
    <div class="discovery-icon">🔍</div>
    <div class="discovery-text">${text}</div>
  `;
  document.body.appendChild(popup);

  // Spawn sparkles
  for (let i = 0; i < 12; i++) {
    const s = document.createElement('div');
    s.className = 'sparkle';
    const angle = (i / 12) * Math.PI * 2;
    const dist = 40 + Math.random() * 30;
    s.style.cssText = `
      left: 50%; top: 50%;
      --tx: ${Math.cos(angle) * dist}px;
      --ty: ${Math.sin(angle) * dist}px;
      animation-delay: ${Math.random() * 0.2}s;
    `;
    document.body.appendChild(s);
    setTimeout(() => s.remove(), 1000);
  }

  setTimeout(() => {
    popup.style.animation = 'popOut 0.3s ease forwards';
    setTimeout(() => popup.remove(), 300);
  }, 1200);
}
```

**Step 3: Integrate into showFeedback()**

Update `showFeedback()` to call the new effects:

```javascript
function showFeedback(isCorrect, message, quizIdx, stageIdx) {
  totalAttempts++;
  if (isCorrect) {
    totalCorrect++;
    score += 100;
    document.getElementById('scoreDisplay').textContent = score;
    screenFlash('green');
  } else {
    screenFlash('red');
  }

  const fb = document.getElementById('quizFeedback');
  fb.className = `feedback ${isCorrect ? 'correct' : 'wrong'}`;
  fb.textContent = message;

  if (isCorrect) {
    const qc = document.getElementById('quizCard');
    qc.classList.remove('correct-flash');
    void qc.offsetWidth;
    qc.classList.add('correct-flash');
    qc.addEventListener('animationend', () => qc.classList.remove('correct-flash'), { once: true });

    const btnNext = document.getElementById('btnNext');
    const stage = stages[stageIdx];
    const isLast = quizIdx >= stage.quizzes.length - 1;
    btnNext.className = 'btn-next visible';
    btnNext.textContent = isLast ? '완료 →' : '다음 문제 →';
    document.getElementById('btnRetry').className = 'btn-retry';
  } else {
    shakeElement(document.getElementById('quizCard'));
    document.getElementById('btnRetry').className = 'btn-retry visible';
    document.getElementById('btnNext').className = 'btn-next';
  }
}
```

**Step 4: Show discovery popup on stage clear**

Update `showStageClear()` — add before the clear panel shows:

```javascript
function showStageClear() {
  const stage = stages[currentStage];
  document.getElementById('quizCard').style.display = 'none';

  // Show discovery effect
  showDiscovery('단서 발견!');

  setTimeout(() => {
    const clearEl = document.getElementById('stageClear');
    clearEl.classList.add('visible');
    clearEl.classList.add('unlock-anim');
    clearEl.addEventListener('animationend', () => clearEl.classList.remove('unlock-anim'), { once: true });

    document.getElementById('clearMessage').textContent = stage.clearMessage;

    const hintEl = document.getElementById('clearHint');
    if (stage.nextHint) {
      hintEl.textContent = stage.nextHint;
      hintEl.style.display = 'block';
    } else {
      hintEl.style.display = 'none';
    }

    const btnNext = document.getElementById('btnNextStage');
    btnNext.textContent = currentStage >= stages.length - 1 ? '🎉 탈출!' : '다음 단계로 →';
  }, 1400);
}
```

**Step 5: Verify in browser**

- Correct answer triggers green screen flash + vibration
- Wrong answer triggers red flash + shake + vibration
- Stage clear shows discovery popup with sparkles
- Clear panel has unlock animation

**Step 6: Commit**

```bash
git add index.html
git commit -m "feat: enhanced quiz interactions with screen flash, discovery popup, and sparkles"
```

---

## Task 6: Hint System

**Files:**
- Modify: `index.html` — game data, CSS, HTML, JS

**Step 1: Add hint data to every quiz**

Add `hints` array to each quiz in `stages`. Example for stages[0].quizzes[0]:

```javascript
hints: [
  { level: 1, text: '이 회사의 정식 이름은 영어 두 단어의 합성어입니다.', cost: 0 },
  { level: 2, text: 'Expert + ??? — "???"는 거대한 나라를 뜻합니다.', cost: 50 },
  { level: 3, text: '정답은 "Empire"입니다. Expert Empire = EXEM.', cost: 100 }
]
```

Full hint data for all 13 quizzes:

```javascript
// stages[0].quizzes[0] (Empire)
hints: [
  { level: 1, text: '회사 이름은 영어 두 단어의 합성어입니다.', cost: 0 },
  { level: 2, text: 'EX___ = Expert + "나라/제국"을 뜻하는 영어 단어', cost: 50 },
  { level: 3, text: '정답: Empire (Expert Empire = EXEM)', cost: 100 }
],

// stages[0].quizzes[1] (전개일여)
hints: [
  { level: 1, text: '프랙탈 도형의 특성과 관련된 한자 4글자입니다.', cost: 0 },
  { level: 2, text: '全(전체 전), 個(낱 개), 一(한 일), 如(같을 여)', cost: 50 },
  { level: 3, text: '정답: 전개일여 — 개인과 전체가 하나', cost: 100 }
],

// stages[1].quizzes[0] (가연성)
hints: [
  { level: 1, text: '세 종류의 불꽃 유형을 떠올려보세요.', cost: 0 },
  { level: 2, text: '"조건이 없으면 동기부여가 안 된다" — 스스로 타는 게 아닙니다.', cost: 50 },
  { level: 3, text: '정답: 가연성 — 불을 갖다 대면 타오르는 유형', cost: 100 }
],

// stages[1].quizzes[1] (탐색)
hints: [
  { level: 1, text: 'Passion의 첫 번째 행동 단계입니다.', cost: 0 },
  { level: 2, text: '더 나은 방법을 끊임없이 찾아보는 것 — 무엇을 하는 걸까요?', cost: 50 },
  { level: 3, text: '정답: 탐색', cost: 100 }
],

// stages[2].quizzes[0] (10% 공유)
hints: [
  { level: 1, text: 'Speed의 핵심은 공유 시점입니다.', cost: 0 },
  { level: 2, text: '카드 A를 다시 읽어보세요: "10%만 됐을 때..."', cost: 50 },
  { level: 3, text: '정답: 방향만 잡았을 때 (10%)', cost: 100 }
],

// stages[2].quizzes[1] (스톡데일)
hints: [
  { level: 1, text: '베트남 전쟁 미군 장교의 이름을 한국어로 쓰세요.', cost: 0 },
  { level: 2, text: 'S로 시작하는 영어 이름의 한국어 음역입니다.', cost: 50 },
  { level: 3, text: '정답: 스톡데일 (Stockdale)', cost: 100 }
],

// stages[2].quizzes[2] (Flow)
hints: [
  { level: 1, text: '몰입 상태와 관련된 핵심가치입니다.', cost: 0 },
  { level: 2, text: '시간도 잊고 배고픔도 잊는 상태 — 심리학에서도 유명한 개념', cost: 50 },
  { level: 3, text: '정답: Flow — 문제와 나만 남는 몰입의 순간', cost: 100 }
],

// stages[3].quizzes[0] (책임의식)
hints: [
  { level: 1, text: 'Work의 핵심 키워드를 생각해보세요.', cost: 0 },
  { level: 2, text: '고객이 기다리고 있다면, 맡은 일에 대한 태도는?', cost: 50 },
  { level: 3, text: '정답: 고객이 기다리고 있다면 해결하고 퇴근한다', cost: 100 }
],

// stages[3].quizzes[1] (수용)
hints: [
  { level: 1, text: 'Relationship의 핵심 원칙 중 하나입니다.', cost: 0 },
  { level: 2, text: '연차와 직급을 떠나 좋은 의견을 대하는 태도', cost: 50 },
  { level: 3, text: '정답: 경력과 관계없이 좋은 방법이면 적극 받아들인다', cost: 100 }
],

// stages[3].quizzes[2] (투명성)
hints: [
  { level: 1, text: 'Communication의 핵심 원칙을 생각해보세요.', cost: 0 },
  { level: 2, text: '"굳이 안 해도 되겠지"라는 판단 — 투명하게 공유하는 것과 반대', cost: 50 },
  { level: 3, text: '정답: "투명하게 적극 공유"하는 원칙에 어긋난다', cost: 100 }
],

// stages[4].quizzes[0] (생활규칙 3가지)
hints: [
  { level: 1, text: '세 가지 모두 관계와 성장에 관한 질문입니다.', cost: 0 },
  { level: 2, text: '조언을 ___? 조언을 ___? ___를 만들고 있나요?', cost: 50 },
  { level: 3, text: '정답: 조언을 구하고/조언을 해주고/친구를 만들고', cost: 100 }
],

// stages[4].quizzes[1] (지식)
hints: [
  { level: 1, text: '필리노베이터의 철학자(Philosopher) 마음과 관련됩니다.', cost: 0 },
  { level: 2, text: '업무를 통해 생산하는 무형의 가치 — 한 글자', cost: 50 },
  { level: 3, text: '정답: 지식', cost: 100 }
],

// stages[4].quizzes[2] (기술은 인격)
hints: [
  { level: 1, text: '두 사람의 "기술"에 대한 관점 차이를 비교해보세요.', cost: 0 },
  { level: 2, text: 'A는 보상 수단, B는 인격/태도/유산 — 엑셈은 어느 쪽?', cost: 50 },
  { level: 3, text: '정답: B — 기술을 인격이자 유산으로 보는 사람', cost: 100 }
]
```

**Step 2: Add hint system CSS**

```css
/* Hint System */
.hint-btn-float {
  position: fixed;
  bottom: 1.5rem;
  right: 1.5rem;
  z-index: 50;
  width: 52px;
  height: 52px;
  border-radius: 50%;
  border: none;
  background: var(--glass-bg);
  backdrop-filter: var(--glass-blur);
  border: 1px solid var(--gold);
  color: var(--gold);
  font-size: 1.5rem;
  cursor: pointer;
  transition: all var(--transition-fast);
  box-shadow: var(--glow-gold);
  display: none;
}

.hint-btn-float.visible { display: flex; align-items: center; justify-content: center; }
.hint-btn-float:hover { transform: scale(1.1); }
.hint-btn-float:active { transform: scale(0.95); }

.hint-badge {
  position: absolute;
  top: -4px;
  right: -4px;
  width: 20px;
  height: 20px;
  border-radius: 50%;
  background: var(--red);
  color: white;
  font-size: 0.65rem;
  font-weight: 700;
  display: flex;
  align-items: center;
  justify-content: center;
}

.hint-panel {
  position: fixed;
  bottom: 0;
  left: 0;
  right: 0;
  z-index: 60;
  background: var(--surface);
  border-top: 1px solid var(--gold);
  border-radius: 16px 16px 0 0;
  padding: 1.5rem;
  transform: translateY(100%);
  transition: transform 0.3s ease;
  max-height: 50vh;
  overflow-y: auto;
}

.hint-panel.open { transform: translateY(0); }

.hint-panel-header {
  display: flex;
  justify-content: space-between;
  align-items: center;
  margin-bottom: 1rem;
}

.hint-panel-title {
  font-weight: 700;
  color: var(--gold);
  font-size: 1rem;
}

.hint-close {
  background: none;
  border: none;
  color: var(--muted);
  font-size: 1.2rem;
  cursor: pointer;
}

.hint-item {
  padding: 0.75rem 1rem;
  border-radius: 10px;
  margin-bottom: 0.5rem;
  cursor: pointer;
  transition: all var(--transition-fast);
}

.hint-item.available {
  background: rgba(245, 158, 11, 0.08);
  border: 1px solid rgba(245, 158, 11, 0.2);
  color: var(--text);
}

.hint-item.available:hover {
  background: rgba(245, 158, 11, 0.15);
}

.hint-item.revealed {
  background: rgba(245, 158, 11, 0.05);
  border: 1px solid rgba(245, 158, 11, 0.1);
  color: var(--muted);
  cursor: default;
}

.hint-item.locked {
  background: var(--surface2);
  border: 1px solid var(--border);
  color: var(--border);
  cursor: not-allowed;
}

.hint-cost {
  font-size: 0.75rem;
  margin-top: 0.25rem;
}

.hint-cost.free { color: var(--green); }
.hint-cost.paid { color: var(--red); }

/* Score deduction animation */
@keyframes scoreDeduct {
  0%   { opacity: 1; transform: translateY(0); }
  100% { opacity: 0; transform: translateY(-30px); }
}

.score-deduct {
  position: fixed;
  z-index: 100;
  color: var(--red);
  font-weight: 700;
  font-size: 1rem;
  pointer-events: none;
  animation: scoreDeduct 1s ease-out forwards;
}
```

**Step 3: Add hint HTML**

Add before `</body>`:

```html
<!-- Hint System -->
<button class="hint-btn-float" id="hintBtn" onclick="toggleHintPanel()">
  💡
  <span class="hint-badge" id="hintBadge">3</span>
</button>

<div class="hint-panel" id="hintPanel">
  <div class="hint-panel-header">
    <div class="hint-panel-title">💡 힌트</div>
    <button class="hint-close" onclick="toggleHintPanel()">✕</button>
  </div>
  <div id="hintList"></div>
</div>
```

**Step 4: Add hint system JS**

```javascript
// ── HINT SYSTEM ──
let hintState = {}; // key: `${stageIdx}-${quizIdx}`, value: number of hints revealed
let totalHintsUsed = 0;
let hintPanelOpen = false;

function getHintKey() {
  return `${currentStage}-${currentQuiz}`;
}

function toggleHintPanel() {
  hintPanelOpen = !hintPanelOpen;
  document.getElementById('hintPanel').classList.toggle('open', hintPanelOpen);
  if (hintPanelOpen) renderHints();
}

function renderHints() {
  const quiz = stages[currentStage].quizzes[currentQuiz];
  if (!quiz.hints) { document.getElementById('hintList').innerHTML = '<p style="color:var(--muted)">이 문제에는 힌트가 없습니다.</p>'; return; }

  const key = getHintKey();
  const revealed = hintState[key] || 0;

  document.getElementById('hintList').innerHTML = quiz.hints.map((h, i) => {
    if (i < revealed) {
      return `<div class="hint-item revealed">
        <div>🔓 힌트 ${i + 1}: ${h.text}</div>
      </div>`;
    } else if (i === revealed) {
      const costLabel = h.cost === 0
        ? '<div class="hint-cost free">무료</div>'
        : `<div class="hint-cost paid">-${h.cost}점</div>`;
      return `<div class="hint-item available" onclick="revealHint(${i})">
        <div>🔒 힌트 ${i + 1}을 열려면 클릭하세요</div>
        ${costLabel}
      </div>`;
    } else {
      return `<div class="hint-item locked">
        <div>🔒 힌트 ${i + 1} (이전 힌트를 먼저 열어주세요)</div>
      </div>`;
    }
  }).join('');

  // Update badge
  const remaining = quiz.hints.length - revealed;
  const badge = document.getElementById('hintBadge');
  badge.textContent = remaining;
  badge.style.display = remaining > 0 ? 'flex' : 'none';
}

function revealHint(idx) {
  const quiz = stages[currentStage].quizzes[currentQuiz];
  const hint = quiz.hints[idx];
  const key = getHintKey();

  // Deduct score
  if (hint.cost > 0) {
    score = Math.max(0, score - hint.cost);
    document.getElementById('scoreDisplay').textContent = score;
    showScoreDeduction(hint.cost);
  }

  hintState[key] = idx + 1;
  totalHintsUsed++;
  renderHints();
}

function showScoreDeduction(amount) {
  const el = document.createElement('div');
  el.className = 'score-deduct';
  el.textContent = `-${amount}`;
  const scoreEl = document.getElementById('scoreDisplay');
  const rect = scoreEl.getBoundingClientRect();
  el.style.left = `${rect.left}px`;
  el.style.top = `${rect.top}px`;
  document.body.appendChild(el);
  setTimeout(() => el.remove(), 1000);
}

function showHintButton() {
  document.getElementById('hintBtn').classList.add('visible');
}

function hideHintButton() {
  document.getElementById('hintBtn').classList.remove('visible');
  document.getElementById('hintPanel').classList.remove('open');
  hintPanelOpen = false;
}
```

**Step 5: Integrate hints into game flow**

In `loadQuiz()`, add at the end:

```javascript
// Show hint button and reset panel
showHintButton();
renderHints();
```

In `showStageClear()`, add:

```javascript
hideHintButton();
```

In `showEnding()`, add:

```javascript
hideHintButton();
```

In `startGame()`, set `gameStarted = true` and add:

```javascript
showHintButton();
```

**Step 6: Verify in browser**

- 💡 button appears during quizzes
- Click opens hint panel from bottom
- Hints reveal one at a time
- Score deduction animation shows on paid hints
- Badge count decrements

**Step 7: Commit**

```bash
git add index.html
git commit -m "feat: 3-tier hint system with score deduction and floating UI"
```

---

## Task 7: New Puzzle Types (Match, Order, Image, Dial)

**Files:**
- Modify: `index.html` — CSS + JS puzzle renderers + game data

**Step 1: Add CSS for new puzzle types**

```css
/* ── MATCH PUZZLE ── */
.match-container {
  display: grid;
  grid-template-columns: 1fr auto 1fr;
  gap: 0.5rem;
  align-items: start;
}

.match-column {
  display: flex;
  flex-direction: column;
  gap: 0.5rem;
}

.match-item {
  padding: 0.7rem 1rem;
  border-radius: 10px;
  border: 1px solid var(--border);
  background: var(--surface2);
  color: var(--text);
  font-size: 0.85rem;
  cursor: grab;
  transition: all var(--transition-fast);
  user-select: none;
  touch-action: none;
}

.match-item:active { cursor: grabbing; }
.match-item.dragging { opacity: 0.5; transform: scale(0.95); }
.match-item.drag-over { border-color: var(--blue); background: rgba(64, 226, 255, 0.1); }
.match-item.matched-correct { border-color: var(--green); background: rgba(16, 185, 129, 0.1); }
.match-item.matched-wrong { border-color: var(--red); background: rgba(239, 68, 68, 0.1); }

.match-arrow-col {
  display: flex;
  flex-direction: column;
  gap: 0.5rem;
  align-items: center;
  justify-content: center;
  color: var(--border);
  font-size: 1.2rem;
}

.match-slot {
  padding: 0.7rem 1rem;
  border-radius: 10px;
  border: 2px dashed var(--border);
  background: transparent;
  min-height: 42px;
  display: flex;
  align-items: center;
  justify-content: center;
  font-size: 0.85rem;
  color: var(--muted);
  transition: all var(--transition-fast);
}

.match-slot.drag-over { border-color: var(--blue); background: rgba(64, 226, 255, 0.05); }
.match-slot.filled { border-style: solid; }

.btn-match-submit {
  grid-column: 1 / -1;
  margin-top: 0.5rem;
  padding: 0.85rem;
  border-radius: 8px;
  border: none;
  background: var(--blue);
  color: #0a0a1a;
  font-weight: 700;
  cursor: pointer;
  transition: all var(--transition-fast);
}

/* ── ORDER PUZZLE ── */
.order-list {
  display: flex;
  flex-direction: column;
  gap: 0.5rem;
}

.order-item {
  display: flex;
  align-items: center;
  gap: 0.75rem;
  padding: 0.75rem 1rem;
  border-radius: 10px;
  border: 1px solid var(--border);
  background: var(--surface2);
  cursor: grab;
  touch-action: none;
  user-select: none;
  transition: all var(--transition-fast);
}

.order-item:active { cursor: grabbing; }
.order-item.dragging { opacity: 0.5; }
.order-item.drag-over { border-color: var(--blue); transform: translateY(2px); }

.order-number {
  width: 24px;
  height: 24px;
  border-radius: 50%;
  background: var(--surface);
  border: 1px solid var(--border);
  display: flex;
  align-items: center;
  justify-content: center;
  font-size: 0.75rem;
  font-weight: 700;
  color: var(--muted);
  flex-shrink: 0;
}

.order-text { font-size: 0.85rem; color: var(--text); }

.btn-order-submit {
  margin-top: 0.75rem;
  padding: 0.85rem;
  border-radius: 8px;
  border: none;
  background: var(--blue);
  color: #0a0a1a;
  font-weight: 700;
  width: 100%;
  cursor: pointer;
}

/* ── DIAL PUZZLE ── */
.dial-container {
  display: flex;
  gap: 0.5rem;
  justify-content: center;
  align-items: center;
  margin: 1rem 0;
}

.dial-wheel {
  width: 56px;
  height: 160px;
  border-radius: 12px;
  border: 2px solid var(--border);
  background: var(--surface);
  overflow: hidden;
  position: relative;
  touch-action: none;
}

.dial-track {
  position: absolute;
  width: 100%;
  transition: transform 0.2s ease;
}

.dial-char {
  width: 100%;
  height: 40px;
  display: flex;
  align-items: center;
  justify-content: center;
  font-size: 1.3rem;
  font-weight: 700;
  color: var(--muted);
  font-family: 'Unica 77 LL', monospace;
}

.dial-char.active { color: var(--blue); font-size: 1.5rem; }

.dial-indicator {
  position: absolute;
  left: 0;
  right: 0;
  top: 50%;
  transform: translateY(-50%);
  height: 42px;
  border-top: 2px solid var(--blue);
  border-bottom: 2px solid var(--blue);
  background: rgba(64, 226, 255, 0.05);
  pointer-events: none;
}

.dial-arrow {
  width: 32px;
  height: 32px;
  border-radius: 50%;
  border: 1px solid var(--border);
  background: var(--surface2);
  color: var(--muted);
  display: flex;
  align-items: center;
  justify-content: center;
  cursor: pointer;
  font-size: 0.8rem;
  transition: all var(--transition-fast);
}

.dial-arrow:hover { border-color: var(--blue); color: var(--blue); }

.dial-col {
  display: flex;
  flex-direction: column;
  align-items: center;
  gap: 0.3rem;
}

.btn-dial-submit {
  margin-top: 1rem;
  padding: 0.85rem 2rem;
  border-radius: 8px;
  border: none;
  background: var(--blue);
  color: #0a0a1a;
  font-weight: 700;
  cursor: pointer;
  width: 100%;
}
```

**Step 2: Add JS puzzle renderers**

Add to the quiz rendering section in `loadQuiz()`. Extend the `if/else` chain:

```javascript
// Add after the 'fill' type handler in loadQuiz():

else if (quiz.type === 'match') {
  const shuffledRight = [...quiz.pairs].sort(() => Math.random() - 0.5);
  area.innerHTML = `
    <div class="match-container">
      <div class="match-column" id="matchLeft">
        ${quiz.pairs.map((p, i) => `<div class="match-item" data-idx="${i}">${p.left}</div>`).join('')}
      </div>
      <div class="match-arrow-col">
        ${quiz.pairs.map(() => '→').join('<br>')}
      </div>
      <div class="match-column" id="matchRight">
        ${shuffledRight.map((p, i) => `<div class="match-slot" data-slot="${i}" data-answer="${quiz.pairs.indexOf(p)}">${p.right}</div>`).join('')}
      </div>
      <button class="btn-match-submit" onclick="answerMatch()">확인</button>
    </div>`;
  initMatchDrag();
}

else if (quiz.type === 'order') {
  const shuffled = quiz.items.map((text, i) => ({ text, correctIdx: i }))
    .sort(() => Math.random() - 0.5);
  area.innerHTML = `
    <div class="order-list" id="orderList">
      ${shuffled.map((item, i) => `<div class="order-item" draggable="true" data-correct="${item.correctIdx}">
        <div class="order-number">${i + 1}</div>
        <div class="order-text">${item.text}</div>
      </div>`).join('')}
    </div>
    <button class="btn-order-submit" onclick="answerOrder()">확인</button>`;
  initOrderDrag();
}

else if (quiz.type === 'dial') {
  const chars = quiz.charset || 'ㄱㄴㄷㄹㅁㅂㅅㅇㅈㅊㅋㅌㅍㅎ';
  const dialCount = quiz.answer.length;
  area.innerHTML = `
    <div class="dial-container" id="dialContainer">
      ${Array.from({ length: dialCount }, (_, i) => `
        <div class="dial-col">
          <button class="dial-arrow" onclick="dialSpin(${i}, -1)">▲</button>
          <div class="dial-wheel" id="dialWheel${i}" data-idx="${i}" data-pos="0">
            <div class="dial-indicator"></div>
            <div class="dial-track" id="dialTrack${i}">
              ${chars.split('').map((c, ci) => `<div class="dial-char${ci === 0 ? ' active' : ''}">${c}</div>`).join('')}
            </div>
          </div>
          <button class="dial-arrow" onclick="dialSpin(${i}, 1)">▼</button>
        </div>
      `).join('')}
    </div>
    <button class="btn-dial-submit" onclick="answerDial()">확인</button>`;
  initDialTouch();
}
```

**Step 3: Add JS logic for new puzzle types**

```javascript
// ── MATCH PUZZLE ──
function initMatchDrag() {
  const items = document.querySelectorAll('#matchLeft .match-item');
  const slots = document.querySelectorAll('#matchRight .match-slot');
  let dragItem = null;

  items.forEach(item => {
    item.draggable = true;
    item.addEventListener('dragstart', (e) => {
      dragItem = item;
      item.classList.add('dragging');
      e.dataTransfer.effectAllowed = 'move';
    });
    item.addEventListener('dragend', () => {
      item.classList.remove('dragging');
      dragItem = null;
    });
    // Touch support
    item.addEventListener('touchstart', (e) => {
      dragItem = item;
      item.classList.add('dragging');
    });
  });

  slots.forEach(slot => {
    slot.addEventListener('dragover', (e) => { e.preventDefault(); slot.classList.add('drag-over'); });
    slot.addEventListener('dragleave', () => slot.classList.remove('drag-over'));
    slot.addEventListener('drop', (e) => {
      e.preventDefault();
      slot.classList.remove('drag-over');
      if (dragItem) {
        slot.textContent = dragItem.textContent;
        slot.dataset.matched = dragItem.dataset.idx;
        slot.classList.add('filled');
        dragItem.style.opacity = '0.4';
        dragItem.draggable = false;
      }
    });
    // Touch support
    slot.addEventListener('touchend', (e) => {
      if (dragItem) {
        slot.textContent = dragItem.textContent;
        slot.dataset.matched = dragItem.dataset.idx;
        slot.classList.add('filled');
        dragItem.style.opacity = '0.4';
        dragItem.classList.remove('dragging');
        dragItem.draggable = false;
        dragItem = null;
      }
    });
  });
}

function answerMatch() {
  const quiz = stages[currentStage].quizzes[currentQuiz];
  const slots = document.querySelectorAll('#matchRight .match-slot');
  let allCorrect = true;
  let allFilled = true;

  slots.forEach((slot, i) => {
    if (!slot.dataset.matched) { allFilled = false; return; }
    const matchedIdx = parseInt(slot.dataset.matched);
    const answerIdx = parseInt(slot.dataset.answer);
    if (matchedIdx === answerIdx) {
      slot.classList.add('matched-correct');
    } else {
      slot.classList.add('matched-wrong');
      allCorrect = false;
    }
  });

  if (!allFilled) return; // not all matched yet

  showFeedback(allCorrect,
    allCorrect ? quiz.feedbackCorrect : quiz.feedbackWrong,
    currentQuiz, currentStage);
}

// ── ORDER PUZZLE ──
function initOrderDrag() {
  const list = document.getElementById('orderList');
  let dragEl = null;

  list.querySelectorAll('.order-item').forEach(item => {
    item.draggable = true;
    item.addEventListener('dragstart', (e) => {
      dragEl = item;
      item.classList.add('dragging');
    });
    item.addEventListener('dragend', () => {
      item.classList.remove('dragging');
      updateOrderNumbers();
    });
    item.addEventListener('dragover', (e) => {
      e.preventDefault();
      if (item === dragEl) return;
      const rect = item.getBoundingClientRect();
      const mid = rect.top + rect.height / 2;
      if (e.clientY < mid) {
        list.insertBefore(dragEl, item);
      } else {
        list.insertBefore(dragEl, item.nextSibling);
      }
    });

    // Touch reorder
    let touchY = 0;
    item.addEventListener('touchstart', (e) => {
      dragEl = item;
      touchY = e.touches[0].clientY;
      item.classList.add('dragging');
    });
    item.addEventListener('touchmove', (e) => {
      e.preventDefault();
      const curY = e.touches[0].clientY;
      const items = [...list.querySelectorAll('.order-item')];
      for (const other of items) {
        if (other === dragEl) continue;
        const rect = other.getBoundingClientRect();
        if (curY > rect.top && curY < rect.bottom) {
          const mid = rect.top + rect.height / 2;
          if (curY < mid) list.insertBefore(dragEl, other);
          else list.insertBefore(dragEl, other.nextSibling);
          break;
        }
      }
    });
    item.addEventListener('touchend', () => {
      item.classList.remove('dragging');
      updateOrderNumbers();
    });
  });
}

function updateOrderNumbers() {
  document.querySelectorAll('#orderList .order-item').forEach((item, i) => {
    item.querySelector('.order-number').textContent = i + 1;
  });
}

function answerOrder() {
  const items = document.querySelectorAll('#orderList .order-item');
  let allCorrect = true;
  items.forEach((item, i) => {
    const correct = parseInt(item.dataset.correct);
    if (correct !== i) allCorrect = false;
  });

  const quiz = stages[currentStage].quizzes[currentQuiz];
  showFeedback(allCorrect,
    allCorrect ? quiz.feedbackCorrect : quiz.feedbackWrong,
    currentQuiz, currentStage);
}

// ── DIAL PUZZLE ──
let dialPositions = [];

function initDialTouch() {
  const quiz = stages[currentStage].quizzes[currentQuiz];
  dialPositions = new Array(quiz.answer.length).fill(0);
}

function dialSpin(wheelIdx, direction) {
  const quiz = stages[currentStage].quizzes[currentQuiz];
  const chars = quiz.charset || 'ㄱㄴㄷㄹㅁㅂㅅㅇㅈㅊㅋㅌㅍㅎ';
  const len = chars.length;

  dialPositions[wheelIdx] = (dialPositions[wheelIdx] + direction + len) % len;
  const pos = dialPositions[wheelIdx];

  const track = document.getElementById(`dialTrack${wheelIdx}`);
  track.style.transform = `translateY(-${pos * 40}px)`;

  // Update active char
  track.querySelectorAll('.dial-char').forEach((ch, i) => {
    ch.classList.toggle('active', i === pos);
  });
}

function answerDial() {
  const quiz = stages[currentStage].quizzes[currentQuiz];
  const chars = quiz.charset || 'ㄱㄴㄷㄹㅁㅂㅅㅇㅈㅊㅋㅌㅍㅎ';

  const userAnswer = dialPositions.map(pos => chars[pos]).join('');
  const isCorrect = userAnswer === quiz.answer;

  showFeedback(isCorrect,
    isCorrect ? quiz.feedbackCorrect : quiz.feedbackWrong,
    currentQuiz, currentStage);
}
```

**Step 4: Add sample match/order quiz to game data**

Replace `stages[3].quizzes[0]` (금고 Work) with a match puzzle, and convert an existing quiz to order type. Or add new quizzes. Here we convert one existing quiz to demonstrate:

Add a new match quiz to `stages[3]` (금고) — replace the first quiz:

```javascript
// Replace stages[3].quizzes[0] with:
{
  type: 'match',
  badge: '🔐 [Work] 버튼',
  question: '금고의 첫 번째 버튼을 열려면 핵심가치와 그 핵심 원칙을 올바르게 연결하세요.',
  pairs: [
    { left: 'Work', right: '책임의식 · 성취추구' },
    { left: 'Relationship', right: '협력 · 수용' },
    { left: 'Communication', right: '투명성 · 진정성' }
  ],
  feedbackCorrect: '✓ 완벽한 매칭! [Work] 버튼이 눌렸습니다!',
  feedbackWrong: '💡 각 핵심가치의 핵심 원칙을 다시 떠올려보세요.',
  hints: [
    { level: 1, text: '세 쌍을 올바르게 연결해야 합니다.', cost: 0 },
    { level: 2, text: 'Work=책임의식, Relationship=수용, Communication=투명성', cost: 50 },
    { level: 3, text: 'Work↔책임의식·성취추구, Relationship↔협력·수용, Communication↔투명성·진정성', cost: 100 }
  ]
},
```

**Step 5: Verify in browser**

- Match puzzle: drag items to connect pairs
- Order puzzle: drag to reorder
- Dial puzzle: arrows spin characters
- Touch works on mobile
- Correct/wrong feedback fires

**Step 6: Commit**

```bash
git add index.html
git commit -m "feat: add match, order, and dial puzzle types with touch support"
```

---

## Task 8: Story Enhancement — Tension + Timer Effects

**Files:**
- Modify: `index.html` — timer function + CSS

**Step 1: Add tension CSS**

```css
/* Tension effects */
.bg-darken {
  transition: opacity 2s ease;
}

.tension-text {
  position: fixed;
  bottom: 80px;
  left: 50%;
  transform: translateX(-50%);
  z-index: 30;
  color: var(--red);
  font-size: 0.85rem;
  font-style: italic;
  opacity: 0;
  animation: tensionFade 3s ease forwards;
  pointer-events: none;
  text-align: center;
  max-width: 300px;
}

@keyframes tensionFade {
  0%   { opacity: 0; transform: translateX(-50%) translateY(10px); }
  20%  { opacity: 1; transform: translateX(-50%) translateY(0); }
  80%  { opacity: 1; }
  100% { opacity: 0; }
}

@keyframes vignette {
  from { box-shadow: inset 0 0 100px rgba(0,0,0,0); }
  to   { box-shadow: inset 0 0 100px rgba(0,0,0,0.5); }
}

.vignette-overlay {
  position: fixed;
  inset: 0;
  pointer-events: none;
  z-index: 2;
  box-shadow: inset 0 0 100px rgba(0,0,0,0.5);
}
```

**Step 2: Add tension logic to timer**

```javascript
const tensionMessages = [
  { at: 180, text: '어디선가 형광등이 깜빡입니다...' },
  { at: 120, text: '복도 끝에서 이상한 소리가 들립니다...' },
  { at: 60,  text: '형광등이 하나씩 꺼지기 시작합니다...' },
  { at: 30,  text: '마지막 불빛마저 꺼지려 합니다!' }
];
let shownTension = new Set();

// Add to updateTimer():
// After the color class update:
tensionMessages.forEach(msg => {
  if (secondsLeft === msg.at && !shownTension.has(msg.at)) {
    shownTension.add(msg.at);
    showTensionText(msg.text);
  }
});

// Darken background as time runs low
if (secondsLeft <= 180) {
  const darkPct = Math.max(0, (180 - secondsLeft) / 180) * 0.3;
  document.getElementById('bg-layer').style.opacity = 1 - darkPct;
}

// Add vignette at 60s
if (secondsLeft === 60) {
  const vig = document.createElement('div');
  vig.className = 'vignette-overlay';
  vig.id = 'vignetteOverlay';
  document.body.appendChild(vig);
}
```

```javascript
function showTensionText(text) {
  const el = document.createElement('div');
  el.className = 'tension-text';
  el.textContent = text;
  document.body.appendChild(el);
  setTimeout(() => el.remove(), 3500);
}
```

**Step 3: Verify in browser**

- At 3:00, 2:00, 1:00, 0:30 — tension text appears
- Background darkens gradually after 3:00
- Vignette overlay at 1:00

**Step 4: Commit**

```bash
git add index.html
git commit -m "feat: tension system with dynamic darkness and atmospheric messages"
```

---

## Task 9: Enhanced Ending Screen — Grades + Graph + Epilogue

**Files:**
- Modify: `index.html` — ending CSS + HTML + JS

**Step 1: Add ending enhancement CSS**

```css
/* Grade badge */
.grade-badge {
  width: 80px;
  height: 80px;
  border-radius: 50%;
  display: flex;
  align-items: center;
  justify-content: center;
  font-size: 2.5rem;
  font-weight: 900;
  font-family: 'Unica 77 LL', sans-serif;
  margin: 0 auto 1rem;
  border: 3px solid;
}

.grade-S { color: #f59e0b; border-color: #f59e0b; box-shadow: 0 0 30px rgba(245,158,11,0.5); background: rgba(245,158,11,0.1); }
.grade-A { color: #10b981; border-color: #10b981; box-shadow: 0 0 20px rgba(16,185,129,0.3); background: rgba(16,185,129,0.1); }
.grade-B { color: #40E2FF; border-color: #40E2FF; box-shadow: 0 0 15px rgba(64,226,255,0.3); background: rgba(64,226,255,0.1); }
.grade-C { color: #94a3b8; border-color: #94a3b8; background: rgba(148,163,184,0.1); }

/* Stage accuracy bars */
.stage-graph {
  width: 100%;
  max-width: 400px;
  margin: 1.5rem auto;
}

.stage-bar-row {
  display: flex;
  align-items: center;
  gap: 0.5rem;
  margin-bottom: 0.5rem;
}

.stage-bar-label {
  font-size: 0.75rem;
  color: var(--muted);
  width: 70px;
  text-align: right;
  flex-shrink: 0;
}

.stage-bar-track {
  flex: 1;
  height: 12px;
  background: var(--surface2);
  border-radius: 6px;
  overflow: hidden;
}

.stage-bar-fill {
  height: 100%;
  border-radius: 6px;
  transition: width 1s ease;
  background: linear-gradient(90deg, var(--blue), var(--green));
}

.stage-bar-pct {
  font-size: 0.75rem;
  color: var(--text);
  width: 35px;
  font-weight: 700;
}

/* Epilogue */
.epilogue {
  margin-top: 1.5rem;
  padding: 1.5rem;
  background: var(--glass-bg);
  backdrop-filter: var(--glass-blur);
  border: 1px solid var(--glass-border);
  border-radius: 12px;
  max-width: 520px;
  font-size: 0.9rem;
  line-height: 1.8;
  color: var(--muted);
  font-style: italic;
}

/* Special badge */
.special-badge {
  display: inline-flex;
  align-items: center;
  gap: 0.4rem;
  padding: 0.4rem 1rem;
  border-radius: 20px;
  font-size: 0.85rem;
  font-weight: 700;
  margin-top: 1rem;
}

.badge-solo {
  background: rgba(245,158,11,0.15);
  border: 1px solid var(--gold);
  color: var(--gold);
}
```

**Step 2: Update ending HTML**

Replace ending section:

```html
<div id="ending">
  <div class="grade-badge" id="gradeBadge">S</div>
  <div class="ending-title" id="endTitle">탈출 성공!</div>
  <div class="ending-sub" id="endSub">당신은 진정한 엑셈인입니다</div>

  <div class="ending-stats">
    <div class="stat-box">
      <div class="stat-value" id="endTime">—</div>
      <div class="stat-label">소요 시간</div>
    </div>
    <div class="stat-box">
      <div class="stat-value" id="endScore">—</div>
      <div class="stat-label">최종 점수</div>
    </div>
    <div class="stat-box">
      <div class="stat-value" id="endCorrect">—</div>
      <div class="stat-label">정답률</div>
    </div>
    <div class="stat-box">
      <div class="stat-value" id="endHints">—</div>
      <div class="stat-label">힌트 사용</div>
    </div>
  </div>

  <div class="stage-graph" id="stageGraph"></div>

  <div id="specialBadges"></div>

  <div class="epilogue" id="epilogue">
    다음 날 아침, 사무실 문이 열렸습니다.<br><br>
    어젯밤의 도전이 꿈이었을까? 하지만 책상 위 메모는 여전히 남아있습니다:<br><br>
    <strong style="color:#e2e8f0;">"오늘부터 당신은 엑셈인입니다."</strong><br><br>
    전개일여(全個一如) — 당신의 성장이 곧 엑셈의 성장입니다.
  </div>

  <div class="ending-message">
    <div class="values-summary">
      <div class="value-tag">전개일여</div>
      <div class="value-tag">Passion</div>
      <div class="value-tag">Speed</div>
      <div class="value-tag">Flow</div>
      <div class="value-tag">Work</div>
      <div class="value-tag">Relationship</div>
      <div class="value-tag">Communication</div>
      <div class="value-tag">Philinnovator</div>
    </div>
  </div>

  <button class="btn-restart" onclick="restartGame()">다시 도전하기</button>
</div>
```

**Step 3: Update showEnding() JS**

```javascript
// Track per-stage stats
let stageStats = []; // { correct: 0, total: 0 } per stage

// Initialize in startGame():
stageStats = stages.map(() => ({ correct: 0, total: 0 }));

// Update showFeedback() — add stat tracking:
stageStats[stageIdx].total++;
if (isCorrect) stageStats[stageIdx].correct++;
```

Replace `showEnding()`:

```javascript
function showEnding(timeout) {
  clearInterval(timerInterval);
  hideHintButton();

  // Remove vignette
  document.getElementById('vignetteOverlay')?.remove();
  document.getElementById('bg-layer').style.opacity = 1;

  document.getElementById('game').style.display = 'none';
  const endingEl = document.getElementById('ending');
  endingEl.style.display = 'flex';

  const elapsed = Math.floor((Date.now() - startTime) / 1000);
  const em = Math.floor(elapsed / 60);
  const es = elapsed % 60;
  document.getElementById('endTime').textContent = `${em}분 ${es}초`;
  document.getElementById('endScore').textContent = score + '점';
  const pct = totalAttempts > 0 ? Math.round((totalCorrect / totalAttempts) * 100) : 0;
  document.getElementById('endCorrect').textContent = `${pct}%`;
  document.getElementById('endHints').textContent = `${totalHintsUsed}회`;

  // Grade
  let grade = 'C';
  if (pct >= 95 && totalHintsUsed === 0 && elapsed <= 300) grade = 'S';
  else if (pct >= 80 && totalHintsUsed <= 3) grade = 'A';
  else if (pct >= 60) grade = 'B';

  const gradeBadge = document.getElementById('gradeBadge');
  gradeBadge.textContent = grade;
  gradeBadge.className = `grade-badge grade-${grade}`;

  // Stage accuracy graph
  const graphEl = document.getElementById('stageGraph');
  graphEl.innerHTML = stageStats.map((s, i) => {
    const stagePct = s.total > 0 ? Math.round((s.correct / s.total) * 100) : 0;
    return `<div class="stage-bar-row">
      <div class="stage-bar-label">${stages[i].icon} ${stages[i].title}</div>
      <div class="stage-bar-track"><div class="stage-bar-fill" style="width:0%;" data-pct="${stagePct}"></div></div>
      <div class="stage-bar-pct">${stagePct}%</div>
    </div>`;
  }).join('');

  // Animate bars after render
  setTimeout(() => {
    graphEl.querySelectorAll('.stage-bar-fill').forEach(bar => {
      bar.style.width = bar.dataset.pct + '%';
    });
  }, 100);

  // Special badges
  const badgesEl = document.getElementById('specialBadges');
  let badges = [];
  if (totalHintsUsed === 0) badges.push('<span class="special-badge badge-solo">🏆 독립 탈출</span>');
  badgesEl.innerHTML = badges.join('');

  if (timeout) {
    document.getElementById('endTitle').textContent = '시간 초과!';
    gradeBadge.textContent = '⏰';
    gradeBadge.className = 'grade-badge grade-C';
    document.getElementById('endSub').textContent = '아쉽지만 다시 도전해보세요';
    document.getElementById('endSub').style.color = '#f59e0b';
    document.getElementById('epilogue').innerHTML = '시간이 다 되어 사무실 보안 시스템이 작동했습니다.<br><br>하지만 걱정 마세요 — 내일 다시 도전할 수 있습니다.<br><br><strong style="color:#e2e8f0;">"포기하지 않는 것이 엑셈인의 첫 번째 자질입니다."</strong>';
  }

  spawnConfetti(80);
}
```

**Step 4: Verify in browser**

- Grade badge shows S/A/B/C
- Stage accuracy bars animate on load
- Hint count shown
- Epilogue narrative displayed
- Timeout has different ending text

**Step 5: Commit**

```bash
git add index.html
git commit -m "feat: enhanced ending with grades, stage graph, epilogue, and special badges"
```

---

## Task 10: Final Polish — Mobile Responsive + Reset

**Files:**
- Modify: `index.html` — responsive CSS + reset function

**Step 1: Enhance responsive CSS**

```css
@media (max-width: 480px) {
  .intro-title { font-size: 1.4rem; }
  .game-header { flex-wrap: wrap; gap: 0.5rem; }
  .match-container { grid-template-columns: 1fr; gap: 0.75rem; }
  .match-arrow-col { display: none; }
  .dial-container { gap: 0.3rem; }
  .dial-wheel { width: 48px; height: 140px; }
  .stage-graph { padding: 0 0.5rem; }
  .ending-stats { gap: 0.75rem; }
  .stat-box { padding: 0.75rem 1rem; }
  .cutscene-text { font-size: 0.95rem; }
  .hint-panel { max-height: 60vh; }
  .timer-circle { width: 48px; height: 48px; }
  .timer-circle svg { width: 48px; height: 48px; }
}

@media (max-width: 360px) {
  .stage-badge { min-width: 40px; padding: 0.4rem; font-size: 1.2rem; }
  .office-map { display: none; }
}
```

**Step 2: Fix restartGame() to reset all new state**

```javascript
function restartGame() {
  // Reset all state
  hintState = {};
  totalHintsUsed = 0;
  hintPanelOpen = false;
  shownTension = new Set();
  dialPositions = [];
  document.getElementById('vignetteOverlay')?.remove();
  document.getElementById('bg-layer').style.opacity = 1;
  location.reload();
}
```

**Step 3: Verify in browser**

- Test at 320px, 375px, 768px, 1280px widths
- All puzzle types work on mobile
- Restart fully resets state
- Hint panel doesn't overflow on small screens

**Step 4: Commit**

```bash
git add index.html
git commit -m "fix: mobile responsive polish and full state reset"
```

---

## Summary

| Task | Description | Est. Lines |
|------|-------------|-----------|
| 1 | CSS Glassmorphism Design System | +60 |
| 2 | Circular SVG Timer + Progress Dots | +80 |
| 3 | 1st-Person 3D Transitions + Parallax + Cutscenes | +150 |
| 4 | Cinematic Intro Screen | +70 |
| 5 | Enhanced Quiz Interactions (Flash, Discovery, Sparkles) | +120 |
| 6 | Hint System (3-tier, UI, scoring) | +200 |
| 7 | New Puzzle Types (Match, Order, Dial) | +300 |
| 8 | Story Tension + Timer Effects | +60 |
| 9 | Enhanced Ending (Grades, Graph, Epilogue) | +150 |
| 10 | Mobile Responsive + Reset | +30 |

**Total estimated additions:** ~1,220 lines → final file ~2,700 lines
