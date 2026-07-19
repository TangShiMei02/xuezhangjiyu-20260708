<template>
  <!-- 彩蛋容器 - 悬浮在所有页面之上 -->
  <div>
    <!-- 时间彩蛋 - 底部提示 -->
    <transition name="egg-toast">
      <div v-if="timeEggVisible"
           class="fixed bottom-24 left-1/2 -translate-x-1/2 z-50
                  bg-black/80 backdrop-blur-md text-white px-6 py-3 rounded-2xl
                  text-sm text-center max-w-xs border border-capsule-gold/20"
           @click="timeEggVisible = false">
        {{ timeEggMessage }}
      </div>
    </transition>

    <!-- 隐藏基地 - 全屏弹窗 -->
    <transition name="egg-modal">
      <div v-if="secretBaseVisible"
           class="fixed inset-0 z-50 flex items-center justify-center bg-black/80 backdrop-blur-md"
           @click.self="secretBaseVisible = false">
        <div class="bg-[#1a1a2e] rounded-3xl p-6 max-w-sm mx-4 border border-capsule-gold/20
                    max-h-[70vh] overflow-y-auto">
          <div class="flex items-center justify-between mb-4">
            <h3 class="text-capsule-gold font-bold text-lg">🤫 学长の秘密基地</h3>
            <button class="text-gray-500 hover:text-white text-xl" @click="secretBaseVisible = false">✕</button>
          </div>
          <div class="space-y-3">
            <div v-for="(item, i) in eggConfig.secretBase" :key="i"
                 class="bg-white/5 rounded-xl p-4">
              <template v-if="item.type === 'link'">
                <a :href="item.url" target="_blank"
                   class="text-capsule-gold hover:text-yellow-300 transition-colors block">
                  🔗 {{ item.title }}
                </a>
              </template>
              <template v-else>
                <p class="text-gray-300 text-sm">{{ item.content }}</p>
              </template>
            </div>
          </div>
        </div>
      </div>
    </transition>

    <!-- 摇一摇触发提示 -->
    <transition name="egg-toast">
      <div v-if="shakeMessageVisible"
           class="fixed top-24 left-1/2 -translate-x-1/2 z-50
                  bg-black/80 backdrop-blur-md text-white px-6 py-3 rounded-2xl
                  text-sm text-center max-w-xs border border-white/10">
        {{ shakeMessage }}
      </div>
    </transition>
  </div>
</template>

<script setup>
import { ref, onMounted, onUnmounted } from 'vue'
import config from '../config.js'

const eggConfig = config.easterEggs

// ===== 时间彩蛋 =====
const timeEggVisible = ref(false)
const timeEggMessage = ref('')

function checkTimeEgg() {
  const now = new Date()
  const hour = String(now.getHours()).padStart(2, '0')

  // 考试周彩蛋优先
  if (eggConfig.examWeek && eggConfig.examWeek.start) {
    const start = new Date(eggConfig.examWeek.start)
    const end = new Date(eggConfig.examWeek.end)
    if (now >= start && now <= end) {
      timeEggMessage.value = '📚 考试周加油！相信自己，你可以的！💪'
      timeEggVisible.value = true
      return
    }
  }

  // 时段彩蛋
  const key = Object.keys(eggConfig.nightMessages).find(k => {
    return hour.startsWith(k) || hour === k
  })
  if (key) {
    timeEggMessage.value = eggConfig.nightMessages[key]
    timeEggVisible.value = true
  }
}

// ===== 隐藏基地（连续点击） =====
const secretBaseVisible = ref(false)
let clickCount = 0
let clickTimer = null

function handleGlobalClick() {
  clickCount++
  if (clickCount >= 5) {
    clickCount = 0
    secretBaseVisible.value = true
    if (navigator.vibrate) navigator.vibrate(50)
  }
  clearTimeout(clickTimer)
  clickTimer = setTimeout(() => {
    clickCount = 0
  }, 2000)
}

// ===== 摇一摇 =====
const shakeMessageVisible = ref(false)
const shakeMessage = ref('')

function initShake() {
  if (!window.DeviceMotionEvent) return

  let lastX = 0, lastY = 0, lastZ = 0
  let lastShakeTime = 0

  window.addEventListener('devicemotion', (e) => {
    const acc = e.accelerationIncludingGravity
    if (!acc) return

    const x = acc.x, y = acc.y, z = acc.z
    if (lastX === 0 && lastY === 0 && lastZ === 0) {
      lastX = x; lastY = y; lastZ = z
      return
    }

    const delta = Math.abs(x - lastX) + Math.abs(y - lastY) + Math.abs(z - lastZ)
    const now = Date.now()

    if (delta > 25 && now - lastShakeTime > 5000) {
      lastShakeTime = now
      const soups = eggConfig.chickenSoup
      shakeMessage.value = soups[Math.floor(Math.random() * soups.length)]
      shakeMessageVisible.value = true
      if (navigator.vibrate) navigator.vibrate(50)
      setTimeout(() => { shakeMessageVisible.value = false }, 3000)
    }

    lastX = x; lastY = y; lastZ = z
  })
}

onMounted(() => {
  checkTimeEgg()
  document.addEventListener('click', handleGlobalClick)
  initShake()
})

onUnmounted(() => {
  document.removeEventListener('click', handleGlobalClick)
  if (clickTimer) clearTimeout(clickTimer)
})
</script>

<style scoped>
.egg-toast-enter-active {
  animation: toastIn 0.4s ease-out;
}
.egg-toast-leave-active {
  animation: toastOut 0.3s ease-in;
}
@keyframes toastIn {
  0% { opacity: 0; transform: translateY(20px) translateX(-50%); }
  100% { opacity: 1; transform: translateY(0) translateX(-50%); }
}
@keyframes toastOut {
  0% { opacity: 1; transform: translateY(0) translateX(-50%); }
  100% { opacity: 0; transform: translateY(-20px) translateX(-50%); }
}
.egg-modal-enter-active {
  animation: modalIn 0.3s ease-out;
}
.egg-modal-leave-active {
  animation: modalOut 0.2s ease-in;
}
@keyframes modalIn {
  0% { opacity: 0; }
  100% { opacity: 1; }
}
@keyframes modalOut {
  0% { opacity: 1; }
  100% { opacity: 0; }
}
</style>
