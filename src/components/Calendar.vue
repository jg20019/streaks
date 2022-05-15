<script setup>

import { computed, reactive } from 'vue'

import { eachDayOfInterval, 
         endOfMonth,
         format,
         nextSaturday, 
         previousSunday, 
         isSameDay,
         isSunday, 
         isSaturday } from 'date-fns'

const state = reactive({
  year: 2022, 
  month: 4,
  tallies: new Set() // list of dates the work was done.
})


const daysInCalendarView = computed(() => {

  const start = new Date(state.year, state.month, 1)
  const end = endOfMonth(start)

  const firstSunday = isSunday(start) ? start : previousSunday(start)
  const lastSaturday = isSaturday(end) ? end : nextSaturday(end)

  const dates = eachDayOfInterval({
    start: firstSunday,
    end: lastSaturday
  })

  return dates
})

const toggleDate = (date) => {
  const key = format(date, "yyyy-MM-dd")
  if (state.tallies.has(key)) {
    state.tallies.delete(key)
  } else {
    state.tallies.add(key)
  }

  saveState()
}

const inTally = (date) => {
  const key = format(date, "yyyy-MM-dd")
  return state.tallies.has(key)
}

const loadState = () => {
  const json = localStorage.getItem('streaks')
  if (json === null) return;

  const s = JSON.parse(localStorage.getItem('streaks'))
  state.year = s.year
  state.month = s.month
  state.tallies = new Set(s.tallies)
}

const saveState = () => {
  const tallies = []
  state.tallies.forEach(date => {
    tallies.push(date)
  })

  localStorage.setItem('streaks', JSON.stringify({
    year: state.year,
    month: state.month,
    tallies: tallies
  }))
}

loadState()

</script>

<template>
<div class="calendar-container"> 
  <h1> May 2022 </h1>
  <div class="days-of-week"> 
    <h2> SUN </h2>
    <h2> MON </h2>
    <h2> TUE </h2>
    <h2> WED </h2>
    <h2> THU </h2>
    <h2> FRI </h2>
    <h2> SAT </h2>
  </div>
  <div class="days">
    <p 
      v-for="date in daysInCalendarView"
      key="date"
      @click="() => toggleDate(date)"
      :class="{ clicked: inTally(date) }"
    >
      <label class="date-marker">
        {{ date.getDate () }}
      </label>
    </p>
  </div>
</div>
</template>

<style scoped>
  .calendar-container {
    max-width: 960px;
    margin: 10px auto;
    border: 5px solid #2c3e50;
    border-radius: 5px;
    padding: 20px;
  }

  .days-of-week {
    display: flex;
    justify-content: space-around;
  }

  .days {
    display: grid;
    grid-template-columns: repeat(7, 1fr);
    grid-gap: 0px;
    gap: 0px;
  }

  .days p {
    margin: 0;
    align-self: stretch;
    height: 80px;
    position: relative;
  }

  .days p:hover {
    background-color: #2c3e50;
    color: white;
  }

  .days .date-marker {
    position: absolute;
    top: 10px;
    right: 10px;
  }
  
  .clicked {
    color: white;
    background-color: #2c3e50;
    font-size: 18px;
  }

</style>
