<template>
  <div class="capsule-scene fixed inset-0 flex flex-col items-center justify-center overflow-hidden">
    <!-- 星空背景 Canvas -->
    <canvas ref="starCanvas" class="absolute inset-0 w-full h-full"></canvas>

    <!-- 装饰光晕 -->
    <div class="absolute w-96 h-96 rounded-full bg-capsule-gold/5 blur-[120px] -top-20 left-1/2 -translate-x-1/2"></div>
    <div class="absolute w-64 h-64 rounded-full bg-blue-500/5 blur-[100px] -bottom-16 left-1/3"></div>

    <!-- 标题 -->
    <div v-show="showTitle && !opening" class="title-enter z-10 mb-12">
      <h1 class="text-capsule-gold text-2xl tracking-[0.4em] font-light">
        ✦ 时空胶囊 ✦
      </h1>
    </div>

    <!-- 胶囊主体 -->
    <div class="relative z-10 cursor-pointer" @click="handleCapsuleClick" ref="capsuleRef">
      <!-- 胶囊发光光环 -->
      <div class="capsule-outer-glow" :class="{ 'capsule-glow-pulse': !opening }"></div>

      <!-- 胶囊 SVG 主体 -->
      <div class="capsule-body-svg" :class="{ 'capsule-opening': opening }">
        <svg width="120" height="200" viewBox="0 0 120 200" fill="none">
          <defs>
            <linearGradient id="bodyGrad" x1="0" y1="0" x2="1" y2="1">
              <stop offset="0%" stop-color="#e8e8e8"/>
              <stop offset="30%" stop-color="#f5f5f5"/>
              <stop offset="50%" stop-color="#d0d0d0"/>
              <stop offset="70%" stop-color="#e0e0e0"/>
              <stop offset="100%" stop-color="#b0b0b0"/>
            </linearGradient>
            <linearGradient id="goldRing" x1="0" y1="0" x2="1" y2="0">
              <stop offset="0%" stop-color="#b8860b"/>
              <stop offset="20%" stop-color="#ffd700"/>
              <stop offset="50%" stop-color="#fff8dc"/>
              <stop offset="80%" stop-color="#ffd700"/>
              <stop offset="100%" stop-color="#b8860b"/>
            </linearGradient>
            <linearGradient id="glowLine" x1="0" y1="0" x2="1" y2="0">
              <stop offset="0%" stop-color="transparent"/>
              <stop offset="50%" stop-color="rgba(255,255,255,0.4)"/>
              <stop offset="100%" stop-color="transparent"/>
            </linearGradient>
            <radialGradient id="topShine" cx="30%" cy="30%">
              <stop offset="0%" stop-color="rgba(255,255,255,0.6)"/>
              <stop offset="100%" stop-color="transparent"/>
            </radialGradient>
            <filter id="capsuleShadow">
              <feDropShadow dx="0" dy="0" stdDeviation="4" flood-color="rgba(255,215,0,0.15)"/>
            </filter>
          </defs>

          <!-- 顶部半球 -->
          <path d="M20 75 C20 40, 100 40, 100 75 L100 80 L20 80 Z"
                fill="url(#bodyGrad)" filter="url(#capsuleShadow)"/>
          <path d="M20 75 C20 40, 100 40, 100 75"
                fill="url(#topShine)" opacity="0.5"/>

          <!-- 底部半球 -->
          <path d="M20 125 L20 130 C20 165, 100 165, 100 130 L100 125 Z"
                fill="url(#bodyGrad)" filter="url(#capsuleShadow)"/>
          <path d="M100 130 C100 165, 20 165, 20 130"
                fill="url(#topShine)" opacity="0.3"
                transform="translate(0,10)"/>

          <!-- 中部连接段 -->
          <rect x="20" y="75" width="80" height="50" fill="url(#bodyGrad)"/>

          <!-- 金色环（上） -->
          <rect x="18" y="72" width="84" height="8" rx="2" fill="url(#goldRing)"/>
          <rect x="18" y="72" width="84" height="2" rx="1" fill="rgba(255,255,255,0.3)"/>

          <!-- 金色环（下） -->
          <rect x="18" y="120" width="84" height="8" rx="2" fill="url(#goldRing)"/>
          <rect x="18" y="120" width="84" height="2" rx="1" fill="rgba(255,255,255,0.3)"/>

          <!-- 中部核心装饰 -->
          <circle cx="60" cy="100" r="16" fill="none" stroke="url(#goldRing)" stroke-width="1.5" opacity="0.4"/>
          <circle cx="60" cy="100" r="8" fill="url(#goldRing)" opacity="0.3"/>
          <circle cx="60" cy="100" r="3" fill="#ffd700" opacity="0.8">
            <animate attributeName="r" values="2;4;2" dur="2s" repeatCount="indefinite"/>
            <animate attributeName="opacity" values="0.5;1;0.5" dur="2s" repeatCount="indefinite"/>
          </circle>

          <!-- 高光线 -->
          <rect x="30" y="76" width="3" height="48" rx="1.5" fill="url(#glowLine)" opacity="0.5"/>
        </svg>
      </div>
    </div>

    <!-- 提示文字 -->
    <div v-show="showTitle && !opening" class="hint-enter z-10 mt-10">
      <p class="text-gray-400 text-sm tracking-wider animate-pulse">
        {{ isMobile ? '轻触胶囊，开启学长的寄语' : '点击胶囊，开启学长的寄语' }}
      </p>
    </div>
  </div>
</template>

<script setup>
import { ref, onMounted, onUnmounted } from 'vue'

const emit = defineEmits(['complete'])
const starCanvas = ref(null)
const capsuleRef = ref(null)
const opening = ref(false)
const showTitle = ref(false)
const isMobile = ref(false)

// 星空粒子
let animFrame = null

function initStars() {
  const canvas = starCanvas.value
  if (!canvas) return
  const ctx = canvas.getContext('2d')
  canvas.width = window.innerWidth
  canvas.height = window.innerHeight

  const stars = Array.from({ length: 150 }, () => ({
    x: Math.random() * canvas.width,
    y: Math.random() * canvas.height,
    r: Math.random() * 1.8 + 0.3,
    speed: Math.random() * 0.2 + 0.03,
    opacity: Math.random() * 0.7 + 0.3,
    phase: Math.random() * Math.PI * 2,
  }))

  function draw() {
    ctx.clearRect(0, 0, canvas.width, canvas.height)
    const now = Date.now() * 0.001
    stars.forEach(s => {
      s.y -= s.speed
      if (s.y < -5) { s.y = canvas.height + 5; s.x = Math.random() * canvas.width }
      const twinkle = Math.sin(now * 2 + s.phase) * 0.3 + 0.7
      ctx.beginPath()
      ctx.arc(s.x, s.y, s.r, 0, Math.PI * 2)
      ctx.fillStyle = `rgba(255, 255, 255, ${s.opacity * twinkle})`
      ctx.fill()
    })
    animFrame = requestAnimationFrame(draw)
  }
  draw()
}

onMounted(() => {
  isMobile.value = 'ontouchstart' in window
  initStars()
  setTimeout(() => { showTitle.value = true }, 400)
  window.addEventListener('resize', initStars)
})

onUnmounted(() => {
  if (animFrame) cancelAnimationFrame(animFrame)
  window.removeEventListener('resize', initStars)
})

function handleCapsuleClick() {
  if (opening.value) return
  opening.value = true

  if (navigator.vibrate) {
    navigator.vibrate([80, 40, 80])
  }

  setTimeout(() => {
    emit('complete')
  }, 1600)
}
</script>

<style scoped>
.capsule-scene {
  background: radial-gradient(ellipse at 50% 40%, #1e1e3a 0%, #1a1a2e 40%, #0a0a1a 100%);
}

/* 标题入场 */
.title-enter {
  animation: titleIn 1s ease-out both;
}
@keyframes titleIn {
  0% { opacity: 0; transform: translateY(-20px); letter-spacing: 0.8em; }
  100% { opacity: 1; transform: translateY(0); letter-spacing: 0.4em; }
}

/* 提示入场 */
.hint-enter {
  animation: hintIn 0.8s 0.5s ease-out both;
}
@keyframes hintIn {
  0% { opacity: 0; transform: translateY(10px); }
  100% { opacity: 1; transform: translateY(0); }
}

/* 胶囊浮动 */
.capsule-body-svg {
  animation: capsuleHover 4s ease-in-out infinite;
  filter: drop-shadow(0 0 20px rgba(255, 215, 0, 0.1));
  transition: filter 0.3s;
}
.capsule-body-svg:hover {
  filter: drop-shadow(0 0 30px rgba(255, 215, 0, 0.25));
}
@keyframes capsuleHover {
  0%, 100% { transform: translateY(0) rotateY(0deg); }
  33% { transform: translateY(-12px) rotateY(3deg); }
  66% { transform: translateY(-4px) rotateY(-2deg); }
}

/* 胶囊外发光环 */
.capsule-outer-glow {
  position: absolute;
  top: 50%;
  left: 50%;
  transform: translate(-50%, -50%);
  width: 160px;
  height: 240px;
  border-radius: 80px / 120px;
  pointer-events: none;
}
.capsule-glow-pulse {
  animation: outerGlow 3s ease-in-out infinite;
}
@keyframes outerGlow {
  0%, 100% {
    box-shadow: 0 0 30px rgba(255, 215, 0, 0.08), 0 0 60px rgba(255, 215, 0, 0.03);
  }
  50% {
    box-shadow: 0 0 50px rgba(255, 215, 0, 0.15), 0 0 100px rgba(255, 215, 0, 0.05);
  }
}

/* 胶囊打开 */
.capsule-opening {
  animation: capsuleOpenAnim 1.6s ease-in-out forwards !important;
}
@keyframes capsuleOpenAnim {
  0% {
    transform: scale(1) rotateY(0deg);
    opacity: 1;
    filter: drop-shadow(0 0 20px rgba(255, 215, 0, 0.1)) brightness(1);
  }
  25% {
    transform: scale(1.3) rotateY(90deg);
    opacity: 0.9;
    filter: drop-shadow(0 0 40px rgba(255, 215, 0, 0.4)) brightness(1.3);
  }
  50% {
    transform: scale(1.6) rotateY(180deg);
    opacity: 0.6;
    filter: drop-shadow(0 0 80px rgba(255, 215, 0, 0.6)) brightness(2);
  }
  75% {
    transform: scale(2.5) rotateY(360deg);
    opacity: 0.3;
    filter: drop-shadow(0 0 150px rgba(255, 215, 0, 0.8)) brightness(3);
  }
  100% {
    transform: scale(4) rotateY(540deg);
    opacity: 0;
    filter: drop-shadow(0 0 200px rgba(255, 215, 0, 1)) brightness(5);
  }
}
</style>
