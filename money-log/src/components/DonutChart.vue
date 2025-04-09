<script setup>
import { Doughnut } from 'vue-chartjs'
import { Chart as ChartJS, ArcElement, Tooltip, Legend } from 'chart.js'
import { computed } from 'vue'

ChartJS.register(ArcElement, Tooltip, Legend)

const props = defineProps({
  goal: Number,
  spent: Number,
})

const percentage = computed(() => {
  if (!props.goal) return 0
  return Math.min(Math.round((props.spent / props.goal) * 100), 100)
})

const chartData = computed(() => ({
  labels: ['소비', '남은 금액'],
  datasets: [
    {
      data: [props.spent, props.goal - props.spent],
      backgroundColor: ['#facc15', '#d4d4d4'],
      borderWidth: 0,
    },
  ],
}))

const chartOptions = {
  cutout: '70%', // 차트 도넛 두께
  plugins: {
    legend: { display: false },
    tooltip: { enabled: false },
  },
  responsive: true,
  maintainAspectRatio: false,
}
</script>

<template>
  <div class="donut-chart-container">
    <Doughnut :data="chartData" :options="chartOptions" />
    <div class="chart-label">{{ percentage }}%</div>
    <!-- 공백 추가 -->
  </div>
</template>

<style scoped>
.donut-chart-container {
  position: relative;
  width: 90px; /* 차트 크기 */
  height: 90px; /* 차트 크기 */
  margin: 0 auto;
}

.chart-label {
  position: absolute;
  top: 50%;
  left: 50%;
  transform: translate(-50%, -50%);
  font-weight: bold;
  font-size: 1rem;
  color: #feba17;
}
</style>
