<template>
  <div class="food-calendar">
    <!-- 页面标题和导出按钮 -->
    <div class="page-header">
      <h1 class="page-title zcool-kuaile-regular">乐乐的辅食日记</h1>
      <button class="export-btn" @click="exportPDF" title="导出PDF">
        <svg width="16" height="16" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2">
          <path d="M21 15v4a2 2 0 0 1-2 2H5a2 2 0 0 1-2-2v-4"/>
          <polyline points="7 10 12 15 17 10"/>
          <line x1="12" y1="15" x2="12" y2="3"/>
        </svg>
      </button>
    </div>
    
    <!-- 头部信息 -->
    <div class="calendar-header">
      <div class="month-nav">
        <h2 class="month-title nanum-pen-script-regular">{{ currentMonthTitle }}</h2>
      </div>
    </div>

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
import { ref, computed, nextTick } from 'vue'
import foodPlanData from '../data/food-plan.json'
import html2canvas from 'html2canvas'
import { jsPDF } from 'jspdf'

interface FoodPlan {
  date: string
  dayDesc: string
  morning: string
  afternoon: string
  tip: string
}

interface BabyInfo {
  startAgeDays: number
  endAgeDays: number
  startDate: string
  note: string
}

const babyInfo = foodPlanData.babyInfo as BabyInfo
const aprilFoodPlan = foodPlanData.aprilFoodPlan as FoodPlan[]

// 构建日期到计划的映射
const planMap = new Map<string, FoodPlan>()
aprilFoodPlan.forEach(plan => {
  planMap.set(plan.date, plan)
})

// 当前显示的年月
const currentYear = ref(2026)
const currentMonth = ref(3) // 0-based, 3=4月

// 选中的日期
const selectedDate = ref<string | null>(null)

// 提示弹窗
const showTipModal = ref(false)
const currentTip = ref('')

// 星期名称
const weekDays = ['日', '一', '二', '三', '四', '五', '六']

// 标题
const currentMonthTitle = computed(() => {
  return `${currentYear.value}-${currentMonth.value + 1}`
})

// 宝宝月龄文本
const babyAgeText = computed(() => {
  if (!babyInfo) return ''
  const startDays = babyInfo.startAgeDays
  const months = Math.floor(startDays / 30)
  const days = startDays % 30
  return `${months}个月${days}天`
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
  return planMap.get(date) || null
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
  if (planMap.has(date)) {
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

// 导航
const canPrev = computed(() => {
  return currentYear.value > 2026 || (currentYear.value === 2026 && currentMonth.value >= 3)
})

const canNext = computed(() => {
  return currentYear.value < 2026 || (currentYear.value === 2026 && currentMonth.value < 3)
})

const prevMonth = () => {
  if (canPrev.value) {
    if (currentMonth.value === 0) {
      currentMonth.value = 11
      currentYear.value--
    } else {
      currentMonth.value--
    }
  }
}

const nextMonth = () => {
  if (canNext.value) {
    if (currentMonth.value === 11) {
      currentMonth.value = 0
      currentYear.value++
    } else {
      currentMonth.value++
    }
  }
}

const exportPDF = () => {
  window.print()
}

// // 导出PDF
// const exportPDF = async () => {
//   const element = document.querySelector('.calendar-grid') as HTMLElement
//   const weekHeader = document.querySelector('.week-header') as HTMLElement
  
//   if (!element) return
  
//   // A4横向尺寸(mm)
//   const pdfWidth = 297  // A4 landscape width
//   const pdfHeight = 210 // A4 landscape height
//   const margin = 10     // margin in mm
//   const contentWidth = pdfWidth - margin * 2
//   const contentHeight = pdfHeight - margin * 2
  
//   // 创建导出容器 - 固定尺寸确保内容在一页内
//   const exportContainer = document.createElement('div')
//   exportContainer.style.cssText = `
//     position: absolute;
//     left: -9999px;
//     top: 0;
//     width: ${contentWidth * 3.78}px;
//     background: white;
//     padding: 0;
//     font-family: Arial, sans-serif;
//     overflow: hidden;
//   `
  
//   // 内容容器带padding
//   const contentWrapper = document.createElement('div')
//   contentWrapper.style.cssText = `
//     padding: 15px;
//     box-sizing: border-box;
//   `
  
//   // 添加标题
//   const title = document.createElement('div')
//   title.style.cssText = `
//     text-align: center;
//     font-size: 20px;
//     font-weight: 600;
//     color: #4A4A4A;
//     margin-bottom: 4px;
//   `
//   title.textContent = '乐乐的辅食日记'
//   contentWrapper.appendChild(title)
  
//   // 添加月份
//   const monthTitle = document.createElement('div')
//   monthTitle.style.cssText = `
//     text-align: center;
//     font-size: 14px;
//     color: #8C8C8C;
//     margin-bottom: 8px;
//   `
//   monthTitle.textContent = currentMonthTitle.value
//   contentWrapper.appendChild(monthTitle)
  
//   // 添加星期标题
//   if (weekHeader) {
//     const weekHeaderClone = weekHeader.cloneNode(true) as HTMLElement
//     weekHeaderClone.style.cssText = `
//       display: grid;
//       grid-template-columns: repeat(7, 1fr);
//       background: #EFECE8;
//       border-radius: 6px 6px 0 0;
//       border: 1px solid #E5E0D8;
//       border-bottom: none;
//       margin-bottom: 0;
//     `
//     const weekDays = weekHeaderClone.querySelectorAll('.week-day')
//     weekDays.forEach(day => {
//       (day as HTMLElement).style.cssText = `
//         padding: 6px 2px;
//         text-align: center;
//         font-weight: 500;
//         color: #8C8C8C;
//         font-size: 12px;
//       `
//     })
//     contentWrapper.appendChild(weekHeaderClone)
//   }
  
//   // 克隆日历网格
//   const gridClone = element.cloneNode(true) as HTMLElement
//   gridClone.style.cssText = `
//     display: grid;
//     grid-template-columns: repeat(7, 1fr);
//     border: 1px solid #E5E0D8;
//     border-top: none;
//     border-radius: 0 0 6px 6px;
//     box-shadow: none;
//     margin-bottom: 0;
//   `
  
//   // 调整单元格样式
//   const cells = gridClone.querySelectorAll('.calendar-cell')
//   cells.forEach(cell => {
//     const cellEl = cell as HTMLElement
//     cellEl.style.cssText = `
//       min-height: 75px;
//       padding: 4px;
//       border: 1px solid #EDEAE5;
//       background: #FFFFFF;
//       display: flex;
//       flex-direction: column;
//       box-sizing: border-box;
//     `
    
//     const dateNum = cellEl.querySelector('.cell-date')
//     if (dateNum) {
//       (dateNum as HTMLElement).style.cssText = `
//         font-size: 12px;
//         font-weight: 500;
//         margin-bottom: 3px;
//       `
//     }
    
//     const content = cellEl.querySelector('.cell-content')
//     if (content) {
//       (content as HTMLElement).style.cssText = `
//         font-size: 7px;
//         line-height: 1.3;
//         flex: 1;
//       `
//     }
    
//     const meals = cellEl.querySelectorAll('.meal')
//     meals.forEach(meal => {
//       (meal as HTMLElement).style.cssText = `
//         margin-bottom: 2px;
//         padding: 2px 3px;
//         border-radius: 3px;
//       `
//     })
    
//     const mealTexts = cellEl.querySelectorAll('.meal-text')
//     mealTexts.forEach(text => {
//       (text as HTMLElement).style.cssText = `
//         font-size: 7px;
//         line-height: 1.2;
//       `
//     })
    
//     const tip = cellEl.querySelector('.cell-tip')
//     if (tip) {
//       (tip as HTMLElement).style.cssText = `
//         font-size: 6px;
//         padding: 2px 3px;
//         border-radius: 3px;
//       `
//     }
//   })
  
//   contentWrapper.appendChild(gridClone)
//   exportContainer.appendChild(contentWrapper)
//   document.body.appendChild(exportContainer)
  
//   try {
//     await nextTick()
    
//     const canvas = await html2canvas(exportContainer, {
//       scale: 2,
//       useCORS: true,
//       backgroundColor: '#ffffff',
//       logging: false,
//     })
    
//     const imgData = canvas.toDataURL('image/png', 1.0)
    
//     const pdf = new jsPDF({
//       orientation: 'landscape',
//       unit: 'mm',
//       format: 'a4'
//     })
    
//     // 按PDF页面比例缩放图片
//     const imgRatio = canvas.width / canvas.height
//     const pdfRatio = contentWidth / contentHeight
    
//     let drawWidth: number
//     let drawHeight: number
    
//     if (imgRatio > pdfRatio) {
//       // 图片更宽，按宽度适配
//       drawWidth = contentWidth
//       drawHeight = contentWidth / imgRatio
//     } else {
//       // 图片更高，按高度适配
//       drawHeight = contentHeight
//       drawWidth = contentHeight * imgRatio
//     }
    
//     // 居中放置
//     const x = margin + (contentWidth - drawWidth) / 2
//     const y = margin + (contentHeight - drawHeight) / 2
    
//     pdf.addImage(imgData, 'PNG', x, y, drawWidth, drawHeight)
//     pdf.save(`乐乐的辅食日记_${currentMonthTitle.value}.pdf`)
//   } catch (error) {
//     console.error('导出PDF失败:', error)
//     alert('导出PDF失败，请重试')
//   } finally {
//     document.body.removeChild(exportContainer)
//   }
// }
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