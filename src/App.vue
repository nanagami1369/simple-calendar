<script setup lang="ts">
const createCalendarDate = (year: number, month: number): CalendarCell[] => {
  const calendar = new Array<CalendarCell>(42);
  let calendarIndex = 0;
  const prevLastDate = new Date(year, month, 1);
  prevLastDate.setDate(0);
  const prevLastDayOfWeek = prevLastDate.getDay();
  // 先月
  for (let index = prevLastDayOfWeek; index >= 0; index--) {
    const day = prevLastDate.getDate() - index;
    calendar[calendarIndex] = new CalendarCell(day, false);
    calendarIndex++;
  }

  // 今月
  const currentLastDate = new Date(year, month, 1);
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
  return calendar;
};

class CalendarCell {
  day: Readonly<number>;
  isCurrent: Readonly<boolean>;

  constructor(day: number, isCurrent: boolean) {
    this.day = day;
    this.isCurrent = isCurrent;
  }
}
// url queryから表示する年月の情報を取得
let query: string;
if (location.search.startsWith("?date=")) {
  query = location.search.substring("?date=".length).split("&")[0];
} else {
  query = "";
}
// 表示するカレンダーの年と月
let current: Date;
if (/^[0-9]{4}-[0-9]{1,2}$/.test(query)) {
  const YearAndMonthPair = query.split("-");
  const year = Number(YearAndMonthPair[0]);
  const month = Number(YearAndMonthPair[1]);
  // 正規表現で数字あることを判定済みなため、文字列が数字以外の可能性を考慮しない
  // month(月)は0始まりなので - 1
  current = new Date(year, month - 1);
} else {
  current = new Date();
}
const calendar = createCalendarDate(current.getFullYear(), current.getMonth());
const prevMonth = (): void => {
  const prevMonth = new Date(current.getFullYear(), current.getMonth());
  prevMonth.setMonth(prevMonth.getMonth() - 1);
  location.search = `?date=${prevMonth.getFullYear()}-${
    prevMonth.getMonth() + 1
  }`;
};
const nextMonth = (): void => {
  const nextMonth = new Date(current.getFullYear(), current.getMonth());
  nextMonth.setMonth(nextMonth.getMonth() + 1);
  location.search = `?date=${nextMonth.getFullYear()}-${
    nextMonth.getMonth() + 1
  }`;
};
const nowMonth = (): void => {
  const now = new Date();
  location.search = `?date=${now.getFullYear()}-${now.getMonth() + 1}`;
};
</script>

<template>
  <main>
    <header class="calendar-header">
      <button @click="prevMonth" class="arrow-button">&lt;</button>
      <section class="current-date">
        {{ current.getFullYear() }}/{{ current.getMonth() + 1 }}
      </section>
      <button @click="nextMonth" class="arrow-button">&gt;</button>
      <button @click="nowMonth">現在</button>
    </header>
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
  header {
    display: flex;
    height: 40px;
    font-size: 2em;
    section.current-date {
      margin: auto 15px;
    }
    button {
      margin: 2px;
      width: 60px;
    }

    button.arrow-button {
      width: 40px;
    }
  }
  table {
    width: 100%;
    height: calc(100% - 40px);
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
