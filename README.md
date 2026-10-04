<!doctype html>
<html lang="zh-CN">
<head>
<meta charset="utf-8">
<meta name="viewport" content="width=device-width, initial-scale=1, viewport-fit=cover">
<title>功德 +1 · 猫耳木鱼</title>
<style>
  * { margin: 0; padding: 0; box-sizing: border-box; }

  html, body {
    height: 100%;
    background: #ffffff;
    overflow: hidden;
    -webkit-tap-highlight-color: transparent;
  }

  /* ============================================================
     背景：白底 + 浅黑色猫脚印
  ============================================================ */
  body {
    background-color: #fcfcfc;
    background-image: url("data:image/svg+xml;utf8,<svg xmlns='http://www.w3.org/2000/svg' width='120' height='120' viewBox='0 0 120 120'><g fill='%23000000' opacity='0.07'><circle cx='60' cy='60' r='10'/><circle cx='38' cy='40' r='6'/><circle cx='60' cy='34' r='6'/><circle cx='82' cy='40' r='6'/></g></svg>");
    background-size: 100px 100px;
    display: grid;
    place-items: center;
    user-select: none;
    font-family: -apple-system, "PingFang SC", "Microsoft YaHei", sans-serif;
  }

  /* ============================================================
     主容器
  ============================================================ */
  #app {
    position: relative;
    width: 100%;
    height: 100%;
    display: grid;
    place-items: center;
    cursor: pointer;
  }

  /* ============================================================
     人物图片容器
  ============================================================ */
  #character {
    position: relative;
    width: min(70vw, 360px);
    height: min(70vw, 360px);
    display: grid;
    place-items: center;
    transition: transform 0.18s cubic-bezier(0.25, 0.8, 0.25, 1);
    will-change: transform;
  }

  #character img {
    position: absolute;
    inset: 0;
    width: 100%;
    height: 100%;
    object-fit: contain;
    transition: opacity 0.1s ease;
    user-select: none;
    -webkit-user-drag: none;
    pointer-events: none;
  }

  /* 默认显示第一个（无猫耳） */
  #char-normal { opacity: 1; }
  /* 默认隐藏第二个（有猫耳） */
  #char-cat { opacity: 0; }

  /* 切换猫耳状态 */
  #character.cat-mode #char-normal { opacity: 0; }
  #character.cat-mode #char-cat { opacity: 1; }

  /* 缩小 */
  #character.shrink {
    transform: scale(0.82);
  }

  /* 放大回弹 */
  #character.pop {
    animation: pop 0.25s ease-out forwards;
  }

  @keyframes pop {
    0%   { transform: scale(0.82); }
    55%  { transform: scale(1.08); }
    100% { transform: scale(1); }
  }

  /* 轻微震动 */
  #character.shake {
    animation: shake 0.22s ease-in-out;
  }

  @keyframes shake {
    0%   { transform: translate(0, 0) rotate(0deg); }
    20%  { transform: translate(-2px, 1px) rotate(-0.8deg); }
    40%  { transform: translate(2px, -1px) rotate(0.8deg); }
    60%  { transform: translate(-1.5px, 1px) rotate(-0.5deg); }
    80%  { transform: translate(1.5px, -1px) rotate(0.5deg); }
    100% { transform: translate(0, 0) rotate(0deg); }
  }

  /* ============================================================
     功德 +1 文字
  ============================================================ */
  #text-layer {
    position: absolute;
    inset: 0;
    pointer-events: none;
    overflow: visible;
  }

  .merit-text {
    position: absolute;
    left: 50%;
    transform: translateX(-50%);
    font-size: 22px;
    font-weight: 700;
    color: #333;
    letter-spacing: 0.08em;
    text-shadow: 0 2px 12px rgba(255, 255, 255, 0.9);
    white-space: nowrap;
    pointer-events: none;
    animation: floatUp 1.1s ease-out forwards;
  }

  @keyframes floatUp {
    0%   { opacity: 0; transform: translate(-50%, 0) scale(0.8); }
    20%  { opacity: 1; transform: translate(-50%, -10px) scale(1.05); }
    60%  { opacity: 1; transform: translate(-50%, -40px) scale(1); }
    100% { opacity: 0; transform: translate(-50%, -80px) scale(0.95); }
  }
</style>
</head>
<body>

<div id="app">
  <div id="character">
    <!-- ⚠️ 请把 src 替换成你自己的图片地址或本地路径 -->
    <img id="char-normal" src="1.jpg" alt="普通形态">
    <img id="char-cat" src="2.jpg" alt="猫耳形态">
  </div>
  <div id="text-layer"></div>
</div>

<script>
(function () {
  'use strict';

  const character = document.getElementById('character');
  const textLayer = document.getElementById('text-layer');
  const app = document.getElementById('app');

  let isCat = false;
  let clickCount = 0;
  let isAnimating = false;

  /* ============================================================
     Web Audio API：合成木鱼敲击声
  ============================================================ */
  let audioCtx = null;

  function initAudio() {
    if (!audioCtx) {
      try {
        audioCtx = new (window.AudioContext || window.webkitAudioContext)();
      } catch (e) {
        console.warn('Web Audio 不支持');
      }
    }
    if (audioCtx && audioCtx.state === 'suspended') {
      audioCtx.resume();
    }
  }

  function playWoodfish() {
    if (!audioCtx) return;
    const now = audioCtx.currentTime;

    // 主音：高频三角波，快速衰减
    const osc = audioCtx.createOscillator();
    const gain = audioCtx.createGain();

    osc.type = 'triangle';
    osc.frequency.setValueAtTime(1180, now);
    osc.frequency.exponentialRampToValueAtTime(620, now + 0.08);

    gain.gain.setValueAtTime(0.0001, now);
    gain.gain.exponentialRampToValueAtTime(0.6, now + 0.005);
    gain.gain.exponentialRampToValueAtTime(0.0001, now + 0.12);

    osc.connect(gain);
    gain.connect(audioCtx.destination);

    osc.start(now);
    osc.stop(now + 0.15);

    // 低频共鸣，让声音更饱满
    const osc2 = audioCtx.createOscillator();
    const gain2 = audioCtx.createGain();

    osc2.type = 'sine';
    osc2.frequency.setValueAtTime(320, now);
    osc2.frequency.exponentialRampToValueAtTime(180, now + 0.06);

    gain2.gain.setValueAtTime(0.0001, now);
    gain2.gain.exponentialRampToValueAtTime(0.25, now + 0.004);
    gain2.gain.exponentialRampToValueAtTime(0.0001, now + 0.09);

    osc2.connect(gain2);
    gain2.connect(audioCtx.destination);

    osc2.start(now);
    osc2.stop(now + 0.1);
  }

  /* ============================================================
     点击处理
  ============================================================ */
  function handleClick() {
    initAudio();
    playWoodfish();

    // 创建功德 +1 文字
    createMeritText();

    // 震动效果
    character.classList.remove('shake');
    // 强制重排以便重新触发动画
    void character.offsetWidth;
    character.classList.add('shake');

    // 缩小 -> 切换 -> 放大
    if (!isAnimating) {
      isAnimating = true;
      character.classList.remove('pop');
      character.classList.add('shrink');

      setTimeout(function () {
        // 切换图片状态
        isCat = !isCat;
        if (isCat) {
          character.classList.add('cat-mode');
        } else {
          character.classList.remove('cat-mode');
        }

        // 放大回弹
        character.classList.remove('shrink');
        character.classList.add('pop');

        setTimeout(function () {
          character.classList.remove('pop');
          isAnimating = false;
        }, 280);
      }, 180);
    }

    clickCount++;
  }

  /* ============================================================
     功德 +1 文字特效
  ============================================================ */
  function createMeritText() {
    const el = document.createElement('div');
    el.className = 'merit-text';
    el.textContent = '功德 +1';

    // 每次点击让文字略微错开，避免完全重叠
    const offsetY = -(Math.random() * 30 + 80);
    const offsetX = (Math.random() - 0.5) * 40;

    el.style.top = '50%';
    el.style.left = '50%';
    el.style.marginTop = offsetY + 'px';
    el.style.marginLeft = offsetX + 'px';

    textLayer.appendChild(el);

    // 动画结束自动移除
    setTimeout(function () {
      if (el.parentNode) el.parentNode.removeChild(el);
    }, 1150);
  }

  /* ============================================================
     事件绑定
  ============================================================ */
  app.addEventListener('pointerdown', function (e) {
    // 避免点击文字层或人物容器以外的区域时误触发
    e.preventDefault();
    handleClick();
  });

  // 禁用右键菜单，提升体验
  app.addEventListener('contextmenu', function (e) {
    e.preventDefault();
  });

  // 移动端防止双击缩放
  let lastTouchEnd = 0;
  document.addEventListener('touchend', function (e) {
    const now = Date.now();
    if (now - lastTouchEnd <= 300) {
      e.preventDefault();
    }
    lastTouchEnd = now;
  }, false);

})();
</script>

</body>
</html>
