<script setup>
import { ref } from 'vue'
import { RouterView } from 'vue-router'
import MoneyInputModal from './components/MoneyInputModal.vue'
import FloatingButton from './components/FloatingButton.vue'
import axios from 'axios'

const isModalOpen = ref(false)

const openModal = () => {
  isModalOpen.value = true
}

const closeModal = () => {
  isModalOpen.value = false
}

const handleSubmit = async formData => {
  try {
    // 1. 기존 거래 내역 가져오기
    const response = await axios.get('http://localhost:5500/transactions')
    const transactions = response.data

    // 2. 새로운 ID 생성 (기존 최대 ID + 1)
    const maxId = Math.max(...transactions.map(t => Number(t.id)), 0)
    const newId = maxId + 1

    // 3. 새로운 거래 내역 객체 생성
    const newTransaction = {
      id: String(newId),
      type: formData.isIncome ? 'income' : 'expense',
      category: formData.category,
      content: formData.content,
      code: formData.code,
      date: formData.date,
      amount: Number(formData.amount),
    }

    // 4. 새로운 거래 내역 저장
    await axios.post('http://localhost:5500/transactions', newTransaction)
    console.log('새로운 거래 내역이 저장되었습니다:', newTransaction)
  } catch (error) {
    console.error('데이터 저장 중 오류가 발생했습니다:', error)
  }
}
</script>

<template>
  <div class="app-container">
    <RouterView />
    <FloatingButton :onClick="openModal" />
    <MoneyInputModal
      :isOpen="isModalOpen"
      @close="closeModal"
      @submit="handleSubmit"
    />
  </div>
</template>

<style scoped></style>
