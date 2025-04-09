<script setup>
import { ref, computed, onMounted } from 'vue'
import axios from 'axios'
import DonutChart from './DonutChart.vue'

// 날짜 → 'YYYY-MM'
const now = new Date()
const currentMonth = `${now.getFullYear()}-${String(now.getMonth() + 1).padStart(2, '0')}`

// 상태값
const goalAmount = ref(0)
const transactions = ref([])

// 데이터 로딩
onMounted(() => {
  // 목표 예산
  axios
    .get('http://localhost:3001/goal')
    .then(response => {
      const goalData = response.data
      if (goalData.month === currentMonth) {
        goalAmount.value = goalData.targetExpense
      }
    })
    .catch(error => {
      console.error('목표 예산 불러오기 실패:', error)
    })

  // 트랜잭션 내역
  axios
    .get('http://localhost:3001/transactions')
    .then(response => {
      transactions.value = response.data
    })
    .catch(error => {
      console.error('트랜잭션 불러오기 실패:', error)
    })
})

// 이번 달 지출만 필터링
const thisMonthExpenses = computed(() =>
  transactions.value.filter(
    t => t.type === 'expense' && t.date?.startsWith(currentMonth),
  ),
)

// 실제 지출 합계
const currentSpending = computed(() =>
  thisMonthExpenses.value.reduce((sum, item) => sum + item.amount, 0),
)
</script>

<template>
  <div class="GoalTracker">
    <h2 class="title">목표 금액까지, <span class="highlight">이만큼</span></h2>
    <br />
    <DonutChart :goal="goalAmount" :spent="currentSpending" />
    <div class="text-wrap">
      <div class="label">
        <p class="label-name">목표금액</p>
        <span class="label-value goal"
          >{{ goalAmount.toLocaleString() }}원</span
        >
      </div>
      <div class="label">
        <p class="label-name">현재까지의 소비</p>
        <span class="label-value nowspend"
          >{{ currentSpending.toLocaleString() }}원</span
        >
      </div>
    </div>
  </div>
</template>

<style scoped>
.GoalTracker {
  font-family: 'Pretendard', sans-serif;
  padding: 1rem;
  text-align: center;
}

.title {
  text-align: center;
  font-size: 1rem;
  line-height: 1.8;
  font-weight: 900;
}

.highlight {
  color: #f5b63c;
  font-weight: 900;
  text-shadow: 0.8px 0 currentColor;
}

/* 하단 정렬을 위한 flex 적용 */
.text-wrap {
  margin-top: 1.5rem;
  display: flex;
  flex-direction: column;
  gap: 0.75rem;
  align-items: center;
}

.label {
  display: flex;
  justify-content: space-between;
  width: 240px;
}

.label-name {
  font-weight: 900;
  font-size: 0.95rem;
  margin: 0;
}

.label-value {
  font-weight: 600;
  font-size: 0.8rem;
}

.goal {
  color: #4c4539;
}

.nowspend {
  color: #feba17;
}
</style>
