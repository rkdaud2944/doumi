<template>
  <div>
    <h1>캘린더시작</h1>
    <FullCalendar :options="calendarOptions" class="calendar"/>

    <h2>캘린더 끝<br>게시판시작</h2>
    <vue-good-table :columns="columns" :rows="rows" :search-options="{enabled: true}" class="calendar"/>
    <h3>게시판끝</h3>
  </div>
</template>

<script>
import { ref } from 'vue'
import FullCalendar from '@fullcalendar/vue3'
import dayGridPlugin from '@fullcalendar/daygrid'
import interactionPlugin from '@fullcalendar/interaction'
// import VueGoodTable from 'vue-good-table'
// import 'vue-good-table/dist/vue-good-table.css'

import 'vue-good-table-next/dist/vue-good-table-next.css'
import { VueGoodTable } from 'vue-good-table-next';

export default {
  methods: {
    handleButtonClick() {
      // 버튼 클릭 처리 로직
    },
  },
  components: {
    FullCalendar,
    VueGoodTable
  },
  setup() {
    
    const columns = ref([
      {
        label: '날짜',
        field: 'createdAt',
        type: 'date',
        dateInputFormat: 'yyyy-MM-dd',
        dateOutputFormat: 'yyyy-MM-dd',
      },
      {
        label: 'title',
        field: 'title'
      },
      {
        label: '현재인원/전체인원',
        field: 'test',
        formatFn: (value, row) => `${value}/${row.test2}`
      },

      
    ])

    const rows = ref([
      { id:1, title:"알버트 봇", test:26, test2: 26, createdAt: '1998-11-23'},
      { id:2, title:"어썸 봇",test:25, test2: 25, createdAt: '1998-11-23'},
      { id:3, title:"알버 봇",test:14, test2: 20, createdAt: '2018-10-30'},
      { id:4, title:"버트 봇",test:7, test2: 24, createdAt: '2021-10-11'},
      { id:5, title:"트 봇",test:0, test2: 18, createdAt: '2013-10-21'},
      { id:6, title:"어썸카트 봇",test:16, test2: 50, createdAt: '2014-10-31'},
    ])

    const calendarOptions = {
      plugins: [dayGridPlugin, interactionPlugin],
      initialView: 'dayGridMonth',
      dateClick: handleDateClick,
      eventContent: customEventContent,
      // dayRender: customDayRender,
      events: [
        { title: '현충일', date: '2023-06-06', classNames: 'holiday-event' },
        { title: '알버트 로봇과 함께하는 메타버스 체험', date: '2023-06-08' },
        { title: '어썸봇 2', date: '2023-06-08' },
      ]
    }

    function handleDateClick(arg) {
      alert('date click! ' + arg.dateStr)
    }

    function customEventContent(arg) {
      return { html: `<span class="event-title">${arg.event.title}</span>` }
    }

    // function customDayRender(arg) {
    //   if (arg.date.getDay() === 0) {
    //     arg.el.style.backgroundColor = 'red'
    //   }
    // } 휴일 빨간색 하려하는데 안됨

    return {
      columns,
      rows,
      calendarOptions,
      handleDateClick,
      customEventContent,
      // customDayRender
    }
  }
}
</script>

<style>
.calendar {
  margin-right: auto;
  margin-left: auto;
  text-align: center;
  width: 80%;
  margin-top: 10px;
}
/* .holiday-event .event-title {
  background-color: red;
  font-weight: bold;
} */
</style>
