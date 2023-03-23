<template>
  <div class="calendar">
  <div>
    <p class="calendar__time">{{ 'London time: ' + londonTime }}</p>

    <img src="@/assets/forCalendar.png" class="calendar__image" />
  </div>

    <div class="calendar__main">
      <div class="calendar__main__header content">
        <h2>{{ monthYear + ' ' + 'р.' }}</h2>

        <div class="calendar__main__buttons">
          <button @click="previousMonth" class="calendar__main__buttons--left button is-warning">
            &lt;
          </button>

          <button @click="nextMonth" class="calendar__main__buttons--right button is-warning">
            &gt;
          </button>
        </div>
      </div>

      <table class="calendar__table">
        <thead>
          <tr class="calendar__table__title">
            <th v-for="(name, index) in daysNames" :key="index">{{ name }}</th>
          </tr>
        </thead>
        <tbody>
          <tr v-for="(week, index) in calendar" :key="index" class="calendar__table__title">
            <td
              v-for="day in week"
              :key="day.date"
              :class="{
                'other-month': day.otherMonth,
                'calendar-day--today': today.getDate() === day.date && today.getMonth() === month,
                'has-events': day.hasEvents
              }"
              @click="showEvents(day)"
            >
              {{ day.date }}
            </td>
          </tr>
        </tbody>
      </table>
      <div v-for="event in events" :key="event.start">
        <h2 class="calendar__event">{{ event.title }}</h2>
        <p class="calendar__event__time">{{ event.start.toLocaleString('en-GB', { timeZone: 'Europe/Kyiv', day: '2-digit', month: '2-digit', year: 'numeric', hour: '2-digit', minute: '2-digit'}).replace(/\//g, '.') }} - {{ event.end.toLocaleString('en-GB', { timeZone: 'Europe/Kyiv', hour: '2-digit', minute: '2-digit'}) + ' ' + 'Часовий пояс (Europe/Kyiv)' }}</p>
      </div>
    </div>
  </div>
</template>

<script>
import moment from 'moment-timezone';

export default {
  data() {
    return {
      today: new Date(),
      month: new Date().getMonth(),
      year: new Date().getFullYear(),
      daysInMonth: null,
      firstDayOfMonth: null,
      lastDayOfMonth: null,
      calendar: [],
      daysNames: ['Пн', 'Вт', 'Ср', 'Чт', 'Пт', 'Сб', 'Нд'],
      events: [
        {
          title: 'Подія: запланувати співбесіду з розробником цього календаря💡',
          start: new Date('2023-03-26T14:00:00+02:00'),
          end: new Date('2023-03-26T15:00:00+02:00')
        }
      ],
      londonTime: null,
    }
  },
  computed: {
    monthYear() {
      const monthNames = [
        'Січень',
        'Лютий',
        'Березень',
        'Квітень',
        'Травень',
        'Червень',
        'Липень',
        'Серпень',
        'Вересень',
        'Жовтень',
        'Листопад',
        'Грудень'
      ]
      return `${monthNames[this.month]} ${this.year}`
    }
  },
  mounted() {
    this.createCalendar();
    this.getLondonTime();
  },
  methods: {
    createCalendar() {
      this.daysInMonth = new Date(this.year, this.month + 1, 0).getDate()
      this.firstDayOfMonth = new Date(this.year, this.month, 0).getDay()
      this.lastDayOfMonth = new Date(this.year, this.month + 1, 0).getDay()

      let date = 1
      let previousMonthDays = new Date(this.year, this.month, 0).getDate()
      let nextMonthDays = 1

      for (let i = 0; i < 6; i++) {
        let week = []

        for (let j = 0; j < 7; j++) {
          if (i === 0 && j < this.firstDayOfMonth) {
            week.push({ date: previousMonthDays - this.firstDayOfMonth + j + 1, otherMonth: true })
          } else if (date > this.daysInMonth) {
            week.push({ date: nextMonthDays++, otherMonth: true })
          } else {
            week.push({
              date: date++,
              otherMonth: false,
              hasEvents: this.events.some(
                (event) =>
                  event.start.getDate() === date - 1 && event.start.getMonth() === this.month
              )
            })
          }
        }

        this.calendar.push(week)
      }
    },
    previousMonth() {
      if (this.month === 0) {
        this.year--
        this.month = 11
      } else {
        this.month--
      }

      this.calendar = []
      this.createCalendar()
    },
    nextMonth() {
      if (this.month === 11) {
        this.year++
        this.month = 0
      } else {
        this.month++
      }

      this.calendar = []
      this.createCalendar()
    },
    showEvents(day) {
      const eventsOnDay = this.events.filter(
        (event) => event.start.getDate() === day.date && event.start.getMonth() === this.month
      )
      console.log(eventsOnDay)
    },
    getLondonTime() {
      this.londonTime = moment().tz('Europe/London').format('h:mm:ss a');
      setTimeout(() => {
        this.getLondonTime();
      }, 1000);
    }
  }
}
</script>