<template>
  <div v-if="isHome" class="min-h-screen bg-[#050a10] text-white font-sans relative selection:bg-[#ff6b6b]/30">
    <!-- StarryBackground -->
    <div class="fixed inset-0 z-0 overflow-hidden bg-[#050a10] pointer-events-none">
      <div v-for="star in stars" :key="star.id" class="absolute rounded-full bg-white" :style="{ left: `${star.x}%`, top: `${star.y}%`, width: `${star.size}px`, height: `${star.size}px`, opacity: star.opacity }" />
      <div class="absolute inset-0 bg-[radial-gradient(circle_at_50%_20%,rgba(255,80,80,0.15),transparent_60%)]" />
    </div>
    
    <div class="relative z-10 flex flex-col items-center pt-24 pb-16 px-4 sm:px-6 lg:px-8 max-w-7xl mx-auto">
      
      <!-- Hero Section -->
      <div class="flex flex-col items-center text-center max-w-3xl fade-in-init">
        <!-- Mascot / Logo -->
        <div 
          @mouseenter="isHovered = true"
          @mouseleave="isHovered = false"
          @click="triggerHappyJump"
          :class="['mb-6 drop-shadow-[0_0_30px_rgba(255,77,77,0.35)] cursor-pointer lobster-anim', { 'hovered': isHovered, 'happy': isHappy }]"
        >
          <svg viewBox="0 0 120 120" fill="none" xmlns="http://www.w3.org/2000/svg" class="w-32 h-32 md:w-40 md:h-40">
            <!-- Lobster Claw Silhouette -->
            <path d="M60 10 C30 10 15 35 15 55 C15 75 30 95 45 100 L45 110 L55 110 L55 100 C55 100 60 102 65 100 L65 110 L75 110 L75 100 C90 95 105 75 105 55 C105 35 90 10 60 10Z" fill="url(#lobster-gradient)" class="claw-body"></path>
            <!-- Left Claw -->
            <path d="M20 45 C5 40 0 50 5 60 C10 70 20 65 25 55 C28 48 25 45 20 45Z" fill="url(#lobster-gradient)" :class="['claw-left', { waving: isHovered }]"></path>
            <!-- Right Claw -->
            <path d="M100 45 C115 40 120 50 115 60 C110 70 100 65 95 55 C92 48 95 45 100 45Z" fill="url(#lobster-gradient)" :class="['claw-right', { waving: isHovered }]"></path>
            <!-- Antenna -->
            <path d="M45 15 Q35 5 30 8" stroke="#d84d4d" stroke-width="2" stroke-linecap="round" class="antenna"></path>
            <path d="M75 15 Q85 5 90 8" stroke="#d84d4d" stroke-width="2" stroke-linecap="round" class="antenna"></path>
            <!-- Eyes -->
            <circle cx="45" cy="35" r="6" fill="#050810" class="eye"></circle>
            <circle cx="75" cy="35" r="6" fill="#050810" class="eye"></circle>
            <circle :cx="46 + eyeOffsetX" :cy="34 + eyeOffsetY" r="2" fill="#00e5cc" class="eye-glow"></circle>
            <circle :cx="76 + eyeOffsetX" :cy="34 + eyeOffsetY" r="2" fill="#00e5cc" class="eye-glow"></circle>
            <path d="M52 45 Q60 52 68 45" stroke="#ffd3d3" stroke-width="2" stroke-linecap="round" :class="['lobster-smile', { visible: isHappy }]"></path>
            <defs>
              <linearGradient id="lobster-gradient" x1="0%" y1="0%" x2="100%" y2="100%">
                <stop offset="0%" stop-color="#ff8c8c"></stop>
                <stop offset="100%" stop-color="#ff4d4d"></stop>
              </linearGradient>
            </defs>
          </svg>
        </div>

        <h1 class="!text-5xl md:!text-7xl !leading-none font-bold !tracking-[0.25em] !mt-0 !mb-4 !pl-[0.25em]">
          <span class="text-[#ff4d4d]">
            HelloClaw
          </span>
        </h1>
        
        <h2 class="!text-6xl md:!text-8xl !leading-none font-bold !tracking-widest uppercase !mt-0 !mb-8 !border-0 !pt-0">
          <span class="animate-gradient-text">
            哈喽！龙虾
          </span> 👋
        </h2>

        <p class="!text-lg md:!text-xl text-gray-400 !mt-0 !mb-10 max-w-2xl !leading-relaxed">
          你的 OpenClaw 终极助手。清理收件箱、发送邮件、管理日历、办理登机牌。一切都在你常用的聊天软件中完成。
        </p>

        <div class="flex flex-col sm:flex-row items-center gap-4 mb-12">
          <a
            :href="withBase('/cn/adopt/chapter0')"
            class="w-full sm:w-auto group flex items-center justify-center gap-2 bg-[#ff4d4d] hover:bg-[#ff6b6b] text-white font-bold rounded-full py-3 px-7 transition-all duration-300 hover:scale-105 active:scale-95"
          >
            领养龙虾 <span class="opacity-80 group-hover:opacity-100 transition-opacity">→</span>
          </a>
          <a
            :href="withBase('/cn/build/')"
            class="w-full sm:w-auto group flex items-center justify-center gap-2 bg-white/5 hover:bg-white/10 border border-white/10 text-white font-bold rounded-full py-3 px-7 transition-all duration-300 hover:scale-105 active:scale-95"
          >
            构建龙虾 <span class="text-[#ff4d4d] group-hover:text-[#ff6b6b] transition-colors">→</span>
          </a>
        </div>

        <a :href="withBase('/cn/build/chapter7')" class="group flex items-center gap-3 bg-white/5 hover:bg-white/10 border border-white/10 rounded-full py-2 px-4 md:px-6 transition-all duration-300 mb-20 hover:scale-105 active:scale-95 cursor-pointer">
          <span class="bg-[#ff4d4d] text-white text-xs font-bold px-2.5 py-1 rounded-full uppercase tracking-wider">
            New
          </span>
          <span class="text-sm md:text-base text-gray-300 group-hover:text-white transition-colors">
            构建篇第 7-9 章上线：轻量化 / 安全加固 / 硬件方案
          </span>
          <span class="text-gray-500 group-hover:text-white transition-colors">→</span>
        </a>
      </div>

      <div v-fade-in class="w-full max-w-5xl">
        <div class="flex items-center justify-between mb-8">
          <h3 class="!text-3xl md:!text-4xl !font-extrabold !leading-tight flex items-center gap-2">
            <span class="text-[#ff4d4d]">&gt;</span> 推荐章节
          </h3>
          <a :href="withBase('/cn/adopt/chapter0')" class="text-[#ff4d4d] hover:text-[#ff6b6b] text-sm font-medium flex items-center gap-1 transition-colors">
            从第 0 章开始 <span>→</span>
          </a>
        </div>

        <div class="relative overflow-hidden rounded-2xl border border-white/5 bg-[#0f1623] shadow-xl">
          <div class="marquee w-max flex items-stretch gap-6 py-6 px-6">
            <a
              v-for="(item, idx) in tickerItems"
              :key="`${item.title}-${idx}`"
              :href="withBase(item.link)"
              class="block w-[280px] md:w-[360px] shrink-0 whitespace-normal bg-[#0a0f18] border border-white/5 rounded-2xl p-6 shadow-lg hover:-translate-y-1 transition-transform"
            >
              <div class="flex items-center justify-between mb-3">
                <span class="text-xs uppercase tracking-wider text-gray-500">{{ item.track }}</span>
                <span class="text-[#ff4d4d] text-xs font-bold">→</span>
              </div>
              <h4 class="!text-xl !font-extrabold !leading-snug !m-0 text-white">{{ item.title }}</h4>
              <p class="text-gray-400 text-sm mt-3 mb-0 leading-relaxed">{{ item.summary }}</p>
            </a>
          </div>
        </div>
      </div>

      <!-- Quick Start Section -->
      <div v-fade-in class="w-full max-w-5xl mt-32">
        <h3 class="!text-3xl md:!text-4xl !font-extrabold !leading-tight flex items-center gap-2 mb-8">
          <span class="text-[#ff4d4d]">&gt;</span> Quick Start
        </h3>
        
        <div class="bg-[#0f1623] border border-white/5 rounded-2xl overflow-hidden shadow-xl">
          <div class="flex items-center justify-between px-4 py-3 border-b border-white/5 bg-[#0a0f18]">
            <div class="flex items-center gap-4">
              <div class="flex gap-1.5">
                <div class="w-3 h-3 rounded-full bg-[#ff5f56]" />
                <div class="w-3 h-3 rounded-full bg-[#ffbd2e]" />
                <div class="w-3 h-3 rounded-full bg-[#27c93f]" />
              </div>
              <div class="flex gap-2">
                <button class="bg-[#00e5ff] text-black text-xs font-bold px-3 py-1 rounded">One-liner</button>
                <button class="text-gray-400 hover:text-white text-xs px-3 py-1 transition-colors">npm</button>
                <button class="text-gray-400 hover:text-white text-xs px-3 py-1 transition-colors">Hackable</button>
                <button class="text-gray-400 hover:text-white text-xs px-3 py-1 transition-colors">macOS</button>
              </div>
            </div>
            <div class="flex items-center gap-4">
              <span class="text-gray-400 text-xs">macOS/Linux</span>
              <button class="text-[#ff4d4d] text-xs hover:text-[#ff6b6b] transition-colors">change</button>
              <span class="border border-white/10 text-gray-400 text-xs px-2 py-0.5 rounded flex items-center gap-1">
                β BETA
              </span>
            </div>
          </div>
          
          <div class="p-6 font-mono text-sm">
            <p class="text-gray-500 mb-4"># Works everywhere. Installs everything. You're welcome. 🦞</p>
            <div class="flex items-center justify-between group">
              <div class="flex items-center gap-3">
                <span class="text-[#ff4d4d]">$</span>
                <span class="text-gray-300">curl -fsSL https://openclaw.ai/install.sh | bash</span>
              </div>
              <button @click="copyCommand" class="p-2 rounded bg-white/5 text-gray-400 hover:text-white opacity-0 group-hover:opacity-100 transition-all">
                <span v-if="copied">✓</span>
                <span v-else>📋</span>
              </button>
            </div>
          </div>
        </div>
        <p class="text-center text-gray-500 text-sm mt-4">
          Works on macOS, Windows & Linux. The one-liner installs Node.js and everything else for you.
        </p>
      </div>

      <!-- What It Does Section -->
      <div v-fade-in class="w-full max-w-5xl mt-32">
        <h3 class="!text-3xl md:!text-4xl !font-extrabold !leading-tight flex items-center gap-2 mb-8">
          <span class="text-[#ff4d4d]">&gt;</span> What It Does
        </h3>
        
        <div class="grid grid-cols-1 md:grid-cols-3 gap-6">
          <div class="bg-[#0f1623] border border-white/5 rounded-2xl p-8 flex flex-col items-center text-center gap-4 shadow-xl hover:-translate-y-1 transition-transform">
            <div class="text-[#ff4d4d] mb-2 text-3xl">🏠</div>
            <h4 class="!text-2xl !font-extrabold text-white">Runs on Your Machine</h4>
            <p class="text-gray-400 !text-base !leading-relaxed">Mac, Windows, or Linux. Anthropic, OpenAI, or local models. Private by default—your data stays yours.</p>
          </div>
          <div class="bg-[#0f1623] border border-white/5 rounded-2xl p-8 flex flex-col items-center text-center gap-4 shadow-xl hover:-translate-y-1 transition-transform">
            <div class="text-[#ff4d4d] mb-2 text-3xl">💬</div>
            <h4 class="!text-2xl !font-extrabold text-white">Any Chat App</h4>
            <p class="text-gray-400 !text-base !leading-relaxed">Talk to it on WhatsApp, Telegram, Discord, Slack, Signal, or iMessage. Works in DMs and group chats.</p>
          </div>
          <div class="bg-[#0f1623] border border-white/5 rounded-2xl p-8 flex flex-col items-center text-center gap-4 shadow-xl hover:-translate-y-1 transition-transform">
            <div class="text-[#ff4d4d] mb-2 text-3xl">🧠</div>
            <h4 class="!text-2xl !font-extrabold text-white">Persistent Memory</h4>
            <p class="text-gray-400 !text-base !leading-relaxed">Remembers you and becomes uniquely yours. Your preferences, your context, your AI.</p>
          </div>
          <div class="bg-[#0f1623] border border-white/5 rounded-2xl p-8 flex flex-col items-center text-center gap-4 shadow-xl hover:-translate-y-1 transition-transform">
            <div class="text-[#ff4d4d] mb-2 text-3xl">🌐</div>
            <h4 class="!text-2xl !font-extrabold text-white">Web Access</h4>
            <p class="text-gray-400 !text-base !leading-relaxed">Browses the web, reads articles, and summarizes content for you instantly.</p>
          </div>
          <div class="bg-[#0f1623] border border-white/5 rounded-2xl p-8 flex flex-col items-center text-center gap-4 shadow-xl hover:-translate-y-1 transition-transform">
            <div class="text-[#ff4d4d] mb-2 text-3xl">⌨️</div>
            <h4 class="!text-2xl !font-extrabold text-white">Command Line</h4>
            <p class="text-gray-400 !text-base !leading-relaxed">Execute scripts, manage files, and control your system through natural language.</p>
          </div>
          <div class="bg-[#0f1623] border border-white/5 rounded-2xl p-8 flex flex-col items-center text-center gap-4 shadow-xl hover:-translate-y-1 transition-transform">
            <div class="text-[#ff4d4d] mb-2 text-3xl">🧩</div>
            <h4 class="!text-2xl !font-extrabold text-white">Extensible</h4>
            <p class="text-gray-400 !text-base !leading-relaxed">Build your own skills and integrations. The possibilities are endless.</p>
          </div>
        </div>
      </div>

      <!-- Works With Everything Section -->
      <div v-fade-in class="w-full max-w-5xl mt-32 flex flex-col items-center">
        <div class="w-full">
          <h3 class="!text-3xl md:!text-4xl !font-extrabold !leading-tight flex items-center gap-2 mb-8">
            <span class="text-[#ff4d4d]">&gt;</span> Works With Everything
          </h3>
        </div>
        
        <div class="flex flex-wrap justify-center gap-4 max-w-4xl mb-8">
          <div v-for="item in integrations" :key="item.name" class="flex items-center gap-2 bg-[#0f1623] border border-white/5 rounded-full px-4 py-2 hover:bg-white/5 transition-colors cursor-pointer">
            <span class="text-lg">{{ item.icon }}</span>
            <span class="text-sm text-gray-300">{{ item.name }}</span>
          </div>
        </div>
        
        <div class="flex items-center gap-6 text-[#ff4d4d] text-sm font-medium">
          <a href="#" class="hover:text-[#ff6b6b] transition-colors flex items-center gap-1">
            View all 50+ integrations <span>→</span>
          </a>
          <span class="text-gray-600">•</span>
          <a href="#" class="hover:text-[#ff6b6b] transition-colors flex items-center gap-1">
            See what people built <span>→</span>
          </a>
        </div>
      </div>

      <!-- Sponsors Section -->
      <div v-fade-in class="w-full max-w-5xl mt-32">
        <h3 class="!text-3xl md:!text-4xl !font-extrabold !leading-tight flex items-center gap-2 mb-8">
          <span class="text-[#ff4d4d]">&gt;</span> Sponsors
        </h3>
        
        <div class="flex flex-wrap justify-center gap-6 mb-12">
          <div class="bg-[#0f1623] border border-white/5 rounded-xl w-48 h-20 flex items-center justify-center hover:bg-white/5 transition-colors cursor-pointer">
            <span class="text-2xl font-bold text-white">OpenAI</span>
          </div>
          <div class="bg-[#0f1623] border border-white/5 rounded-xl w-48 h-20 flex items-center justify-center hover:bg-white/5 transition-colors cursor-pointer">
            <span class="text-2xl font-bold text-white flex items-center gap-2">
              <span class="text-3xl">▲</span> Vercel
            </span>
          </div>
          <div class="bg-[#0f1623] border border-white/5 rounded-xl w-48 h-20 flex items-center justify-center hover:bg-white/5 transition-colors cursor-pointer">
            <span class="text-sm text-gray-400 mr-2">CI powered by</span>
            <span class="text-xl font-bold text-white tracking-widest">BLACK<br/>SMITH</span>
          </div>
          <div class="bg-[#0f1623] border border-white/5 rounded-xl w-48 h-20 flex items-center justify-center hover:bg-white/5 transition-colors cursor-pointer">
            <span class="text-3xl font-black text-white tracking-tighter">convex</span>
          </div>
        </div>
      </div>

      <!-- Footer -->
      <footer class="mt-32 pb-16 text-center w-full max-w-3xl">
        <div class="flex justify-center gap-6 text-[#ff4d4d] text-sm mb-8">
          <a href="#" class="hover:text-[#ff6b6b] transition-colors">Blog</a>
          <span class="text-gray-600">•</span>
          <a href="#" class="hover:text-[#ff6b6b] transition-colors">Showcase</a>
          <span class="text-gray-600">•</span>
          <a href="#" class="hover:text-[#ff6b6b] transition-colors">Shoutouts</a>
          <span class="text-gray-600">•</span>
          <a href="#" class="hover:text-[#ff6b6b] transition-colors">Integrations</a>
          <span class="text-gray-600">•</span>
          <a href="#" class="hover:text-[#ff6b6b] transition-colors">Trust</a>
        </div>
        
        <p class="text-gray-400 text-sm mb-4">
          Built by <span class="text-[#ff4d4d]">Molty 🦞</span>, a space lobster AI with a <span class="text-[#ff4d4d]">soul</span>, by <span class="text-[#ff4d4d]">Peter Steinberger</span> & <span class="text-[#ff4d4d]">community</span>.
        </p>
        
        <p class="text-gray-600 text-xs">
          Formerly known as Clawdbot and Moltbot. Independent project, not affiliated with Anthropic.
        </p>
      </footer>
    </div>
  </div>
  <Layout v-else />
</template>

<script setup>
import { ref, computed, onMounted, onUnmounted } from 'vue'
import { useRoute, withBase } from 'vitepress'
import DefaultTheme from 'vitepress/theme'

const { Layout } = DefaultTheme
const route = useRoute()
const isHome = computed(() => {
  const path = route.path
  return path === '/' || path === '/index.html' || 
         path === '/hello-claw/' || path === '/hello-claw/index.html'
})

const isHovered = ref(false)
const isHappy = ref(false)
const copied = ref(false)
const eyeOffsetX = ref(0)
const eyeOffsetY = ref(0)

const stars = Array.from({ length: 100 }).map((_, i) => ({
  id: i,
  x: Math.random() * 100,
  y: Math.random() * 100,
  size: Math.random() * 2 + 0.5,
  opacity: Math.random() * 0.5 + 0.1,
}))

const integrations = [
  { icon: '💬', name: 'WhatsApp' },
  { icon: '✈️', name: 'Telegram' },
  { icon: '👾', name: 'Discord' },
  { icon: '#️⃣', name: 'Slack' },
  { icon: '🔵', name: 'Signal' },
  { icon: '💬', name: 'iMessage' },
  { icon: '🤖', name: 'Claude' },
  { icon: '🟩', name: 'GPT' },
  { icon: '🎵', name: 'Spotify' },
  { icon: '💡', name: 'Hue' },
  { icon: '💎', name: 'Obsidian' },
  { icon: '𝕏', name: 'Twitter' },
  { icon: '🌐', name: 'Browser' },
  { icon: '📧', name: 'Gmail' },
  { icon: '🐙', name: 'GitHub' }
]

const baseRecommendedSections = [
  { track: '领养龙虾', title: '第 1 章：十分钟上手 OpenClaw', summary: '一键安装、配置模型、常用命令与 Coding Plan 模式。', link: '/cn/adopt/chapter1' },
  { track: '领养龙虾', title: '第 3 章：移动端接入', summary: '把龙虾接到 Telegram / 飞书 / QQ，随时随地发一句话就能办事。', link: '/cn/adopt/chapter3' },
  { track: '领养龙虾', title: '第 4 章：自动化任务入门', summary: '定时提醒、自动化报告、周期性工作流，从“会聊”到“会做”。', link: '/cn/adopt/chapter4' },
  { track: '领养龙虾', title: '第 5 章：Skills 技能系统', summary: '理解技能层次、安装市场技能、开发自定义技能并调试。', link: '/cn/adopt/chapter5' },
  { track: '领养龙虾', title: '第 7 章：生产环境部署', summary: 'VPS/Docker/24×7 运行、隔离与常见部署排障要点。', link: '/cn/adopt/chapter7' },
  { track: '构建龙虾', title: '第 1 章：核心定位与设计理念', summary: 'Agent Runtime vs Chatbot，四个原语工具的设计哲学。', link: '/cn/build/chapter1' },
  { track: '构建龙虾', title: '第 3 章：提示词系统', summary: '7 个 Markdown 文件如何组成提示词、热更新与 Token 预算。', link: '/cn/build/chapter3' },
  { track: '构建龙虾', title: '第 5 章：消息循环与事件驱动', summary: 'ReAct 循环、心跳机制、重试与超时，Agent 如何“活起来”。', link: '/cn/build/chapter5' },
  { track: '构建龙虾', title: '第 7 章：轻量化方案', summary: 'NanoClaw/Nanobot/ZeroClaw 路线拆解：更小、更快、更易维护。', link: '/cn/build/chapter7' },
  { track: '构建龙虾', title: '第 8 章：安全加固方案', summary: '权限控制、沙箱隔离、审计与防注入：让龙虾在生产环境更稳。', link: '/cn/build/chapter8' },
  { track: '构建龙虾', title: '第 9 章：硬件方案', summary: '树莓派/迷你主机/ARM 设备选型，低功耗部署与本地模型协同。', link: '/cn/build/chapter9' },
]

const shuffle = (items) => {
  const arr = [...items]
  for (let i = arr.length - 1; i > 0; i--) {
    const j = Math.floor(Math.random() * (i + 1))
    ;[arr[i], arr[j]] = [arr[j], arr[i]]
  }
  return arr
}

const recommendedSections = ref(shuffle(baseRecommendedSections))
const tickerItems = computed(() => [...recommendedSections.value, ...recommendedSections.value])

const copyCommand = () => {
  navigator.clipboard.writeText('curl -fsSL https://openclaw.ai/install.sh | bash')
  copied.value = true
  setTimeout(() => copied.value = false, 2000)
}

const triggerHappyJump = () => {
  isHappy.value = false
  requestAnimationFrame(() => {
    isHappy.value = true
    setTimeout(() => {
      isHappy.value = false
    }, 700)
  })
}

const updateEyeTracking = (event) => {
  const maxOffset = 1.8
  const normalizedX = (event.clientX / window.innerWidth - 0.5) * 2
  const normalizedY = (event.clientY / window.innerHeight - 0.5) * 2
  eyeOffsetX.value = Math.max(-maxOffset, Math.min(maxOffset, normalizedX * maxOffset))
  eyeOffsetY.value = Math.max(-maxOffset, Math.min(maxOffset, normalizedY * maxOffset))
}

onMounted(() => {
  window.addEventListener('mousemove', updateEyeTracking)
})

onUnmounted(() => {
  window.removeEventListener('mousemove', updateEyeTracking)
})

// Simple intersection observer for fade-in animations
const vFadeIn = {
  mounted: (el) => {
    el.style.opacity = '0'
    el.style.transform = 'translateY(40px)'
    el.style.transition = 'opacity 0.8s ease-out, transform 0.8s ease-out'
    
    const observer = new IntersectionObserver((entries) => {
      entries.forEach(entry => {
        if (entry.isIntersecting) {
          el.style.opacity = '1'
          el.style.transform = 'translateY(0)'
          observer.unobserve(el)
        }
      })
    }, { threshold: 0.1 })
    
    observer.observe(el)
  }
}
</script>

<style>
@keyframes twitch-left {
  0%, 85%, 100% { transform: rotate(0deg); }
  88% { transform: rotate(-20deg); }
  91% { transform: rotate(10deg); }
  94% { transform: rotate(-10deg); }
  97% { transform: rotate(5deg); }
}
@keyframes twitch-right {
  0%, 85%, 100% { transform: rotate(0deg); }
  88% { transform: rotate(20deg); }
  91% { transform: rotate(-10deg); }
  94% { transform: rotate(10deg); }
  97% { transform: rotate(-5deg); }
}
.claw-left {
  transform-origin: 25px 55px;
  animation: twitch-left 6s infinite ease-in-out;
}
.claw-right {
  transform-origin: 95px 55px;
  animation: twitch-right 6s infinite ease-in-out;
}
@keyframes wave-right {
  0%, 100% { transform: rotate(0deg); }
  20% { transform: rotate(34deg); }
  40% { transform: rotate(-18deg); }
  60% { transform: rotate(30deg); }
  80% { transform: rotate(-12deg); }
}
@keyframes wave-left {
  0%, 100% { transform: rotate(0deg); }
  50% { transform: rotate(-6deg); }
}
.claw-right.waving {
  animation: wave-right 0.7s infinite ease-in-out;
}
.claw-left.waving {
  animation: wave-left 1.2s infinite ease-in-out;
}
@keyframes float {
  0%, 100% { transform: translateY(0) rotate(0deg); }
  25% { transform: translateY(-5px) rotate(2.5deg); }
  50% { transform: translateY(-10px) rotate(5deg); }
  75% { transform: translateY(-5px) rotate(-2.5deg); }
}
.lobster-anim {
  animation: float 4s infinite ease-in-out;
  transition: transform 0.5s ease-out;
}
.lobster-anim.hovered {
  transform: scale(1.1);
}
.lobster-anim.happy {
  animation: happy-bounce 0.7s ease-out;
}
.lobster-smile {
  opacity: 0;
  transition: opacity 0.15s ease;
}
.lobster-smile.visible {
  opacity: 1;
}
@keyframes happy-bounce {
  0% { transform: translateY(0) scale(1); }
  20% { transform: translateY(-16px) scale(1.06); }
  45% { transform: translateY(0) scale(1); }
  65% { transform: translateY(-8px) scale(1.03); }
  100% { transform: translateY(0) scale(1); }
}
.fade-in-init {
  animation: fadeInInit 0.8s ease-out forwards;
}
@keyframes fadeInInit {
  from { opacity: 0; transform: translateY(20px); }
  to { opacity: 1; transform: translateY(0); }
}
@keyframes gradient-loop {
  0% { background-position: 0% 50%; }
  100% { background-position: 100% 50%; }
}
.animate-gradient-text {
  background: linear-gradient(to right, #ffffff, #ff4d4d, #52b79a);
  background-size: 200% auto;
  animation: gradient-loop 3s linear infinite alternate;
  -webkit-background-clip: text;
  -webkit-text-fill-color: transparent;
  background-clip: text;
}

.marquee {
  animation: marquee-loop 42s linear infinite;
}

.marquee:hover {
  animation-play-state: paused;
}

@keyframes marquee-loop {
  0% { transform: translateX(0); }
  100% { transform: translateX(-50%); }
}
</style>
