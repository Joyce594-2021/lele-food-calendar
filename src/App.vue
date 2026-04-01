<script setup lang="ts">
import { ref } from 'vue'
import MonthSelector from './components/MonthSelector.vue'
import FoodCalendar from './components/FoodCalendar.vue'

const currentView = ref<'home' | 'calendar'>('home')
const selectedMonth = ref<number>(4)

const handleSelectMonth = (month: number) => {
  selectedMonth.value = month
  currentView.value = 'calendar'
}

const handleBackToHome = () => {
  currentView.value = 'home'
}
</script>

<template>
  <MonthSelector 
    v-if="currentView === 'home'" 
    @select-month="handleSelectMonth" 
  />
  <FoodCalendar 
    v-else 
    :month="selectedMonth" 
    @back-to-home="handleBackToHome" 
  />
</template>