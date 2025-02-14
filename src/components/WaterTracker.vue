<script setup lang="ts">
import { ref, computed } from 'vue'
import { format, startOfMonth, endOfMonth, eachDayOfInterval, startOfWeek, startOfYear, addMonths, subMonths, addWeeks, subWeeks } from 'date-fns'


interface WaterLog {
  date: string;
  pints: number;
}

type ViewType = 'day' | 'week' | 'month' | 'year'

// Manually define the entries here
const waterLogs = ref<WaterLog[]>([
  { date: '2025-01-01', pints: 9 },
  { date: '2025-02-11', pints: 4 },
  { date: '2025-02-03', pints: 1 },

]);

const currentView = ref<ViewType>('year')
const currentDate = ref(new Date())

const getStatusColor = (pints: number) => {
  if (pints >= 7) return 'bg-slate-950 text-white'
  if (pints >= 4) return 'bg-orange-600 text-white'
  if (pints >= 1) return 'bg-yellow-400 text-black'
  return 'bg-emerald-800 text-white'
}

const getStatusText = (pints: number) => {
  if (pints >= 7) return 'Blackout'
  if (pints >= 4) return 'Bad'
  if (pints >= 1) return 'Okay'
  return 'Sober'
}

const getDayColor = (date: Date) => {
  const dateStr = format(date, 'yyyy-MM-dd')
  const log = waterLogs.value.find(log => log.date === dateStr)
  if (!log) return 'bg-emerald-700 text-white hover:bg-emerald-600' // Set default color for sober
  if (log.pints >= 7) return 'bg-slate-950 text-white hover:bg-slate-800'
  if (log.pints >= 4) return 'bg-orange-600 text-white hover:bg-orange-500'
  if (log.pints >= 1) return 'bg-yellow-400 text-black hover:bg-yellow-300'
  return 'bg-emerald-700 text-white hover:bg-emerald-600' // Sober
}




const monthsInYear = computed(() => {
  const months = []
  let currentMonth = startOfYear(currentDate.value)
  for (let i = 0; i < 12; i++) {
    months.push(addMonths(currentMonth, i))
  }
  return months
})

const navigatePrevious = () => {
  switch (currentView.value) {
    case 'day':
      currentDate.value = subWeeks(currentDate.value, 1)
      break
    case 'week':
      currentDate.value = subWeeks(currentDate.value, 1)
      break
    case 'month':
      currentDate.value = subMonths(currentDate.value, 1)
      break
    case 'year':
      currentDate.value = new Date(currentDate.value.getFullYear() - 1, 0, 1)
      break
  }
}

const navigateNext = () => {
  switch (currentView.value) {
    case 'day':
      currentDate.value = addWeeks(currentDate.value, 1)
      break
    case 'week':
      currentDate.value = addWeeks(currentDate.value, 1)
      break
    case 'month':
      currentDate.value = addMonths(currentDate.value, 1)
      break
    case 'year':
      currentDate.value = new Date(currentDate.value.getFullYear() + 1, 0, 1)
      break
  }
}

const viewTitle = computed(() => {
  switch (currentView.value) {
    case 'day':
      return format(currentDate.value, 'MMMM d, yyyy')
    case 'week':
      return `Week of ${format(startOfWeek(currentDate.value), 'MMMM d, yyyy')}`
    case 'month':
      return format(currentDate.value, 'MMMM yyyy')
    case 'year':
      return format(currentDate.value, 'yyyy')
  }
})

const sortedLogs = computed(() => {
  return [...waterLogs.value].sort((a, b) => 
    new Date(b.date).getTime() - new Date(a.date).getTime()
  )
})
</script>

<template>
  <div class="max-w-6xl mx-auto p-6">
    <h1 class="text-3xl font-bold mb-8 text-center">Kit's Pint Tracker🍺🍻🍻</h1>
    
    <!-- View Switcher -->
    <div class="flex justify-between items-center mb-6">
      <div class="flex gap-2">
        <button
          v-for="view in ['year']"
          :key="view"
          @click="currentView = view as ViewType"
          :class="[
            'px-4 py-2 rounded-md text-sm font-medium',
            currentView === view
              ? 'bg-blue-500 text-white'
              : 'bg-gray-100 text-gray-700 hover:bg-gray-200'
          ]"
        >
          {{ view.charAt(0).toUpperCase() + view.slice(1) }}
        </button>
      </div>
      
      <div class="flex items-center gap-4">
        <button
          @click="navigatePrevious"
          class="p-2 rounded-md hover:bg-gray-100"
        >
          ←
        </button>
        <h2 class="text-xl font-semibold">{{ viewTitle }}</h2>
        <button
          @click="navigateNext"
          class="p-2 rounded-md hover:bg-gray-100"
        >
          →
        </button>
      </div>
    </div>

    <!-- Calendar View -->
    <div class="bg-white rounded-lg shadow-md p-6 mb-8">

        <!-- Color Coding Reference -->
        <div class="mb-2 p-4 bg-gray-50 rounded-md">

<div class=" p-4 bg-gray-50 rounded-md">
<div class="flex flex-col sm:flex-row gap-6">
<div class="flex items-center">
<div class="w-4 h-4 rounded bg-emerald-700 mr-2"></div>
<span>0 pints: 😇 Sober</span>
</div>

<div class="flex items-center">
<div class="w-4 h-4 rounded bg-yellow-400 mr-2"></div>
<span>1-3 pints: 🙃 Good</span>
</div>

<div class="flex items-center">
<div class="w-4 h-4 rounded bg-orange-600 mr-2"></div>
<span>4-6 pints: ☹️ Bad</span>
</div>

<div class="flex items-center">
<div class="w-4 h-4 rounded bg-slate-950 mr-2"></div>
<span>7+ pints: 🥴 BlackOut</span>
</div>
</div>
</div>
</div>
      <template v-if="currentView === 'year'">
        <div class="grid grid-cols-1 sm:grid-cols-3 gap-4">
          <div v-for="month in monthsInYear" :key="format(month, 'MM')" class="space-y-2">
            <h3 class="font-semibold text-center">{{ format(month, 'MMMM') }}</h3>
            <div class="grid grid-cols-7 gap-1">
              <div v-for="day in ['S', 'M', 'T', 'W', 'T', 'F', 'S']" 
                   class="text-center text-xs font-medium text-gray-500 p-1">
                {{ day }}
              </div>
              <button v-for="date in eachDayOfInterval({
                start: startOfMonth(month),
                end: endOfMonth(month)
              })" 
                      :key="date.toISOString()"
                      :class="[
                        getDayColor(date),
                        'p-2 rounded transition-colors text-center text-sm hover:shadow-sm'
                      ]">
                {{ format(date, 'd') }}
              </button>
            </div>
          </div>
        </div>
      </template>
    </div>

    <!-- Recent History -->
    <div class="bg-white rounded-lg shadow-md p-6">
      <h2 class="text-xl font-semibold mb-4">Recent History</h2>
      <div class="space-y-4">
        <div v-for="log in sortedLogs.slice(0, 5)" :key="log.date" 
             class="flex items-center justify-between p-4 rounded-lg"
             :class="getStatusColor(log.pints)">
          <span class="font-medium">{{ format(new Date(log.date), 'MMM dd, yyyy') }}</span>
          <div class="flex items-center gap-4">
            <span>{{ log.pints }} pints</span>
            <span class="text-sm font-medium">{{ getStatusText(log.pints) }}</span>
          </div>
        </div>
      </div>
    </div>
  </div>
</template>