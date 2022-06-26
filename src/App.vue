<script setup lang="ts">
import { onMounted } from "vue";

const fetchSyukujituData = async (): Promise<Map<string, string>> => {
  const response = await fetch("./syukujitsu.csv");
  const syukujituBuffer = await response.arrayBuffer();
  const syukujituMap = new Map();
  const syukujituData = new TextDecoder("shift-jis").decode(syukujituBuffer);
  syukujituData.split("\r\n").forEach((line) => {
    const dateAndSyukujituPair = line.split(",");
    syukujituMap.set(dateAndSyukujituPair[0], dateAndSyukujituPair[1]);
  });
  return syukujituMap;
};
const createCalendarDate = (year: number, month: number): CalendarCell[] => {
  const calendar = new Array<CalendarCell>(42);
  let calendarIndex = 0;
  const prevLastDate = new Date(year, month, 1);
  prevLastDate.setDate(0);
  const prevLastDayOfWeek = prevLastDate.getDay();
  // 先月
  for (let index = prevLastDayOfWeek; index >= 0; index--) {
    const day = prevLastDate.getDate() - index;
    calendar[calendarIndex] = new CalendarCell(
      prevLastDate.getFullYear(),
      prevLastDate.getMonth(),
      day,
      false
    );
    calendarIndex++;
  }

  // 今月
  const currentLastDate = new Date(year, month, 1);
  currentLastDate.setMonth(currentLastDate.getMonth() + 1);
  currentLastDate.setDate(0);
  for (let day = 1; day <= currentLastDate.getDate(); day++) {
    calendar[calendarIndex] = new CalendarCell(
      currentLastDate.getFullYear(),
      currentLastDate.getMonth(),
      day,
      true
    );
    calendarIndex++;
  }
  // 来月
  const nextDate = new Date(year, month, 1);
  nextDate.setMonth(nextDate.getMonth() + 1);
  const remainingDate = calendar.length - calendarIndex;
  for (let day = 1; day <= remainingDate; day++) {
    calendar[calendarIndex] = new CalendarCell(
      nextDate.getFullYear(),
      nextDate.getMonth() + 1,
      day,
      false
    );
    calendarIndex++;
  }
  return calendar;
};

class CalendarCell {
  year: Readonly<number>;
  month: Readonly<number>;
  day: Readonly<number>;
  isCurrent: Readonly<boolean>;

  constructor(year: number, month: number, day: number, isCurrent: boolean) {
    this.year = year;
    this.month = month;
    this.day = day;
    this.isCurrent = isCurrent;
  }

  display(): string {
    return `${this.year}/${this.month + 1}/${this.day}`;
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
onMounted(async () => {
  // 祝日情報がわかったら後からJSでつける
  await fetchSyukujituData().then((syukujituMap) => {
    const calendarCells = Array.from(
      document.querySelectorAll<HTMLElement>("table tbody td")
    );
    calendarCells
      .filter((cell) => syukujituMap.get(cell.dataset.date ?? ""))
      .map((cell) => {
        cell.classList.add("is-syukujitu");
        cell.insertAdjacentHTML(
          "beforeend",
          `<section>${
            syukujituMap.get(cell.dataset.date ?? "") ?? ""
          }</section>`
        );
      });
  });
});
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
            :data-date="calendar[x + 7 * y - 8].display()"
          >
            <section>{{ calendar[x + 7 * y - 8].day }}</section>
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
      font-weight: bold;
      font-size: 2em;
      border: solid 2px #535360;
      background-color: #f2f2f6;
      width: calc(100% / 7);
    }
    td.sunday {
      color: #cc1b1b;
    }
    td.is-syukujitu {
      color: #cc1b1b;
    }
    td.saturday {
      color: #351bcc;
    }
    thead.day-of-week {
      tr {
        height: 40px;
      }
    }
    tbody {
      td {
        height: calc(100% / 7);
        vertical-align: top;
      }
      td:not(.is-current-day) {
        span {
          opacity: 0.3;
        }
      }
    }
  }
}
</style>
