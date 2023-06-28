<template>
  <div>
    <FullCalendar :options="calendarOptions" class="calendar"/>
    <br>
    <button @click="showAll" class="show-all-btn">모든 데이터 보기</button>
    <vue-good-table :columns="columns" :rows="rows" :search-options="{enabled: true}" @row-click="onRowClick" class="calendar">
      <template #table-row="{row, column}">
        <template v-if="column.label === '신청하기'">
          <span v-if="row.test === row.test2">-</span>
          <button v-else @click="yourButtonClickHandler(row)">신청하기</button>
        </template>
      </template>

    </vue-good-table>
    <div class="modal" v-if="showModal">
      <div class="modal-content">
        <span class="close" @click="closeModal">&times;</span>
        <p>제목: {{ selectedRow.title }}</p>
        <p>날짜: {{ selectedRow.createdAt }}</p>
        <p>신청현황: {{ selectedRow.test }}/{{ selectedRow.test2 }}</p>
      </div>
    </div>
  </div>
</template>



<script>
import { ref, watch  } from 'vue'
import FullCalendar from '@fullcalendar/vue3'
import dayGridPlugin from '@fullcalendar/daygrid'
import interactionPlugin from '@fullcalendar/interaction'
import 'vue-good-table-next/dist/vue-good-table-next.css'
import { VueGoodTable } from 'vue-good-table-next';

export default {

  data() {
    return {
      selectedRow: null,
      showModal: false,
    };
  },

  methods: {
    onRowClick(params) {
      this.selectedRow = params.row;
      this.showModal = true;
    },
    closeModal() {
      this.showModal = false;
    },
    yourButtonClickHandler() {
      console.log("버튼 눌림");
    },
  },

  components: {
    FullCalendar,
    VueGoodTable,
  },

  setup() {
    function showAll() {
      rows.value = [...originalRows.value];
    }

    const columns = ref([
      {
        label: '날짜',
        field: 'createdAt',
        type: 'date',
        dateInputFormat: 'yyyy-MM-dd',
        dateOutputFormat: 'yyyy-MM-dd',
        width: '15%',
        align: 'center'
      },
      {
        label: '제목',
        field: 'title',
        width: 'auto',
        align: 'left'
      },
      {
        label: '신청현황',
        field: 'test',
        width: '15%',
        align: 'center',
        formatFn: (value, row) => {
          if (row.test === row.test2) {
            return '마감';
          } else {
            return `${value}/${row.test2}`;
          }
        },
      },
      {
        label: '신청하기',
        field: 'button',
        width: '10%',
        align: 'center',
        slot: 'table-row'
      },
    ]);

    const originalRows = ref([
      { id: 1, title: "4차시 틴커캐드를 활용한 3D 모델링", test: 26, test2: 26, createdAt: '2023-06-13'},
      { id: 2, title: "3차시 메타버스 타고 떠나는 세계여행_휴대폰용", test: 25, test2: 25, createdAt: '2023-06-14'},
      { id: 3, title: "4차시 틴커캐드를 활용한 3D 모델링", test: 14, test2: 20, createdAt: '2023-06-12'},
      { id: 4, title: "4차시 메타버스 타고 떠나는 세계여행", test: 7, test2: 24, createdAt: '2023-06-13'},
      { id: 5, title: "4차시 인벤이의 하루", test: 1, test2: 18, createdAt: '2023-06-14'},
      { id: 6, title: "8차시 메타버스 타고 떠나는 세계여행", test: 50, test2: 50, createdAt: '2023-06-12'},
      { id: 7, title: "4차시 메타버스 타고 떠나는 세계여행", test: 16, test2: 50, createdAt: '2023-06-13'},
      { id: 8, title: "4차시 메타버스 타고 떠나는 세계여행", test: 16, test2: 50, createdAt: '2023-06-14'},
    ]);
    
    const rows = ref([...originalRows.value]);

    const convertRowsToEvents = (rows) => {
      const eventsFromRows = rows.map(row => ({
        title: row.title,
        date: row.createdAt,
        test: row.test,
        test2: row.test2,
      }));

      const holidays = [
        { title: '신정', date: '2023-01-01' },
        { title: '설날 연휴', date: '2023-01-21' },
        { title: '설날', date: '2023-01-22' },
        { title: '설날 연휴', date: '2023-01-23' },
        { title: '설날 대체공휴일', date: '2023-01-24' },
        { title: '삼일절', date: '2023-03-01' },
        { title: '근로자의 날', date: '2023-05-01' },
        { title: '어린이 날', date: '2023-05-05' },
        { title: '부처님오신날', date: '2023-05-27' },
        { title: '현충일', date: '2023-06-06' },
        { title: '광복절', date: '2023-08-15' },
        { title: '추석 연휴', date: '2023-09-28' },
        { title: '추석', date: '2023-09-29' },
        { title: '추석 연휴', date: '2023-09-30' },
        { title: '개천절', date: '2023-10-03' },
        { title: '한글날', date: '2023-10-09' },
        { title: '성탄절', date: '2023-12-25' }
      ];
        for (const holiday of holidays) {
          eventsFromRows.push({
            title: holiday.title,
            date: holiday.date,
            classNames: 'holiday-event',
            textColor: 'red' 
          });
        }
      return eventsFromRows;
    };

    const calendarEvents = ref(convertRowsToEvents(rows.value));

    watch(rows, (newRows) => {
      calendarEvents.value = convertRowsToEvents(newRows);
    });

    const calendarOptions = {
      plugins: [dayGridPlugin, interactionPlugin],
      initialView: 'dayGridMonth',
      dateClick: handleDateClick,
      eventContent: customEventContent,
      events: calendarEvents.value ,
      dayMaxEvents: 4,

      dayCellContent: (arg) => {
        const date = arg.date;

        const holidays = [
          { month: 0, day: 1 },  
          { month: 0, day: 21 }, 
          { month: 0, day: 22 }, 
          { month: 0, day: 23 },  
          { month: 0, day: 24 },  
          { month: 2, day: 1 }, 
          { month: 4, day: 1 }, 
          { month: 4, day: 5 }, 
          { month: 4, day: 27 },
          { month: 5, day: 6 },  
          { month: 7, day: 15 },  
          { month: 8, day: 28 },  
          { month: 8, day: 29 },
          { month: 8, day: 30 },
          { month: 9, day: 3 },
          { month: 9, day: 9 },
          { month: 11, day: 25 },
        ];

        const isHoliday = holidays.some(
          holiday => date.getFullYear() === 2023 && date.getMonth() === holiday.month && date.getDate() === holiday.day
        );

        const isSunday = date.getDay() === 0;
        if (isHoliday || isSunday) {
          return {html: `<div style="color: red;">${date.getDate()}</div>`};
        } else {
          return {html: `<div>${date.getDate()}</div>`};
        }
      },
    }

    function handleDateClick(arg) {
      const clickedDate = arg.dateStr;  
        rows.value = originalRows.value.filter(row => row.createdAt === clickedDate);
        window.scrollTo({
          top: document.body.scrollHeight,
          behavior: 'smooth'
      });
    }

    function customEventContent(arg) {
      const date = arg.event.start;

      const formattedDate = `${date.getFullYear()}-${String(date.getMonth() + 1).padStart(2, '0')}-${String(date.getDate()).padStart(2, '0')}`;

      const holidays = [
        '2023-01-01',
        '2023-01-21',
        '2023-01-22',
        '2023-01-23',
        '2023-01-24',
        '2023-03-01',
        '2023-05-01',
        '2023-05-05',
        '2023-05-27',
        '2023-06-06',
        '2023-08-15',
        '2023-09-28',
        '2023-09-29',
        '2023-09-30',
        '2023-10-03',
        '2023-10-09',
        '2023-12-25'
      ];

      const isHoliday = holidays.includes(formattedDate);
      if (isHoliday) {
        return { html: `<span class="event-title">${arg.event.title}</span>` };
      } else {
        const test = arg.event.extendedProps.test;
        const test2 = arg.event.extendedProps.test2;
        let content = '';

        if (test === test2) {
            content = `
                <div class="event-content" style="display: flex; justify-content: space-between;">
                    <span class="event-title" style="text-decoration: line-through; margin-left: 5px; text-overflow: "";">${arg.event.title}</span>
                </div>`;
        } else {
            content = `
                <div class="event-content" style="display: flex; justify-content: space-between;">
                    <span class="event-title" style="margin-left: 5px; white-space: nowrap; overflow: hidden; text-overflow: ellipsis;">${arg.event.title}</span>
                    <span class="event-ratio" style="margin-right: 5px;">(${test ? test : ''}${test2 ? '/' + test2 : ''})</span>
                </div>`;
        }
        return { html: content };
      }
    }
    return {
      columns,
      rows,
      calendarOptions,
      handleDateClick,
      customEventContent,
      calendarEvents,
      showAll
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
.vgt-left-align {
  text-align: center ;
}
.vgt-right-align {
  text-align: center ;
}
.fc-col-header-cell:nth-of-type(1) .fc-col-header-cell-cushion {
  color: red;
}
.holiday-event{
  background-color: rgba(255,255,255,0.0) !important;
  border: none !important;
  color: red !important;
  bottom : 25px;
  text-align: right !important;
  margin-right: 5px !important;
}
.fc-daygrid-day-top {
   flex-direction: row !important;
}
.fc-event-main {
  text-align: left;
}
.holiday-event .fc-event-main{
  text-align: right;
}
.fc-daygrid-day-frame {
  height: 100px;
  overflow: hidden;
}
.show-all-btn {
  display: block;
  margin: 0 auto 10px;
  padding: 10px 20px;
  background-color: #4caf50;
  color: white;
  border: none;
  border-radius: 5px;
  cursor: pointer;
}
.show-all-btn:hover {
  background-color: #45a049;
}
.modal {
  display: block;
  position: fixed;
  z-index: 1;
  padding-top: 100px;
  left: 0;
  top: 0;
  width: 100%;
  height: 100%;
  overflow: auto;
  background-color: rgba(0, 0, 0, 0.4);
}

.modal-content {
  background-color: #fefefe;
  margin: auto;
  padding: 20px;
  border: 1px solid #888;
  width: 80%;
}

.close {
  color: #aaaaaa;
  float: right;
  font-size: 28px;
  font-weight: bold;
}

.close:hover,
.close:focus {
  color: #000;
  text-decoration: none;
  cursor: pointer;
}
</style>
