<template>
  <div class="app-root w-full h-full relative">
    <!-- 用 Transition 包裹实现场景淡入淡出 -->
    <div class="scene-container w-full h-full relative" :key="'scene-' + step">
      <!-- 步骤0: 加载页 -->
      <LoadingScreen v-if="step === 0" @complete="goTo(1)" />

      <!-- 步骤1: 时空胶囊 -->
      <CapsuleUnlock v-else-if="step === 1" @complete="goTo(2)" />

      <!-- 步骤2: 学长现身 -->
      <HostAppearance v-else-if="step === 2" @complete="goTo(3)" />

      <!-- 步骤3: 记忆碎片 -->
      <MemoryFragments v-else-if="step === 3" @complete="goTo(4)" />

      <!-- 步骤4: 传承仪式 -->
      <HandoverCeremony v-else-if="step === 4" @complete="goTo(5)" />

      <!-- 步骤5: 寄语墙 -->
      <MessageWall v-else-if="step === 5" @complete="goTo(6)" />

      <!-- 步骤6: 结束画面 -->
      <div v-else class="fixed inset-0 flex flex-col items-center justify-center bg-gradient-to-b from-[#1a1a2e] to-[#0a0a1a]">
        <div class="text-center px-6" style="animation: endFadeIn 1s ease-out both;">
          <div class="text-6xl mb-6" style="animation: endFloat 3s ease-in-out infinite;">🎓</div>
          <h2 class="text-2xl font-bold text-white mb-3">感谢你的到来</h2>
          <p class="text-gray-400 text-sm leading-relaxed mb-8 max-w-xs">
            愿你的大学生活同样精彩。<br/>
            这个座位的故事，将由你来续写。
          </p>
          <p class="text-capsule-gold text-xl font-signature">—— {{ config.senior.name }}</p>
        </div>
      </div>
    </div>

    <!-- 彩蛋系统始终存在 -->
    <EasterEgg />
  </div>
</template>

<script setup>
import { ref } from 'vue'
import config from './config.js'
import LoadingScreen from './components/LoadingScreen.vue'
import CapsuleUnlock from './components/CapsuleUnlock.vue'
import HostAppearance from './components/HostAppearance.vue'
import MemoryFragments from './components/MemoryFragments.vue'
import HandoverCeremony from './components/HandoverCeremony.vue'
import MessageWall from './components/MessageWall.vue'
import EasterEgg from './components/EasterEgg.vue'

const step = ref(0)

function goTo(s) {
  step.value = s
}
</script>

<style>
/* 全局场景淡入动画 */
.scene-container > div {
  animation: sceneFadeIn 0.8s ease-out both;
}
@keyframes sceneFadeIn {
  0% {
    opacity: 0;
    transform: scale(1.02);
  }
  100% {
    opacity: 1;
    transform: scale(1);
  }
}

@keyframes endFadeIn {
  0% { opacity: 0; transform: translateY(30px); }
  100% { opacity: 1; transform: translateY(0); }
}
@keyframes endFloat {
  0%, 100% { transform: translateY(0); }
  50% { transform: translateY(-15px); }
}
</style>
