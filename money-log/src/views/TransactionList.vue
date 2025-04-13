<script setup>
import ListFilter from '@/components/ListFilter.vue'
import { useRoute } from 'vue-router'
import { useTransactionStore } from '@/stores/transactionStore'
import { computed, onMounted, ref, watch } from 'vue'

import { getCategoryIcon } from '@/stores/categoryIcons'

const route = useRoute()
const dashboardMonth = route.params.month
const dashboardYear = route.params.year
// transactionStore에 있는 데이터 가져오기
const transactionStore = useTransactionStore()
const currentMonth = ref(4)
const currentYear = ref(2025)

// computed 옵션으로 currentYear나 currentMonth가 바뀌면 자동으로 selectedMonth도 갱신
const selectedMonth = computed(
  () =>
    `${currentYear.value}-${currentMonth.value.toString().padStart(2, '0')}`,
)

watch(selectedMonth, newVal => {
  // console.log('newVal:', newVal)
  transactionStore.getTransactionInfo(newVal)
})

function prevMonth() {
  const date = new Date(currentYear.value, currentMonth.value - 2)
  currentMonth.value = date.getMonth() + 1
  currentYear.value = date.getFullYear()

  // )
}
function nextMonth() {
  const date = new Date(currentYear.value, currentMonth.value)
  currentMonth.value = date.getMonth() + 1
  currentYear.value = date.getFullYear()
}

// 해당 월의 총 수입 계산
const selectedMonthIncome = computed(() =>
  transactionStore.monthTotalIncome(selectedMonth.value),
)
// 해당 월의 총 지출 계산
const selectedMonthExpense = computed(() =>
  transactionStore.monthTotalExpense(selectedMonth.value),
)

onMounted(() => {
  // 스크롤을 맨 위로 이동
  window.scrollTo(0, 0)

  // 대시보드 달력에서 선택된 날짜의 리스트로 이동
  currentMonth.value = dashboardMonth
  currentYear.value = dashboardYear
  transactionStore.getTransactionInfo(`${dashboardYear}-0${dashboardMonth}`)
})
</script>

<template>
  <div class="transaction-list">
    <div class="transaction__bg"></div>
    <div class="transaction-filter__container">
      <!-- 달력 상단 바 -->
      <div class="top-bar">
        <div class="logo"></div>
      </div>
      <!-- 달력 -->
      <div class="month-box">
        <div class="nav">
          <span @click="prevMonth">&lt;</span>
          <span>{{ `${currentMonth}월` }}</span>
          <span @click="nextMonth">&gt;</span>
        </div>
        <div class="year">{{ currentYear }}</div>
        <div class="description">이번 달에는 이렇게 썼어요!</div>
      </div>
      <ListFilter
        :currentYear="currentYear"
        :currentMonth="currentMonth"
        :selectedMonth="selectedMonth"
      />
    </div>

    <!-- 리스트 -->
    <div class="transaction-list__container">
      <div class="transaction-list__top-container">
        전체 내역 {{ transactionStore.countTransactionData(selectedMonth) }} 건
        <div class="amount__container">
          <span
            ><span class="amount__container-label">수입</span>
            <span class="income-amount__text"
              >{{ selectedMonthIncome.toLocaleString() }}&nbsp;원</span
            ></span
          >
          |
          <span>
            <span class="amount__container-label">지출</span>
            <span class="expense-amount__text">{{
              selectedMonthExpense.toLocaleString()
            }}</span
            >&nbsp;원
          </span>
        </div>
      </div>
      <div class="transaction-list__bottom-container">
        <ul class="transaction-list__list">
          <li>
            <div class="transaction-list__title">
              <span>Category</span>
              <span>Date</span>
              <span>Memo</span>
            </div>
          </li>
          <li class="list__line"></li>
          <!-- 여기 아래 부분 v-for문 돌리면 됩니다.-->
          <li
            v-for="list in transactionStore.filteredTransactionData"
            :key="list.id"
            class="list-content"
          >
            <RouterLink :to="`/detail/${list.id}`" class="list-link">
              <div class="list-content__ctg">
                <img :src="getCategoryIcon(list.code)" alt="아이콘" />
                <div class="list-content__text">
                  <!-- 수입인지 지출인지 판단하여 색상 변경, +, - 붙여줌 -->
                  <p
                    :class="
                      list.type === 'income'
                        ? 'income-amount'
                        : 'expense-amount'
                    "
                  >
                    {{ list.type === 'income' ? '+' : '-' }}
                    {{ list.amount.toLocaleString() }}원
                  </p>
                  <p>{{ list.category }}</p>
                </div>
              </div>
              <div class="list-content__date">{{ list.date }}</div>
              <div class="list-content__memo">{{ list.content }}</div>
            </RouterLink>
          </li>
        </ul>
      </div>
    </div>
  </div>
</template>

<style scoped>
.transaction-list {
  position: relative;
  padding-top: 40px;
}
.transaction__bg {
  background-color: var(--yellow-200);
  height: 400px;
  top: -125px;
  position: relative;
  z-index: -100;
}
.transaction-filter__container {
  display: flex;
  flex-direction: column;
  position: relative;
  top: -20rem;
  align-items: center;
}
.top-bar {
  display: flex;
  justify-content: space-between;
  align-items: center;
  padding: 20px;
}

.month-box {
  text-align: center;
  margin-top: 10px;
  position: relative;
  top: -90px;
}

.month-box .nav {
  font-size: 3rem;
  font-weight: bold;
  display: flex;
  justify-content: center;
  align-items: center;
  gap: 20px;
  margin-bottom: 2rem;
}

.month-box .nav span {
  cursor: pointer;
}

.month-box .year {
  font-size: 18px;
  margin-top: 8px;
  font-weight: 800;
  color: #444;
}

.month-box .description {
  margin-top: 10px;
  font-size: 14px;
  color: #333;
  font-weight: 800;
}

.transaction-list__container {
  display: flex;
  flex-direction: column;
  align-items: center;
  justify-content: center;
  max-width: 1280px;
  width: calc(100% - 2rem);
  margin: auto;
  position: relative;
  top: -20rem;
}
.transaction-list__top-container {
  display: flex;
  justify-content: space-between;
  width: 100%;
  font-size: 1.5rem;
  font-weight: 800;
  color: var(--gray-500);
  padding: 0 0.8rem;
}
.amount__container {
  display: flex;
  justify-content: flex-end;
  gap: 2rem;
  flex-grow: 0.2;
  font-size: 1.2rem;
  font-weight: 800;
  color: var(--gray-500);
}
.amount__container-label {
  margin-right: 1rem;
}
.income-amount__text {
  color: var(--blue);
}
.expense-amount__text {
  color: var(--point-color);
}
.transaction-list__bottom-container {
  width: 100%;
  margin-top: 2rem;
}
.transaction-list__list {
  display: flex;
  flex-direction: column;
}
.transaction-list__title {
  display: flex;
  width: 100%;
  height: 2rem;
  padding: 0 1.2rem;
  justify-content: space-between;
  align-items: center;
  color: black;
  font-size: 1.2rem;
  font-weight: 800;
}

.transaction-list__title {
  display: flex;
  width: 100%;
  padding: 2rem;
  justify-content: space-between;
  align-items: center;
  color: black;
  font-size: 1.2rem;
  font-weight: 800;
}

.transaction-list__title span:nth-child(1) {
  /* text-align: center; */
  padding-left: 7rem;
  width: 50%;
}

.transaction-list__title span:nth-child(2) {
  /* text-align: center; */
  padding-left: 3rem;
  width: 32%;
}

.transaction-list__title span:nth-child(3) {
  text-align: center;
  flex-grow: 1;
}

.list__line {
  margin-top: 1rem;
  border-bottom: 1px solid var(--gray-300);
}
.list-content {
  display: flex;
  border-bottom: 1px solid var(--gray-300);
  padding: 2rem;
  justify-content: space-between;
  align-items: center;
  color: black;
  font-size: 1.5rem;
  font-weight: 400;
}
.list-content__ctg {
  display: flex;
  align-items: center;
  gap: 2rem;
  width: 50%;
  /* flex-grow: 0.4; */
}
.list-content__ctg img {
  width: 4.375rem;
}
.list-content__date {
  display: flex;
  /* flex-grow: 0.6; */
  align-items: center;
  width: 32%;
}
.list-content__memo {
  display: flex;
  align-items: center;
  flex-grow: 1;
}
.list-link {
  display: flex;
  justify-content: space-between;
  align-items: center;
  text-decoration: none;
  color: inherit;
  width: 100%;
}
.income-amount {
  color: var(--blue);
}

.expense-amount {
  color: var(--point-color);
}
.list-content__text {
  display: flex;
  flex-direction: column;
  gap: 0.5rem;
}
</style>
