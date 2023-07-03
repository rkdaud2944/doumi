<template>
  <FullCalendar :options="calendarOptions"  class="calendar"/>
  <!-- 모달창 -->
  <div v-if="showModal" class="modal">
      <div class="modal-content">
          <div class="modal-header">
              <h4>신청</h4>
              <span @click="showModal = false" class="close">&times;</span>
          </div>
          <div class="modal-body">
            <h3>날짜정해주세요.</h3>
            <input type="radio" id="one-day" value="one" v-model="checkedValue" @change="onCheckboxChange">
            <label for="one-day">하루에 다 오기</label>

            <input type="radio" id="two-day" value="two" v-model="checkedValue" @change="onCheckboxChange">
            <label for="two-day">2일에 걸쳐서</label>

            <div v-if="checkedValue === 'one'">
              <h3>신청하실 날짜입니다.</h3>
              <div style="margin-bottom: 4%;">날짜 : {{selectedDate}}</div>
                <div>
                  <form>
                    <h3>신청하실 시간대를 선택해주세요.</h3>
                    <input type="radio" name="1" value="a"> 09:00 ~ 11:00<br>
                    <input type="radio" name="2" value="b"> 11:00 ~ 13:00<br>
                    <input type="radio" name="3" value="c"> 14:00 ~ 16:00<br>
                    <input type="radio" name="4" value="d"> 16:00 ~ 18:00<br>
                    <input type="radio" name="5" value="e"> 18:00 ~ 20:00<br>
                    <br>
                  </form>
                </div>
            </div>
            <div v-if="checkedValue === 'two'">
              <div style="display: flex; justify-content: space-between;">
                <div>
                  <h3>첫번째 신청하실 날짜입니다.</h3>
                  <div style="margin-bottom: 10%;">날짜 : {{selectedDate}}</div>
                  <div>
                    <form>
                      <h3>첫번째 신청하실 시간대를 선택해주세요.</h3>
                      <input type="radio" name="1" value="a"> 09:00 ~ 11:00<br>
                      <input type="radio" name="2" value="b"> 11:00 ~ 13:00<br>
                      <input type="radio" name="3" value="c"> 14:00 ~ 16:00<br>
                      <input type="radio" name="4" value="d"> 16:00 ~ 18:00<br>
                      <input type="radio" name="5" value="e"> 18:00 ~ 20:00<br>
                      <br>
                    </form>
                  </div>
                </div>
                <div style="margin-right: 10%;">
                  <h3>두번째 신청하실 날짜를 선택해주세요.</h3>
                  <input type="date" id="newDate" v-model="newDate" :min="minDate" @change="updateMinDate">
                  <div>
                    <form>
                      <h3>두번째 신청하실 시간대를 선택해주세요.</h3>
                      <input type="radio" name="1" value="a"> 09:00 ~ 11:00<br>
                      <input type="radio" name="2" value="b"> 11:00 ~ 13:00<br>
                      <input type="radio" name="3" value="c"> 14:00 ~ 16:00<br>
                      <input type="radio" name="4" value="d"> 16:00 ~ 18:00<br>
                      <input type="radio" name="5" value="e"> 18:00 ~ 20:00<br>
                      <br>
                    </form>
                  </div>
                </div>
              </div>
            </div>
          </div>
          <div class="modal-footer">
              <button @click="showModal = false">Close</button>
          </div>
      </div>
  </div>
</template>

<script>
// import { ref, watch } from 'vue'
import FullCalendar from '@fullcalendar/vue3'
import dayGridPlugin from '@fullcalendar/daygrid'
import interactionPlugin from '@fullcalendar/interaction'
import axios from 'axios';
import moment from 'moment';


export default {
  watch: {
    showModal(value) {
      if (value === true) {
        this.updateMinDate();
      } else {
        this.minDate = null;
      }
    },
  },
  
  data() {
    return {
      data: []
    }
  },
  
  components: {
    FullCalendar,
  },

  computed: {
    calendarOptions() {
      return {
        plugins: [dayGridPlugin, interactionPlugin],
        initialView: 'dayGridMonth',
        dateClick: this.handleDateClick,
        events: this.data,
      };
    },
  },

  beforeMount() {
    this.fetchData();
  },
  
  methods: {
    handleDateClick: function(arg) {
      alert('date click! ' + arg.dateStr)
    },

    fetchData(){
      axios
        .post('http://localhost:8081/reserve/api/v1/admin/findAllLocalChildren?curriculumSn=12')
        .then(response => {
          var data = response.data.result.data

          const events = Object.entries(data).map(([date, value]) => {
            let startDate = moment(date, 'YYYY-MM-DD').format('YYYY-MM-DD')
            let event = {
              title: `Event: ${value}`,
              date: startDate
            }
            return event
          })
          this.data = events
        })
        .catch(error => {
          console.error(error)
        })
    },

    updateMinDate() {
      const selectedDate = new Date(this.selectedDate);
      const minDate = new Date(selectedDate.getTime() + 24 * 60 * 60 * 1000); // Add 1 day to the selected date
      this.minDate = minDate.toISOString().split("T")[0];
    },
    
  },

  // setup() {
    
  //   const showModal = ref(false);
  //   const selectedDate = ref(null);
  //   const checkedValue = ref("one");
  //   const resultEvent = ref(null);

  //   const calendarOptions = ref(null);
  //   // const calendarRef = ref(null);
    
  //   watch(showModal, (value) => {
  //     if (value === true) {
  //       checkedValue.value = "one";
  //     }
  //   }); 
    

    
  //   // watch(resultEvent, (value) => {
  //   //   const calendarEl = document.getElementById('existing-calendar');
  //   //   calendarRef.value = calendarEl;
      
  //   //   console.log("와치안에 밸류 ㅣ: "+value)
      
  //   //   const calendar = new FullCalendar(calendarEl, calendarOptions.value);
  //   //   calendar.render();
  //   // }); 

    
  //   watch(resultEvent, () => {
  //   }); 

  //   const renderCalendar = () => {
  //     console.log("순서 여기는 랜더링된 캘린더~")
  //     const calendarEl = document.getElementById('calendarID');
  //     // calendarEl.render();
  //     // console.log("aa")

      
  //     const calendar = new FullCalendar(calendarEl, {
  //       plugins: [dayGridPlugin, interactionPlugin],
  //       initialView: 'dayGridMonth',
  //       dateClick: handleDateClick,
  //       eventContent: customEventContent,
  //       events: originalRows.value , //resultEvent
  //       dayMaxEvents: 4,
        
  //       // dayCellContent: () => {
  //       //   return {html: `<div>Test</div>`};
  //       // },

  //       dayCellContent: (arg) => {
  //         console.log("resultEvent.value : "+JSON.stringify(resultEvent.value))
  //         console.log("calendarEvents : "+JSON.stringify(calendarEvents.value))
  //         console.log("순서:" + 2)
  //         const date = arg.date;

  //         const holidays = [
  //           { month: 0, day: 1 },  
  //           { month: 0, day: 21 }, 
  //           { month: 0, day: 22 }, 
  //           { month: 0, day: 23 },  
  //           { month: 0, day: 24 },  
  //           { month: 2, day: 1 }, 
  //           { month: 4, day: 1 }, 
  //           { month: 4, day: 5 }, 
  //           { month: 4, day: 27 },
  //           { month: 5, day: 6 },  
  //           { month: 7, day: 15 },  
  //           { month: 8, day: 28 },  
  //           { month: 8, day: 29 },
  //           { month: 8, day: 30 },
  //           { month: 9, day: 3 },
  //           { month: 9, day: 9 },
  //           { month: 11, day: 25 },
  //         ];

  //         const isHoliday = holidays.some(
  //           holiday => date.getFullYear() === 2023 && date.getMonth() === holiday.month && date.getDate() === holiday.day
  //         );

  //         const isSunday = date.getDay() === 0;
  //         if (isHoliday || isSunday) {
  //           return {html: `<div style="color: red;">${date.getDate()}</div>`};
  //         } else {
  //           return {html: `<div>${date.getDate()}</div>`};
  //         }
  //       },
  //     });
  //     console.log("순서 123123")
  //     calendar.render();
  //   }
    

  //   const fetchData = () => {
  //     axios
  //       .post('http://localhost:8081/reserve/api/v1/admin/findAllLocalChildren?curriculumSn=12')
  //       .then(response => {
  //         var data = response.data.result.data

  //         const events = Object.entries(data).map(([date, value]) => {
  //           let startDate = moment(date, 'YYYY-MM-DD').format('YYYY-MM-DD')
  //           let event = {
  //             title: `Event: ${value}`,
  //             date: startDate,
  //             test: null, // Set the appropriate value for this property
  //             test2: null // Set the appropriate value for this property
  //           }
  //           // console.log("이벤트즈 : " +JSON.stringify(event))
  //           return event
  //         })
  //         resultEvent.value = events
  //         console.log("순서 :  " +1)
  //         // console.log('resultEvent.value 2  '+JSON.stringify(resultEvent.value))
  //         // console.log(rows.value)
          
  //         renderCalendar();
          


  //       })
  //       .catch(error => {
  //         console.error(error)
  //       })
  //   }

  //   // calendarOptions.value = {
  //   //   plugins: [dayGridPlugin, interactionPlugin],
  //   //   initialView: 'dayGridMonth',
  //   //   dateClick: handleDateClick,
  //   //   eventContent: customEventContent,
  //   //   events: resultEvent.value ,
  //   //   dayMaxEvents: 4,

  //   //   dayCellContent: (arg) => {
  //   //     console.log("resultEvent.value : "+JSON.stringify(resultEvent.value))
  //   //     console.log("calendarEvents : "+JSON.stringify(calendarEvents.value))
  //   //     console.log("순서:" + 2)
  //   //     const date = arg.date;

  //   //     const holidays = [
  //   //       { month: 0, day: 1 },  
  //   //       { month: 0, day: 21 }, 
  //   //       { month: 0, day: 22 }, 
  //   //       { month: 0, day: 23 },  
  //   //       { month: 0, day: 24 },  
  //   //       { month: 2, day: 1 }, 
  //   //       { month: 4, day: 1 }, 
  //   //       { month: 4, day: 5 }, 
  //   //       { month: 4, day: 27 },
  //   //       { month: 5, day: 6 },  
  //   //       { month: 7, day: 15 },  
  //   //       { month: 8, day: 28 },  
  //   //       { month: 8, day: 29 },
  //   //       { month: 8, day: 30 },
  //   //       { month: 9, day: 3 },
  //   //       { month: 9, day: 9 },
  //   //       { month: 11, day: 25 },
  //   //     ];

  //   //     const isHoliday = holidays.some(
  //   //       holiday => date.getFullYear() === 2023 && date.getMonth() === holiday.month && date.getDate() === holiday.day
  //   //     );

  //   //     const isSunday = date.getDay() === 0;
  //   //     if (isHoliday || isSunday) {
  //   //       return {html: `<div style="color: red;">${date.getDate()}</div>`};
  //   //     } else {
  //   //       return {html: `<div>${date.getDate()}</div>`};
  //   //     }
  //   //   },
  //   // }

    
  //   fetchData();

  //   const originalRows = ref([
  //     { id: 1, test: 0, test2: 99, createdAt: '2023-07-09'},
  //     { id: 2, test: 0, test2: 99, createdAt: '2023-07-08'},
  //     { id: 3, test: 0, test2: 99, createdAt: '2023-07-12'},
  //     { id: 4, test: 7, test2: 99, createdAt: '2023-07-13'},
  //     { id: 5, test: 1, test2: 99, createdAt: '2023-07-14'},
  //     { id: 6, test: 50, test2: 50, createdAt: '2023-07-15'},
  //     { id: 7, test: 16, test2: 50, createdAt: '2023-07-16'},
  //     { id: 8, test: 16, test2: 50, createdAt: '2023-07-19'},
  //   ]);
    
  //   const rows = ref([...originalRows.value]);

  //   const convertRowsToEvents = (rows) => {
  //     const eventsFromRows = rows.map(row => ({
  //       title: row.title,
  //       date: row.createdAt,
  //       test: row.test,
  //       test2: row.test2,
  //     }));

  //     const holidays = [
  //       { title: '신정', date: '2023-01-01' },
  //       { title: '설날 연휴', date: '2023-01-21' },
  //       { title: '설날', date: '2023-01-22' },
  //       { title: '설날 연휴', date: '2023-01-23' },
  //       { title: '설날 대체공휴일', date: '2023-01-24' },
  //       { title: '삼일절', date: '2023-03-01' },
  //       { title: '근로자의 날', date: '2023-05-01' },
  //       { title: '어린이 날', date: '2023-05-05' },
  //       { title: '부처님오신날', date: '2023-05-27' },
  //       { title: '현충일', date: '2023-06-06' },
  //       { title: '광복절', date: '2023-08-15' },
  //       { title: '추석 연휴', date: '2023-09-28' },
  //       { title: '추석', date: '2023-09-29' },
  //       { title: '추석 연휴', date: '2023-09-30' },
  //       { title: '개천절', date: '2023-10-03' },
  //       { title: '한글날', date: '2023-10-09' },
  //       { title: '성탄절', date: '2023-12-25' }
  //     ];
  //     for (const holiday of holidays) {
  //       eventsFromRows.push({
  //         title: holiday.title,
  //         date: holiday.date,
  //         classNames: 'holiday-event',
  //         textColor: 'red' 
  //       });
  //     }
  //     return eventsFromRows;
  //   };

  //   const calendarEvents = ref(convertRowsToEvents(rows.value));
    

  //   watch(rows, (newRows) => {
  //     calendarEvents.value = convertRowsToEvents(newRows);
  //   });
  //   console.log(calendarEvents);

    

  //   function handleDateClick(arg) {
  //     selectedDate.value = arg.dateStr;
  //     showModal.value = true;
  //     const clickedDate = arg.dateStr;  
  //       rows.value = originalRows.value.filter(row => row.createdAt === clickedDate);
  //       window.scrollTo({
  //         top: document.body.scrollHeight,
  //         behavior: 'smooth'
  //     });
  //   }

  //   function customEventContent(arg) {
  //     const date = arg.event.start;

  //     const formattedDate = `${date.getFullYear()}-${String(date.getMonth() + 1).padStart(2, '0')}-${String(date.getDate()).padStart(2, '0')}`;

  //     const holidays = [
  //       '2023-01-01',
  //       '2023-01-21',
  //       '2023-01-22',
  //       '2023-01-23',
  //       '2023-01-24',
  //       '2023-03-01',
  //       '2023-05-01',
  //       '2023-05-05',
  //       '2023-05-27',
  //       '2023-06-06',
  //       '2023-08-15',
  //       '2023-09-28',
  //       '2023-09-29',
  //       '2023-09-30',
  //       '2023-10-03',
  //       '2023-10-09',
  //       '2023-12-25'
  //     ];

  //     const isHoliday = holidays.includes(formattedDate);
  //     if (isHoliday) {
  //       return { html: `<span class="event-title">${arg.event.title}</span>` };
  //     } else {
  //       const test = arg.event.extendedProps.test;
  //       const test2 = arg.event.extendedProps.test2;
  //       let content = '';

  //       if (test === test2) {
  //           content = `
  //               <div class="event-content" style="display: flex; justify-content: space-between;">
  //                   <span class="event-title" style=" margin-right: auto; margin-left: auto; text-align:center;">마감</span>
  //               </div>`;
  //       } else {
  //           content = `
  //               <div class="event-content" style="display: flex; justify-content: space-between;">
  //                 <span class="event-ratio" style="margin-right: auto; margin-left: auto; text-align: center;">${(test2 - test) || ''}
  //                   <span style="margin-left:2px;">자리남음</span>
  //                 </span>
  //               </div>`;
  //       }
  //       return { html: content };
  //     }
  //   }
  //   return {
  //           rows,
  //           calendarOptions,
  //           handleDateClick,
  //           customEventContent,
  //           calendarEvents,
  //           showModal,
  //           selectedDate,
  //           checkedValue,
  //           fetchData,
  //           resultEvent
  //   }
  // }



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
.modal {
  display: block;
  position: fixed;
  top: 0;
  left: 0;
  width: 100%;
  height: 100%;
  background-color: rgba(0, 0, 0, 0.5);
}
.modal-content {
  background-color: #fefefe;
  margin: 15% auto;
  padding: 20px;
  border: 1px solid #888;
  width: 80%;
  bottom:80px;
}
.modal-header{
  padding: 10px;
  text-align: center;
}
.modal-footer {
  padding: 10px;
  text-align: right;
}
#two-day{
  margin-left: 5%;
}
.close {
  position: absolute;
  right: 15px;
  top: 10px;
  font-size: 28px;
  font-weight: bold;
  color: #000;
}
.close:hover,
.close:focus {
  color: #000;
  text-decoration: none;
  cursor: pointer;
}

</style>