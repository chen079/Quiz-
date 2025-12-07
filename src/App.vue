<template>
  <div
    class="min-h-screen bg-gradient-to-br from-indigo-50 via-purple-50 to-pink-50 py-10 px-4 font-sans text-slate-800 flex items-center justify-center">

    <div v-if="loading" class="fixed inset-0 bg-white/50 backdrop-blur-sm z-50 flex items-center justify-center">
      <div class="animate-spin rounded-full h-16 w-16 border-t-4 border-b-4 border-indigo-600"></div>
    </div>

    <div v-if="view === 'home'" class="max-w-5xl mx-auto w-full">
      <h1
        class="text-5xl font-extrabold text-center mb-4 text-transparent bg-clip-text bg-gradient-to-r from-indigo-600 to-purple-600">
        ✨ 趣味心理实验室</h1>
      <p class="text-center text-slate-500 text-lg mb-12">实时更新 · 探索未知</p>

      <div class="grid grid-cols-1 md:grid-cols-2 lg:grid-cols-3 gap-8">
        <div v-for="quiz in quizList" :key="quiz.id" @click="loadAndStartQuiz(quiz)"
          class="group bg-white/70 backdrop-blur-lg rounded-3xl shadow-xl hover:shadow-2xl hover:-translate-y-2 transition-all duration-300 cursor-pointer overflow-hidden border border-white/50">
          <div class="h-48 overflow-hidden relative">
            <div class="absolute inset-0 bg-black/20 group-hover:bg-black/0 transition-all duration-300 z-10"></div>
            <img :src="quiz.coverImage"
              class="w-full h-full object-cover transform group-hover:scale-110 transition-transform duration-500" />
          </div>
          <div class="p-6 relative z-20">
            <h3 class="text-2xl font-bold mb-3 text-slate-800">{{ quiz.title }}</h3>
            <p class="text-slate-500 text-sm leading-relaxed line-clamp-2">{{ quiz.description }}</p>
            <div
              class="mt-6 inline-flex items-center text-indigo-600 font-semibold group-hover:text-purple-600 transition-colors">
              开始测试
              <svg xmlns="http://www.w3.org/2000/svg" class="h-5 w-5 ml-2" fill="none" viewBox="0 0 24 24"
                stroke="currentColor">
                <path stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="M13 10V3L4 14h7v7l9-11h-7z" />
              </svg>
            </div>
          </div>
        </div>
      </div>
    </div>

    <div v-else-if="view === 'playing'" class="max-w-3xl mx-auto w-full">
      <div
        class="bg-white/80 backdrop-blur-xl rounded-[2.5rem] shadow-[0_20px_50px_rgba(8,_112,_184,_0.1)] p-8 md:p-12 border border-white/60 animate-fade-in-up relative overflow-hidden">

        <div
          class="absolute -top-10 -right-10 w-40 h-40 bg-purple-300 rounded-full mix-blend-multiply filter blur-3xl opacity-30 animate-blob">
        </div>
        <div
          class="absolute -bottom-10 -left-10 w-40 h-40 bg-indigo-300 rounded-full mix-blend-multiply filter blur-3xl opacity-30 animate-blob animation-delay-2000">
        </div>

        <div class="relative z-10 mb-8 flex justify-between items-center">
          <button @click="view = 'home'"
            class="text-sm text-slate-400 hover:text-indigo-600 transition-colors flex items-center">&larr;
            退出测试</button>
          <span class="text-sm font-medium text-slate-500">进度 {{ currentQIndex + 1 }} / {{ activeQuiz.questions.length
            }}</span>
        </div>

        <div class="relative z-10 mb-8 w-full bg-slate-200/60 rounded-full h-3 overflow-hidden">
          <div
            class="bg-gradient-to-r from-indigo-500 to-purple-500 h-full rounded-full transition-all duration-700 ease-out"
            :style="{ width: progress + '%' }"></div>
        </div>

        <h2
          class="relative z-10 text-3xl md:text-4xl font-extrabold text-slate-800 mb-10 leading-tight text-center min-h-[4rem] flex items-center justify-center">
          {{ currentQuestion.text }}
        </h2>

        <div class="relative z-10 space-y-4 mb-10">
          <button v-for="(opt, idx) in currentQuestion.options" :key="idx" @click="selectOption(opt)" :class="[
            'group w-full text-left p-5 rounded-2xl border-2 transition-all duration-200 font-semibold text-lg flex items-center justify-between active:scale-[0.98]',
            isOptionSelected(opt)
              ? 'border-indigo-500 bg-indigo-50 text-indigo-700 shadow-md'
              : 'border-indigo-100 bg-white/50 text-slate-700 hover:bg-indigo-50 hover:border-indigo-300'
          ]">
            <span class="mr-4">{{ opt.text }}</span>

            <div v-if="isOptionSelected(opt)" class="h-6 w-6 shrink-0 text-indigo-500">
              <svg xmlns="http://www.w3.org/2000/svg" viewBox="0 0 20 20" fill="currentColor">
                <path fill-rule="evenodd"
                  d="M10 18a8 8 0 100-16 8 8 0 000 16zm3.707-9.293a1 1 0 00-1.414-1.414L9 10.586 7.707 9.293a1 1 0 00-1.414 1.414l2 2a1 1 0 001.414 0l4-4z"
                  clip-rule="evenodd" />
              </svg>
            </div>
            <div v-else class="h-6 w-6 shrink-0 rounded-full border-2 border-indigo-200 group-hover:border-indigo-400">
            </div>
          </button>
        </div>

        <div class="relative z-10 flex justify-between items-center pt-4 border-t border-indigo-100/50">
          <button @click="prevQuestion" :disabled="currentQIndex === 0"
            class="px-6 py-3 rounded-xl font-bold text-slate-500 hover:bg-slate-100 disabled:opacity-30 disabled:cursor-not-allowed transition-colors flex items-center">
            <svg xmlns="http://www.w3.org/2000/svg" class="h-5 w-5 mr-1" viewBox="0 0 20 20" fill="currentColor">
              <path fill-rule="evenodd"
                d="M12.707 5.293a1 1 0 010 1.414L9.414 10l3.293 3.293a1 1 0 01-1.414 1.414l-4-4a1 1 0 010-1.414l4-4a1 1 0 011.414 0z"
                clip-rule="evenodd" />
            </svg>
            上一题
          </button>

          <button @click="nextQuestion" :disabled="!currentAnswer"
            class="bg-gradient-to-r from-indigo-600 to-purple-600 text-white px-8 py-3 rounded-xl font-bold hover:shadow-lg hover:shadow-indigo-500/30 hover:-translate-y-0.5 disabled:opacity-50 disabled:cursor-not-allowed disabled:shadow-none disabled:translate-y-0 transition-all flex items-center">
            {{ isLastQuestion ? '查看结果 🎉' : '下一题' }}
            <svg v-if="!isLastQuestion" xmlns="http://www.w3.org/2000/svg" class="h-5 w-5 ml-1" viewBox="0 0 20 20"
              fill="currentColor">
              <path fill-rule="evenodd"
                d="M7.293 14.707a1 1 0 010-1.414L10.586 10 7.293 6.707a1 1 0 011.414-1.414l4 4a1 1 0 010 1.414l-4 4a1 1 0 01-1.414 0z"
                clip-rule="evenodd" />
            </svg>
          </button>
        </div>

      </div>
    </div>

    <div v-else-if="view === 'result'" class="max-w-2xl mx-auto w-full text-center my-10">
      <div
        class="bg-white/95 backdrop-blur-xl rounded-[3rem] shadow-2xl p-8 md:p-12 border border-white/60 animate-fade-in-up">
        <div class="text-6xl mb-4 animate-bounce-slow">🎉</div>
        <h2
          class="text-3xl md:text-4xl font-extrabold text-transparent bg-clip-text bg-gradient-to-r from-indigo-700 to-purple-700 mb-4">
          {{ finalResult.name }}
        </h2>
        <p class="text-lg text-slate-600 leading-relaxed mb-8 font-medium">
          {{ finalResult.description }}
        </p>

        <div v-if="(!activeQuiz.type || activeQuiz.type === 'dimensions')" class="mb-10 h-64 md:h-80 relative z-10">
          <RadarChart :scores="finalScores" :meta="activeQuiz.dimensionMeta" />
        </div>

        <div v-else-if="activeQuiz.type === 'scoring'" class="mb-12 relative z-10">
          <div class="inline-block relative">
            <div class="absolute inset-0 bg-indigo-400 blur-2xl opacity-20 rounded-full"></div>
            <div
              class="relative bg-white/50 backdrop-blur-sm border-4 border-indigo-100 rounded-full w-40 h-40 flex flex-col items-center justify-center shadow-xl">
              <span class="text-5xl font-black text-indigo-600 tracking-tighter">
                {{ finalScores['总分'] }}
              </span>
              <span class="text-sm font-bold text-slate-400 uppercase mt-1">总得分</span>
            </div>
          </div>
        </div>

        <div v-if="activeQuiz.dimensionMeta"
          class="bg-indigo-50/60 p-6 rounded-3xl text-left space-y-4 border border-indigo-100">
          <h3 class="font-bold text-indigo-900 text-lg flex items-center mb-4">
            <svg xmlns="http://www.w3.org/2000/svg" class="h-6 w-6 mr-2" fill="none" viewBox="0 0 24 24"
              stroke="currentColor">
              <path stroke-linecap="round" stroke-linejoin="round" stroke-width="2"
                d="M9 19v-6a2 2 0 00-2-2H5a2 2 0 00-2 2v6a2 2 0 002 2h2a2 2 0 002-2zm0 0V9a2 2 0 012-2h2a2 2 0 012 2v10m-6 0a2 2 0 002 2h2a2 2 0 002-2m0 0V5a2 2 0 012-2h2a2 2 0 012 2v14a2 2 0 01-2 2h-2a2 2 0 01-2-2z" />
            </svg>
            详细指标分析
          </h3>
          <div v-for="(score, key) in finalScores" :key="key"
            class="bg-white p-4 rounded-2xl shadow-sm border border-indigo-50 relative overflow-hidden">
            <div class="absolute -right-2 -bottom-4 text-6xl font-black text-slate-100 z-0 opacity-50">{{ key }}</div>
            <div class="relative z-10">
              <div class="flex justify-between items-center mb-2">
                <span class="font-bold text-slate-800 text-lg">{{ activeQuiz.dimensionMeta?.[key]?.label || key
                  }}</span>
                <span class="font-mono font-bold text-indigo-600 text-xl">{{ score.toFixed(0) }}</span>
              </div>
              <p v-if="activeQuiz.dimensionMeta?.[key]?.desc" class="text-slate-500 text-sm leading-relaxed">
                {{ activeQuiz.dimensionMeta[key].desc }}
              </p>
            </div>
          </div>
        </div>

        <button @click="view = 'home'"
          class="mt-8 bg-gradient-to-r from-indigo-600 to-purple-600 text-white px-10 py-4 rounded-full font-bold text-lg hover:shadow-lg hover:shadow-indigo-500/40 hover:-translate-y-1 transition-all active:scale-95">
          返回测试列表
        </button>
      </div>
    </div>

  </div>
</template>

<script setup>
import { ref, computed, onMounted, defineAsyncComponent } from 'vue'

// 异步组件：按需加载雷达图，优化首屏速度
const RadarChart = defineAsyncComponent(() => import('./components/RadarChart.vue'))

// --- 状态管理 ---
const view = ref('home')
const loading = ref(false)
const quizList = ref([])
const activeQuiz = ref(null)
const currentQIndex = ref(0)
const finalResult = ref(null)
const finalScores = ref({})
const userAnswers = ref([])

// --- 初始化 ---
onMounted(async () => {
  await fetchQuizList()
})

const fetchQuizList = async () => {
  try {
    loading.value = true
    // 加时间戳防止缓存
    const res = await fetch('/quizzes/index.json?t=' + Date.now())
    if (!res.ok) throw new Error('无法加载测试列表')
    quizList.value = await res.json()
  } catch (e) {
    alert('加载列表失败: ' + e.message)
  } finally {
    loading.value = false
  }
}

// --- 核心：加载测试并合并元数据 ---
const loadAndStartQuiz = async (quizInfo) => {
  try {
    loading.value = true
    const res = await fetch(quizInfo.file + '?t=' + Date.now())
    if (!res.ok) throw new Error('测试文件丢失')

    const fileData = await res.json()

    // 合并 index.json 的信息和具体文件的信息
    const mergedQuiz = {
      ...fileData,
      ...quizInfo,
      dimensionMeta: fileData.dimensionMeta || {}
    }

    activeQuiz.value = mergedQuiz
    currentQIndex.value = 0
    userAnswers.value = new Array(mergedQuiz.questions.length).fill(null)
    view.value = 'playing'
  } catch (e) {
    alert('无法开始测试: ' + e.message)
  } finally {
    loading.value = false
  }
}

// --- 计算属性 ---
const currentQuestion = computed(() => {
  if (!activeQuiz.value) return {}
  return activeQuiz.value.questions[currentQIndex.value]
})

const currentAnswer = computed(() => userAnswers.value[currentQIndex.value])

const progress = computed(() => {
  if (!activeQuiz.value) return 0
  return ((currentQIndex.value + 1) / activeQuiz.value.questions.length) * 100
})

const isLastQuestion = computed(() => {
  return activeQuiz.value && currentQIndex.value === activeQuiz.value.questions.length - 1
})

// --- 交互方法 ---
const selectOption = (option) => {
  userAnswers.value[currentQIndex.value] = option
}

const isOptionSelected = (option) => currentAnswer.value === option

const nextQuestion = () => {
  if (!currentAnswer.value) return
  if (isLastQuestion.value) {
    calculateResult()
  } else {
    currentQIndex.value++
  }
}

const prevQuestion = () => {
  if (currentQIndex.value > 0) currentQIndex.value--
}

// --- 双模式计算引擎 ---
const calculateResult = () => {
  const quiz = activeQuiz.value
  const type = quiz.type || 'dimensions' // 默认为维度模式

  // === 模式 A: 计分模式 (Scoring) ===
  if (type === 'scoring') {
    let totalScore = 0
    userAnswers.value.forEach(ans => {
      if (ans && typeof ans.score === 'number') {
        totalScore += ans.score
      }
    })

    finalScores.value = { "总分": totalScore }

    // 查找落在区间的 range: [min, max]
    const matched = quiz.results.find(res => {
      const [min, max] = res.range || [-Infinity, Infinity]
      return totalScore >= min && totalScore <= max
    })
    finalResult.value = matched || { name: '结果计算失败', description: '未命中任何分数段' }
  }

  // === 模式 B: 维度分析 (Dimensions) ===
  else {
    let scores = { ...quiz.dimensions }
    userAnswers.value.forEach(answer => {
      if (answer && answer.scores) {
        for (const [key, value] of Object.entries(answer.scores)) {
          if (scores[key] !== undefined) { scores[key] += value }
        }
      }
    })
    finalScores.value = scores

    let matched = null
    for (const res of quiz.results) {
      if (res.condition === 'default') { if (!matched) matched = res; continue }
      try {
        const logic = new Function('scores', 'Math', `return ${res.condition}`)
        if (logic(scores, Math)) { matched = res; break }
      } catch (e) { console.error('逻辑判定出错:', res.condition, e) }
    }
    finalResult.value = matched || { name: '未知结果', description: '无法计算' }
  }

  view.value = 'result'
}
</script>

<style>
@keyframes fadeInUp {
  from {
    opacity: 0;
    transform: translateY(30px) scale(0.98);
  }

  to {
    opacity: 1;
    transform: translateY(0) scale(1);
  }
}

.animate-fade-in-up {
  animation: fadeInUp 0.6s cubic-bezier(0.22, 1, 0.36, 1) forwards;
}

@keyframes blob {
  0% {
    transform: translate(0px, 0px) scale(1);
  }

  33% {
    transform: translate(30px, -50px) scale(1.1);
  }

  66% {
    transform: translate(-20px, 20px) scale(0.9);
  }

  100% {
    transform: translate(0px, 0px) scale(1);
  }
}

.animate-blob {
  animation: blob 7s infinite;
}

.animation-delay-2000 {
  animation-delay: 2s;
}

@keyframes bounceSlow {

  0%,
  100% {
    transform: translateY(0);
  }

  50% {
    transform: translateY(-10px);
  }
}

.animate-bounce-slow {
  animation: bounceSlow 3s infinite ease-in-out;
}

.line-clamp-2 {
  display: -webkit-box;
  -webkit-line-clamp: 2;
  -webkit-box-orient: vertical;
  overflow: hidden;
}
</style>