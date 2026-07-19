<template>
  <div class="handover-scene fixed inset-0 flex flex-col items-center justify-center overflow-hidden">
    <!-- 背景 -->
    <div class="absolute inset-0 bg-gradient-to-b from-[#1a1a2e] via-[#151515] to-[#1a1212]"></div>

    <!-- 烟花粒子 Canvas -->
    <canvas ref="particleCanvas" class="absolute inset-0 w-full h-full pointer-events-none"></canvas>

    <!-- 装饰光晕 -->
    <div class="absolute w-[400px] h-[400px] rounded-full bg-capsule-gold/5 blur-[120px] top-1/2 left-1/2 -translate-x-1/2 -translate-y-1/2"></div>

    <!-- 内容 -->
    <div class="relative z-10 flex flex-col items-center px-6 max-w-md w-full">
      <!-- 未激活状态 -->
      <template v-if="!activated">
        <div class="text-center mb-8" style="animation: fadeUp 0.8s ease-out both;">
          <div class="text-6xl mb-6" style="animation: giftFloat 3s ease-in-out infinite;">🎁</div>
          <h2 class="text-xl font-bold text-white mb-2">准备好接受传承了吗？</h2>
          <p class="text-gray-500 text-sm">这个座位的故事，将由你续写</p>
        </div>
        <button class="btn-gold text-lg px-12 py-3.5 shadow-2xl shadow-capsule-gold/20
                       transition-transform active:scale-95"
                @click="activateCeremony">
          ✨ 接受传承
        </button>
      </template>

      <!-- 激活后 -->
      <template v-else>
        <div class="text-center" style="animation: fadeUp 0.6s ease-out both;">
          <!-- 庆祝文字 -->
          <p class="text-capsule-gold text-2xl font-bold mb-6 celebration-text">🎉 传承完成！</p>

          <!-- 证书 -->
          <div ref="certificateRef"
               v-show="certVisible"
               class="certificate-card bg-gradient-to-b from-white/[0.06] to-white/[0.02]
                      rounded-2xl p-6 border border-capsule-gold/20 mb-6 text-center
                      shadow-2xl shadow-black/30">

            <p class="text-capsule-gold/60 text-[10px] tracking-[0.3em] uppercase mb-1">
              {{ config.certificate.subtitle }}
            </p>
            <h3 class="text-white text-lg font-bold mb-1">{{ config.certificate.title }}</h3>
            <div class="w-10 h-px bg-capsule-gold/40 mx-auto mb-4"></div>

            <div class="text-gray-300 text-sm space-y-1 mb-5">
              <p class="text-gray-500 text-xs">兹证明：</p>
              <p class="text-white font-bold text-base">{{ config.seat.campus }}</p>
              <p>{{ config.seat.building }} · {{ config.seat.dormitory }} · {{ config.seat.room }}</p>
              <p class="text-gray-400">{{ config.seat.position }}</p>
              <div class="w-10 h-px bg-capsule-gold/20 mx-auto my-3"></div>
              <p class="text-gray-500 text-xs">已由</p>
              <p class="text-capsule-gold font-bold text-base">{{ config.senior.name }}</p>
              <p class="text-gray-500 text-xs">传承至新一届学子</p>
            </div>

            <div class="flex justify-between items-center text-[10px] text-gray-600 mt-4 pt-4 border-t border-white/[0.06]">
              <span>编号：{{ config.certificate.number }}</span>
              <span class="font-signature text-capsule-gold text-sm">{{ config.senior.name }}</span>
            </div>
          </div>

          <!-- 按钮 -->
          <div class="flex flex-col gap-3 items-center">
            <button class="btn-gold text-sm px-6 py-2.5" @click="saveCertificate">
              💾 保存证书
            </button>
            <button class="text-gray-500 hover:text-white text-xs transition-colors"
                    @click="$emit('complete')">
              继续 → 留下你的寄语
            </button>
          </div>
        </div>
      </template>
    </div>
  </div>
</template>

<script setup>
import { ref, onMounted, onUnmounted } from 'vue'
import html2canvas from 'html2canvas'
import config from '../config.js'

defineEmits(['complete'])

const activated = ref(false)
const certVisible = ref(false)
const certificateRef = ref(null)
const particleCanvas = ref(null)

// 烟花粒子
let animFrame = null
let particles = []

function createFirework() {
  const canvas = particleCanvas.value
  if (!canvas) return
  const ctx = canvas.getContext('2d')
  const cx = Math.random() * canvas.width
  const cy = Math.random() * canvas.height * 0.35

  const colors = ['#ffd700', '#ff6b6b', '#69ffb4', '#00d4ff', '#ff69b4', '#fff']

  for (let i = 0; i < 40; i++) {
    const angle = (Math.PI * 2 / 40) * i + (Math.random() - 0.5) * 0.3
    const speed = Math.random() * 3.5 + 1.5
    particles.push({
      x: cx, y: cy,
      vx: Math.cos(angle) * speed,
      vy: Math.sin(angle) * speed,
      life: 1,
      decay: Math.random() * 0.015 + 0.008,
      color: colors[Math.floor(Math.random() * colors.length)],
      r: Math.random() * 2.5 + 1,
    })
  }
}

function animateParticles() {
  const canvas = particleCanvas.value
  if (!canvas) return
  const ctx = canvas.getContext('2d')
  ctx.clearRect(0, 0, canvas.width, canvas.height)

  particles = particles.filter(p => p.life > 0)
  particles.forEach(p => {
    p.x += p.vx
    p.y += p.vy
    p.vy += 0.035
    p.life -= p.decay
    ctx.beginPath()
    ctx.arc(p.x, p.y, p.r * p.life + 0.5, 0, Math.PI * 2)
    ctx.fillStyle = p.color
    ctx.globalAlpha = p.life
    ctx.fill()
  })
  ctx.globalAlpha = 1

  animFrame = requestAnimationFrame(animateParticles)
}

function activateCeremony() {
  activated.value = true
  if (navigator.vibrate) navigator.vibrate(100)

  // 连放烟花
  for (let i = 0; i < 6; i++) {
    setTimeout(() => createFirework(), i * 250)
  }
  animateParticles()

  setTimeout(() => { certVisible.value = true }, 500)
}

async function saveCertificate() {
  if (!certificateRef.value) return
  try {
    const canvas = await html2canvas(certificateRef.value, {
      backgroundColor: '#1a1a2e',
      scale: 2,
      useCORS: true,
    })
    const link = document.createElement('a')
    link.download = '座位传承证书.png'
    link.href = canvas.toDataURL()
    link.click()
  } catch (e) {
    console.error('保存证书失败:', e)
  }
}

onMounted(() => {
  const canvas = particleCanvas.value
  if (!canvas) return
  canvas.width = window.innerWidth
  canvas.height = window.innerHeight
})

onUnmounted(() => {
  if (animFrame) cancelAnimationFrame(animFrame)
})
</script>

<style scoped>
@keyframes fadeUp {
  0% { opacity: 0; transform: translateY(20px); }
  100% { opacity: 1; transform: translateY(0); }
}
@keyframes giftFloat {
  0%, 100% { transform: translateY(0); }
  50% { transform: translateY(-12px); }
}

.celebration-text {
  animation: celebratePop 0.5s ease-out both;
}
@keyframes celebratePop {
  0% { opacity: 0; transform: scale(0.5); }
  60% { transform: scale(1.15); }
  100% { opacity: 1; transform: scale(1); }
}

.certificate-card {
  animation: certAppear 0.8s ease-out both;
}
@keyframes certAppear {
  0% { opacity: 0; transform: scale(0.85) rotateY(15deg); }
  100% { opacity: 1; transform: scale(1) rotateY(0); }
}
</style>
