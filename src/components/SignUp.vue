<template>
  <FullCalendar :options="calendarOptions"  class="calendar" @eventClick="handleDateClick"/>
  <!-- 모달창 -->
  <div v-if="showModal" class="modal" style=" ">
      <div class="modal-content" style="width: 350px; top: 10px; height: 70%; overflow-y: auto; ">
          <div class="modal-header"><div style="font-weight: bold; text-align: left;">날짜 : {{selectedDate}} <div> 남은 {{getEventTitle()}}개 </div></div>
              <h4 style="font-family: 'Courier New', Courier, monospace; font-size: 35px; font-weight: bold;">수업 신청하기
              </h4>
              <p style="font-family: unset; font-size: 18px;">해당 내용은 추후 수정이 불가합니다. 아래의 내용을 확인하시고, 상황을 고려하여 신청 바랍니다.</p>
              
              <span @click="showModal = false" class="close">&times;</span>
          </div>
          <div class="modal-body" style=" margin-right: auto;">
            <h3 style="margin-top: 5px; font-weight: bold;">방문 유형</h3>
            
            <label for="one-day" style="margin-left: 2px;">1일 (4시간 1번 수업)
              <input style="width: auto; margin: 5px;" type="radio" id="one-day" value="one" v-model="checkedValue" @change="onCheckboxChange">
            </label>

            <label for="two-day" style="margin-left: 2px;">2일 (2시간 2번 수업)
              <input style="width: auto; margin: 5px;" type="radio" id="two-day" value="two" v-model="checkedValue" @change="onCheckboxChange">
            </label>

          </div>      
          <div>
            <div v-if="checkedValue === 'one'">

                <form style=" margin-left: 1%; margin-right: auto;">
                  <label for="school">기관명</label>
                    <select id="school" name="school" v-model="selectedSchool" required>
                      <option value="">기관 선택</option>
                      <option value="school1">기관 1</option>
                      <option value="school2">기관 2</option>
                      <option value="school3">기관 3</option>
                    </select>

                    <label for="teacher">교사명</label>
                    <input type="text" id="teacher" name="teacher" v-model="teacherName" required>

                    <label for="phone">핸드폰번호 (- 생략)</label>
                    <input type="tel" id="phone" name="phone" v-model="phoneNumber" pattern="[0-9]{3}[0-9]{4}[0-9]{4}" required>

                    <label for="email">이메일</label>
                    <input type="email" id="email" name="email" v-model="email" required>

                    <label for="studentCount">학생수</label>
                    <input type="number" id="studentCount" name="studentCount" v-model="studentCount" :max="getEventMaxStudentCount()" min="1" required>  
                    <div>
                      <h3>시간대를 선택해주세요.</h3>
                      <input style="width: auto;"  type="radio" id="time-a" name="time" value="a" v-model="selectedTime" required>
                      <label style="display: inline;" for="time-a">09:00 ~ 11:00</label><br>
                      <input style="width: auto;"  type="radio" id="time-b" name="time" value="b" v-model="selectedTime" required>
                      <label style="display: inline;" for="time-b">11:00 ~ 13:00</label><br>
                      <input style="width: auto;"  type="radio" id="time-c" name="time" value="c" v-model="selectedTime" required>
                      <label style="display: inline;" for="time-c">14:00 ~ 16:00</label><br>
                      <input style="width: auto;"  type="radio" id="time-d" name="time" value="d" v-model="selectedTime" required>
                      <label style="display: inline;" for="time-d">16:00 ~ 18:00</label><br>
                      <input style="width: auto;"  type="radio" id="time-e" name="time" value="e" v-model="selectedTime" required>
                      <label style="display: inline;" for="time-e">18:00 ~ 20:00</label><br>
                    </div>
                </form>
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
            <button @click="submitApplication" :disabled="!isFormValid">신청</button>
          </div>
      </div>
  </div>
</template>

<script>
import { ref, watch } from 'vue'
import FullCalendar from '@fullcalendar/vue3'
import dayGridPlugin from '@fullcalendar/daygrid'
import interactionPlugin from '@fullcalendar/interaction'
import axios from 'axios'
import moment from 'moment'

export default {
  watch: {
    showModal(value) {
      if (value === true) {
        this.updateMinDate()
      } else {
        this.minDate = null
      }
    },
  },
  
  data() {
    return {
      selectedSchool: "",
      teacherName: "",
      phoneNumber: "",
      email: "",
      studentCount: null,
      selectedTime: null,
      newDate: null,
      minDate: null,
      data: [], // 원본 데이터
      events: [], // 이벤트 데이터
      curriculumSn:12
      
    }
  },
  
  components: {
    FullCalendar,
  },

  computed: {
    isFormValid() {
    return (
      this.selectedSchool !== "" &&
      /^[가-힣]{2,4}$/.test(this.teacherName) &&
      /^[0-9]{3}[0-9]{4}[0-9]{4}$/.test(this.phoneNumber) &&
      /^[^\s@]+@[^\s@]+\.[^\s@]+$/.test(this.email) &&
      this.studentCount !== null &&
      this.studentCount > 0 &&
      this.selectedTime !== null
    );
  },
    calendarOptions() {
      return {
        plugins: [dayGridPlugin, interactionPlugin],
        initialView: 'dayGridMonth',
        dateClick: this.handleDateClick,
        dayMaxEvents: 4,
        events: this.events, // events 배열 사용
        dayCellContent: (arg) => {
          const date = arg.date

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
          ]

          const isHoliday = holidays.some(
            holiday => date.getFullYear() === 2023 && date.getMonth() === holiday.month && date.getDate() === holiday.day
          )

          const isSunday = date.getDay() === 0
          if (isHoliday || isSunday) {
            return { html: `<div style="color: red;">${date.getDate()}</div>` }
          } else {
            return { html: `<div>${date.getDate()}</div>` }
          }
        },
      }
    },
  },

  beforeMount() {
    this.fetchData()
  },

  methods: {
    submitApplication() {
    if (!this.isFormValid) {
      // 경고 표시 또는 필요한 동작 수행
      alert("입력값을 확인해주세요.");
      return;
    }

    // 신청 로직 수행
    // ...
  },
    getEventMaxStudentCount() {
  const eventData = this.events.find(event => event.date === this.selectedDate);
  if (eventData) {
    const match = eventData.title.match(/\d+/);
    if (match) {
      return parseInt(match[0]);
    }
  }
  return 0;
},



    getEventTitle() {
    const eventData = this.events.find(event => event.date === this.selectedDate);
    return eventData ? eventData.title : '';
    
  },
    convertDataToEvents(data) {
      const events = Object.entries(data).map(([date, value]) => {
        let startDate = moment(date, 'YYYY-MM-DD').format('YYYY-MM-DD')
        let event = {
          title: `태블릿: ${value}`,
          date: startDate
        }
        return event
      })
      if (events.length === 0) {
        events.push({
          title: '마감',
          date: moment().format('YYYY-MM-DD'),
          classNames: 'closed-event',
          textColor: 'red'
        })
      }
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
      ]

      for (const holiday of holidays) {
        events.push({
          title: holiday.title,
          date: holiday.date,
          classNames: 'holiday-event',
          textColor: 'red' 
        })
      }

      return events
    },
    fetchData() {
      axios
        .post(`http://localhost:8081/reserve/api/v1/admin/findAllLocalChildren?curriculumSn=${this.curriculumSn}`)
        .then(response => {
          var data = response.data.result.data
          this.data = data // 원본 데이터 저장
          const events = this.convertDataToEvents(data)
          this.events = events // 이벤트 데이터 업데이트
        })
        .catch(error => {
          console.error(error)
        })
    },

    handleDateClick(arg) {
      const clickedDate = arg.dateStr;
      const eventData = this.events.find(event => event.date === clickedDate);
      if (eventData) {
        this.selectedDate = clickedDate;
        this.showModal = true;
        this.selectedDate = arg.dateStr
        this.showModal = true
    

        const data = this.data // 저장된 원본 데이터 사용

        const events = this.convertDataToEvents(data)
        this.events = events // 이벤트 데이터 업데이트
        this.updateMinDate() // 최소 날짜 업데이트

        // API 요청을 통해 데이터베이스와 통신
        axios
        .post('http://localhost:8081/reserve/api/v1/admin/findAllLocalChildren?curriculumSn=12', {
          clickedDate: clickedDate // 클릭한 날짜 정보를 API 요청에 전달
        })
        .then(response => {
          const data = response.data.result.data
          const events = this.convertDataToEvents(data)
          this.events = events // 이벤트 데이터 업데이트
          this.updateMinDate() // 최소 날짜 업데이트
        })
        .catch(error => {
          console.error(error)
        })
      }
    },

    updateMinDate() {
      const selectedDate = new Date(this.selectedDate)
      const minDate = new Date(selectedDate.getTime() + 24 * 60 * 60 * 1000) // Add 1 day to the selected date
      this.minDate = minDate.toISOString().split("T")[0]
    },
  },

  setup() {
    const showModal = ref(false)
    const selectedDate = ref(null)
    const checkedValue = ref("one")
    const resultEvent = ref(null)
    
    watch(showModal, (value) => {
      if (value === true) {
        checkedValue.value = "one"
      }
    })


    return {
      showModal,
      selectedDate,
      checkedValue,
      resultEvent
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
/* 전체 폼 스타일 */
form {
  max-width: 400px;
  margin: 0 auto;
}

/* 레이블 스타일 */
label {
  display: block;
  margin-bottom: 5px;
  font-weight: bold;
}

/* 입력 필드 스타일 */
input,
select {
  width: 100%;
  padding: 8px;
  border: 1px solid #ccc;
  border-radius: 4px;
  font-size: 16px;
  margin-bottom: 10px;
}

/* 제출 버튼 스타일 */
input[type="submit"] {
  background-color: #4CAF50;
  color: white;
  padding: 10px 20px;
  border: none;
  border-radius: 4px;
  cursor: pointer;
}

input[type="submit"]:hover {
  background-color: #45a049;
}

/* 오류 표시 스타일 */
input:invalid {
  border-color: #ff0000;
}
</style>