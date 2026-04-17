<script setup>
import { ref, computed, onMounted, onUnmounted } from 'vue'

// 狀態
const isDarkMode = ref(false)
const showZhuyin = ref(false)
const isCountdown = ref(false)
const currentTime = ref(new Date())
const countdownSeconds = ref(0)
const countdownActive = ref(false)

// 輸入資料
const inputStartHour = ref('09')
const inputStartMinute = ref('00')
const inputEndHour = ref('10')
const inputEndMinute = ref('30')
const inputSubject = ref('')
const inputTeacher = ref('')
const exams = ref([])

const hourOptions = Array.from({ length: 24 }, (_, i) => String(i).padStart(2, '0'))
const minuteOptions = Array.from({ length: 60 }, (_, i) => String(i).padStart(2, '0'))

const selectedTime = computed(() => `${inputStartHour.value}:${inputStartMinute.value} - ${inputEndHour.value}:${inputEndMinute.value}`)

const selectedDuration = computed(() => {
  const startMinutes = parseInt(inputStartHour.value) * 60 + parseInt(inputStartMinute.value)
  const endMinutes = parseInt(inputEndHour.value) * 60 + parseInt(inputEndMinute.value)
  const duration = endMinutes - startMinutes
  return duration > 0 ? duration * 60 : 0
})

// 定時器ID
let timeInterval = null
let countdownInterval = null

// 注音符号映射（簡單示例）
const zhuyinMap = {
  '互': '互ㄏㄨˋ',
  '動': '動ㄉㄨㄥˋ',
  '程': '程ㄔㄥˊ',
  '式': '式ㄕˋ',
  '設': '設ㄕㄜˋ',
  '計': '計ㄐㄧˋ',
  '時': '時ㄕˊ',
  '間': '間ㄐㄧㄢ',
  '倒': '倒ㄉㄠˋ',
  '數': '數ㄕㄨˋ',
  '現': '現ㄒㄧㄢˋ',
  '在': '在ㄗㄞˋ',
  '注': '注ㄓㄨˋ',
  '音': '音ㄧㄣ',
  '符': '符ㄈㄨˊ',
  '號': '號ㄏㄠˋ',
  '主': '主ㄓㄨˇ',
  '題': '題ㄊㄧˊ',
  '字': '字ㄗˋ',
  '內': '內ㄋㄟˋ',
  '容': '容ㄖㄨㄥˊ',
  '按': '按ㄢˋ',
  '鈕': '鈕ㄋㄧㄡˇ',
  '可': '可ㄎㄜˇ',
  '連': '連ㄌㄧㄢˊ',
  '線': '線ㄒㄧㄢˋ',
  '到': '到ㄉㄠˋ',
  '淡': '淡ㄉㄢˋ',
  '江': '江ㄐㄧㄤ',
  '教': '教ㄐㄧㄠˋ',
  '科': '科ㄎㄜ',
  '系': '系ㄒㄧˋ',
  '定': '定ㄉㄧㄥˋ',
  '幫': '幫ㄅㄤ',
  '老': '老ㄌㄠˇ',
  '師': '師ㄕ',
  '監': '監ㄐㄧㄢ',
  '考': '考ㄎㄠˇ',
  '小': '小ㄒㄧㄠˇ',
  '工': '工ㄍㄨㄥ',
  '具': '具ㄐㄩˋ',
  '第': '第ㄉㄧˋ',
  '個': '個ㄍㄜˋ',
  '切': '切ㄑㄧㄝ',
  '換': '換ㄏㄨㄢˋ',
  '改': '改ㄍㄞˇ',
  '變': '變ㄅㄧㄢˋ',
  '整': '整ㄓㄥˇ',
  '網': '網ㄨㄤˇ',
  '頁': '頁ㄧㄝˋ',
  '文': '文ㄨㄣˊ',
  '加': '加ㄐㄧㄚ',
  '沒': '沒ㄇㄟˊ',
  '有': '有ㄧㄡˇ',
  '黑': '黑ㄏㄟ',
  '白': '白ㄅㄞˊ',
  '底': '底ㄉㄧˇ',
  '下': '下ㄒㄧㄚˋ',
  '方': '方ㄈㄤ',
  '做': '做ㄗㄨㄛˋ',
  '讓': '讓ㄖㄤˋ',
  '使': '使ㄕˇ',
  '用': '用ㄩㄥˋ',
  '者': '者ㄓㄜˇ',
  '輸': '輸ㄕㄨ',
  '入': '入ㄖㄨˋ',
  '目': '目ㄇㄨˋ',
  '前': '前ㄑㄧㄢˊ',
  '試': '試ㄕˋ',
  '上': '上ㄕㄤˋ',
  '午': '午ㄨˇ',
  '資': '資ㄗ',
  '訊': '訊ㄒㄩㄣˋ',
  '息': '息ㄒㄧˊ',
  '開': '開ㄎㄞ',
  '啟': '啟ㄑㄧˇ',
  '關': '關ㄍㄨㄢ',
  '閉': '閉ㄅㄧˋ',
  '亮': '亮ㄌㄧㄤˋ',
  '暗': '暗ㄢˋ',
  '尚': '尚ㄕㄤˋ',
  '無': '無ㄨˊ',
  '排': '排ㄆㄞˊ',
  '操': '操ㄘㄠ',
  '作': '作ㄗㄨㄛˋ',
  '刪': '刪ㄕㄢ',
  '除': '除ㄔㄨˊ',
  '例': '例ㄌㄧˋ',
  '如': '如ㄖㄨˊ',
  '新': '新ㄒㄧㄣ',
  '增': '增ㄗㄥ',
  '輸': '輸ㄕㄨ',
  '選': '選ㄒㄩㄢˇ',
  '項': '項ㄒㄧㄤˋ',
  '匯': '匯ㄏㄨㄟˋ',
  '出': '出ㄔㄨ',
  '名': '名ㄇㄧㄥˊ',
  '稱': '稱ㄔㄥ',
}

const addZhuyin = (text) => {
  if (!showZhuyin.value) return text
  let result = text
  for (const [char, zhuyin] of Object.entries(zhuyinMap)) {
    const zhuyinPart = zhuyin.substring(1)
    result = result.replace(new RegExp(char, 'g'), `${char}<ruby><rt style="font-size: 0.65em; color: #666;">${zhuyinPart}</rt></ruby>`)
  }
  return result
}

// 顯示標題（帶注音）
const displayTitle = computed(() => {
  const title = '互動程式設計_413730069'
  return addZhuyin(title)
})

// 顯示按鈕文字（帶注音）
const displayButtonTime = computed(() => {
  return addZhuyin(isCountdown.value ? '倒數' : '時間')
})

const displayButtonZhuyin = computed(() => {
  return addZhuyin('注音')
})

const displayButtonTheme = computed(() => {
  return addZhuyin(isDarkMode.value ? '亮' : '暗')
})

const displayCurrentTimeLabel = computed(() => {
  return addZhuyin('現在時間')
})

const displayCountdownLabel = computed(() => {
  return addZhuyin('倒數時間')
})

const displayFormTitle = computed(() => {
  return addZhuyin('考試資訊輸入')
})

const displayTimeLabel = computed(() => {
  return addZhuyin('時間')
})

const displaySubjectLabel = computed(() => {
  return addZhuyin('科目')
})

const displayTeacherLabel = computed(() => {
  return addZhuyin('監考老師')
})

const displayListTitle = computed(() => {
  return addZhuyin('考試排程')
})

const displayNoExams = computed(() => {
  return addZhuyin('目前尚無考試排程')
})

const displayTableTimeHeader = computed(() => {
  return addZhuyin('時間')
})

const displayTableSubjectHeader = computed(() => {
  return addZhuyin('科目')
})

const displayTableTeacherHeader = computed(() => {
  return addZhuyin('監考老師')
})

const displayTableActionHeader = computed(() => {
  return addZhuyin('操作')
})

const displayDeleteButton = computed(() => {
  return addZhuyin('刪除')
})

const displayAddButton = computed(() => {
  return addZhuyin('新增考科')
})

const displayStartButton = computed(() => {
  return addZhuyin('開始')
})

const displayStopButton = computed(() => {
  return addZhuyin('停止')
})

// 更新當前時間
const updateTime = () => {
  currentTime.value = new Date()
}

// 啟動倒數計時
const startCountdown = () => {
  if (!countdownActive.value) {
    countdownSeconds.value = selectedDuration.value
    countdownActive.value = selectedDuration.value > 0
  }
}

// 停止倒數計時
const stopCountdown = () => {
  countdownActive.value = false
  countdownSeconds.value = 0
}

// 格式化顯示時間
const formatTime = (seconds) => {
  const hours = Math.floor(seconds / 3600)
  const minutes = Math.floor((seconds % 3600) / 60)
  const secs = seconds % 60
  return `${String(hours).padStart(2, '0')}:${String(minutes).padStart(2, '0')}:${String(secs).padStart(2, '0')}`
}

// 格式化當前時間
const displayTime = computed(() => {
  return currentTime.value.toLocaleTimeString('zh-TW')
})

// 新增考試
const addExam = () => {
  if (selectedTime.value && inputSubject.value && inputTeacher.value) {
    exams.value.push({
      id: Date.now(),
      time: selectedTime.value,
      subject: inputSubject.value,
      teacher: inputTeacher.value
    })
    inputSubject.value = ''
    inputTeacher.value = ''
  }
}

// 刪除考試
const deleteExam = (id) => {
  exams.value = exams.value.filter(exam => exam.id !== id)
}

onMounted(() => {
  updateTime()
  timeInterval = setInterval(updateTime, 1000)
  
  countdownInterval = setInterval(() => {
    if (countdownActive.value && countdownSeconds.value > 0) {
      countdownSeconds.value--
    } else if (countdownActive.value && countdownSeconds.value === 0) {
      countdownActive.value = false
    }
  }, 1000)
})

onUnmounted(() => {
  if (timeInterval) clearInterval(timeInterval)
  if (countdownInterval) clearInterval(countdownInterval)
})
</script>

<template>
  <div class="app" :class="{ dark: isDarkMode }">
    <!-- 頂部導航欄 -->
    <div class="header">
      <a href="https://www.et.tku.edu.tw/" class="btn-link" title="淡江教科系">
        ☰ 淡江教科
      </a>
      <h1 class="title" v-html="displayTitle"></h1>
    </div>

    <!-- 功能按鈕區域 -->
    <div class="button-group">
      <button 
        class="btn btn-time"
        @click="isCountdown = !isCountdown"
        :title="isCountdown ? '切換到現在時間' : '切換到倒數計時'"
      >
        ⏱️ <span v-html="displayButtonTime"></span>
      </button>
      
      <button 
        class="btn btn-zhuyin"
        @click="showZhuyin = !showZhuyin"
        :title="showZhuyin ? '關閉注音符號' : '開啟注音符號'"
      >
        🔤 <span v-html="displayButtonZhuyin"></span>{{ showZhuyin ? '✓' : '' }}
      </button>
      
      <button 
        class="btn btn-theme"
        @click="isDarkMode = !isDarkMode"
        :title="isDarkMode ? '切換到淺色主題' : '切換到深色主題'"
      >
        🌙 <span v-html="displayButtonTheme"></span>
      </button>
    </div>

    <div class="main-content">
      <div class="left-panel">
        <div class="card input-card">
          <div class="card-header">
            <div class="card-title"><span v-html="displayFormTitle"></span></div>
          </div>
          <div class="input-grid">
            <div class="form-group time-select-group">
              <label><span v-html="displayTimeLabel"></span></label>
              <div class="time-period-box">
                <div class="time-period-item">
                  <span class="time-period-label">開始</span>
                  <div class="time-selects">
                    <select v-model="inputStartHour" class="input-field time-select">
                      <option v-for="hour in hourOptions" :key="hour" :value="hour">{{ hour }}</option>
                    </select>
                    <span class="time-separator">:</span>
                    <select v-model="inputStartMinute" class="input-field time-select">
                      <option v-for="minute in minuteOptions" :key="minute" :value="minute">{{ minute }}</option>
                    </select>
                  </div>
                </div>
                <div class="time-period-item">
                  <span class="time-period-label">結束</span>
                  <div class="time-selects">
                    <select v-model="inputEndHour" class="input-field time-select">
                      <option v-for="hour in hourOptions" :key="hour" :value="hour">{{ hour }}</option>
                    </select>
                    <span class="time-separator">:</span>
                    <select v-model="inputEndMinute" class="input-field time-select">
                      <option v-for="minute in minuteOptions" :key="minute" :value="minute">{{ minute }}</option>
                    </select>
                  </div>
                </div>
              </div>
            </div>

            <div class="form-group">
              <label for="subject"><span v-html="displaySubjectLabel"></span></label>
              <input
                id="subject"
                v-model="inputSubject"
                type="text"
                placeholder="輸入科目名稱"
                class="input-field"
              />
            </div>

            <div class="form-group">
              <label for="teacher"><span v-html="displayTeacherLabel"></span></label>
              <input
                id="teacher"
                v-model="inputTeacher"
                type="text"
                placeholder="輸入老師名字"
                class="input-field"
              />
            </div>
          </div>
          <div class="card-footer">
            <button @click="addExam" class="btn btn-add">
              ➕ <span v-html="displayAddButton"></span>
            </button>
          </div>
        </div>

        <div class="card schedule-card">
          <div class="card-header">
            <div class="card-title"><span v-html="displayListTitle"></span></div>
            <div class="card-subtitle">已新增的考試排程</div>
          </div>
          <div class="exam-list">
            <div v-if="exams.length === 0" class="no-exams">
              <span v-html="displayNoExams"></span>
            </div>
            <div v-else class="exams-table">
              <table>
                <thead>
                  <tr>
                    <th v-html="displayTableTimeHeader"></th>
                    <th v-html="displayTableSubjectHeader"></th>
                    <th v-html="displayTableTeacherHeader"></th>
                    <th v-html="displayTableActionHeader"></th>
                  </tr>
                </thead>
                <tbody>
                  <tr v-for="exam in exams" :key="exam.id">
                    <td>{{ exam.time }}</td>
                    <td>{{ exam.subject }}</td>
                    <td>{{ exam.teacher }}</td>
                    <td>
                      <button 
                        @click="deleteExam(exam.id)"
                        class="btn btn-delete"
                      >
                        ✕ <span v-html="displayDeleteButton"></span>
                      </button>
                    </td>
                  </tr>
                </tbody>
              </table>
            </div>
          </div>
        </div>
      </div>

      <div class="right-panel">
        <div class="card clock-card">
          <div class="clock-top">
            <div class="clock-title">📅 <span v-html="displayCurrentTimeLabel"></span></div>
            <div class="clock-meta">{{ new Date().toLocaleDateString('zh-TW', { year: 'numeric', month: '2-digit', day: '2-digit' }) }} 星期{{ ['日','一','二','三','四','五','六'][new Date().getDay()] }}</div>
          </div>
          <div class="time-display">
            <div v-if="!isCountdown" class="current-time">
              {{ displayTime }}
            </div>
            <div v-else class="countdown">
              <div class="countdown-title">⏳ <span v-html="displayCountdownLabel"></span></div>
              <div class="countdown-value">
                {{ countdownActive ? formatTime(countdownSeconds) : '00:00:00' }}
              </div>
              <div class="countdown-controls">
                <button 
                  v-if="!countdownActive && countdownSeconds === 0"
                  @click="startCountdown"
                  class="btn btn-small"
                >
                  <span v-html="displayStartButton"></span>
                </button>
                <button 
                  v-if="countdownActive"
                  @click="stopCountdown"
                  class="btn btn-small"
                >
                  <span v-html="displayStopButton"></span>
                </button>
              </div>
            </div>
          </div>
        </div>
      </div>
    </div>
  </div>
</template>

<style scoped>
* {
  box-sizing: border-box;
}

.app {
  min-height: 100vh;
  font-family: 'Microsoft YaHei', Arial, sans-serif;
  background: linear-gradient(135deg, #ffffff 0%, #f5f5f5 100%);
  color: #333;
  transition: background 0.3s ease, color 0.3s ease;
  padding: 0;
  margin: 0;
}

.app.dark {
  background: linear-gradient(135deg, #1a1a1a 0%, #2d2d2d 100%);
  color: #fff;
}

/* 頂部導航欄 */
.header {
  display: flex;
  align-items: center;
  gap: 20px;
  background: linear-gradient(90deg, #4CAF50 0%, #45a049 100%);
  color: white;
  padding: 15px 20px;
  box-shadow: 0 2px 8px rgba(0, 0, 0, 0.1);
}

.app.dark .header {
  background: linear-gradient(90deg, #2a5f2a 0%, #1f4620 100%);
}

.btn-link {
  display: inline-block;
  padding: 8px 16px;
  background: rgba(255, 255, 255, 0.2);
  color: white;
  text-decoration: none;
  border-radius: 4px;
  transition: background 0.3s ease;
  font-weight: 500;
}

.btn-link:hover {
  background: rgba(255, 255, 255, 0.3);
}

.title {
  font-size: 24px;
  font-weight: bold;
  flex: 1;
  margin: 0;
  text-shadow: 1px 1px 2px rgba(0, 0, 0, 0.2);
}

/* 按鈕組 */
.button-group {
  display: flex;
  flex-wrap: wrap;
  gap: 10px;
  padding: 15px 20px;
  background: #f9f9f9;
  border-bottom: 1px solid #ddd;
  justify-content: center;
}

.app.dark .button-group {
  background: #333;
  border-bottom-color: #555;
}

.main-content {
  display: grid;
  grid-template-columns: 1.25fr 0.95fr;
  gap: 20px;
  margin: 20px;
}

.left-panel,
.right-panel {
  display: flex;
  flex-direction: column;
  gap: 20px;
}

.card {
  background: #fff;
  border-radius: 28px;
  box-shadow: 0 20px 50px rgba(0, 0, 0, 0.08);
  overflow: hidden;
}

.app.dark .card {
  background: #1f2129;
  box-shadow: 0 20px 50px rgba(0, 0, 0, 0.3);
}

.card-header {
  padding: 24px 28px 0;
}

.schedule-card .exam-list {
  background: transparent;
  padding: 0 28px 28px;
  margin: 0;
}

.schedule-card .no-exams {
  padding: 30px 0;
}

.schedule-card .exams-table {
  overflow-x: auto;
}

.schedule-card table {
  width: 100%;
  border-collapse: collapse;
}

.schedule-card th,
.schedule-card td {
  padding: 16px 12px;
  border-bottom: 1px solid #f1f3f6;
}

.schedule-card th {
  background: transparent;
  color: #7f8a9d;
  text-align: left;
  font-weight: 700;
}

.schedule-card td {
  color: #232e4a;
}

.schedule-card tbody tr:hover {
  background: rgba(33, 150, 243, 0.08);
}

.card-title {
  font-size: 18px;
  font-weight: 700;
  letter-spacing: 0.03em;
  color: #1f2734;
}

.app.dark .card-title {
  color: #f7f7f7;
}

.card-subtitle {
  margin-top: 10px;
  color: #7f8a9d;
  font-size: 13px;
}

.card-footer {
  padding: 20px 28px 26px;
  display: flex;
  justify-content: flex-end;
}

.clock-card {
  display: flex;
  flex-direction: column;
  min-height: 100%;
}

.clock-top {
  padding: 24px 28px 0;
  display: flex;
  flex-direction: column;
  gap: 10px;
}

.clock-title {
  font-size: 18px;
  font-weight: 700;
  color: #1f2734;
}

.app.dark .clock-title {
  color: #f7f7f7;
}

.clock-meta {
  color: #7f8a9d;
  font-size: 13px;
}

.time-display {
  background: transparent;
  padding: 0 28px 28px;
}

.current-time,
.countdown-value {
  margin: 20px 0 0;
}

.current-time {
  font-size: 72px;
  font-weight: 900;
  letter-spacing: -0.04em;
  color: #141823;
}

.app.dark .current-time {
  color: #f7f7f7;
}

.countdown {
  display: flex;
  flex-direction: column;
  align-items: flex-start;
  gap: 18px;
}

.countdown-title {
  font-size: 16px;
  font-weight: 700;
  color: #ff9800;
}

.countdown-value {
  font-size: 52px;
  font-weight: 900;
  color: #1a1f3b;
}

.countdown-controls {
  display: flex;
  gap: 12px;
}

.btn {
  padding: 10px 16px;
  border: none;
  border-radius: 999px;
  font-size: 14px;
  font-weight: 500;
  cursor: pointer;
  transition: all 0.3s ease;
  background: #2196F3;
  color: white;
}

.btn:hover:not(:disabled) {
  transform: translateY(-2px);
  box-shadow: 0 4px 12px rgba(33, 150, 243, 0.3);
}

.btn:disabled {
  opacity: 0.5;
  cursor: not-allowed;
}

.btn-time {
  background: #FF9800;
}

.btn-time:hover {
  box-shadow: 0 4px 12px rgba(255, 152, 0, 0.3);
}

.btn-zhuyin {
  background: #9C27B0;
}

.btn-zhuyin:hover {
  box-shadow: 0 4px 12px rgba(156, 39, 176, 0.3);
}

.btn-theme {
  background: #673AB7;
}

.btn-theme:hover {
  box-shadow: 0 4px 12px rgba(103, 58, 183, 0.3);
}

.btn-add {
  background: #4CAF50;
  padding: 10px 20px;
  font-size: 15px;
}

.btn-add:hover {
  box-shadow: 0 4px 12px rgba(76, 175, 80, 0.3);
}

.btn-small {
  padding: 8px 12px;
  font-size: 13px;
  background: #2196F3;
}

.btn-delete {
  padding: 6px 10px;
  font-size: 12px;
  background: #f44336;
}

.btn-delete:hover {
  box-shadow: 0 4px 12px rgba(244, 67, 54, 0.3);
}

/* 时间显示区域 */
.time-display {
  padding: 30px 20px;
  text-align: center;
  background: #fff;
  border-bottom: 2px solid #eee;
  margin-bottom: 20px;
}

.app.dark .time-display {
  background: #222;
  border-bottom-color: #444;
}

.current-time {
  font-size: 28px;
  font-weight: bold;
  color: #4CAF50;
}

.app.dark .current-time {
  color: #66BB6A;
}

.countdown {
  display: flex;
  flex-direction: column;
  align-items: center;
  gap: 15px;
}

.countdown-title {
  font-size: 18px;
  font-weight: bold;
  color: #FF9800;
}

.app.dark .countdown-title {
  color: #FFB74D;
}

.countdown-value {
  font-size: 48px;
  font-weight: bold;
  font-family: 'Courier New', monospace;
  color: #f44336;
  text-shadow: 0 2px 4px rgba(0, 0, 0, 0.1);
}

.countdown-controls {
  display: flex;
  gap: 10px;
}

/* 输入表单区域 */
.input-section {
  background: #fff;
  padding: 25px 20px;
  border-radius: 8px;
  margin: 0 20px 20px;
  box-shadow: 0 2px 8px rgba(0, 0, 0, 0.1);
}

.app.dark .input-section {
  background: #222;
  box-shadow: 0 2px 8px rgba(0, 0, 0, 0.3);
}

.input-section h2 {
  margin: 0 0 15px;
  font-size: 18px;
  color: #333;
  border-bottom: 2px solid #4CAF50;
  padding-bottom: 10px;
}

.app.dark .input-section h2 {
  color: #fff;
  border-bottom-color: #66BB6A;
}

.form-group {
  display: flex;
  flex-direction: column;
  margin-bottom: 12px;
}

.form-group label {
  font-weight: 600;
  margin-bottom: 5px;
  font-size: 14px;
}

.input-field {
  padding: 10px 12px;
  border: 1px solid #ddd;
  border-radius: 4px;
  font-size: 14px;
  font-family: inherit;
  transition: all 0.3s ease;
}

.time-select-group .input-field {
  padding: 10px 10px;
}

.input-grid {
  display: grid;
  grid-template-columns: minmax(180px, 260px) minmax(220px, 1fr) minmax(220px, 1fr);
  gap: 18px;
  align-items: start;
}

.form-group {
  display: flex;
  flex-direction: column;
  min-width: 0;
}

.form-group label {
  margin-bottom: 10px;
  font-size: 14px;
  font-weight: 600;
}

.time-selects {
  display: flex;
  align-items: center;
  gap: 8px;
  flex-wrap: wrap;
}

.time-select {
  width: 78px;
  min-width: 78px;
  text-align: center;
}

.time-select-group {
  justify-self: start;
  max-width: 320px;
}

.time-period-box {
  display: grid;
  gap: 12px;
}

.time-period-item {
  display: flex;
  align-items: center;
  gap: 10px;
  padding: 12px 14px;
  border-radius: 16px;
  background: #f7f8fb;
}

.app.dark .time-period-item {
  background: #2a2d38;
}

.time-period-label {
  width: 54px;
  color: #556179;
  font-size: 13px;
  font-weight: 600;
}

.form-actions {
  margin-top: 18px;
  display: flex;
  justify-content: flex-end;
}

@media (max-width: 860px) {
  .input-grid {
    grid-template-columns: 1fr;
  }
}
.input-field:focus {
  outline: none;
  border-color: #4CAF50;
  box-shadow: 0 0 0 3px rgba(76, 175, 80, 0.1);
}

.app.dark .input-field {
  background: #333;
  border-color: #555;
  color: #fff;
}

.app.dark .input-field:focus {
  border-color: #66BB6A;
  box-shadow: 0 0 0 3px rgba(102, 187, 106, 0.1);
}

/* 考试列表区域 */
.exam-list {
  background: #fff;
  padding: 25px 20px;
  border-radius: 8px;
  margin: 0 20px 20px;
  box-shadow: 0 2px 8px rgba(0, 0, 0, 0.1);
}

.app.dark .exam-list {
  background: #222;
  box-shadow: 0 2px 8px rgba(0, 0, 0, 0.3);
}

.exam-list h2 {
  margin: 0 0 15px;
  font-size: 18px;
  color: #333;
  border-bottom: 2px solid #4CAF50;
  padding-bottom: 10px;
}

.app.dark .exam-list h2 {
  color: #fff;
  border-bottom-color: #66BB6A;
}

.no-exams {
  text-align: center;
  padding: 30px;
  color: #999;
  font-size: 16px;
}

.exams-table {
  overflow-x: auto;
}

table {
  width: 100%;
  border-collapse: collapse;
  margin-top: 15px;
}

thead {
  background: #f5f5f5;
}

.app.dark thead {
  background: #333;
}

th {
  padding: 12px;
  text-align: left;
  font-weight: 600;
  border-bottom: 2px solid #ddd;
  color: #333;
}

.app.dark th {
  border-bottom-color: #555;
  color: #fff;
}

td {
  padding: 12px;
  border-bottom: 1px solid #eee;
}

.app.dark td {
  border-bottom-color: #444;
}

tbody tr:hover {
  background: #f9f9f9;
}

.app.dark tbody tr:hover {
  background: #2a2a2a;
}

/* 响应式设计 */
@media (max-width: 768px) {
  .header {
    flex-direction: column;
    gap: 10px;
    padding: 10px 15px;
  }

  .title {
    font-size: 20px;
  }

  .button-group {
    padding: 10px 15px;
  }

  .btn {
    padding: 8px 12px;
    font-size: 12px;
  }

  .time-display {
    padding: 20px 15px;
  }

  .current-time {
    font-size: 24px;
  }

  .countdown-value {
    font-size: 36px;
  }

  .input-section,
  .exam-list {
    margin: 0 15px 15px;
    padding: 15px;
  }

  table {
    font-size: 13px;
  }

  th, td {
    padding: 8px;
  }

  .form-group {
    margin-bottom: 10px;
  }
}

@media (max-width: 480px) {
  .title {
    font-size: 16px;
  }

  .button-group {
    gap: 8px;
  }

  .btn {
    padding: 6px 10px;
    font-size: 11px;
    flex: 1;
    min-width: 80px;
  }

  .current-time {
    font-size: 20px;
  }

  .countdown-value {
    font-size: 32px;
  }
}
</style>
