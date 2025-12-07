<template>
  <div class="min-h-screen bg-gradient-to-br from-indigo-50 via-purple-50 to-pink-50 py-10 px-4 font-sans text-slate-800 flex items-center justify-center">

    <div v-if="view === 'home'" class="max-w-5xl mx-auto w-full">
      <h1 class="text-5xl font-extrabold text-center mb-4 text-transparent bg-clip-text bg-gradient-to-r from-indigo-600 to-purple-600">✨ 趣味心理实验室</h1>
      <p class="text-center text-slate-500 text-lg mb-12">探索你未知的另一面</p>

      <div class="grid grid-cols-1 md:grid-cols-2 lg:grid-cols-3 gap-8">
        <div
          v-for="quiz in quizList"
          :key="quiz.id"
          @click="startQuiz(quiz)"
          class="group bg-white/70 backdrop-blur-lg rounded-3xl shadow-xl hover:shadow-2xl hover:-translate-y-2 transition-all duration-300 cursor-pointer overflow-hidden border border-white/50"
        >
          <div class="h-48 overflow-hidden relative">
            <div class="absolute inset-0 bg-black/20 group-hover:bg-black/0 transition-all duration-300 z-10"></div>
            <img :src="quiz.coverImage" class="w-full h-full object-cover transform group-hover:scale-110 transition-transform duration-500" />
          </div>
          <div class="p-6 relative z-20">
            <h3 class="text-2xl font-bold mb-3 text-slate-800">{{ quiz.title }}</h3>
            <p class="text-slate-500 text-sm leading-relaxed line-clamp-2">{{ quiz.description }}</p>
            <div class="mt-6 inline-flex items-center text-indigo-600 font-semibold group-hover:text-purple-600 transition-colors">
              开始测试
              <svg xmlns="http://www.w3.org/2000/svg" class="h-5 w-5 ml-2 group-hover:translate-x-1 transition-transform" fill="none" viewBox="0 0 24 24" stroke="currentColor"><path stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="M14 5l7 7m0 0l-7 7m7-7H3" /></svg>
            </div>
          </div>
        </div>
      </div>
    </div>

    <div v-else-if="view === 'playing'" class="max-w-3xl mx-auto w-full">
      <div class="bg-white/80 backdrop-blur-xl rounded-[2.5rem] shadow-[0_20px_50px_rgba(8,_112,_184,_0.1)] p-8 md:p-12 border border-white/60 animate-fade-in-up relative overflow-hidden">
        
        <div class="absolute -top-10 -right-10 w-40 h-40 bg-purple-300 rounded-full mix-blend-multiply filter blur-3xl opacity-30 animate-blob"></div>
        <div class="absolute -bottom-10 -left-10 w-40 h-40 bg-indigo-300 rounded-full mix-blend-multiply filter blur-3xl opacity-30 animate-blob animation-delay-2000"></div>

        <div class="relative z-10 mb-8">
          <div class="flex justify-between text-sm font-medium text-slate-500 mb-3">
            <span>进度</span>
            <span>{{ currentQIndex + 1 }} / {{ activeQuiz.questions.length }}</span>
          </div>
          <div class="w-full bg-slate-200/60 rounded-full h-3 overflow-hidden">
            <div class="bg-gradient-to-r from-indigo-500 to-purple-500 h-full rounded-full transition-all duration-700 ease-out" :style="{ width: progress + '%' }"></div>
          </div>
        </div>

        <h2 class="relative z-10 text-3xl md:text-4xl font-extrabold text-slate-800 mb-10 leading-tight text-center">
          {{ currentQuestion.text }}
        </h2>

        <div class="relative z-10 space-y-5">
          <button
            v-for="(opt, idx) in currentQuestion.options"
            :key="idx"
            @click="handleAnswer(opt)"
            class="group w-full text-left p-5 rounded-2xl border-2 border-indigo-100 bg-white/50 hover:bg-indigo-50 hover:border-indigo-400 hover:shadow-lg hover:shadow-indigo-100/50 transition-all duration-300 font-semibold text-lg text-slate-700 hover:text-indigo-700 active:scale-[0.98] flex items-center justify-between"
          >
            <span>{{ opt.text }}</span>
            <svg xmlns="http://www.w3.org/2000/svg" class="h-6 w-6 opacity-0 group-hover:opacity-100 group-hover:translate-x-1 transition-all text-indigo-500" fill="none" viewBox="0 0 24 24" stroke="currentColor">
              <path stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="M9 5l7 7-7 7" />
            </svg>
          </button>
        </div>
      </div>
    </div>

    <div v-else-if="view === 'result'" class="max-w-2xl mx-auto w-full text-center">
      <div class="bg-white/90 backdrop-blur-xl rounded-[3rem] shadow-2xl p-12 border border-white/60 animate-fade-in-up">
        <div class="text-7xl mb-6 animate-bounce-slow">🎉</div>
        <h2 class="text-4xl font-extrabold text-transparent bg-clip-text bg-gradient-to-r from-indigo-700 to-purple-700 mb-6">
          {{ finalResult.name }}
        </h2>
        <div class="w-20 h-1 bg-gradient-to-r from-indigo-500 to-purple-500 mx-auto mb-8 rounded-full"></div>
        <p class="text-xl text-slate-600 leading-relaxed mb-10 font-medium">
          {{ finalResult.description }}
        </p>

        <button
          @click="view = 'home'"
          class="bg-gradient-to-r from-indigo-600 to-purple-600 text-white px-10 py-4 rounded-full font-bold text-lg hover:shadow-lg hover:shadow-indigo-500/40 hover:-translate-y-1 transition-all active:scale-95"
        >
          返回首页
        </button>
      </div>
    </div>

  </div>
</template>

<script setup>
import { ref, computed } from 'vue'

const quizFiles = import.meta.glob('./quizzes/*.json', { eager: true })
const quizList = Object.values(quizFiles).map(module => module.default || module)

const view = ref('home')
const activeQuiz = ref(null)
const currentScores = ref({})
const currentQIndex = ref(0)
const finalResult = ref(null)

const currentQuestion = computed(() => {
  if (!activeQuiz.value) return {}
  return activeQuiz.value.questions[currentQIndex.value]
})

const progress = computed(() => {
  if (!activeQuiz.value) return 0
  return ((currentQIndex.value) / activeQuiz.value.questions.length) * 100
})

const startQuiz = (quiz) => {
  activeQuiz.value = quiz
  currentScores.value = { ...quiz.dimensions }
  currentQIndex.value = 0
  view.value = 'playing'
}

const handleAnswer = (option) => {
  if (option.scores) {
    for (const [key, value] of Object.entries(option.scores)) {
      if (currentScores.value[key] !== undefined) {
        currentScores.value[key] += value
      }
    }
  }

  if (currentQIndex.value < activeQuiz.value.questions.length - 1) {
    currentQIndex.value++
  } else {
    currentQIndex.value++
    setTimeout(() => {
        calculateResult()
    }, 500)
  }
}

const calculateResult = () => {
  const results = activeQuiz.value.results
  const scores = currentScores.value
  let matched = null

  for (const res of results) {
    if (res.condition === 'default') {
      if (!matched) matched = res
      continue
    }
    try {
      const check = new Function('scores', `return ${res.condition}`)(scores)
      if (check) {
        matched = res
        break
      }
    } catch (e) {
      console.error('逻辑错误', e)
    }
  }
  finalResult.value = matched || { name: '未知结果', description: '无法计算' }
  view.value = 'result'
}
</script>

<style>
@keyframes fadeInUp {
  from { opacity: 0; transform: translateY(30px) scale(0.98); }
  to { opacity: 1; transform: translateY(0) scale(1); }
}
.animate-fade-in-up {
  animation: fadeInUp 0.6s cubic-bezier(0.22, 1, 0.36, 1) forwards;
}
@keyframes blob {
  0% { transform: translate(0px, 0px) scale(1); }
  33% { transform: translate(30px, -50px) scale(1.1); }
  66% { transform: translate(-20px, 20px) scale(0.9); }
  100% { transform: translate(0px, 0px) scale(1); }
}
.animate-blob {
  animation: blob 7s infinite;
}
.animation-delay-2000 {
  animation-delay: 2s;
}
@keyframes bounceSlow {
  0%, 100% { transform: translateY(0); }
  50% { transform: translateY(-10px); }
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