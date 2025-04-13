<script setup>
import axios from 'axios'
import { ref, reactive, onMounted, computed } from 'vue'
import basicIcon from '../assets/profile-icon.svg'

const emits = defineEmits(['submit', 'close'])

const amount = ref(0)
const formattedAmount = ref(0)
const date = ref(new Date().toISOString().split('T')[0])
const content = ref('')
const code = ref('')
const isIncome = ref(false)
let categories = reactive([])

const selectedCategoryIcon = ref('')
const selectedCategoryIconName = ref('')
// 선택된 아이콘이 없으면 기본 코인 돼지 모양 아이콘 사용
const resolvedIcon = computed(() => {
  if (selectedCategoryIcon.value) {
    return new URL(
      `../assets/${selectedCategoryIcon.value}.svg`,
      import.meta.url,
    ).href
  } else {
    return basicIcon
  }
})

// 선택된 카테고리의 아이콘 코드 이름 업데이트
function updateSelectedCategory(event) {
  const selectedCtgValue = event.target.value
  selectedCategoryIconName.value = event.target.value

  categories.map(ctg => {
    if (ctg.name === selectedCtgValue) selectedCategoryIcon.value = ctg.code
  })
}

// 금액 입력 시 유효성 검사
function handleAmountInput(event) {
  // 숫자가 아닌 모든 문자 제거
  const value = event.target.value.replace(/[^0-9]/g, '')

  // 숫자만 있는 경우에만 처리
  if (value) {
    // 숫자로 변환하여 저장
    amount.value = value
    // 천 단위 쉼표 추가하여 표시
    formattedAmount.value = Number(value).toLocaleString('ko-KR')
  } else {
    amount.value = ''
    formattedAmount.value = ''
  }
}

function resetForm() {
  amount.value = ''
  formattedAmount.value = ''
  date.value = new Date().toISOString().split('T')[0]
  content.value = ''
  code.value = ''
  selectedCategoryIconName.value = ''
  isIncome.value = false
}

function submitForm() {
  const formData = {
    amount: amount.value,
    date: date.value,
    content: content.value,
    code: selectedCategoryIcon.value,
    category: selectedCategoryIconName.value,
    isIncome: isIncome.value,
  }
  emits('submit', formData)
  closeModal()
}

function closeModal() {
  resetForm()
  emits('close')
}

// 카테고리 정보 불러오기
const API_URL = 'http://localhost:5500/categories'

async function getCategory() {
  try {
    const response = await axios.get(API_URL)
    const data = response.data
    categories = data
  } catch (error) {
    console.error('카테고리 데이터를 불러오는데 실패했습니다:', error)
    throw error
  }
}

onMounted(() => {
  getCategory()
})
</script>

<template>
  <div class="modal-overlay" v-if="isOpen" @click.self="closeModal">
    <div class="modal-content">
      <div class="modal-header">
        <!-- 모달 창 헤더 -->
        <img src="@/assets/coin-icon.svg" alt="코인 아이콘" class="coin-icon" />
        <h2>거래 내역을 입력하세요</h2>
        <p class="subtitle">지출 또는 수입 내역을 입력하는 곳이에요</p>
        <!-- 수입/지출 토글 스위치 -->
        <div class="toggle-container">
          <span :class="{ active: !isIncome }">지출</span>
          <label class="toggle-switch">
            <input type="checkbox" v-model="isIncome" />
            <span class="toggle-slider"></span>
          </label>
          <span :class="{ active: isIncome }">수입</span>
        </div>
      </div>
      <!-- 금액, 날짜 input 모음 -->
      <div class="form-group">
        <div class="input-row">
          <div class="input-group">
            <label
              >금액 amount
              <input
                type="text"
                v-model="formattedAmount"
                placeholder="금액을 입력하세요"
                @input="handleAmountInput"
              />
            </label>
          </div>
          <div class="input-group">
            <label
              >날짜 Date
              <input type="date" v-model="date" />
            </label>
          </div>
        </div>
        <!-- 메모, 카테고리 input 모음 -->
        <div class="input-row">
          <div class="input-group content-group">
            <label
              >메모 memo
              <input
                type="text"
                v-model="content"
                placeholder="메모를 입력하세요"
              />
            </label>
          </div>
          <!-- 카테고리 선택 -->
          <div class="input-group">
            <label
              >카테고리 Category
              <div class="category-select">
                <img
                  :src="resolvedIcon"
                  alt="카테고리 아이콘"
                  class="category-icon"
                />
                <select
                  v-model="category"
                  @change="updateSelectedCategory($event)"
                >
                  <option
                    v-for="cat in categories"
                    :key="cat.id"
                    :value="cat.name"
                  >
                    {{ cat.name }}
                  </option>
                </select>
              </div>
            </label>
          </div>
        </div>
      </div>
      <!-- 버튼 그룹 -->
      <div class="button-group">
        <button class="cancel-btn" @click="closeModal">취소</button>
        <button class="submit-btn" @click="submitForm">저장</button>
      </div>
    </div>
  </div>
</template>

<style scoped>
.modal-overlay {
  position: fixed;
  top: 0;
  left: 0;
  width: 100%;
  height: 100%;
  background-color: rgba(0, 0, 0, 0.5);
  display: flex;
  justify-content: center;
  align-items: center;
  z-index: 1000;
}

.modal-content {
  background: white;
  padding: 2rem;
  border-radius: 16px;
  width: 90%;
  max-width: 500px;
}

.modal-header {
  text-align: center;
  margin-bottom: 2rem;
  display: flex;
  flex-direction: column;
  align-items: center;
}

.coin-icon {
  width: 48px;
  height: 48px;
  margin-bottom: 0.5rem;
}

h2 {
  font-size: 1.5rem;
  margin-bottom: 0.3rem;
}

.subtitle {
  color: #666;
  font-size: 0.8rem;
  margin-bottom: 1rem;
}

.form-group {
  margin-bottom: 2rem;
}

.input-row {
  display: flex;
  gap: 0.5rem;
  margin-bottom: 1rem;
}

.input-group {
  flex: 1;
}

.content-group {
  flex: 1;
}

label {
  display: block;
  margin-bottom: 0.3rem;
  font-size: 0.9rem;
}

input,
select {
  width: 100%;
  padding: 0.75rem;
  border: 1px solid #ddd;
  border-radius: 8px;
  font-size: 1rem;
  margin: 0.5rem 0;
}

/* 숫자 입력 화살표 제거 */
input[type='number']::-webkit-inner-spin-button,
input[type='number']::-webkit-outer-spin-button {
  -webkit-appearance: none;
  margin: 0;
}

input[type='number'] {
  -moz-appearance: textfield;
}

.button-group {
  display: flex;
  gap: 1rem;
}

button {
  flex: 1;
  padding: 1rem;
  border: none;
  border-radius: 8px;
  font-size: 1rem;
  cursor: pointer;
}

.cancel-btn {
  background-color: #f5f5f5;
  color: #333;
}

.submit-btn {
  background-color: #ffc107;
  color: white;
}

.submit-btn:hover {
  background-color: #ffb300;
}

.toggle-container {
  display: flex;
  align-items: center;
  justify-content: center;
  gap: 1rem;
  margin: 0.5rem 0;
}

.toggle-switch {
  position: relative;
  display: inline-block;
  width: 60px;
  height: 34px;
}

.toggle-switch input {
  opacity: 0;
  width: 0;
  height: 0;
}

.toggle-slider {
  position: absolute;
  cursor: pointer;
  top: 0;
  left: 0;
  right: 0;
  bottom: 0;
  background-color: #ffc107;
  transition: 0.4s;
  border-radius: 34px;
}

.toggle-slider:before {
  position: absolute;
  content: '';
  height: 26px;
  width: 26px;
  left: 4px;
  bottom: 4px;
  background-color: white;
  transition: 0.4s;
  border-radius: 50%;
}

input:checked + .toggle-slider {
  background-color: #4caf50;
}

input:checked + .toggle-slider:before {
  transform: translateX(26px);
}

.toggle-container span {
  font-size: 1rem;
  color: #666;
  transition: color 0.3s;
}

.toggle-container span.active {
  color: #333;
  font-weight: bold;
}

.category-select {
  display: flex;
  align-items: center;
  gap: 0.5rem;
}

.category-icon {
  width: 40px;
  height: 40px;
  object-fit: contain;
}

select {
  flex: 1;
  padding: 0.75rem;
  border: 1px solid #ddd;
  border-radius: 8px;
  font-size: 1rem;
  margin: 0.5rem 0;
  background-color: white;
  cursor: pointer;
}
</style>
