<template>
  <div class="message-scene fixed inset-0 flex flex-col overflow-hidden">
    <!-- 背景 -->
    <div class="absolute inset-0 bg-gradient-to-b from-[#1a1a2e] to-[#0a0a1a]"></div>

    <!-- 顶部 -->
    <div class="relative z-10 p-4 text-center shrink-0">
      <h2 class="text-xl font-bold text-white mb-1">💬 跨届寄语墙</h2>
      <p class="text-gray-500 text-xs">给学弟留下一句话，或者看看别人的留言</p>
    </div>

    <!-- Twikoo 评论区域 -->
    <div ref="twikooContainer" class="relative z-10 flex-1 overflow-y-auto scrollable px-4 pb-20">
      <div id="twikoo-comments" class="max-w-md mx-auto"></div>
    </div>

    <!-- 底部操作 -->
    <div class="relative z-10 p-4 border-t border-white/5 shrink-0 bg-black/20">
      <div class="flex items-center justify-center gap-4 max-w-md mx-auto">
        <!-- 时空信箱 -->
        <button class="flex-1 bg-white/5 hover:bg-white/10 rounded-xl py-3 text-center transition-all
                       active:scale-95"
                @click="scrollToComment">
          <div class="text-2xl mb-0.5">✉️</div>
          <div class="text-gray-400 text-xs">给学长写信</div>
        </button>
        <!-- 完成 -->
        <button class="flex-1 bg-capsule-gold/10 hover:bg-capsule-gold/20 rounded-xl py-3 text-center
                       transition-all active:scale-95"
                @click="$emit('complete')">
          <div class="text-2xl mb-0.5">🎓</div>
          <div class="text-capsule-gold text-xs">结束旅程</div>
        </button>
      </div>
    </div>
  </div>
</template>

<script setup>
import { ref, onMounted, onUnmounted } from 'vue'
import config from '../config.js'

const emit = defineEmits(['complete'])
const twikooContainer = ref(null)

let twikooInitialized = false
let twikooInstance = null

onMounted(async () => {
  if (!config.twikoo.envId || config.twikoo.envId.includes('your-twikoo-api')) {
    // 无配置时显示占位
    const placeholder = document.getElementById('twikoo-comments')
    if (placeholder) {
      placeholder.innerHTML = `
        <div class="text-center py-12">
          <div class="text-4xl mb-4">📬</div>
          <p class="text-gray-400 text-sm mb-2">寄语墙暂未开启</p>
          <p class="text-gray-600 text-xs">学长还没有配置留言系统</p>
        </div>
      `
    }
    return
  }

  try {
    const twikoo = await import('twikoo')
    const res = await twikoo.init({
      envId: config.twikoo.envId,
      el: '#twikoo-comments',
      path: config.twikoo.path,
      lang: config.twikoo.lang || 'zh-CN',
    })
    twikooInstance = res
    twikooInitialized = true

    // 自定义样式覆盖
    injectTwikooStyles()
  } catch (e) {
    console.error('Twikoo 初始化失败:', e)
    const placeholder = document.getElementById('twikoo-comments')
    if (placeholder) {
      placeholder.innerHTML = `
        <div class="text-center py-12">
          <p class="text-gray-500 text-sm">留言加载失败，请稍后再试</p>
        </div>
      `
    }
  }
})

function injectTwikooStyles() {
  const style = document.createElement('style')
  style.textContent = `
    #twikoo-comments {
      --twikoo-primary-color: #ffd700;
      --twikoo-primary-bg: rgba(255, 215, 0, 0.1);
      --twikoo-bg: transparent;
      --twikoo-bg-light: rgba(255, 255, 255, 0.03);
      --twikoo-text-color: #e0e0e0;
      --twikoo-text-secondary: #999;
      --twikoo-border-color: rgba(255, 255, 255, 0.08);
    }
    #twikoo-comments .twikoo {
      background: transparent !important;
    }
    #twikoo-comments .twikoo .el-input__inner,
    #twikoo-comments .twikoo .el-textarea__inner {
      background: rgba(255,255,255,0.05) !important;
      border-color: rgba(255,255,255,0.1) !important;
      color: #e0e0e0 !important;
    }
    #twikoo-comments .twikoo .el-button--primary {
      background: linear-gradient(135deg, #ffd700, #f0c000) !important;
      border-color: #ffd700 !important;
      color: #1a1a2e !important;
    }
    #twikoo-comments .twikoo .twikoo-comment {
      background: rgba(255,255,255,0.03) !important;
      border-radius: 12px !important;
      padding: 12px !important;
      margin-bottom: 8px !important;
      border: 1px solid rgba(255,255,255,0.05) !important;
    }
    #twikoo-comments .twikoo .twikoo-head,
    #twikoo-comments .twikoo .twikoo-footer {
      display: none !important;
    }
  `
  document.head.appendChild(style)
}

function scrollToComment() {
  const input = document.querySelector('#twikoo-comments .el-textarea__inner')
  if (input) {
    input.focus()
    input.scrollIntoView({ behavior: 'smooth' })
  }
  if (twikooContainer.value) {
    twikooContainer.value.scrollTo({
      top: twikooContainer.value.scrollHeight,
      behavior: 'smooth',
    })
  }
}

onUnmounted(() => {
  // 清理 Twikoo 注入的样式
})
</script>

<style scoped>
/* Twikoo 评论区样式覆盖 */
:deep(.twikoo) {
  color: #e0e0e0;
}
:deep(.twikoo .tk-comments-container) {
  padding: 0 !important;
}
</style>
