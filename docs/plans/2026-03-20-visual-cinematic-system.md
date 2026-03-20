# Visual Cinematic Intro & Cutscene System Implementation Plan

> **For Claude:** REQUIRED SUB-SKILL: Use superpowers:executing-plans to implement this plan task-by-task.

**Goal:** Replace the text-only intro and cutscenes with a cinematic scene sequencer that plays multi-scene sequences with background images, transitions, and visual effects.

**Architecture:** A data-driven scene sequencer engine (`playCinematic`) renders scenes sequentially with configurable transitions (fadeIn, slideLeft, zoomIn, blur, glitch), text effects (typewriter, fadeLines, reveal), and visual effects (vignette, particles, glitch). The intro becomes a 6-scene cinematic, and each stage cutscene becomes a 2-3 scene cinematic. All changes are in the single `index.html` file.

**Tech Stack:** Vanilla HTML/CSS/JS, CSS animations, Canvas API (particles only)

---

## Task 1: Add Cinematic CSS Styles

**Files:**
- Modify: `index.html:856-948` (after existing cutscene CSS, replace intro animation CSS)

**Step 1: Add cinematic overlay and scene CSS**

Insert the following CSS after the existing `.cutscene-skip:hover` rule (line 901) and before the `@keyframes fadeIn` (line 903). This replaces the old cutscene overlay styles with the new cinematic system styles.

Replace the entire block from line 857 (`.cutscene-overlay {`) through line 948 (last `intro-story-line` delay) with:

```css
/* ── CINEMATIC SYSTEM ── */
.cinematic-overlay {
  position: fixed;
  inset: 0;
  z-index: 200;
  background: #000;
  overflow: hidden;
}

.cinematic-scene {
  position: absolute;
  inset: 0;
  display: flex;
  flex-direction: column;
  align-items: center;
  justify-content: center;
  padding: 2rem;
  opacity: 0;
}

.cinematic-bg {
  position: absolute;
  inset: 0;
  background-size: cover;
  background-position: center;
  z-index: 0;
}

.cinematic-bg-overlay {
  position: absolute;
  inset: 0;
  z-index: 1;
}

.cinematic-bg-overlay.dark {
  background: rgba(0, 0, 0, 0.6);
}

.cinematic-bg-overlay.darker {
  background: rgba(0, 0, 0, 0.75);
}

.cinematic-vignette {
  position: absolute;
  inset: 0;
  box-shadow: inset 0 0 150px rgba(0,0,0,0.7);
  z-index: 2;
  pointer-events: none;
}

.cinematic-content {
  position: relative;
  z-index: 3;
  max-width: 560px;
  text-align: center;
  display: flex;
  flex-direction: column;
  align-items: center;
  gap: 1rem;
}

.cinematic-text {
  font-size: 1.15rem;
  line-height: 2;
  color: var(--muted);
  font-style: italic;
}

.cinematic-text .highlight {
  color: var(--blue-light);
  font-style: normal;
  font-weight: 600;
}

.cinematic-text-line {
  opacity: 0;
  transform: translateY(12px);
}

.cinematic-text-line.visible {
  opacity: 1;
  transform: translateY(0);
  transition: opacity 0.5s ease, transform 0.5s ease;
}

.cinematic-skip {
  position: fixed;
  top: 1.5rem;
  right: 1.5rem;
  z-index: 210;
  padding: 0.5rem 1.2rem;
  border-radius: 8px;
  border: 1px solid rgba(255,255,255,0.2);
  background: rgba(0,0,0,0.5);
  color: var(--muted);
  font-size: 0.82rem;
  cursor: pointer;
  transition: all var(--transition-fast);
  backdrop-filter: blur(8px);
}

.cinematic-skip:hover {
  border-color: var(--blue);
  color: var(--blue);
}

.cinematic-dots {
  position: fixed;
  bottom: 2rem;
  left: 50%;
  transform: translateX(-50%);
  z-index: 210;
  display: flex;
  gap: 0.5rem;
}

.cinematic-dot {
  width: 8px;
  height: 8px;
  border-radius: 50%;
  background: rgba(255,255,255,0.25);
  transition: all 0.3s;
}

.cinematic-dot.active {
  background: var(--blue);
  box-shadow: 0 0 8px var(--blue);
}

.cinematic-dot.done {
  background: rgba(255,255,255,0.5);
}

/* Cinematic logo */
.cinematic-logo {
  height: 80px;
  animation: glowPulse 3s ease-in-out infinite;
}

/* Cinematic start button (intro scene 6) */
.cinematic-start-area {
  display: flex;
  flex-direction: column;
  align-items: center;
  gap: 1.5rem;
}

.cinematic-stage-list {
  display: flex;
  gap: 0.75rem;
  flex-wrap: wrap;
  justify-content: center;
}

.cinematic-stage-icon {
  width: 48px;
  height: 48px;
  border-radius: 12px;
  background: var(--glass-bg);
  border: 1px solid var(--glass-border);
  display: flex;
  align-items: center;
  justify-content: center;
  font-size: 1.4rem;
  opacity: 0;
  transform: scale(0.5);
  transition: opacity 0.4s ease, transform 0.4s ease;
}

.cinematic-stage-icon.visible {
  opacity: 1;
  transform: scale(1);
}

/* ── TRANSITION ANIMATIONS ── */
@keyframes cin-fadeIn {
  from { opacity: 0; }
  to   { opacity: 1; }
}

@keyframes cin-fadeOut {
  from { opacity: 1; }
  to   { opacity: 0; }
}

@keyframes cin-slideLeft {
  from { opacity: 0; transform: translateX(100px); }
  to   { opacity: 1; transform: translateX(0); }
}

@keyframes cin-slideLeftOut {
  from { opacity: 1; transform: translateX(0); }
  to   { opacity: 0; transform: translateX(-100px); }
}

@keyframes cin-zoomIn {
  from { opacity: 0; transform: scale(1.3); }
  to   { opacity: 1; transform: scale(1); }
}

@keyframes cin-zoomInOut {
  from { opacity: 1; transform: scale(1); }
  to   { opacity: 0; transform: scale(0.8); }
}

@keyframes cin-blur {
  from { opacity: 0; filter: blur(20px); }
  to   { opacity: 1; filter: blur(0); }
}

@keyframes cin-blurOut {
  from { opacity: 1; filter: blur(0); }
  to   { opacity: 0; filter: blur(20px); }
}

@keyframes cin-glitch {
  0%   { opacity: 0; transform: translate(0); filter: hue-rotate(0deg); }
  10%  { opacity: 0.8; transform: translate(-5px, 3px); filter: hue-rotate(90deg); }
  20%  { opacity: 0.6; transform: translate(5px, -2px); filter: hue-rotate(180deg); }
  30%  { opacity: 0.9; transform: translate(-3px, 0); filter: hue-rotate(270deg); }
  40%  { opacity: 0.7; transform: translate(2px, 2px); filter: hue-rotate(0deg); }
  50%  { opacity: 1; transform: translate(0); filter: hue-rotate(0deg); }
  100% { opacity: 1; transform: translate(0); filter: hue-rotate(0deg); }
}

@keyframes cin-shake {
  0%, 100% { transform: translateX(0); }
  10%, 30%, 50%, 70%, 90% { transform: translateX(-4px); }
  20%, 40%, 60%, 80% { transform: translateX(4px); }
}

/* Intro logo & button animations (kept) */
@keyframes glowPulse {
  0%, 100% { filter: drop-shadow(0 0 8px rgba(64, 226, 255, 0.4)); }
  50%      { filter: drop-shadow(0 0 24px rgba(64, 226, 255, 0.8)); }
}

@keyframes btnPulse {
  0%, 100% { box-shadow: 0 0 0 0 rgba(64, 226, 255, 0.4); }
  50%      { box-shadow: 0 0 0 12px rgba(64, 226, 255, 0); }
}

/* Particles canvas */
.cinematic-particles {
  position: absolute;
  inset: 0;
  z-index: 2;
  pointer-events: none;
}

/* Responsive */
@media (max-width: 600px) {
  .cinematic-text { font-size: 0.95rem; }
  .cinematic-logo { height: 60px; }
  .cinematic-skip { top: 1rem; right: 1rem; }
  .cinematic-dots { bottom: 1.5rem; }
}
```

**Step 2: Verify no CSS syntax errors**

Open the page in browser, confirm no broken styles. The old intro and cutscene elements will still exist but won't be used until the JS is updated.

**Step 3: Commit**

```bash
git add index.html
git commit -m "feat: add cinematic system CSS styles

Replaces old cutscene overlay and intro animation CSS with new
cinematic system styles including transitions, effects, and responsive design."
```

---

## Task 2: Implement Scene Sequencer Engine (JS)

**Files:**
- Modify: `index.html:2376-2408` (replace `showCutscene` function)

**Step 1: Add scene sequencer engine**

Replace the `showCutscene` function (lines 2376-2408) with the cinematic engine. Insert this code at line 2376:

```javascript
// ── CINEMATIC ENGINE ──
let cinematicActive = false;
let cinematicCleanup = null;

function playCinematic(scenes, callback, options = {}) {
  if (cinematicActive) return;
  cinematicActive = true;

  const overlay = document.createElement('div');
  overlay.className = 'cinematic-overlay';

  // Skip button
  const skipBtn = document.createElement('button');
  skipBtn.className = 'cinematic-skip';
  skipBtn.textContent = '건너뛰기 >>';
  overlay.appendChild(skipBtn);

  // Progress dots
  const dots = document.createElement('div');
  dots.className = 'cinematic-dots';
  scenes.forEach((_, i) => {
    const dot = document.createElement('div');
    dot.className = 'cinematic-dot' + (i === 0 ? ' active' : '');
    dots.appendChild(dot);
  });
  overlay.appendChild(dots);

  document.body.appendChild(overlay);

  let currentScene = 0;
  let sceneTimeout = null;
  let particlesCtx = null;
  let particlesAnim = null;
  let particles = [];

  function updateDots(idx) {
    const allDots = dots.querySelectorAll('.cinematic-dot');
    allDots.forEach((d, i) => {
      d.className = 'cinematic-dot';
      if (i < idx) d.classList.add('done');
      else if (i === idx) d.classList.add('active');
    });
  }

  function createParticles(container) {
    const canvas = document.createElement('canvas');
    canvas.className = 'cinematic-particles';
    canvas.width = window.innerWidth;
    canvas.height = window.innerHeight;
    container.appendChild(canvas);
    particlesCtx = canvas.getContext('2d');
    particles = [];
    for (let i = 0; i < 25; i++) {
      particles.push({
        x: Math.random() * canvas.width,
        y: Math.random() * canvas.height,
        r: Math.random() * 2 + 0.5,
        dx: (Math.random() - 0.5) * 0.5,
        dy: (Math.random() - 0.5) * 0.3 - 0.2,
        alpha: Math.random() * 0.5 + 0.2
      });
    }
    function animParticles() {
      particlesCtx.clearRect(0, 0, canvas.width, canvas.height);
      particles.forEach(p => {
        p.x += p.dx;
        p.y += p.dy;
        if (p.x < 0) p.x = canvas.width;
        if (p.x > canvas.width) p.x = 0;
        if (p.y < 0) p.y = canvas.height;
        if (p.y > canvas.height) p.y = 0;
        particlesCtx.beginPath();
        particlesCtx.arc(p.x, p.y, p.r, 0, Math.PI * 2);
        particlesCtx.fillStyle = `rgba(64, 226, 255, ${p.alpha})`;
        particlesCtx.fill();
      });
      particlesAnim = requestAnimationFrame(animParticles);
    }
    animParticles();
  }

  function applyTextEffect(container, text, effect, speed) {
    if (effect === 'fadeLines') {
      const lines = text.split('\n').filter(l => l.trim());
      container.innerHTML = '';
      lines.forEach((line, i) => {
        const lineEl = document.createElement('div');
        lineEl.className = 'cinematic-text-line';
        lineEl.textContent = line;
        container.appendChild(lineEl);
        setTimeout(() => lineEl.classList.add('visible'), (i + 1) * 400);
      });
    } else if (effect === 'reveal') {
      container.style.clipPath = 'inset(0 100% 0 0)';
      container.textContent = text;
      container.style.opacity = '1';
      container.style.transition = 'clip-path 1.2s ease';
      requestAnimationFrame(() => {
        container.style.clipPath = 'inset(0 0% 0 0)';
      });
    } else {
      // typewriter (default)
      typeText(container, text, speed || 30);
    }
  }

  function getTransitionIn(type) {
    const map = {
      fadeIn: 'cin-fadeIn 0.8s ease forwards',
      slideLeft: 'cin-slideLeft 0.6s ease forwards',
      zoomIn: 'cin-zoomIn 0.8s ease forwards',
      blur: 'cin-blur 1s ease forwards',
      glitch: 'cin-glitch 0.8s ease forwards'
    };
    return map[type] || map.fadeIn;
  }

  function getTransitionOut(type) {
    const map = {
      fadeIn: 'cin-fadeOut 0.5s ease forwards',
      slideLeft: 'cin-slideLeftOut 0.5s ease forwards',
      zoomIn: 'cin-zoomInOut 0.5s ease forwards',
      blur: 'cin-blurOut 0.5s ease forwards',
      glitch: 'cin-fadeOut 0.4s ease forwards'
    };
    return map[type] || map.fadeIn;
  }

  function showScene(idx) {
    if (idx >= scenes.length) {
      closeCinematic();
      return;
    }

    currentScene = idx;
    updateDots(idx);

    const scene = scenes[idx];
    const sceneEl = document.createElement('div');
    sceneEl.className = 'cinematic-scene';

    // Background
    if (scene.bg) {
      const bg = document.createElement('div');
      bg.className = 'cinematic-bg';
      if (scene.bg.startsWith('http') || scene.bg.startsWith('/') || scene.bg.startsWith('.')) {
        bg.style.backgroundImage = `url('${scene.bg}')`;
      } else {
        bg.style.background = scene.bg;
      }
      sceneEl.appendChild(bg);

      // Dark overlay
      const bgOverlay = document.createElement('div');
      bgOverlay.className = 'cinematic-bg-overlay ' + (scene.overlay || 'dark');
      sceneEl.appendChild(bgOverlay);
    }

    // Vignette
    if (!scene.effects || scene.effects.includes('vignette')) {
      const vig = document.createElement('div');
      vig.className = 'cinematic-vignette';
      sceneEl.appendChild(vig);
    }

    // Particles
    if (scene.effects && scene.effects.includes('particles')) {
      createParticles(sceneEl);
    }

    // Content
    const content = document.createElement('div');
    content.className = 'cinematic-content';

    // Custom HTML content (for intro scenes with logo, buttons, etc.)
    if (scene.html) {
      content.innerHTML = scene.html;
      sceneEl.appendChild(content);
      overlay.appendChild(sceneEl);
      sceneEl.style.animation = getTransitionIn(scene.transition);

      // Initialize elements within custom HTML
      if (scene.onMount) {
        setTimeout(() => scene.onMount(content, overlay), 100);
      }

      // No auto-advance for interactive scenes (like start button)
      if (scene.interactive) return;
    } else if (scene.text) {
      const textEl = document.createElement('div');
      textEl.className = 'cinematic-text';
      content.appendChild(textEl);
      sceneEl.appendChild(content);
      overlay.appendChild(sceneEl);
      sceneEl.style.animation = getTransitionIn(scene.transition);

      // Apply text effect
      setTimeout(() => {
        applyTextEffect(textEl, scene.text, scene.textEffect, scene.textSpeed);
      }, 300);
    } else {
      sceneEl.appendChild(content);
      overlay.appendChild(sceneEl);
      sceneEl.style.animation = getTransitionIn(scene.transition);
    }

    // Shake effect
    if (scene.effects && scene.effects.includes('shake')) {
      setTimeout(() => {
        sceneEl.style.animation = 'cin-shake 0.5s ease';
        setTimeout(() => {
          sceneEl.style.animation = '';
          sceneEl.style.opacity = '1';
        }, 500);
      }, scene.shakeDelay || 1500);
    }

    // Auto-advance
    const duration = scene.duration || 3500;
    sceneTimeout = setTimeout(() => {
      advanceScene(sceneEl, scene);
    }, duration);
  }

  function advanceScene(sceneEl, scene) {
    if (sceneTimeout) { clearTimeout(sceneTimeout); sceneTimeout = null; }
    if (particlesAnim) { cancelAnimationFrame(particlesAnim); particlesAnim = null; }

    sceneEl.style.animation = getTransitionOut(scene.transition);
    setTimeout(() => {
      sceneEl.remove();
      showScene(currentScene + 1);
    }, 500);
  }

  function closeCinematic() {
    if (!cinematicActive) return;
    cinematicActive = false;
    if (sceneTimeout) { clearTimeout(sceneTimeout); sceneTimeout = null; }
    if (particlesAnim) { cancelAnimationFrame(particlesAnim); particlesAnim = null; }
    if (typingTimer) { clearTimeout(typingTimer); typingTimer = null; }

    overlay.style.animation = 'cin-fadeOut 0.5s ease forwards';
    setTimeout(() => {
      overlay.remove();
      if (callback) callback();
    }, 500);
  }

  // Skip button
  skipBtn.onclick = closeCinematic;

  // Click/tap to advance scene
  overlay.addEventListener('click', (e) => {
    if (e.target === skipBtn || e.target.closest('.cinematic-skip')) return;
    if (e.target.closest('.btn-start') || e.target.closest('.cinematic-stage-icon')) return;

    const currentSceneEl = overlay.querySelector('.cinematic-scene');
    if (currentSceneEl && currentScene < scenes.length) {
      const scene = scenes[currentScene];
      if (scene.interactive) return; // don't skip interactive scenes
      advanceScene(currentSceneEl, scene);
    }
  });

  cinematicCleanup = closeCinematic;

  // Start first scene
  showScene(0);
}

// Legacy wrapper for backward compatibility
function showCutscene(text, callback) {
  const scenes = [{
    text: text,
    transition: 'fadeIn',
    duration: Math.max(text.length * 35 + 2000, 5000),
    textEffect: 'typewriter',
    overlay: 'dark'
  }];
  playCinematic(scenes, callback);
}
```

**Step 2: Verify the legacy `showCutscene` wrapper still works**

Test by playing the game and advancing past stage 1. The existing cutscene text should display in the new cinematic overlay.

**Step 3: Commit**

```bash
git add index.html
git commit -m "feat: implement cinematic scene sequencer engine

Adds playCinematic() function with support for multiple scene types,
transitions (fadeIn, slideLeft, zoomIn, blur, glitch), text effects
(typewriter, fadeLines, reveal), particles, vignette, skip button,
and progress dots. Maintains backward-compatible showCutscene() wrapper."
```

---

## Task 3: Replace Intro with Cinematic Sequence

**Files:**
- Modify: `index.html:1635-1655` (intro HTML)
- Modify: `index.html:2267-2295` (startGame function)
- Modify: `index.html:60-71` (intro CSS)

**Step 1: Simplify the intro HTML**

Replace lines 1635-1655 (the `<div id="intro">...</div>`) with a minimal placeholder:

```html
<div id="intro">
  <button class="btn-start" onclick="showIntro()" style="display:none" id="introAutoStart"></button>
</div>
```

**Step 2: Simplify intro CSS**

Replace the `#intro` CSS block (lines 60-71) with:

```css
/* ── INTRO SCREEN ── */
#intro {
  width: 100%;
  min-height: 100vh;
  display: flex;
  align-items: center;
  justify-content: center;
  background: #000;
}
```

**Step 3: Add intro scene data and `showIntro()` function**

Insert the following right before the `startGame()` function (at line 2267):

```javascript
// ── INTRO CINEMATIC ──
const introScenes = [
  {
    bg: '#000',
    html: '<img src="logo.png" alt="EXEM" class="cinematic-logo">',
    transition: 'fadeIn',
    duration: 3000,
    overlay: 'none',
    effects: ['vignette']
  },
  {
    bg: 'https://images.unsplash.com/photo-1497366216548-37526070297c?auto=format&fit=crop&w=1920&q=80',
    text: '2026년 3월, 입사 첫날.\n\n퇴근 시간이 지났지만, 사무실에 혼자 남겨졌다.',
    transition: 'blur',
    duration: 4000,
    textEffect: 'typewriter',
    textSpeed: 35,
    overlay: 'darker',
    effects: ['vignette']
  },
  {
    bg: 'https://images.unsplash.com/photo-1518455027359-f3f8164ba6bd?auto=format&fit=crop&w=1920&q=80',
    text: '책상 위에 놓인 메모 한 장이 눈에 들어온다.\n\n"엑셈의 핵심가치를 증명하라."',
    transition: 'zoomIn',
    duration: 4000,
    textEffect: 'typewriter',
    textSpeed: 35,
    overlay: 'darker',
    effects: ['vignette']
  },
  {
    bg: 'https://images.unsplash.com/photo-1558618666-fcd25c85f82e?auto=format&fit=crop&w=1920&q=80',
    text: '문을 열려 했지만—\n\n딸깍. 잠겨 있다.',
    transition: 'slideLeft',
    duration: 3500,
    textEffect: 'typewriter',
    textSpeed: 40,
    overlay: 'darker',
    effects: ['vignette', 'shake'],
    shakeDelay: 2000
  },
  {
    bg: 'https://images.unsplash.com/photo-1497215842964-222b430dc094?auto=format&fit=crop&w=1920&q=80',
    text: 'EXEM의 핵심가치를 증명하면\n탈출할 수 있다.',
    transition: 'glitch',
    duration: 4000,
    textEffect: 'fadeLines',
    overlay: 'darker',
    effects: ['vignette']
  },
  {
    bg: 'radial-gradient(ellipse at 50% 30%, rgba(30,42,74,0.8) 0%, rgba(0,0,0,0.95) 70%)',
    html: '<div class="cinematic-start-area"></div>',
    transition: 'fadeIn',
    duration: 999999,
    interactive: true,
    overlay: 'none',
    effects: ['vignette', 'particles'],
    onMount: function(content, overlay) {
      const area = content.querySelector('.cinematic-start-area');
      // Stage icons
      const icons = document.createElement('div');
      icons.className = 'cinematic-stage-list';
      const stageEmojis = ['📁', '🗄️', '💻', '🔒', '🚪'];
      const stageNames = ['책상 서랍', '파일 캐비닛', '컴퓨터', '금고', '탈출 출구'];
      stageEmojis.forEach((emoji, i) => {
        const icon = document.createElement('div');
        icon.className = 'cinematic-stage-icon';
        icon.textContent = emoji;
        icon.title = stageNames[i];
        icons.appendChild(icon);
        setTimeout(() => icon.classList.add('visible'), (i + 1) * 200);
      });
      area.appendChild(icons);

      // Start button
      const btn = document.createElement('button');
      btn.className = 'btn-start';
      btn.textContent = '탈출 시작하기 →';
      btn.style.animation = 'btnPulse 2s ease-in-out infinite';
      btn.style.opacity = '0';
      btn.style.transition = 'opacity 0.5s ease';
      btn.onclick = () => {
        cinematicActive = false;
        overlay.style.animation = 'cin-fadeOut 0.8s ease forwards';
        setTimeout(() => {
          overlay.remove();
          startGame();
        }, 800);
      };
      area.appendChild(btn);
      setTimeout(() => { btn.style.opacity = '1'; }, 1200);
    }
  }
];

function showIntro() {
  playCinematic(introScenes, null);
}
```

**Step 4: Modify the page load to auto-play intro**

Add a DOMContentLoaded listener or modify the existing init to call `showIntro()` on page load. Add right after the `showIntro` function definition:

```javascript
// Auto-play intro on page load
document.addEventListener('DOMContentLoaded', () => {
  showIntro();
});
```

**Step 5: Update `startGame()` to work without the old intro HTML**

Modify the `startGame()` function at line 2267. Change the first line:

```javascript
function startGame() {
  document.getElementById('intro').style.display = 'none';
```

Keep this line as-is — it hides the placeholder `#intro` div.

**Step 6: Test the complete intro flow**

1. Reload the page — cinematic should auto-play
2. Click through scenes or wait for auto-advance
3. Scene 6 should show stage icons + start button
4. Clicking "탈출 시작하기" should start the game normally
5. Test "건너뛰기 >>" button at any point

**Step 7: Commit**

```bash
git add index.html
git commit -m "feat: replace text intro with 6-scene cinematic sequence

Intro now plays as a cinematic with background images, transitions
(blur, zoomIn, slideLeft, glitch, fadeIn), typewriter text, vignette
effects, and particle effects on the final scene."
```

---

## Task 4: Convert Stage Cutscenes to Cinematic Sequences

**Files:**
- Modify: `index.html:1997-2252` (stages array — change `cutscene` string → `cutsceneScenes` array)
- Modify: `index.html` (around line 2927 — `nextStage` function)

**Step 1: Replace cutscene strings with scene arrays in stages data**

For each stage that has a `cutscene` property, replace it with `cutsceneScenes` array.

**Stage 2 (파일 캐비닛) — line ~2042:**

Replace:
```javascript
cutscene: '서랍에서 찾은 단서 카드를 손에 쥐고 일어섰습니다.\n\n사무실 구석, 파일 캐비닛의 금속 표면이 희미하게 빛납니다.\n\n발걸음을 옮기자 복도의 형광등이 한 번 깜빡입니다...',
```

With:
```javascript
cutsceneScenes: [
  {
    bg: 'https://images.unsplash.com/photo-1749301630912-efc7d86b5699?auto=format&fit=crop&w=1920&q=80',
    text: '서랍에서 찾은 단서 카드를 손에 쥐고 일어섰습니다.',
    transition: 'zoomIn',
    duration: 3000,
    textEffect: 'typewriter',
    textSpeed: 30,
    overlay: 'darker',
    effects: ['vignette']
  },
  {
    bg: 'https://images.unsplash.com/photo-1569235186275-626cb53b83ce?auto=format&fit=crop&w=1920&q=80',
    text: '사무실 구석, 파일 캐비닛의 금속 표면이 희미하게 빛납니다.\n\n발걸음을 옮기자 복도의 형광등이 한 번 깜빡입니다...',
    transition: 'slideLeft',
    duration: 3500,
    textEffect: 'typewriter',
    textSpeed: 30,
    overlay: 'darker',
    effects: ['vignette']
  }
],
```

**Stage 3 (컴퓨터) — line ~2084:**

Replace:
```javascript
cutscene: '캐비닛에서 찾은 카드 3장을 주머니에 넣었습니다.\n\n책상 위 컴퓨터 화면이 대기 모드로 깜빡이고 있습니다.\n\n비밀번호 입력창이 당신을 기다리고 있습니다...',
```

With:
```javascript
cutsceneScenes: [
  {
    bg: 'https://images.unsplash.com/photo-1569235186275-626cb53b83ce?auto=format&fit=crop&w=1920&q=80',
    text: '캐비닛에서 찾은 카드 3장을 주머니에 넣었습니다.',
    transition: 'fadeIn',
    duration: 3000,
    textEffect: 'typewriter',
    textSpeed: 30,
    overlay: 'darker',
    effects: ['vignette']
  },
  {
    bg: 'https://images.unsplash.com/photo-1520583457224-aee11bad5112?auto=format&fit=crop&w=1920&q=80',
    text: '컴퓨터 화면이 깜빡이며 켜집니다.\n\n비밀번호 입력창이 당신을 기다리고 있습니다...',
    transition: 'glitch',
    duration: 3500,
    textEffect: 'typewriter',
    textSpeed: 30,
    overlay: 'darker',
    effects: ['vignette']
  }
],
```

**Stage 4 (금고) — line ~2142:**

Replace:
```javascript
cutscene: '컴퓨터 화면에 나타난 지도를 따라 책장 뒤로 향합니다.\n\n무거운 책장을 밀자 — 먼지가 날리며 금고가 드러났습니다.\n\n세 개의 버튼이 차가운 금속 위에서 빛나고 있습니다...',
```

With:
```javascript
cutsceneScenes: [
  {
    bg: 'https://images.unsplash.com/photo-1520583457224-aee11bad5112?auto=format&fit=crop&w=1920&q=80',
    text: '모니터에 나타난 지도를 따라 책장 뒤로 향합니다.',
    transition: 'blur',
    duration: 3000,
    textEffect: 'typewriter',
    textSpeed: 30,
    overlay: 'darker',
    effects: ['vignette']
  },
  {
    bg: 'https://images.unsplash.com/photo-1636572644856-219441002f7f?auto=format&fit=crop&w=1920&q=80',
    text: '무거운 책장을 밀자 — 먼지가 날리며 금고가 드러났습니다.\n\n세 개의 버튼이 차가운 금속 위에서 빛나고 있습니다...',
    transition: 'slideLeft',
    duration: 3500,
    textEffect: 'typewriter',
    textSpeed: 30,
    overlay: 'darker',
    effects: ['vignette']
  }
],
```

**Stage 5 (탈출 출구) — line ~2198:**

Replace:
```javascript
cutscene: '금고에서 꺼낸 열쇠를 꽉 쥐고 출구로 달려갑니다.\n\n복도 끝, 마지막 문 앞에 도착했습니다.\n\n열쇠를 넣으려는 순간 — 마지막 잠금장치가 보입니다...',
```

With:
```javascript
cutsceneScenes: [
  {
    bg: 'https://images.unsplash.com/photo-1636572644856-219441002f7f?auto=format&fit=crop&w=1920&q=80',
    text: '금고에서 꺼낸 열쇠를 꽉 쥐고 출구로 달려갑니다.',
    transition: 'zoomIn',
    duration: 3000,
    textEffect: 'typewriter',
    textSpeed: 30,
    overlay: 'darker',
    effects: ['vignette']
  },
  {
    bg: 'https://images.unsplash.com/photo-1497215842964-222b430dc094?auto=format&fit=crop&w=1920&q=80',
    text: '복도 끝, 출구를 향해 마지막 발걸음을 옮깁니다.',
    transition: 'slideLeft',
    duration: 3000,
    textEffect: 'typewriter',
    textSpeed: 30,
    overlay: 'darker',
    effects: ['vignette']
  },
  {
    bg: 'https://images.unsplash.com/photo-1681837026486-8964ee84b6b9?auto=format&fit=crop&w=1920&q=80',
    text: '열쇠를 넣으려는 순간—\n\n마지막 잠금장치가 보입니다...',
    transition: 'fadeIn',
    duration: 3500,
    textEffect: 'fadeLines',
    overlay: 'darker',
    effects: ['vignette']
  }
],
```

**Step 2: Update `nextStage()` to use `cutsceneScenes`**

In the `nextStage()` function (around line 2920), replace the cutscene handling:

Replace:
```javascript
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
```

With:
```javascript
  const nextScenes = stages[nextIdx].cutsceneScenes;

  if (nextScenes) {
    playCinematic(nextScenes, () => {
      addFlicker();
      setTimeout(() => {
        loadStage(nextIdx, true);
        window.scrollTo({ top: 0, behavior: 'smooth' });
      }, 150);
    });
  } else {
```

**Step 3: Test each stage transition**

Play through the game and verify:
1. Stage 1→2 cutscene: 2 scenes (zoomIn → slideLeft)
2. Stage 2→3 cutscene: 2 scenes (fadeIn → glitch)
3. Stage 3→4 cutscene: 2 scenes (blur → slideLeft)
4. Stage 4→5 cutscene: 3 scenes (zoomIn → slideLeft → fadeIn)
5. Each cutscene has skip button and click-to-advance
6. Progress dots show correctly

**Step 4: Commit**

```bash
git add index.html
git commit -m "feat: convert stage cutscenes to multi-scene cinematics

Each inter-stage cutscene now plays as 2-3 scene cinematic sequence
with unique backgrounds, transitions, and text effects instead of
a single text overlay."
```

---

## Task 5: Clean Up Old CSS and Test Full Flow

**Files:**
- Modify: `index.html` (remove unused old intro/cutscene CSS classes)

**Step 1: Remove old unused CSS**

Remove the following CSS rules that are no longer used (they were replaced in Task 1):
- `.intro-logo` (line ~73)
- `.intro-sub` (line ~82)
- `.intro-title` (line ~89)
- `.intro-story` (line ~96)
- `.intro-story em` (line ~108)
- `.stage-list` (line ~113)
- `.stage-badge` and related styles (line ~951-980)
- Old `.btn-start` styles can be kept (reused in cinematic)

Also remove the responsive `.cutscene-text` rule at line ~1595.

Keep the `.btn-start` CSS as it's reused in the cinematic start button.

**Step 2: Full end-to-end test**

1. Reload page → intro cinematic plays (6 scenes)
2. Skip button works at any point
3. Click-to-advance works
4. Start button on scene 6 launches game
5. Complete stage 1 → cinematic plays before stage 2
6. Complete stage 2 → cinematic plays before stage 3
7. Complete stage 3 → cinematic plays before stage 4
8. Complete stage 4 → cinematic plays before stage 5
9. All transitions animate correctly
10. Progress dots update
11. Mobile responsiveness works
12. Timer continues during cutscenes

**Step 3: Commit**

```bash
git add index.html
git commit -m "refactor: remove unused old intro/cutscene CSS

Cleans up CSS rules for the old text-based intro and cutscene overlay
that were replaced by the cinematic system."
```

---

## Summary

| Task | Description | Est. Lines Changed |
|------|------------|-------------------|
| 1 | Cinematic CSS styles | ~250 lines added, ~90 removed |
| 2 | Scene sequencer engine (JS) | ~200 lines added, ~30 removed |
| 3 | Intro cinematic scenes | ~100 lines added, ~20 removed |
| 4 | Stage cutscene scenes | ~80 lines modified |
| 5 | Cleanup & test | ~30 lines removed |

**Total: ~630 lines added, ~170 removed**
