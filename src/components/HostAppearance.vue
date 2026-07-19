<template>
  <div class="host-scene fixed inset-0 flex flex-col items-center justify-center overflow-hidden">
    <!-- 渐变背景 -->
    <div class="absolute inset-0 bg-gradient-to-b from-[#1a1a2e] via-[#1e1a18] to-[#0a0a0a]"></div>

    <!-- 装饰光晕 -->
    <div class="absolute w-[500px] h-[500px] rounded-full bg-orange-500/5 blur-[150px] -top-32 right-[-20%]"></div>
    <div class="absolute w-[400px] h-[400px] rounded-full bg-yellow-500/5 blur-[120px] -bottom-24 left-[-10%]"></div>

    <!-- 浮动粒子 -->
    <canvas ref="particleCanvas" class="absolute inset-0 w-full h-full opacity-30 pointer-events-none"></canvas>

    <!-- 内容 -->
    <div class="relative z-10 flex flex-col items-center px-6 max-w-md w-full">
      <!-- 头像区域 -->
      <div class="mb-10 avatar-section" ref="avatarRef">
        <div class="relative w-32 h-32">
          <!-- 外圈光环 -->
          <div class="absolute inset-[-8px] rounded-full border border-capsule-gold/20 avatar-ring"></div>
          <!-- 头像 -->
          <div class="w-full h-full rounded-full overflow-hidden ring-2 ring-capsule-gold/30 shadow-xl shadow-capsule-gold/10">
            <img :src="config.senior.avatar"
                 class="w-full h-full object-cover"
                 alt="学长头像"
                 @error="onAvatarError"
                 ref="avatarImg" />
          </div>
          <!-- 全息扫描线 -->
          <div class="absolute inset-0 rounded-full overflow-hidden pointer-events-none">
            <div class="hologram-line"></div>
          </div>
        </div>
        <div class="text-center mt-4">
          <h2 class="text-xl font-bold text-white tracking-wider">{{ config.senior.name }}</h2>
          <p class="text-gray-500 text-sm mt-0.5">{{ config.senior.grade }} · {{ config.senior.major }}</p>
        </div>
      </div>

      <!-- 打字机区域 -->
      <div class="w-full max-w-sm mb-6">
        <div class="bg-black/50 backdrop-blur-xl rounded-2xl p-5 border border-white/5 shadow-xl">
          <p class="text-gray-200 text-sm leading-relaxed whitespace-pre-line min-h-[100px]">
            {{ displayedText }}<span v-if="!typingDone" class="inline-block text-capsule-gold animate-pulse">▊</span>
          </p>
        </div>
      </div>

      <!-- 继续按钮 -->
      <div v-show="typingDone" class="btn-enter">
        <button class="btn-gold text-base px-10 py-3.5 shadow-2xl"
                @click="goToBlog">
          继续探索 →
        </button>
      </div>
    </div>
  </div>
</template>

<script setup>
import { ref, onMounted, nextTick, onUnmounted } from 'vue'
import config from '../config.js'

defineEmits(['complete'])

function goToBlog() {
  window.open(config.senior.redirectUrl, '_blank')
}

const displayedText = ref('')
const typingDone = ref(false)
const avatarImg = ref(null)
const particleCanvas = ref(null)
const avatarRef = ref(null)

const fullText = config.senior.message.join('\n')
let charIndex = 0
let typeTimer = null
let animFrame = null

function startTyping() {
  typeTimer = setInterval(() => {
    if (charIndex < fullText.length) {
      displayedText.value += fullText[charIndex]
      charIndex++
    } else {
      clearInterval(typeTimer)
      typingDone.value = true
    }
  }, 45)
}

function onAvatarError(e) {
  e.target.style.display = 'none'
  e.target.parentElement.innerHTML += '<div class="w-full h-full flex items-center justify-center bg-gradient-to-br from-yellow-400/20 to-orange-500/20 text-4xl">👤</div>'
}

// 粒子背景
function initParticles() {
  const canvas = particleCanvas.value
  if (!canvas) return
  const ctx = canvas.getContext('2d')
  canvas.width = window.innerWidth
  canvas.height = window.innerHeight

  const particles = Array.from({ length: 30 }, () => ({
    x: Math.random() * canvas.width,
    y: Math.random() * canvas.height,
    r: Math.random() * 2 + 0.5,
    speed: Math.random() * 0.15 + 0.02,
    opacity: Math.random() * 0.3 + 0.1,
  }))

  function draw() {
    ctx.clearRect(0, 0, canvas.width, canvas.height)
    particles.forEach(p => {
      p.y -= p.speed
      if (p.y < -5) { p.y = canvas.height + 5; p.x = Math.random() * canvas.width }
      ctx.beginPath()
      ctx.arc(p.x, p.y, p.r, 0, Math.PI * 2)
      ctx.fillStyle = `rgba(255, 215, 0, ${p.opacity})`
      ctx.fill()
    })
    animFrame = requestAnimationFrame(draw)
  }
  draw()
}

onMounted(() => {
  initParticles()
  window.addEventListener('resize', initParticles)
  setTimeout(() => { avatarRef.value?.classList.add('avatar-visible') }, 200)
  setTimeout(startTyping, 600)
})

onUnmounted(() => {
  if (typeTimer) clearInterval(typeTimer)
  if (animFrame) cancelAnimationFrame(animFrame)
  window.removeEventListener('resize', initParticles)
})
</script>

<style scoped>
.avatar-section {
  opacity: 0;
  transform: translateY(20px);
  transition: all 0.8s ease-out;
}
.avatar-section.avatar-visible {
  opacity: 1;
  transform: translateY(0);
}

.avatar-ring {
  animation: ringPulse 3s ease-in-out infinite;
}
@keyframes ringPulse {
  0%, 100% { box-shadow: 0 0 15px rgba(255, 215, 0, 0.1); }
  50% { box-shadow: 0 0 35px rgba(255, 215, 0, 0.25); }
}

.hologram-line {
  position: absolute;
  left: 0;
  right: 0;
  height: 2px;
  background: linear-gradient(90deg, transparent, rgba(0, 255, 255, 0.4), transparent);
  animation: scanLine 2.5s linear infinite;
}
@keyframes scanLine {
  0% { top: -5%; }
  100% { top: 105%; }
}

/* 头像缩放发光 */
.hologram-line {
  box-shadow: 0 0 8px rgba(0, 255, 255, 0.15);
}

.btn-enter {
  animation: btnSlideIn 0.6s 0.2s ease-out both;
}
@keyframes btnSlideIn {
  0% { opacity: 0; transform: translateY(20px) scale(0.95); }
  100% { opacity: 1; transform: translateY(0) scale(1); }
}
</style>
