<script setup lang="ts">
class CalendarCell {
  day: Readonly<number>;
  isCurrent: Readonly<boolean>;

  constructor(day: number, isCurrent: boolean) {
    this.day = day;
    this.isCurrent = isCurrent;
  }
}

const calendar = new Array<CalendarCell>(42);
let calendarIndex = 0;
const prevLastDate = new Date();
prevLastDate.setDate(0);
const prevLastDayOfWeek = prevLastDate.getDay();
// 先月
for (let index = prevLastDayOfWeek; index >= 0; index--) {
  const day = prevLastDate.getDate() - index;
  calendar[calendarIndex] = new CalendarCell(day, false);
  calendarIndex++;
}

// 今月
const currentLastDate = new Date();
currentLastDate.setMonth(currentLastDate.getMonth() + 1);
currentLastDate.setDate(0);
for (let day = 1; day <= currentLastDate.getDate(); day++) {
  calendar[calendarIndex] = new CalendarCell(day, true);
  calendarIndex++;
}
// 来月
const remainingDate = calendar.length - calendarIndex;
for (let day = 1; day <= remainingDate; day++) {
  calendar[calendarIndex] = new CalendarCell(day, false);
  calendarIndex++;
}
console.log(calendar);
</script>

<template>
  <main>
    <table>
      <thead class="day-of-week">
        <tr>
          <td class="sunday">日</td>
          <td>月</td>
          <td>火</td>
          <td>水</td>
          <td>木</td>
          <td>金</td>
          <td class="saturday">土</td>
        </tr>
      </thead>
      <tbody>
        <tr v-for="y in 6" :key="y">
          <td
            v-for="x in 7"
            :key="x"
            class="calendar-cell"
            :class="{
              'is-current-day': calendar[x + 7 * y - 8].isCurrent,
              saturday: x == 7,
              sunday: x == 1,
            }"
          >
            <span>{{ calendar[x + 7 * y - 8].day }}</span>
          </td>
        </tr>
      </tbody>
    </table>
  </main>
</template>

<style lang="scss">
main {
  height: 100%;
  color: #191920;
  table {
    width: 100%;
    height: 100%;
    padding: 0px;
    border-collapse: collapse;
    td {
      text-align: center;
      vertical-align: top;
      font-weight: bold;
      font-size: 2em;
      border: solid 2px #535360;
      background-color: #f2f2f6;
    }
    td.sunday {
      color: #cc1b1b;
    }
    td.saturday {
      color: #351bcc;
    }
    thead.day-of-week {
      tr {
        height: 20px;
      }
    }
    tbody {
      td:not(.is-current-day) {
        span {
          opacity: 0.3;
        }
      }
    }
  }
}
</style>
