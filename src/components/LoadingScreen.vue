<template>
  <div class="loading-screen fixed inset-0 z-50 flex flex-col items-center justify-center bg-[#0a0a1a]">
    <!-- 旋转圆环 -->
    <div class="relative w-24 h-24" v-if="!burstStarted">
      <svg class="w-full h-full rotate-svg" viewBox="0 0 100 100">
        <circle cx="50" cy="50" r="42"
                fill="none" stroke="rgba(255,215,0,0.15)"
                stroke-width="3" />
        <circle cx="50" cy="50" r="42"
                fill="none" stroke="#ffd700"
                stroke-width="3"
                stroke-linecap="round"
                :stroke-dasharray="circumference"
                :stroke-dashoffset="dashOffset"
                class="transition-all duration-300" />
      </svg>
      <div class="absolute inset-0 flex items-center justify-center">
        <span class="text-capsule-gold text-sm font-bold">{{ progress }}%</span>
      </div>
    </div>

    <!-- 加载文字 -->
    <p class="text-gray-400 text-sm mt-6 tracking-[0.3em] loading-text" v-if="!burstStarted">
      {{ loadingText }}
    </p>

    <!-- 光芒爆发 -->
    <div v-if="burstStarted" class="absolute inset-0 flex items-center justify-center pointer-events-none">
      <div class="w-4 h-4 bg-capsule-gold rounded-full light-burst"></div>
    </div>
  </div>
</template>

<script setup>
import { ref, computed, onMounted, onUnmounted } from 'vue'

const emit = defineEmits(['complete'])

const progress = ref(0)
const burstStarted = ref(false)
const loadingText = ref('正在连接时空胶囊...')
const circumference = 2 * Math.PI * 42
const dashOffset = ref(circumference)

const loadingTexts = [
  '正在连接时空胶囊...',
  '正在定位时空坐标...',
  '正在加载学长信息...',
  '正在准备寄语...',
  '传输通道已建立 ✓',
]

let timer = null
let burstTimer = null
let completeTimer = null

onMounted(() => {
  timer = setInterval(() => {
    if (progress.value < 100) {
      progress.value += Math.floor(Math.random() * 5) + 1
      if (progress.value > 100) progress.value = 100
      dashOffset.value = circumference - (progress.value / 100) * circumference
      const idx = Math.min(
        Math.floor(progress.value / 25),
        loadingTexts.length - 1
      )
      loadingText.value = loadingTexts[idx]
    } else {
      clearInterval(timer)
      burstTimer = setTimeout(() => {
        burstStarted.value = true
        // 光爆动画 1s 后用 timeout 触发完成（比 animationend 更可靠）
        completeTimer = setTimeout(() => {
          emit('complete')
        }, 1200)
      }, 300)
    }
  }, 300)
})

onUnmounted(() => {
  if (timer) clearInterval(timer)
  if (burstTimer) clearTimeout(burstTimer)
  if (completeTimer) clearTimeout(completeTimer)
})
</script>

<style scoped>
.loading-screen {
  transition: opacity 0.5s ease;
}
.rotate-svg {
  animation: spin 2s linear infinite;
}
@keyframes spin {
  100% { transform: rotate(360deg); }
}
.loading-text {
  animation: textPulse 1.5s ease-in-out infinite;
}
@keyframes textPulse {
  0%, 100% { opacity: 0.5; }
  50% { opacity: 1; }
}
</style>
