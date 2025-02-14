<script setup lang="ts">
import { ref, computed, onMounted } from 'vue'
import { format, startOfMonth, endOfMonth, eachDayOfInterval, startOfWeek, endOfWeek, startOfYear, endOfYear, addMonths, subMonths, addWeeks, subWeeks } from 'date-fns'
import { Dialog, DialogPanel, TransitionChild, TransitionRoot } from '@headlessui/vue'

interface WaterLog {
  date: string;
  pints: number;
}

type ViewType = 'day' | 'week' | 'month' | 'year'

const waterLogs = ref<WaterLog[]>([])
const currentPints = ref(0)
const isOpen = ref(false)
const selectedDate = ref<Date | null>(null)
const confirmationMessage = ref('')
const currentView = ref<ViewType>('year')
const currentDate = ref(new Date())

// Generate some random data for the year
onMounted(() => {
  // Clear existing water logs
  waterLogs.value = [] // Remove all random data

  // Optionally, you can keep this part if you want to generate new data later
  // const start = startOfYear(new Date())
  // const end = endOfYear(new Date())
  // const days = eachDayOfInterval({ start, end })
  
  // days.forEach(day => {
  //   if (Math.random() > 0.3) { // 70% chance of having a log
  //     waterLogs.value.push({
  //       date: format(day, 'yyyy-MM-dd'),
  //       pints: Math.floor(Math.random() * 10) + 1 // 1-10 pints
  //     })
  //   }
  // })
})

const addWaterLog = () => {
  if (!selectedDate.value) return
  
  const dateStr = format(selectedDate.value, 'yyyy-MM-dd')
  const existingLogIndex = waterLogs.value.findIndex(log => log.date === dateStr)
  
  if (existingLogIndex !== -1) {
    if (currentPints.value === 0) {
      waterLogs.value.splice(existingLogIndex, 1)
      confirmationMessage.value = 'Removed beer intake log!'
    } else {
      waterLogs.value[existingLogIndex].pints = currentPints.value
      confirmationMessage.value = 'Updated beer intake!'
    }
  } else {
    if (currentPints.value > 0) {
      waterLogs.value.push({
        date: dateStr,
        pints: currentPints.value
      })
      confirmationMessage.value = 'Added new beer intake log!'
    }
  }
  isOpen.value = false
}

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

const openDialog = (date: Date) => {
  selectedDate.value = date
  const dateStr = format(date, 'yyyy-MM-dd')
  const log = waterLogs.value.find(log => log.date === dateStr)
  currentPints.value = log ? log.pints : 0 // Default to 0 pints if no log exists
  isOpen.value = true
}

const calendarDays = computed(() => {
  switch (currentView.value) {
    case 'day':
      return [currentDate.value]
    case 'week':
      return eachDayOfInterval({
        start: startOfWeek(currentDate.value),
        end: endOfWeek(currentDate.value)
      })
    case 'month':
      return eachDayOfInterval({
        start: startOfMonth(currentDate.value),
        end: endOfMonth(currentDate.value)
      })
    case 'year':
      return eachDayOfInterval({
        start: startOfYear(currentDate.value),
        end: endOfYear(currentDate.value)
      })
  }
})

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
          v-for="view in ['day', 'week', 'month', 'year']"
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

      <div class="mt-6 text-sm text-gray-600 flex gap-4 mb-10 align-middle items-center w-full bg-gray-50 p-4">

       
        <div class="flex items-center gap-2">
          <div class="w-4 h-4 rounded bg-emerald-800"></div>
          <span>0 pints <span style="font-weight: bold;">: 😇 Sober</span></span>
        </div>
       
        <div class="flex items-center gap-2">
          <div class="w-4 h-4 rounded bg-yellow-400"></div>
          <span>1-3 pints <span style="font-weight: bold;">: 🙃 Okay</span></span>
        </div>
        <div class="flex items-center gap-2">
          <div class="w-4 h-4 rounded bg-orange-600"></div>
          <span>4-6 pints <span style="font-weight: bold;">: ☹️ Bad</span></span>
        </div>
        <div class="flex items-center gap-2">
          <div class="w-4 h-4 rounded bg-neutral-900"></div>
          <span>7+ pints <span style="font-weight: bold;">: 🥴 BlackOut</span></span>
        </div>


      </div>


      <template v-if="currentView === 'year'">
        <div class="grid grid-cols-3 gap-8">
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
                      @click="openDialog(date)"
                      :class="[
                        getDayColor(date),
                        'p-1 rounded transition-colors text-center text-sm hover:shadow-sm'
                      ]">
                {{ format(date, 'd') }}
              </button>
            </div>
          </div>
        </div>
      </template>
      
      <template v-else>
        <div class="grid grid-cols-7 gap-2">
          <div v-for="day in ['Sun', 'Mon', 'Tue', 'Wed', 'Thu', 'Fri', 'Sat']" 
               class="text-center font-medium text-gray-500 text-sm p-2">
            {{ day }}
          </div>
          <button v-for="date in calendarDays" 
                  :key="date.toISOString()"
                  @click="openDialog(date)"
                  :class="[
                    getDayColor(date),
                    'p-4 rounded-lg transition-colors text-center hover:shadow-md'
                  ]">
            {{ format(date, 'd') }}
          </button>
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

    <!-- Headless UI Dialog for adding/editing water intake -->
    <TransitionRoot appear :show="isOpen" as="template">
      <Dialog as="div" @close="isOpen = false" class="relative z-10">
        <TransitionChild
          as="template"
          enter="duration-300 ease-out"
          enter-from="opacity-0"
          enter-to="opacity-100"
          leave="duration-200 ease-in"
          leave-from="opacity-100"
          leave-to="opacity-0"
        >
          <div class="fixed inset-0 bg-black bg-opacity-25" />
        </TransitionChild>

        <div class="fixed inset-0 overflow-y-auto">
          <div class="flex min-h-full items-center justify-center p-4 text-center">
            <TransitionChild
              as="template"
              enter="duration-300 ease-out"
              enter-from="opacity-0 scale-95"
              enter-to="opacity-100 scale-100"
              leave="duration-200 ease-in"
              leave-from="opacity-100 scale-100"
              leave-to="opacity-0 scale-95"
            >
              <DialogPanel class="w-full max-w-md transform overflow-hidden rounded-2xl bg-white p-6 text-left align-middle shadow-xl transition-all">
                <h3 class="text-lg font-medium leading-6 text-gray-900 mb-4">
                  Beer Intake for {{ selectedDate ? format(selectedDate, 'MMMM d, yyyy') : '' }}
                </h3>
                
                <div class="mt-4">
                  <label class="block text-sm font-medium text-gray-700 mb-2">
                    Number of Pints
                  </label>
                  <input
                    type="number"
                    v-model="currentPints"
                    min="0"
                    class="w-full px-3 py-2 border rounded-md"
                    placeholder="Enter number of pints"
                  />
                  <p class="mt-2 text-sm text-gray-500">
                    Enter 0 to remove the log for this date
                  </p>
                </div>

                <div class="mt-6 flex gap-3">
                  <button
                    type="button"
                    class="inline-flex justify-center rounded-md border border-transparent bg-blue-500 px-4 py-2 text-sm font-medium text-white hover:bg-blue-600 focus:outline-none focus-visible:ring-2 focus-visible:ring-blue-500 focus-visible:ring-offset-2"
                    @click="addWaterLog"
                  >
                    Save
                  </button>
                  <button
                    type="button"
                    class="inline-flex justify-center rounded-md border border-gray-300 bg-white px-4 py-2 text-sm font-medium text-gray-700 hover:bg-gray-50 focus:outline-none focus-visible:ring-2 focus-visible:ring-blue-500 focus-visible:ring-offset-2"
                    @click="isOpen = false"
                  >
                    Cancel
                  </button>
                </div>
              </DialogPanel>
            </TransitionChild>
          </div>
        </div>
      </Dialog>
    </TransitionRoot>
  </div>
</template>