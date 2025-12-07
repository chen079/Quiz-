<template>
  <div class="w-full h-full">
    <Radar :data="chartData" :options="chartOptions" />
  </div>
</template>

<script setup>
import { computed } from 'vue'
import {
  Chart as ChartJS,
  RadialLinearScale,
  PointElement,
  LineElement,
  Filler,
  Tooltip,
  Legend
} from 'chart.js'
import { Radar } from 'vue-chartjs'

ChartJS.register(RadialLinearScale, PointElement, LineElement, Filler, Tooltip, Legend)

const props = defineProps({
  scores: { type: Object, required: true },
  meta: { type: Object, default: () => ({}) }
})

const chartData = computed(() => {
  const labels = []
  const data = []
  for (const key in props.scores) {
    labels.push(props.meta[key]?.label || key)
    data.push(props.scores[key])
  }

  return {
    labels,
    datasets: [{
      label: '得分',
      backgroundColor: 'rgba(99, 102, 241, 0.2)',
      borderColor: '#6366F1',
      pointBackgroundColor: '#6366F1',
      pointBorderColor: '#fff',
      pointHoverBackgroundColor: '#fff',
      pointHoverBorderColor: '#6366F1',
      data
    }]
  }
})

const chartOptions = {
  responsive: true,
  maintainAspectRatio: false,
  scales: {
    r: {
      angleLines: { color: 'rgba(0,0,0,0.1)' },
      grid: { color: 'rgba(0,0,0,0.1)' },
      pointLabels: {
        font: { size: 14, family: 'system-ui, sans-serif' },
        color: '#475569'
      },
      ticks: { display: false, beginAtZero: true }
    }
  },
  plugins: { legend: { display: false } }
}
</script>