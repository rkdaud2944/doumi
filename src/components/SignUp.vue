<template>
    <FullCalendar :options="calendarOptions" class="calendar" @eventClick="handleDateClick" />
    <!-- 모달창 -->
    <div v-if="showModal" class="modal">
      <div class="modal-content">
        <div class="modal-header">
          <div style="font-weight: bold; text-align: left;">
            날짜 : {{ selectedDate }}
            <div> 남은 자리 : {{ getEventTitle() }}
            </div>
          </div>
          <br />
          <h4 style="font-family: 'Courier New', Courier, monospace; font-size: 35px; font-weight: bold;">
            수업 신청하기
          </h4>
          <p style="font-family: unset; font-size: 16px;">
            해당 내용은
              <a style="color: red;">추후 수정이 불가
              </a>
            합니다. <br>아래의 내용을 확인하시고,<br>상황을 고려하여 신청 바랍니다.
          </p>
          <span @click="showModal = false" class="close">&times;
          </span>
        </div>
        <div class="modal-body" style="margin-right: auto;">
          <h3 style="margin-top: 5px; font-weight: bold;">
            방문 유형
          </h3>
          <label for="one-day" style="margin-left: 2px;">
            1일 (4차시 1번 수업)
            <input style="width: auto; margin: 5px;" type="radio" id="one-day" value="one" v-model="checkedValue" @change="onCheckboxChange" />
          </label>
          <label for="two-day" style="margin-left: 2px;">
            2일 (2차시 2번 수업)
            <input style="width: auto; margin: 5px;" type="radio" id="two-day" value="two" v-model="checkedValue" @change="onCheckboxChange" />
          </label>
        </div>
        <div>
          
  
          <!-- 하루 방문 모달창 -->
          <div v-if="checkedValue === 'one'">
            <form style="margin-left:auto; margin-right: auto;">
              <label for="school">과목명</label>
              
              <select id="school" name="school" v-model="curriculumSn" required>
                <option value="1">3D프린팅과 모델링</option>
                <option value="2">할로코드로 만드는 사물인터넷</option>
                <option value="3">햄스터봇으로 만나는 자율주행</option>
              </select>
  
              <label for="school">기관명</label>
                <div class="dropdown-container">
                  <input type="text" v-model="searchText" ref="schoolInput" @input="filterData" @focus="showDropdown = false" placeholder="검색어를 입력하세요" />
                  <div class="dropdown-toggle" @click="toggleDropdown">
                    <i class="arrow-icon" :class="{'arrow-up': showDropdown, 'arrow-down': !showDropdown}"></i>
                  </div>
                  <ul class="dropdown-menu" v-show="!showDropdown">
                    <li v-for="item in filteredData" :key="item.sn" @click="selectData(item)">
                      {{ item.schoolName }}
                    </li>
                  </ul>
              </div>
    
              <label for="teacher">교사명</label>
              <input type="text" id="teacher" name="teacher" ref="teacherInput" v-model="teacherName" required />
  
              <label for="phone">핸드폰번호 (- 생략)</label>
              <input type="tel" id="phone" name="phone" ref="phoneInput" v-model="phoneNumber" pattern="[0-9]{3}[0-9]{4}[0-9]{4}" required />
  
              
              <label for="email">이메일</label>
              <div style="display: flex;">
                      <input type="email" id="email" name="email" ref="emailInput" v-model="email" required />
              <div class="email-button" @click="sendEmail" v-show="email && buttonDisabled" :style="{cursor: email ? 'pointer' : 'default'}">
                  <p style="font-size: 10px; line-height: 25px;">인증</p>
              </div>
              </div>
  
          
              <div style="display: flex;" v-if="showEmailInput" >
              <label for="text">인증번호</label>
                      <input type="text" ref="securitypwInput" v-model="emailSecuritypw" required />
              <div class="email-button" @click="emailSecurity">
                  <p style="font-size: 6px; line-height: 25px; margin: 0px;">Check</p>
              </div>
              </div>
              <p v-if="showCountdown">남은시간: {{ formatCountdown() }}</p>
              <p v-if="emailclear">인증 완료</p>
  
              <label for="studentCount">학생수</label>
              <input type="number" id="studentCount" name="studentCount" ref="studentCountInput" v-model="studentCount" :max="getEventMaxStudentCount()" min="1" required />
              <div><br />
                <h3>시간대를 선택해주세요.</h3><br />
                <label style="display: block" v-for="time in timeOptions" :key="time.id">
                  <input style="width: auto;" type="checkbox" :id="time.id" :value="time.label" v-model="selectedTimes"
                    :disabled="selectedTimes.length >= 2 && !selectedTimes.includes(time.label)" required />
                  {{ time.label }}
                </label>
              </div>
              <div class="modal-footer">
                <div style="
                    background-color: #4caf50;
                    color: white;
                    border: none;
                    border-radius: 4px;
                    padding: 10px 10px;
                    cursor: pointer;
                    height: 40px;
                    width: 50px;
                    margin-left: auto;
                    margin-right: 1%;"
                    v-if="checkedValue === 'one'" @click="onedayclick">신청</div>
              </div>
            </form>
          </div>
  
          <!-- 이틀 방문 모달창 -->
          <div v-if="checkedValue === 'two'">
              <div style="display: flex; justify-content: space-between;">
                <form @submit.prevent="onSubmit" style=" margin-left:auto; margin-right: auto;">
                    <label for="school">과목명</label>
                
                    <select id="school" name="school" v-model="curriculumSn" required>
                        <option value="1">3D프린팅과 모델링</option>
                        <option value="2">할로코드로 만드는 사물인터넷</option>
                        <option value="3">햄스터봇으로 만나는 자율주행</option>
                    </select>

                  <label for="school">기관명</label>
                    <div class="dropdown-container">
                      <input type="text" v-model="searchText" ref="schoolInputTwoDay" @input="filterData" @focus="showDropdown = false" placeholder="검색어를 입력하세요" />
                      <div class="dropdown-toggle" @click="toggleDropdown">
                        <i class="arrow-icon" :class="{'arrow-up': showDropdown, 'arrow-down': !showDropdown}"></i>
                      </div>
                      <ul class="dropdown-menu" v-show="!showDropdown">
                        <li v-for="item in filteredData" :key="item.sn" @click="selectData(item)">
                          {{ item.schoolName }}
                        </li>
                      </ul>
                  </div>
                    <label for="teacher">교사명</label>
                    <input type="text" id="teacher" name="teacher" ref="teacherInputTwoDay" v-model="teacherName" required>
                    <label for="phone">핸드폰번호 (- 생략)</label>
                    <input type="tel" id="phone" name="phone" ref="phoneInputTwoDay" v-model="phoneNumber" pattern="[0-9]{3}[0-9]{4}[0-9]{4}" required>
                    
                    <label for="email">이메일</label>
                    <div style="display: flex;">
                      <input type="email" id="email" name="email" ref="emailInputTwoDay" v-model="email" required />
                      <div class="email-button" @click="sendEmail" v-show="email && buttonDisabled" :style="{cursor: email ? 'pointer' : 'default'}">
                        <p style="font-size: 10px; line-height: 25px;">인증</p>
                      </div>
                    </div>
  
                  
                    <div style="display: flex;" v-if="showEmailInput" >
                      <label for="text">인증번호</label>
                      <input type="text" ref="securitypwInputTwoDay" v-model="emailSecuritypw" required />
                      <div class="email-button" @click="emailSecurity">
                        <p style="font-size: 6px; line-height: 25px; margin: 0px;">Check</p>
                      </div>
                    </div>
                    <p v-if="showCountdown">남은시간: {{ formatCountdown() }}</p>
                    <p v-if="emailclear">인증 완료</p>
                    
  
                    <label for="studentCount">학생수</label>
                    <input type="number" id="studentCount" name="studentCount" ref="studentCountInputTwoDay" v-model="studentCount" :max="getEventMaxStudentCount()" min="1" required>  
                    <div>
                      <h3>{{selectedDate}}에 방문하시는 시간대를 선택해주세요.</h3>
  
                      <label style="display: block" v-for="time in timeOptions" :key="time.id">
                        <input style="width: auto;" type="checkbox" :id="time.id" :value="time.label" ref="selectTimesTwoDay1" v-model="selectedTimes"
                          :disabled="selectedTimes.length >= 1 && !selectedTimes.includes(time.label)" required />
                        {{ time.label }}
                      </label>
                      
                    </div>
                      <div>
                          <h3>두번째 신청하실 날짜를 선택해주세요.</h3>
                          <select id="second-date" name="second-date" ref="twoDay" v-model="selectedSecondDate" @change="displaySelectedDateTitle" required>
                              <option value="">날짜 선택</option>
                              <option v-for="dateOption in filteredDateOptions" :key="dateOption.value" :value="dateOption.value">
                              {{ dateOption.label }}
                              </option>
                          </select>
                          <p v-if="selectedSecondDate !== '' && selectedDateTitle !== ''">
                              {{ selectedDateTitle }} 남음
                          </p>
                      </div>
  
                      <div>
                          <h3>두번째 신청하실 시간대를 선택해주세요.</h3>
  
                      <label style="display: block" v-for="time in timeOptions" :key="time.id">
                        <input style="width: auto;" type="checkbox" :id="time.id" :value="time.label" ref="selectTimesTwoDay1" v-model="selectedTimes2"
                          :disabled="selectedTimes2.length >= 1 && !selectedTimes2.includes(time.label)" required />
                        {{ time.label }}
                      </label>
                      
                    </div>
                    <div class="modal-footer">
                      <button style="
                        background-color: #4caf50;
                        color: white;
                        border: none;
                        border-radius: 4px;
                        padding: 10px 10px;
                        cursor: pointer;
                        height: 40px;
                        width: 50px;
                        margin-left: auto;
                        margin-right: 1%;" type="submit" v-if="checkedValue === 'two'" @click="twodayclick">신청</button>
                    </div>
                </form>  
              </div>
            </div>
          </div>
      </div>
    </div>
    <!-- 표 -->
    <h2 style="margin-left: auto; margin-right: auto;  text-align:center; margin-top: 20px; ">아래 과목을 참고하고, 위 달력을 클릭하여 남은 자리를 참고하고 예약하세요.</h2>
    <div class="curidata">
  
    
        <div class="poster">
          <div class="poster-header">
            <h2 class="poster-title">4차시 3D프린팅과 모델링</h2>
            <p class="poster-date">교육 대상 : 초등학생 / 중학생</p>
            <p class="poster-date">관련 분야 : 3D 모델링, 3D 프린팅</p>
            <p class="poster-date">준비물 : 태블릿, 인터넷</p><br>
            <p class="poster-goal">학습목표 : <br>- 3D 프린팅의 원리를 이해한다.<br>- 틴커캐드를 이용하여 3D 모델링 작업을 한다.<br>- 틴커캐드를 이용하여 나만의 작품을 만들 수 있다.<br></p>
          </div><br>
          <div class="poster-description">
            <p>
              1차시 : 3D 프린팅이란 ?<br>
              - 3D 프린팅이란?<br>
              - 3D 프린팅의 원리와 활용<br>
              - 틴커캐드 가입과 인터페이스 알아보기<br><br>
  
              2차시 : 틴커캐드의 기본기능 익히기<br>
              - 모델링 기초 수업 <br>
              - 여러 가지 도형 만들어 보기<br><br>
  
              3차시 : 글자모델링<br>
              - 글자 모델링 해보기<br>
              - 글자와 도형을 이용한 나만의 도장 만들어 보기<br><br>
  
              4차시 : 나만의 컵 만들기<br>
              - 투명도형을 이용해서 공간 설계하기<br>
              - 투명도형을 이용한 나만의 컵 만들어 보기<br>
            </p>
          </div>
        </div>
  
        <div style="margin-left: 10px; margin-right: 10px;" class="poster">
          <div class="poster-header">
            <h2 class="poster-title">할로코드로 만드는 사물인터넷</h2>
            <p class="poster-date">교육 대상 : 초등학생 / 중학생</p>
            <p class="poster-date">관련 분야 : 피지컬 컴퓨팅, 인공지능</p>
            <p class="poster-date">준비물 : 태블릿, 인터넷</p><br>
            <p class="poster-goal">학습목표 : <br>- 할로코드를 활용하여 LED를 제어할 수 있다.<br>- 다양한 센서를 활용하여 사물인터넷 장치를 개발할 수 있다.<br></p>
          </div><br>
          <div class="poster-description">
            <p>
              1차시 : 엠블럭 시작과 할로코드 연결 <br>
              - 엠블록 프로그램을 실행하기<br>
              - 엠블록 사용자 계정등록하기<br>
              - 엠블록 화면 구성 알아보기<br><br>
  
              2차시 : 할로코드 기본 구성 및 LED 표현<br>
              - 엠블럭 코딩해보기 <br>
              - 할로코드 기초 및 기능 확인하기<br>
              - 할로코드 LED 표현해보기<br><br>
  
              3차시 : 할로코드 코딩 LED, 마이크<br>
              - LED링으로 스마일 만들기  <br>
              - 빛의 3원색 표현하기<br>
              - 소리 감지기 만들기<br><br>
  
              4차시 : 할로코드 코딩 모션 센서<br>
              - 에너지링 만들기<br>
              - 자동차 핸들 만들기<br>
            </p>
          </div>
        </div>
        <div class="poster">
          <div class="poster-header">
            <h2 class="poster-title">햄스터로 만나는 자율주행</h2>
            <p class="poster-date">교육 대상 : 초등학생 / 중학생</p>
            <p class="poster-date">관련 분야 : SW , 코딩, 컴퓨터 </p>
            <p class="poster-date">준비물 : 햄스터봇, 태블릿, 자율주행 말판, 축구장 공, 색연필, 종이등</p><br>
            <p class="poster-goal">학습목표 : <br>- 햄스터봇을 활용한 프로그래밍 활동을 통해 컴퓨팅사고력 향상과 문제해결력을 키웁니다.<br>- 햄스터봇의 센서를 활용하여 자율주행차를 만들어봅니다.<br></p>
          </div><br>
          <div class="poster-description">
            <p>
              1차시 : 안녕 햄스터봇<br>
              - 햄스터봇의 기본 구성 및 명칭을 알아보기<br>
              - 스택을 이용한 햄스터봇 제어하기<br>
              - 햄스터봇과 태블릿 페어링 하기<br>
              - LED 색 변경하는  코딩하기<br>
  
              2차시 : 햄스터봇 움직이기<br>
              - 키보드로 햄스터봇 움직이기 <br>
              - 라인을 따라 움직이는 자율주행 햄스터만들기<br><br>
  
              3차시 : 노래하는 햄스터봇<br>
              - 햄스터봇의 스피커를 이용하여 노래하기<br>
              - 장애물 인식 자율주행 자동차 만들기<br><br>
  
              4차시 : 햄스터봇 축구대회<br>
              - 햄스터봇으로 축구게임하기<br>
              - 대시보드로 햄스터봇 제어 리모컨 만들기<br>
              - 리모컨으로 햄스터봇 축구대회하기<br>
            </p>
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
      data() {
          return {
              selectedSchool: '',
              teacherName: '',
              phoneNumber: '',
              email: '',
              studentCount: null,
              curriculumSn: '',
              classDate: '2023-07-20',
              selectedTimes: [],
              selectedTimes2: [],
              emailSecuritypw : '',
              data: [], // 원본 데이터
              events: [], // 이벤트 데이터
              curriculumSna: 12,
              selectedLabels: [],
              dateOptions: [],
              selectedDateTitle: '',
              selectedSecondDate: '',
              showEmailInput: false,
              countdown: 300, // 5 minutes in seconds
              countdownInterval: null,
              buttonDisabled: true,
              emailclear : false,
  
              schoolData: [], // 초기 데이터
              filteredData: [], // 검색어로 필터링된 데이터
              selectedData: null, // 선택된 데이터
              showDropdown: true, // 드롭다운 메뉴 표시 여부
              searchText: '', // 검색어
              api:'http://localhost:8081', // 로컬 테스트용
              // api:'http://43.202.4.224:8081', // 실사용 서버 주소
          }
      },
  
      components: {
          FullCalendar,
      },
  
      beforeMount() {
          this.fetchData()
      },
  
      mounted() {
          this.getSchoolData(); // 초기 데이터 가져오기
      },
  
      computed: {
          filteredDateOptions() {
              if (this.selectedDate === '') {
                  return []; // 선택된 첫번째 날짜가 없을 경우 빈 배열 반환
              }
              // 선택된 첫번째 날짜 이후의 날짜들만 필터링하여 반환
                  return this.dateOptions.filter(dateOption => dateOption.value > this.selectedDate);
          },
          showCountdown() {
              return this.showEmailInput && this.countdown > 0;
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
                  (holiday) =>
                  date.getFullYear() === 2023 && date.getMonth() === holiday.month && date.getDate() === holiday.day
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
  
          timeOptions() {
              return [
                  { id: 'time-a', label: '10:00 ~ 11:30' },
                  { id: 'time-b', label: '13:00 ~ 14:30' },
                  { id: 'time-c', label: '14:00 ~ 15:30' },
                  { id: 'time-d', label: '14:30 ~ 16:00' },
                  { id: 'time-e', label: '15:00 ~ 16:30' },
                  { id: 'time-f', label: '16:00 ~ 17:30' },
                  { id: 'time-g', label: '16:30 ~ 18:00' },
              ]
          },
  
      },
  
      methods: {
          toggleTable(tableNumber) {
              if (tableNumber === 1) {
                  this.showTable1 = !this.showTable1;
              } else if (tableNumber === 2) {
                  this.showTable2 = !this.showTable2;
              } else if (tableNumber === 3) {
                  this.showTable3 = !this.showTable3;
              }
          },
  
          formatCountdown() {
              const minutes = Math.floor(this.countdown / 60);
              const seconds = this.countdown % 60;
              return `${minutes.toString().padStart(2, '0')}:${seconds.toString().padStart(2, '0')}`;
          },
  
          sendEmail() {
              axios
              .get(
            `${this.api}/sendVerification?email=${this.email}`
              )
              .then((response) => {
                  this.showEmailInput = true;
                  this.buttonDisabled = false;
                  // 카운트다운 시작
                  this.countdownInterval = setInterval(() => {
                      if (this.countdown > 0) {
                          this.countdown--;
                      } else {
                          clearInterval(this.countdownInterval);
                          this.showModal = false; // 모달 닫기
                      }
                  }, 1000);
          console.log(response.data)
              })
              .catch((error) => {
                  console.error(error)
              })
          },
  
          emailSecurity() {
              axios
              .get(
            `${this.api}/verifyEmail?email=${this.email}&code=${this.emailSecuritypw}`
              )
              .then((response) => {
                  if (response.data.result.title == "인증성공") {
                      this.countdown = 300;
                      this.showEmailInput = false;
                      this.emailclear = true;
                      clearInterval(this.countdownInterval);
                      return
                  }
  
                  alert("인증번호를 다시 한번 확인해주세요.");
              })
              .catch((error) => {
                  console.error(error)
              })
          },
  
          displaySelectedDateTitle() {
              const eventData = this.events.find((event) => event.date === this.selectedSecondDate)
              const match = eventData ? eventData.title.match(/\d+/) : null
              this.selectedDateTitle = match ? match[0] : ''
          },
  
          getSelectedDateTitle() {
              const eventData = this.events.find((event) => event.date === this.selectedSecondDate)
              return eventData ? eventData.title : ''
          },
  
          refreshModalData() {
        this.selectedSchool = null
              this.teacherName = ''
              this.phoneNumber = ''
              this.email = ''
              this.studentCount = null
              this.selectedTimes = []
              this.selectedTimes2 = []
              clearInterval(this.countdownInterval);
              this.countdown = 300;
              this.showEmailInput = false;
              this.buttonDisabled = true;
          this.searchText = ''
          },
  
          clearModalData() {
          // Reset the modal data to initial values
        this.selectedSchool = null
              this.teacherName = ''
              this.phoneNumber = ''
              this.email = ''
              this.studentCount = null
              this.selectedTimes = []
              this.selectedTimes2 = []
              clearInterval(this.countdownInterval);
              this.countdown = 300;
              this.showEmailInput = false;
              this.buttonDisabled = true;
        this.searchText = ''
          },
  
      // 유효성 검사
      validateFormOneDay() {
        // 학교명(기관명) 유효성 검사
        if (!this.searchText.trim()) {
          alert('기관명을 입력해주세요.');
          this.$refs.schoolInput.focus();
          return false;
        }
        
        // 학교명(기관명) 수정 검증
        let validSchool = this.schoolData.find((item) =>
          item.schoolName === this.searchText
        );
        if (!validSchool) {
          alert('입력하신 기관명은 유효하지 않습니다. 목록에서 선택해주세요. \n해당기관이 리스트에 없으면 아래 번호로 문의주세요. \n * 인천미래채움 운영센터: 070-7726-3346/5256');
          this.$refs.schoolInput.focus();
          return false;
        }
  
        // 만약 학교명(기관명)이 유효하다면, 선택된 학교의 sn을 다시 설정합니다.
        this.selectedSchool = validSchool.sn;
  
        // 교사명 유효성 검사
        if (!this.teacherName.trim()) {
          alert('교사명을 입력해주세요.');
          this.$refs.teacherInput.focus();
          return false;
        }
  
        // 핸드폰번호 유효성 검사
        const phoneRegEx = /^[0-9]{3}[0-9]{4}[0-9]{4}$/;
        if (!this.phoneNumber.trim() || !phoneRegEx.test(this.phoneNumber)) {
          alert('핸드폰번호 11자리를 입력해주세요. ("-")는 생략해주세요.');
          this.$refs.phoneInput.focus();
          return false;
        }
  
        // 이메일 유효성 검사
        if (!this.email.trim() || !this.email.includes('@')) {
          alert('유효한 이메일을 입력해주세요.');
          this.$refs.emailInput.focus();
          return false;
        }
  
        // 이메일 인증번호 유효성 검사
        if (!this.emailclear) {
          alert('이메일 인증을 완료해주세요.');
          // this.$refs.securitypwInput.focus();
          return false;
        }
  
        // 학생수 유효성 검사
        if (!this.studentCount || this.studentCount < 1) {
          alert('학생수는 1명 이상이어야 합니다.');
          this.$refs.studentCountInput.focus();
          return false;
        }
  
        // 시간대 선택 유효성 검사
        if (this.selectedTimes.length !== 2) {
          alert('시간대를 반드시 두 개 선택해주세요.');
          return false;
        }
  
  
        return true;
      },
  
      validateFormTwoDay() {
        // 기관명 유효성 검사
        if (!this.searchText.trim()) {
          alert('기관명을 입력해주세요.');
          this.$refs.schoolInputTwoDay.focus();
          return false;
        }
        
        // 기관명 수정 검증
        let validSchool = this.schoolData.find((item) =>
          item.schoolName === this.searchText
        );
        if (!validSchool) {
          alert('입력하신 기관명은 유효하지 않습니다. 목록에서 선택해주세요. \n해당기관이 리스트에 없으면 아래 번호로 문의주세요. \n * 인천미래채움 운영센터: 070-7726-3346/5256');
          this.$refs.schoolInputTwoDay.focus();
          return false;
        }
  
        // 기관명이 유효하다면, 선택된 기관의 sn을 다시 설정합니다.
        this.selectedSchool = validSchool.sn;
  
        // 교사명 유효성 검사
        if (!this.teacherName.trim()) {
          alert('교사명을 입력해주세요.');
          this.$refs.teacherInputTwoDay.focus();
          return false;
        }
  
        // 핸드폰번호 유효성 검사
        const phoneRegEx = /^[0-9]{3}[0-9]{4}[0-9]{4}$/;
        if (!this.phoneNumber.trim() || !phoneRegEx.test(this.phoneNumber)) {
          alert('핸드폰번호 11자리를 입력해주세요. ("-")는 생략해주세요.');
          this.$refs.phoneInputTwoDay.focus();
          return false;
        }
  
        // 이메일 유효성 검사
        if (!this.email.trim() || !this.email.includes('@')) {
          alert('유효한 이메일을 입력해주세요.');
          this.$refs.emailInputTwoDay.focus();
          return false;
        }
  
        // 이메일 인증번호 유효성 검사
        if (!this.emailclear) {
          alert('이메일 인증을 완료해주세요.');
          // this.$refs.securitypwInputTwoDay.focus();
          return false;
        }
  
        // 학생수 유효성 검사
        if (!this.studentCount || this.studentCount < 1) {
          alert('학생수는 1명 이상이어야 합니다.');
          this.$refs.studentCountInputTwoDay.focus();
          return false;
        }
  
        // 첫번째 시간대 선택 유효성 검사
        if (this.selectedTimes.length === 0) {
          alert('첫번째 방문 시간대를 최소 하나 이상 선택해주세요.');
          return false;
        }
  
        // 두번째 날짜 선택 유효성 검사
        if (!this.selectedSecondDate) {
          alert('두번째 신청할 날짜를 선택해주세요.');
          this.$refs.twoDay.focus();
          return false;
        }
  
        // 두번째 시간대 선택 유효성 검사
        if (this.selectedTimes2.length === 0) {
          alert('두번째 방문 시간대를 최소 하나 이상 선택해주세요.');
          return false;
        }
  
        return true;
      },
  
  
          onedayclick() {
        // 유효성 검사가 통과되지 않을 시 데이터를 DB에 보내지 않고 리턴
        if (!this.validateFormOneDay()) {
          return;
        }
                  axios
                  .post(
            `${this.api}/reserve/api/v1/admin/addReservation?teacherName=${this.teacherName}&phone=${this.phoneNumber}&email=${this.email}&studentsPerGroup=${this.studentCount}&curriculumSn=${this.curriculumSn}&schoolSn=${this.selectedSchool}&classDate=${this.selectedDate}&classDate2=${this.classDate}&classSchedule=${this.selectedTimes.join(',')}`
                  )
                  .then((response) => {
            if (response.data.result.msg != 'SUCCESS') {
              // 서버로부터 받은 데이터가 'SUCCESS'가 아니면 아래 처리를 실행
              alert('서버로부터 예상치 않은 응답을 받았습니다. \n'+response.data.result.msg)
            } else {
              if (confirm('신청이 완료되어 입력한 이메일로 예약확인 메일이 발송되었습니다.\n확인을 누르시면 새로고침이 됩니다.')) {
                location.reload();
              }
            }
                  })
                  .catch((error) => {
                      console.error(error)
                  })
          },
  
          twodayclick() {
        // 유효성 검사가 통과되지 않을 시 데이터를 DB에 보내지 않고 리턴
        if (!this.validateFormTwoDay()) {
          return;
        }
                  axios
                  .post(
          `${this.api}/reserve/api/v1/admin/addDoubleReservation?teacherName=${this.teacherName}&phone=${this.phoneNumber}&email=${this.email}&studentsPerGroup=${this.studentCount}&curriculumSn=${this.curriculumSn}&schoolSn=${this.selectedSchool}&classDate=${this.selectedDate}&classDate2=${this.selectedSecondDate}&classSchedule=${this.selectedTimes.join(',')}`
                  )
                  .then((response) => {
            if (response.data.result.msg != 'SUCCESS') {
              // 서버로부터 받은 데이터가 'SUCCESS'가 아니면 아래 처리를 실행
              alert('서버로부터 예상치 않은 응답을 받았습니다. \n'+response.data.result.msg)
            } else {
              if (confirm('신청이 완료되어 입력한 이메일로 예약확인 메일이 발송되었습니다.\n확인을 누르시면 새로고침이 됩니다.')) {
                location.reload();
              }
            }
                  })
                  .catch((error) => {
                      console.error(error)
                  })
        
          },
  
          getEventMaxStudentCount() {
              const eventData = this.events.find((event) => event.date === this.selectedDate)
              if (eventData) {
                  const match = eventData.title.match(/\d+/)
                  if (match) {
                      return parseInt(match[0])
                  }
              }
              return 0
          },
  
          getEventTitle() {
              const eventData = this.events.find((event) => event.date === this.selectedDate)
              return eventData ? eventData.title : ''
          },
  
          convertDataToEvents(data) {
              const events = Object.entries(data).map(([date, value]) => {
                  let startDate = moment(date, 'YYYY-MM-DD').format('YYYY-MM-DD')
                  let event = {
                      title: `${value} 남음`,
                      date: startDate,
                  }
                  return event
              })
              if (events.length === 0) {
                  events.push({
                  title: '마감',
                  date: moment().format('YYYY-MM-DD'),
                  classNames: 'closed-event',
                  textColor: 'red',
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
                  { title: '성탄절', date: '2023-12-25' },
              ]
  
              for (const holiday of holidays) {
                  events.push({
                  title: holiday.title,
                  date: holiday.date,
                  classNames: 'holiday-event',
                  textColor: 'red',
                  })
              }
  
              return events
          },
  
          fetchData() {
              axios
        .post(`${this.api}/reserve/api/v1/admin/findAllLocalChildren?curriculumSn=${this.curriculumSna}`)
              .then((response) => {
                  const data = response.data.result.data;
                  const dates = Object.keys(data).map((date) => date.split('T')[0]); // Extract date without time
                  const sortedDates = dates.sort((a, b) => new Date(a) - new Date(b)); // Sort dates in ascending order
                  this.dateOptions = sortedDates.map((date) => ({
                      value: date,
                      label: date,
                  }));
  
                  const events = this.convertDataToEvents(data)
                  this.events = events // Update the event data
              })
              .catch((error) => {
                  console.error(error)
              })
          },
  
          handleDateClick(arg) {
              const clickedDate = arg.dateStr
              const eventData = this.events.find((event) => event.date === clickedDate)
              if (eventData) {
                  this.selectedDate = clickedDate
                  this.showModal = true
                  this.selectedDate = arg.dateStr
                  this.showModal = true
  
                  const data = this.data // 저장된 원본 데이터 사용
  
                  const events = this.convertDataToEvents(data)
                  this.events = events // 이벤트 데이터 업데이트
  
                  // API 요청을 통해 데이터베이스와 통신
                  axios
            .post(`${this.api}/reserve/api/v1/admin/findAllLocalChildren?curriculumSn=12`, {
                      clickedDate: clickedDate, // 클릭한 날짜 정보를 API 요청에 전달
                  })
                  .then((response) => {
                      const data = response.data.result.data
                      const events = this.convertDataToEvents(data)
                      this.events = events // 이벤트 데이터 업데이트
                  })
                  .catch((error) => {
                      console.error(error)
                  })
              }
          },
  
          // 기관 필터 검색 (toggleDropdown함수까지)
          async getSchoolData() {
              try {
          const response = await axios.get(`${this.api}/reserve/api/v1/admin/findSchool`); 
                  this.schoolData = response.data.result.msg;
                  this.filteredData = this.schoolData;
              } catch (error) {
                  console.error(error);
              }
          },
  
          filterData() {
              if (this.searchText === '') {
                  this.filteredData = this.schoolData; // 검색어가 없으면 전체 데이터 표시
              } else {
                  this.filteredData = this.schoolData.filter((item) =>
                      item.schoolName.includes(this.searchText) // 검색어로 필터링된 데이터만 표시
                  );
              }
          },
          selectData(data) {
        this.searchText = data.schoolName; // 입력필드에 선택한 학교 이름을 보여줍니다.
        this.selectedSchool = data.sn; // 선택한 학교의 sn 값을 selectedSchool에 설정합니다.
        this.showDropdown = true; // 드롭다운 메뉴를 닫습니다.
          },
          toggleDropdown() {
              this.showDropdown = !this.showDropdown; // 드롭다운 토글
          },
      },
  
      watch: {
          showModal(value) {
              if (value === true) {
                  this.refreshModalData()
              } else {
                  this.clearModalData()
              }
          },
  
          checkedValue() {
              if (this.showModal) {
                  this.refreshModalData()
              }
          },
  
      },
  
      setup() {
      const showModal = ref(false)
      const selectedDate = ref(null)
      const checkedValue = ref('one')
      const resultEvent = ref(null)
  
      watch(showModal, (value) => {
          if (value === true) {
              checkedValue.value = 'one'
          }
      })
  
          return {
              showModal,
              selectedDate,
              checkedValue,
              resultEvent,
          }
      },
  }
  </script>
  
<style>

.dropdown-container {
    width : 100%;
    position: relative;
    display: inline-block;
}

.dropdown-container input{
    width: 90%;
}

.dropdown-toggle {
    display: inline-block;
    width: 10%;
    background-color: transparent;
    border: none;
    cursor: pointer;
    padding: 5px;
}

.arrow-icon {
    display: inline-block;
    width: 0;
    height: 0;
    border-style: solid;
    border-width: 5px 5px 0 5px;
    border-color: #999 transparent transparent transparent;
    transition: transform 0.3s;
}

.arrow-up {
    transform: rotate(180deg);
}

.arrow-down {
    transform: rotate(0);
}

.dropdown-menu {
    position: absolute;
    top: 100%;
    left: 0;
    z-index: 999;
    background-color: #fff;
    border: 1px solid #ccc;
    max-height: 200px;
    display: block; /* ul 요소가 화면에 표시되도록 수정 */
    overflow-y: auto;
}

.dropdown-menu li {
    padding: 10px;
    cursor: pointer;
}

.dropdown-menu li:hover {
    background-color: #f1f1f1;
}

/* ############# */

.calendar {
    margin-right: auto;
    margin-left: auto;
    text-align: center;
    width: 80%;
    margin-top: 10px;
}
.vgt-left-align {
    text-align: center;
}
.vgt-right-align {
    text-align: center;
}
.fc-col-header-cell:nth-of-type(1) .fc-col-header-cell-cushion {
    color: red;
}
.holiday-event {
    background-color: rgba(255, 255, 255, 0.0) !important;
    border: none !important;
    color: red !important;
    bottom: 25px;
    text-align: right !important;
    margin-right: 5px !important;
}
.fc-daygrid-day-top {
    flex-direction: row !important;
}
.fc-event-main {
    text-align: left;
}
.holiday-event .fc-event-main {
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
    top: -5vh;
    width: 500px;
    height: 70%;
    overflow-y: auto;
}
.modal-header {
    padding: 10px;
    text-align: center;
}
.modal-footer {
    padding: 10px;
    text-align: right;
}
#two-day {
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
input[type='submit'] {
    background-color: #4caf50;
    color: white;
    padding: 10px 20px;
    border: none;
    border-radius: 4px;
    cursor: pointer;
}
input[type='submit']:hover {
    background-color: #45a049;
}
/* 오류 표시 스타일 */
input:invalid {
    border-color: #ff0000;
}
.email-button {
    background-color: #4caf50;
    color: white;
    border: none;
    border-radius: 4px;
    padding: 8px 12px;
    margin-left: 5px;
    cursor: pointer;
    height: 40px;
    width: 50px;
}
.email-button:hover {
    background-color: #45a049;
}
.table {
    border-collapse: collapse;
    width: 100%;
    margin-bottom: 1em;
}
.table th,
.table td {
    border: 1px solid #ddd;
    padding: 8px;
}
.table th {
    background-color: #f2f2f2;
    font-weight: bold;
}
.poster {
    width: 26vw;
    min-width: 300px;
    background-color: #f3f3f3;
    border-radius: 8px;
    box-shadow: 0 2px 4px rgba(0, 0, 0, 0.2);
    text-align: center;
    margin-top : 30px;
    padding: 20px;
    display:inline-block;
    height: 800px;
}
.poster-header {
    text-align: center;
    margin-bottom: 10px;
}
.poster-title {
    font-size: 22px;
    font-weight: bold;
    margin-bottom: 5px;
    color: #333;
}
.poster-date {
    font-size: 16px;
    color: #888;
}
.poster-goal{
    font-size: 14.5px;
    color: #2c2c2c;
    text-align: left;
}
.poster-description {
    text-align: justify;
    line-height: 1.5;
    color: #555;
}
.custom-button {
    display: inline-block;
    background-color: #3498db;
    color: #fff;
    font-size: 16px;
    font-weight: bold;
    padding: 10px 20px;
    border: none;
    border-radius: 4px;
    cursor: pointer;
}
.custom-button:hover {
    background-color: #2980b9;
}
.toggle-icon {
    margin-left: 5px;
}
@media (max-width: 576px) {
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
        margin: 3%;
        margin-top : 15%;
        padding: 20px;
        border: 1px solid #888;
        bottom: 10px;
        width: 350px;
        height: 70%;
        overflow-y: auto;
    }
    .modal-header {
        padding: 10px;
        text-align: center;
    }
    .modal-footer {
        padding: 10px;
        text-align: right;
    }
    .curidata{
        margin-left: auto;
        margin-right: auto;
        text-align:center;
        margin-bottom: 20px;
        justify-content: center;
        display: block !important;
    }

}
.curidata{
    margin-left: auto;
    margin-right: auto;
    text-align:center;
    margin-bottom: 20px;
    display:flex;
    justify-content: center;
}
</style>