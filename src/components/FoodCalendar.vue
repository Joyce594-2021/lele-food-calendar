<template>
  <div class="food-calendar">
    <!-- 页面标题和导出按钮 -->
    <div class="page-header">
      <button class="back-btn" @click="emit('back-to-home')" title="返回首页">
        <svg width="20" height="20" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2">
          <polyline points="15 18 9 12 15 6"/>
        </svg>
      </button>
      <h1 class="page-title zcool-kuaile-regular">{{ currentMonthTitle }}</h1>
      <button class="export-btn" @click="exportPDF" title="导出PDF">
        <svg width="16" height="16" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2">
          <path d="M21 15v4a2 2 0 0 1-2 2H5a2 2 0 0 1-2-2v-4"/>
          <polyline points="7 10 12 15 17 10"/>
          <line x1="12" y1="15" x2="12" y2="3"/>
        </svg>
      </button>
    </div>

    <!-- 头部信息 -->
    <!-- <div class="calendar-header">
      <div class="month-nav">
        <h2 class="month-title nanum-pen-script-regular">{{ currentMonthTitle }}</h2>
      </div>
    </div> -->

    <!-- 星期标题 -->
    <div class="week-header">
      <div class="week-day zcool-kuaile-regular" v-for="day in weekDays" :key="day">{{ day }}</div>
    </div>

    <!-- 日历网格 -->
    <div class="calendar-grid">
      <!-- 空白占位 -->
      <div class="calendar-cell empty" v-for="n in firstDayOffset" :key="'empty-' + n"></div>
      
      <!-- 日期单元格 -->
      <div 
        class="calendar-cell" 
        :class="{ 
          'has-plan': getPlan(date),
          'today': isToday(date),
          'selected': isSelected(date)
        }"
        v-for="date in monthDates" 
        :key="date"
        @click="selectDate(date)"
      >
        <div class="cell-date caveat-brush-regular">{{ getDateNum(date) }}</div>
        <div class="cell-content zcool-kuaile-regular" v-if="getPlan(date)">
          <div class="meal morning">
            <span class="meal-text">{{ getPlan(date)!.morning }}</span>
          </div>
          <div class="meal afternoon" v-if="getPlan(date)!.afternoon !== '无辅食'">
            <span class="meal-text">{{ getPlan(date)!.afternoon }}</span>
          </div>
        </div>
        <div class="cell-tip zcool-kuaile-regular" v-if="getPlan(date)?.tip">
          {{ getPlan(date)!.tip }}
        </div>
      </div>
    </div>

  </div>
</template>

<script setup lang="ts">
import { ref, computed, onMounted, watch } from 'vue'
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

interface MonthMap {
  [key: string]: MonthData
}

// interface BabyInfo {
//   birthDate: string
//   note: string
// }

const props = defineProps<{
  month?: number
}>()

const emit = defineEmits<{
  'back-to-home': []
}>()

// const babyInfo = (foodPlanData as any).babyInfo as BabyInfo
const monthsData = (foodPlanData as any).months as MonthMap

// 当前显示的月份数据
const currentMonthData = ref<FoodPlan[]>([])
const currentMonthLabel = ref('')

// 构建日期到计划的映射
const planMap = ref(new Map<string, FoodPlan>())

function updateMonthData(month: number) {
  const monthKey = String(month)
  const data = monthsData[monthKey]
  if (data) {
    currentMonthData.value = data.plans || []
    currentMonthLabel.value = data.monthName || `${month}月`
  } else {
    currentMonthData.value = []
    currentMonthLabel.value = `${month}月`
  }
  
  // 重建映射
  const newMap = new Map<string, FoodPlan>()
  currentMonthData.value.forEach(plan => {
    newMap.set(plan.date, plan)
  })
  planMap.value = newMap
}

// 初始化月份数据
updateMonthData(props.month || 4)

// 当前显示的年月
const currentYear = ref(2026)
const currentMonth = ref((props.month || 4) - 1) // 0-based

// 选中的日期
const selectedDate = ref<string | null>(null)

// 星期名称
const weekDays = ['日', '一', '二', '三', '四', '五', '六']

// 标题
const currentMonthTitle = computed(() => {
  return `${currentYear.value}年${currentMonthLabel.value}`
})

// 获取当月所有日期
const monthDates = computed(() => {
  const year = currentYear.value
  const month = currentMonth.value
  const daysInMonth = new Date(year, month + 1, 0).getDate()
  
  const dates: string[] = []
  for (let day = 1; day <= daysInMonth; day++) {
    const date = new Date(year, month, day)
    const dateStr = formatDate(date)
    dates.push(dateStr)
  }
  return dates
})

// 当月第一天是星期几
const firstDayOffset = computed(() => {
  const year = currentYear.value
  const month = currentMonth.value
  return new Date(year, month, 1).getDay()
})

// 获取某天的计划
const getPlan = (date: string): FoodPlan | null => {
  return planMap.value.get(date) || null
}

// 是否是今天
const isToday = (date: string) => {
  const today = new Date()
  const todayStr = formatDate(today)
  return date === todayStr
}

// 是否选中
const isSelected = (date: string) => {
  return date === selectedDate.value
}

// 获取日期数字
const getDateNum = (date: string) => {
  return new Date(date).getDate()
}

// 选择日期
const selectDate = (date: string) => {
  if (planMap.value.has(date)) {
    selectedDate.value = date
  }
}

// 格式化日期为 YYYY-MM-DD
function formatDate(date: Date): string {
  const year = date.getFullYear()
  const month = String(date.getMonth() + 1).padStart(2, '0')
  const day = String(date.getDate()).padStart(2, '0')
  return `${year}-${month}-${day}`
}

// 监听月份 prop 变化
watch(() => props.month, (newMonth) => {
  if (newMonth !== undefined) {
    currentMonth.value = newMonth - 1
    updateMonthData(newMonth)
  }
})

// 初始化时默认选中今天（如果今天有计划）
onMounted(() => {
  // 如果传入了月份 prop，使用它
  if (props.month !== undefined) {
    currentMonth.value = props.month - 1
    updateMonthData(props.month)
  }
  
  // 检查今天是否有辅食计划，有则选中
  const today = formatDate(new Date())
  if (planMap.value.has(today)) {
    selectedDate.value = today
  }
})

const exportPDF = () => {
  window.print()
}
</script>

<style scoped>
.food-calendar {
  width: 100%;
  max-width: 900px;
  margin: 0 auto;
  padding: 24px;
  box-sizing: border-box;
  position: relative;
}

/* 页面头部 - 标题和导出按钮 */
.page-header {
  display: flex;
  align-items: center;
  justify-content: center;
  margin-bottom: 20px;
  position: relative;
}

/* 返回按钮 */
.back-btn {
  position: absolute;
  left: 0;
  top: 50%;
  transform: translateY(-50%);
  background: none;
  border: none;
  padding: 6px;
  cursor: pointer;
  color: var(--text-secondary, #8C8C8C);
  opacity: 0.6;
  transition: all 0.25s ease;
  border-radius: 6px;
  display: flex;
  align-items: center;
}

.back-btn:hover {
  opacity: 1;
  background: rgba(0, 0, 0, 0.05);
  color: var(--text-primary, #4A4A4A);
}

.page-title {
  font-size: 28px;
  font-weight: 600;
  color: var(--text-primary, #4A4A4A);
  text-align: center;
  margin: 0;
  letter-spacing: 2px;
}

/* 导出按钮 - 低调的图标按钮，放在右上角 */
.export-btn {
  position: absolute;
  right: 0;
  top: 50%;
  transform: translateY(-50%);
  background: none;
  border: none;
  padding: 6px;
  cursor: pointer;
  color: var(--text-secondary, #8C8C8C);
  opacity: 0.5;
  transition: all 0.25s ease;
  border-radius: 6px;
}

.export-btn:hover {
  opacity: 0.8;
  background: rgba(0, 0, 0, 0.05);
  color: var(--text-primary, #4A4A4A);
}

.export-btn:active {
  opacity: 1;
}

.export-btn svg {
  display: block;
}

/* 当前日期展示板块 */
.current-date-section {
  background: linear-gradient(135deg, rgba(167, 199, 215, 0.15) 0%, rgba(184, 197, 176, 0.15) 100%);
  padding: 14px 20px;
  border-radius: 14px;
  margin-bottom: 20px;
  border: 1px solid var(--border, #E5E0D8);
  display: flex;
  align-items: center;
  justify-content: center;
  box-shadow: 0 2px 10px rgba(74, 74, 74, 0.04);
}

.date-display {
  display: flex;
  align-items: center;
  gap: 10px;
}

.calendar-icon {
  color: var(--primary, #A7C7D7);
  flex-shrink: 0;
}

.date-text {
  font-size: 18px;
  font-weight: 500;
  color: var(--text-primary, #4A4A4A);
  letter-spacing: 1px;
}

/* 头部样式 */
.calendar-header {
  text-align: center;
  margin-bottom: 24px;
}

.month-nav {
  display: flex;
  align-items: center;
  justify-content: center;
  gap: 16px;
  margin-bottom: 16px;
}

.month-title {
  font-size: 22px;
  font-weight: 500;
  color: var(--text-primary, #4A4A4A);
  margin: 0;
  min-width: 120px;
}

.nav-btn {
  width: 44px;
  height: 44px;
  border: 1px solid var(--border, #E5E0D8);
  border-radius: 12px;
  background: var(--bg-surface, #FFFFFF);
  color: var(--text-primary, #4A4A4A);
  cursor: pointer;
  display: flex;
  align-items: center;
  justify-content: center;
  transition: all 0.25s ease;
  touch-action: manipulation;
  box-shadow: 0 2px 8px rgba(74, 74, 74, 0.05);
}

.nav-btn:hover:not(:disabled) {
  background: var(--bg-surface-alt, #EFECE8);
  border-color: var(--primary, #A7C7D7);
}

.nav-btn:active:not(:disabled) {
  transform: scale(0.96);
  box-shadow: none;
}

.nav-btn:disabled {
  opacity: 0.4;
  cursor: not-allowed;
}

/* 宝宝信息卡片 */
.baby-info {
  display: flex;
  flex-direction: column;
  gap: 4px;
  background: linear-gradient(135deg, rgba(220, 174, 150, 0.12) 0%, rgba(167, 199, 215, 0.12) 100%);
  padding: 12px 16px;
  border-radius: 12px;
  margin: 0 8px;
}

.baby-age {
  font-size: 13px;
  color: var(--secondary-dark, #C99A82);
  font-weight: 500;
}

.baby-note {
  font-size: 12px;
  color: var(--text-secondary, #8C8C8C);
}

/* 星期标题 */
.week-header {
  display: grid;
  grid-template-columns: repeat(7, 1fr);
  background: var(--bg-surface-alt, #EFECE8);
  border-radius: 12px 12px 0 0;
  border: 1px solid var(--border, #E5E0D8);
  border-bottom: none;
}

.week-day {
  padding: 12px 4px;
  text-align: center;
  font-weight: 500;
  color: var(--text-secondary, #8C8C8C);
  font-size: 14px;
}

.week-day:first-child {
  color: var(--secondary, #DCAE96);
}

.week-day:last-child {
  color: var(--primary, #A7C7D7);
}

/* 日历网格 */
.calendar-grid {
  display: grid;
  grid-template-columns: repeat(7, 1fr);
  border: 1px solid var(--border, #E5E0D8);
  border-top: none;
  border-radius: 0 0 12px 12px;
  overflow: hidden;
  box-shadow: 0 4px 16px rgba(74, 74, 74, 0.06);
}

.calendar-cell {
  min-height: 115px;
  padding: 8px;
  border: 1px solid var(--border-light, #EDEAE5);
  background: var(--bg-surface, #FFFFFF);
  cursor: pointer;
  transition: all 0.25s ease;
  position: relative;
  touch-action: manipulation;
  -webkit-tap-highlight-color: transparent;
  display: flex;
  flex-direction: column;
}

.calendar-cell:hover:not(.empty) {
  background: var(--bg-surface-alt, #EFECE8);
  transform: translateY(-1px);
  box-shadow: 0 2px 8px rgba(74, 74, 74, 0.05);
  z-index: 1;
}

.calendar-cell.empty {
  background: var(--bg-primary, #F7F5F3);
  cursor: default;
  border-color: var(--border-light, #EDEAE5);
}

.calendar-cell:not(.empty):active {
  background: var(--bg-surface-alt, #EFECE8);
  transform: scale(0.98);
}

.calendar-cell.has-plan {
  cursor: pointer;
}

.calendar-cell.today {
  border-color: var(--accent, #B8C5B0);
  border-width: 2px;
}

.calendar-cell.selected {
  background: rgba(167, 199, 215, 0.15);
  border-color: var(--primary, #A7C7D7);
}

.cell-date {
  font-size: 15px;
  font-weight: 500;
  color: var(--text-primary, #4A4A4A);
  margin-bottom: 6px;
}

.cell-content {
  font-size: 10px;
  line-height: 1.4;
  flex: 1;
  display: flex;
  flex-direction: column;
  justify-content: flex-start;
  gap: 3px;
}

.meal {
  display: flex;
  gap: 3px;
  margin-bottom: 3px;
  align-items: flex-start;
  flex-wrap: wrap;
  border-radius: 8px;
  padding: 4px 5px;
}

/* 早餐 - 温暖的豆沙红色调 */
.meal.morning {
  background: linear-gradient(135deg, rgba(220, 174, 150, 0.15) 0%, rgba(220, 174, 150, 0.08) 100%);
  border-left: 3px solid var(--secondary, #DCAE96);
}

/* 下午餐 - 柔和的橄榄绿色调 */
.meal.afternoon {
  background: linear-gradient(135deg, rgba(184, 197, 176, 0.2) 0%, rgba(184, 197, 176, 0.1) 100%);
  border-left: 3px solid var(--accent, #B8C5B0);
}

.meal-text {
  color: var(--text-primary, #4A4A4A);
  overflow: visible;
  white-space: normal;
  word-break: break-word;
  font-size: 9px;
  line-height: 1.3;
}

.cell-tip {
  font-size: 9px;
  color: var(--text-secondary, #8C8C8C);
  background: rgba(220, 174, 150, 0.12);
  padding: 3px 5px;
  border-radius: 6px;
  margin-top: auto;
  line-height: 1.3;
  word-break: break-word;
}

/* 平板适配 (768px - 1024px) */
@media (min-width: 768px) {
  .food-calendar {
    padding: 28px;
  }

  .month-title {
    font-size: 26px;
  }

  .calendar-cell {
    min-height: 130px;
    padding: 10px;
  }

  .cell-date {
    font-size: 16px;
  }

  .cell-content {
    font-size: 11px;
  }
}

/* 小屏幕手机适配 (小于375px) */
@media (max-width: 374px) {
  .food-calendar {
    padding: 12px;
  }

  .current-date-section {
    padding: 10px 14px;
    margin-bottom: 14px;
  }

  .date-text {
    font-size: 15px;
  }

  .month-nav {
    gap: 10px;
  }

  .month-title {
    font-size: 18px;
    min-width: 100px;
  }

  .nav-btn {
    width: 40px;
    height: 40px;
  }

  .calendar-cell {
    min-height: 95px;
    padding: 5px;
  }

  .cell-date {
    font-size: 13px;
    margin-bottom: 3px;
  }

  .cell-content {
    font-size: 9px;
  }

  .meal {
    gap: 2px;
    margin-bottom: 2px;
  }

  .meal-text {
    font-size: 8px;
  }

  .week-day {
    font-size: 11px;
    padding: 8px 2px;
  }
}

/* 横屏适配 */
@media (max-height: 500px) and (orientation: landscape) {
  .calendar-cell {
    min-height: 65px;
  }

  .cell-content {
    font-size: 9px;
  }

  .meal {
    margin-bottom: 1px;
  }
}

/* 打印样式 - 优化PDF导出效果 */
@media print {
  .page-header {
    margin-bottom: 16pt !important;
  }
  
  .page-title {
    font-size: 24pt !important;
    margin-bottom: 0 !important;
  }
  /* 隐藏不需要打印的元素 */
  .export-btn,
  .nav-btn {
    display: none !important;
  }

  /* 设置页面边距和大小 */
  @page {
    size: landscape;
    margin: 1cm;
  }

  body {
    margin: 0;
    padding: 0;
    background: white !important;
  }

  .food-calendar {
    padding: 0 !important;
    max-width: 100% !important;
  }

  /* 确保背景色打印 */
  .calendar-cell,
  .week-header,
  .meal {
    -webkit-print-color-adjust: exact !important;
    print-color-adjust: exact !important;
  }

  /* 调整标题 */
  .month-title {
    font-size: 20pt !important;
    margin-bottom: 12pt !important;
  }

  /* 调整日历网格 */
  .calendar-grid {
    break-inside: avoid;
  }

  .calendar-cell {
    min-height: 80px !important;
    border: 1px solid #ddd !important;
  }

  .cell-date {
    font-size: 11pt !important;
  }

  .cell-content {
    font-size: 8pt !important;
  }

  .meal-text {
    font-size: 8pt !important;
  }

  .week-day {
    font-size: 10pt !important;
  }
}
</style>