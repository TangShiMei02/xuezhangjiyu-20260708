<template>
  <div class="memories-scene fixed inset-0 overflow-hidden">
    <!-- 背景 -->
    <div class="absolute inset-0 bg-gradient-to-b from-[#1a1a2e] via-[#1a1e2e] to-[#0a0a1a]"></div>

    <!-- 顶部导航 -->
    <div class="absolute top-0 left-0 right-0 z-20 px-4 pt-5 pb-3">
      <div class="flex items-center justify-between max-w-md mx-auto">
        <button class="text-gray-400 text-sm hover:text-white transition-colors"
                @click="prev"
                :disabled="currentIndex === 0"
                :class="{ 'opacity-30': currentIndex === 0 }">
          ← 返回
        </button>
        <div class="flex items-center gap-1.5">
          <span v-for="(_, i) in cards" :key="i"
                class="w-1.5 h-1.5 rounded-full transition-all duration-500"
                :class="i === currentIndex ? 'bg-capsule-gold w-5' : 'bg-gray-600'">
          </span>
        </div>
        <span class="text-gray-500 text-xs">{{ currentIndex + 1 }}/{{ cards.length }}</span>
      </div>
    </div>

    <!-- 卡片主区域 -->
    <div class="absolute inset-0 flex items-center justify-center px-5"
         @touchstart="onTouchStart"
         @touchend="onTouchEnd"
         @mousedown="onTouchStart"
         @mouseup="onTouchEnd">

      <div class="w-full max-w-sm">
        <div class="card-wrapper" :key="currentIndex">
          <div class="bg-white/[0.04] backdrop-blur-2xl rounded-3xl p-7 border border-white/[0.06]
                      shadow-2xl shadow-black/30 min-h-[380px] flex flex-col">
            <!-- 图标 + 标题 -->
            <div class="flex items-center gap-3 mb-5">
              <span class="text-3xl">{{ cards[currentIndex].icon }}</span>
              <h3 class="text-lg font-bold text-white">{{ cards[currentIndex].title }}</h3>
            </div>

            <!-- 分隔线 -->
            <div class="w-full h-px bg-gradient-to-r from-capsule-gold/20 via-capsule-gold/10 to-transparent mb-5"></div>

            <!-- 内容 -->
            <div class="flex-1 text-gray-300 text-sm leading-relaxed card-content">
              <component :is="cards[currentIndex].component" />
            </div>
          </div>
        </div>

        <!-- 底部按钮 -->
        <div class="flex justify-center mt-5">
          <button v-if="currentIndex < cards.length - 1"
                  class="bg-white/5 hover:bg-white/10 text-white px-8 py-2.5 rounded-full text-sm
                         transition-all active:scale-95 border border-white/10"
                  @click="next">
            下一张 →
          </button>
          <button v-else
                  class="btn-gold text-sm px-8 py-2.5 shadow-2xl shadow-capsule-gold/20"
                  @click="$emit('complete')">
            继续 →
          </button>
        </div>
      </div>
    </div>
  </div>
</template>

<script setup>
import { ref, h, onMounted } from 'vue'
import config from '../config.js'

defineEmits(['complete'])
const currentIndex = ref(0)

// 触摸滑动
let touchX = 0, touchY = 0
function onTouchStart(e) {
  const p = e.touches ? e.touches[0] : e
  touchX = p.clientX; touchY = p.clientY
}
function onTouchEnd(e) {
  const p = e.changedTouches ? e.changedTouches[0] : e
  const dx = p.clientX - touchX
  const dy = p.clientY - touchY
  if (Math.abs(dx) > 50 && Math.abs(dx) > Math.abs(dy) * 1.5) {
    dx < 0 ? next() : prev()
  }
}

function next() {
  if (currentIndex.value < cards.length - 1) currentIndex.value++
}
function prev() {
  if (currentIndex.value > 0) currentIndex.value--
}

// ===== 构建卡片 =====
const hasMusic = config.memories.musicId && config.memories.musicId.length > 0
const hasPhotos = config.memories.photos && config.memories.photos.length > 0

const cards = ref([])

function buildCards() {
  const list = []

  // 座位档案
  list.push({
    icon: '📍',
    title: '座位坐标',
    component: () => h('div', { class: 'space-y-3' }, [
      h('div', { class: 'bg-white/[0.03] rounded-xl p-4 border border-white/[0.04]' }, [
        h('p', { class: 'text-gray-200' }, [
          h('span', { class: 'text-capsule-gold' }, config.seat.campus),
          h('br'),
          `${config.seat.building} · ${config.seat.dormitory} · ${config.seat.room}`,
          h('br'),
          config.seat.position,
        ]),
      ]),
      h('p', { class: 'text-gray-600 text-xs text-center mt-3' }, `📅 ${config.seat.period}`),
    ]),
  })

  // 桌面印记
  list.push({
    icon: '🖊️',
    title: '桌面上的小秘密',
    component: () => h('div', { class: 'space-y-3' }, [
      h('div', { class: 'bg-white/[0.03] rounded-xl p-4 border border-white/[0.04] italic text-gray-400 border-l-2 border-l-capsule-gold/30' },
        `"${config.memories.deskMark}"`
      ),
      h('p', { class: 'text-gray-600 text-xs' }, '不知道还在不在，你去看看吧 😄'),
    ]),
  })

  // 音乐记忆
  if (hasMusic) {
    list.push({
      icon: '🎵',
      title: '这个位置最常播放的歌',
      component: () => h('div', { class: 'space-y-3' }, [
        h('div', { class: 'bg-white/[0.03] rounded-xl overflow-hidden' }, [
          h('iframe', {
            src: `//music.163.com/outchain/player?type=2&id=${config.memories.musicId}&auto=0&height=66`,
            style: 'width: 100%; height: 66px; border: none;',
            loading: 'lazy',
          }),
        ]),
        h('p', { class: 'text-gray-600 text-xs' }, '戴上耳机，感受一下学长的品位 🎧'),
      ]),
    })
  }

  // 照片墙
  if (hasPhotos) {
    list.push({
      icon: '📸',
      title: '在这个位置的瞬间',
      component: () => h('div', { class: 'grid gap-2' },
        config.memories.photos.map(src =>
          h('div', { class: 'bg-white/[0.03] rounded-xl overflow-hidden aspect-video flex items-center justify-center' }, [
            h('img', {
              src: `/assets/photos/${src}`,
              class: 'w-full h-full object-cover',
              alt: '照片',
              loading: 'lazy',
            }),
          ])
        )
      ),
    })
  }

  // 生存指南
  list.push({
    icon: '💡',
    title: '学长の生存指南',
    component: () => h('div', { class: 'space-y-2' },
      config.memories.tips.map((tip, i) =>
        h('div', {
          class: `bg-white/[0.03] rounded-xl px-4 py-2.5 flex items-start gap-3 border border-white/[0.04]`
        }, [
          h('span', { class: 'text-capsule-gold shrink-0 font-bold' }, `${i + 1}.`),
          h('span', { class: 'text-gray-300' }, tip),
        ])
      )
    ),
  })

  cards.value = list
}

onMounted(buildCards)
</script>

<style scoped>
.card-wrapper {
  animation: cardIn 0.45s ease-out;
}
@keyframes cardIn {
  0% { opacity: 0; transform: translateX(30px) scale(0.97); }
  100% { opacity: 1; transform: translateX(0) scale(1); }
}
.card-content {
  animation: contentFade 0.35s 0.15s ease-out both;
}
@keyframes contentFade {
  0% { opacity: 0; }
  100% { opacity: 1; }
}
</style>
