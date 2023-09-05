<template>
  <n-config-provider :locale="zhCN" :date-locale="dateZhCN">
    <n-card title="排班表">
      <n-date-picker
        type="date"
        v-model:value="timestamp"
        placeholder="选择最近一次日班时间"
        style="margin-bottom: 20px"
      />
      <n-calendar
        v-model:value="value"
        :is-date-disabled="isDateDisabled"
        @update:value="handleUpdateValue"
      >
        <template #default="{ year, month, date }">
          <!-- {{ year }}-{{ month }}-{{ date }} -->
          <div
            class="font-bold"
            :style="{
              backgroundColor: calcRestDay(year, month, date, firstWhiteShift) ? 'green' : ''
            }"
          >
            {{ schedule(year, month, date, firstWhiteShift) }}
          </div>
        </template>
      </n-calendar>
    </n-card>
  </n-config-provider>
</template>

<script lang="ts" setup>
import { ref, watch } from 'vue'
import { NCalendar, NDatePicker, NCard } from 'naive-ui'
import { NConfigProvider } from 'naive-ui'
import { zhCN, dateZhCN } from 'naive-ui'
import {
  isYesterday,
  addDays,
  parse,
  differenceInDays,
  getUnixTime,
  format
} from 'date-fns/esm'

const value = ref(addDays(Date.now(), 1).valueOf())

// 设置白班时间
const firstWhiteShift = ref(localStorage.getItem('date') || '2023-09-04')
const timestamp = ref(getUnixTime(parse(firstWhiteShift.value, 'yyyy-MM-dd', new Date())) * 1000)

watch(
  () => timestamp.value,
  () => {
    firstWhiteShift.value = format(new Date(timestamp.value), 'yyyy-MM-dd')
    localStorage.setItem('date', firstWhiteShift.value)
  }
)

function handleUpdateValue(
  _: number,
  { year, month, date }: { year: number; month: number; date: number }
) {
  console.log(`${year}-${month}-${date} ${schedule(year, month, date, firstWhiteShift.value)}`)
}

function isDateDisabled(timestamp: number) {
  if (isYesterday(timestamp)) {
    return true
  }
  return false
}


const calcDate = (year: number, month: number, date: number, firstWhiteShift: string) => {
  const firstWhiteShiftDate = parse(firstWhiteShift, 'yyyy-MM-dd', new Date())
  let _date = differenceInDays(new Date(year, month - 1, date), firstWhiteShiftDate)
  _date = _date < 0 ? Math.abs(_date % 4 + 4 ) : _date
  return _date
}

const calcRestDay = (year: number, month: number, date: number, firstWhiteShift: string) => {
  let _date = calcDate(year, month, date, firstWhiteShift)
  return calc(_date) === 2 || calc(_date) === 3
}

function calc(date: number) {
  return date % 4
}

function schedule(year: number, month: number, date: number, firstWhiteShift: string) {
  return formateSchedule(
    calcDate(year, month, date, firstWhiteShift)
  )
}

function formateSchedule(date: number) {
  const arrange = ['白班', '夜班', '休息', '休息']
  return `${arrange[calc(date)]}`
}
</script>
