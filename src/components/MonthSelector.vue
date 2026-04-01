<template>
  <div class="month-selector">
    <header class="page-header">
      <h1 class="page-title zcool-kuaile-regular">乐乐的辅食日记</h1>
    </header>

    <div class="months-grid">
      <div
        class="month-card"
        v-for="month in months"
        :key="month.num"
        :class="{ 
          'has-plan': month.hasPlan,
          'current-month': month.isCurrentMonth
        }"
        @click="selectMonth(month.num)"
      >
        <div class="month-icon">
          <svg v-if="month.hasPlan" width="32" height="32" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="1.5">
            <rect x="3" y="4" width="18" height="18" rx="2" ry="2"/>
            <line x1="16" y1="2" x2="16" y2="6"/>
            <line x1="8" y1="2" x2="8" y2="6"/>
            <line x1="3" y1="10" x2="21" y2="10"/>
            <path d="M8 14h.01M12 14h.01M16 14h.01M8 18h.01M12 18h.01"/>
          </svg>
          <svg v-else width="32" height="32" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="1.5" opacity="0.3">
            <rect x="3" y="4" width="18" height="18" rx="2" ry="2"/>
            <line x1="16" y1="2" x2="16" y2="6"/>
            <line x1="8" y1="2" x2="8" y2="6"/>
            <line x1="3" y1="10" x2="21" y2="10"/>
          </svg>
        </div>
        <div class="month-name zcool-kuaile-regular">{{ month.name }}</div>
        <!-- <div class="month-plan-count" v-if="month.hasPlan">
          {{ month.planCount }}天记录
        </div>
        <div class="month-plan-count empty" v-else>
          暂无记录
        </div> -->
      </div>
    </div>

    <!-- <footer class="page-footer">
      <p class="footer-text">点击有记录的月份查看辅食详情</p>
    </footer> -->
  </div>
</template>

<script setup lang="ts">
import { computed } from 'vue'
import foodPlanData from '../data/food-plan.json'

interface FoodPlan {
  date: string
  dayDesc: string
  morning: string
  afternoon: string
  tip: string
}

interface MonthData {
  monthName: string
  plans: FoodPlan[]
  babyAge?: string
}

interface MonthInfo {
  num: number
  name: string
  hasPlan: boolean
  planCount: number
  isCurrentMonth: boolean
}

interface MonthMap {
  [key: string]: MonthData
}

const year = computed(() => foodPlanData.year as number)
const monthsData = computed(() => (foodPlanData.months as MonthMap))

const now = new Date()
const currentYear = now.getFullYear()
const currentMonth = now.getMonth() + 1

const months = computed(() => {
  const result: MonthInfo[] = []
  for (let i = 1; i <= 12; i++) {
    const monthData = monthsData.value[String(i)]
    const planCount = monthData?.plans?.length || 0
    result.push({
      num: i,
      name: monthData?.monthName || `${i}月`,
      hasPlan: planCount > 0,
      planCount,
      isCurrentMonth: i === currentMonth && year.value === currentYear
    })
  }
  return result
})

const emit = defineEmits<{
  'select-month': [month: number]
}>()

const selectMonth = (month: number) => {
  const monthData = monthsData.value[String(month)]
  if (monthData && monthData.plans && monthData.plans.length > 0) {
    emit('select-month', month)
  }
}
</script>

<style scoped>
.month-selector {
  width: 100%;
  max-width: 700px;
  margin: 0 auto;
  padding: 32px 24px;
  box-sizing: border-box;
  min-height: 100vh;
  display: flex;
  flex-direction: column;
}

.page-header {
  text-align: center;
  margin-bottom: 40px;
}

.page-title {
  font-size: 32px;
  font-weight: 600;
  color: var(--text-primary, #4A4A4A);
  margin: 0;
  letter-spacing: 3px;
}

.months-grid {
  display: grid;
  grid-template-columns: repeat(4, 1fr);
  gap: 16px;
  flex: 1;
}

.month-card {
  aspect-ratio: 1;
  display: flex;
  flex-direction: column;
  align-items: center;
  justify-content: center;
  gap: 8px;
  background: var(--bg-surface, #FFFFFF);
  border: 2px solid var(--border, #E5E0D8);
  border-radius: 16px;
  cursor: pointer;
  transition: all 0.3s ease;
  padding: 12px;
  box-shadow: 0 2px 12px rgba(74, 74, 74, 0.05);
}

.month-card:hover {
  transform: translateY(-4px);
  box-shadow: 0 6px 20px rgba(74, 74, 74, 0.1);
}

.month-card.has-plan {
  border-color: var(--primary, #A7C7D7);
  background: linear-gradient(135deg, rgba(167, 199, 215, 0.08) 0%, rgba(184, 197, 176, 0.08) 100%);
}

.month-card.has-plan:hover {
  border-color: var(--secondary, #DCAE96);
  background: linear-gradient(135deg, rgba(220, 174, 150, 0.12) 0%, rgba(167, 199, 215, 0.12) 100%);
}

.month-card.current-month {
  box-shadow: 0 0 0 3px rgba(167, 199, 215, 0.3);
}

.month-card:not(.has-plan) {
  opacity: 0.5;
  cursor: default;
}

.month-card:not(.has-plan):hover {
  transform: none;
  box-shadow: 0 2px 12px rgba(74, 74, 74, 0.05);
}

.month-icon {
  color: var(--primary, #A7C7D7);
  margin-bottom: 4px;
}

.month-card.has-plan .month-icon {
  color: var(--secondary, #DCAE96);
}

.month-name {
  font-size: 18px;
  font-weight: 600;
  color: var(--text-primary, #4A4A4A);
}

.month-plan-count {
  font-size: 11px;
  color: var(--text-secondary, #8C8C8C);
  background: rgba(167, 199, 215, 0.15);
  padding: 3px 8px;
  border-radius: 10px;
}

.month-plan-count.empty {
  background: rgba(0, 0, 0, 0.04);
  color: #B0B0B0;
}

.page-footer {
  text-align: center;
  margin-top: 40px;
  padding-top: 20px;
  border-top: 1px solid var(--border, #E5E0D8);
}

.footer-text {
  color: var(--text-secondary, #8C8C8C);
  font-size: 13px;
  margin: 0;
}

/* 平板适配 */
@media (min-width: 768px) {
  .months-grid {
    grid-template-columns: repeat(4, 1fr);
    gap: 20px;
  }

  .month-card {
    border-radius: 20px;
    padding: 16px;
  }

  .month-name {
    font-size: 20px;
  }
}

/* 小屏幕适配 */
@media (max-width: 480px) {
  .month-selector {
    padding: 24px 16px;
  }

  .page-header {
    margin-bottom: 28px;
  }

  .page-title {
    font-size: 24px;
  }

  .months-grid {
    grid-template-columns: repeat(3, 1fr);
    gap: 12px;
  }

  .month-name {
    font-size: 16px;
  }
}

@media (max-width: 374px) {
  .months-grid {
    grid-template-columns: repeat(2, 1fr);
  }
}
</style>
