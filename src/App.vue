<script setup lang="ts">
const calendar = new Array<number>(42);
let calendarIndex = 0;
const prevLastDate = new Date();
prevLastDate.setDate(0);
const prevLastDayOfWeek = prevLastDate.getDay();
// 先月
for (let index = prevLastDayOfWeek; index >= 0; index--) {
  calendar[calendarIndex] = prevLastDate.getDate() - index;
  calendarIndex++;
}

// 今月
const currentLastDate = new Date();
currentLastDate.setMonth(currentLastDate.getMonth() + 1);
currentLastDate.setDate(0);
for (let index = 1; index <= currentLastDate.getDate(); index++) {
  calendar[calendarIndex] = index;
  calendarIndex++;
}
// 来月
const remainingDate = calendar.length - calendarIndex;
for (let index = 1; index <= remainingDate; index++) {
  calendar[calendarIndex] = index;
  calendarIndex++;
}
console.log(calendar);
</script>

<template>
  <main>
    <table>
      <thead class="day-of-week">
        <tr>
          <td>日</td>
          <td>月</td>
          <td>火</td>
          <td>水</td>
          <td>木</td>
          <td>金</td>
          <td>土</td>
        </tr>
      </thead>
      <tbody>
        <tr v-for="y in 6" :key="y">
          <td v-for="x in 7" :key="x" class="calendar-cell">
            {{ calendar[x + 7 * y - 8] }}
          </td>
        </tr>
      </tbody>
    </table>
  </main>
</template>

<style lang="scss">
main {
  height: 100%;
  table {
    width: 100%;
    height: 100%;
    padding: 0px;
    border-collapse: collapse;
  }

  td {
    text-align: center;
    vertical-align: top;
    font-weight: bold;
    font-size: 2em;
    border: solid 2px #535360;
  }

  thead.day-of-week {
    tr {
      height: 20px;
    }
  }
}
</style>
