<script setup>
import { ref, computed, onMounted, onUnmounted } from 'vue'

// 狀態：模式切換
const isCountdownMode = ref(false)
const showZhuyin = ref(false)
const isDarkMode = ref(false)

// 狀態：時間與倒數計時
const currentTime = ref('')
const countdownSeconds = ref(3600) // 預設倒數 60 分鐘
let timerInterval = null
const targetStartTime = ref(null) // 考試開始時間
const targetEndTime = ref(null)   // 考試結束時間
const countdownState = ref('none') // 'waiting'(即將開始), 'running'(剩餘時間), 'ended'(結束)

// 計算屬性：格式化倒數時間
const formattedCountdown = computed(() => {
  const h = Math.floor(countdownSeconds.value / 3600)
  const m = Math.floor((countdownSeconds.value % 3600) / 60)
  const s = countdownSeconds.value % 60
  return `${h.toString().padStart(2, '0')}:${m.toString().padStart(2, '0')}:${s.toString().padStart(2, '0')}`
})

// 更新時間的邏輯
const updateTime = () => {
  const now = new Date()
  // 更新現在時間
  currentTime.value = now.toLocaleTimeString('zh-TW', { hour12: false })
  
  // 依據時間動態計算倒數狀態與剩餘秒數
  if (isCountdownMode.value && targetEndTime.value) {
    if (targetStartTime.value && now < targetStartTime.value) {
      countdownState.value = 'waiting'
      countdownSeconds.value = Math.floor((targetStartTime.value - now) / 1000)
    } else if (now < targetEndTime.value) {
      countdownState.value = 'running'
      countdownSeconds.value = Math.floor((targetEndTime.value - now) / 1000)
    } else {
      countdownState.value = 'ended'
      countdownSeconds.value = 0
    }
  }
}

onMounted(() => {
  updateTime()
  timerInterval = setInterval(updateTime, 1000)
})

onUnmounted(() => {
  clearInterval(timerInterval)
})

// 狀態：表單資料與紀錄
const formData = ref({
  time: '',
  subject: '',
  teacher: ''
})

const records = ref([])

// 解析輸入的時間字串並取得開始與結束時間
const parseExamTimes = (timeStr) => {
  const now = new Date()

  // 1. 嘗試解析時間範圍 (例如: 10:00-12:00 或 10:00~12:00)
  const rangeMatch = timeStr.match(/(\d{1,2}):(\d{2})\s*[-~]\s*(\d{1,2}):(\d{2})/)
  if (rangeMatch) {
    const start = new Date(now)
    start.setHours(parseInt(rangeMatch[1]), parseInt(rangeMatch[2]), 0, 0)
    
    const end = new Date(now)
    end.setHours(parseInt(rangeMatch[3]), parseInt(rangeMatch[4]), 0, 0)
    
    if (end < start) end.setDate(end.getDate() + 1) // 跨日處理
    return { start, end }
  }
  
  // 2. 嘗試解析純分鐘數 (例如: 60, 90分鐘)
  const minMatch = timeStr.match(/(\d+)/)
  if (minMatch) {
    const mins = parseInt(minMatch[1])
    return { start: now, end: new Date(now.getTime() + mins * 60000) }
  }
  
  return { start: now, end: new Date(now.getTime() + 3600000) } // 預設 60 分鐘
}

// 提交表單資料
const submitData = () => {
  if (!formData.value.time && !formData.value.subject && !formData.value.teacher) return
  
  // 根據輸入的時間更新倒數計時，並自動切換為倒數模式
  if (formData.value.time) {
    const { start, end } = parseExamTimes(formData.value.time)
    targetStartTime.value = start
    targetEndTime.value = end
    isCountdownMode.value = true
    updateTime() // 立即更新狀態
  }

  // 將新資料加入紀錄陣列
  records.value.push({ ...formData.value })
  
  // 清空表單
  formData.value.time = ''
  formData.value.subject = ''
  formData.value.teacher = ''
}
</script>

<template>
  <div :class="['app-container', { 'dark-mode': isDarkMode, 'hide-zhuyin': !showZhuyin }]">
    <!-- 標題與外部連結區域 -->
    <header class="app-header">
      <a href="https://www.et.tku.edu.tw/" target="_blank" class="link-btn" title="淡江教育科技學系">
        🎓 <ruby>教<rt>ㄐㄧㄠˋ</rt>科<rt>ㄎㄜ</rt></ruby>
      </a>
      <h1 class="main-title">
        💻 <ruby>互<rt>ㄏㄨˋ</rt>動<rt>ㄉㄨㄥˋ</rt>程<rt>ㄔㄥˊ</rt>式<rt>ㄕˋ</rt></ruby>_413730382
      </h1>
    </header>

    <!-- 功能切換按鈕區 (工具列) -->
    <div class="toolbar">
      <button @click="isCountdownMode = !isCountdownMode" class="tool-btn" title="切換時間模式">
        <span v-if="!isCountdownMode">⏳ <ruby>考<rt>ㄎㄠˇ</rt>試<rt>ㄕˋ</rt>倒<rt>ㄉㄠˋ</rt>數<rt>ㄕㄨˇ</rt></ruby></span>
        <span v-else>🕒 <ruby>現<rt>ㄒㄧㄢˋ</rt>在<rt>ㄗㄞˋ</rt>時<rt>ㄕˊ</rt>間<rt>ㄐㄧㄢ</rt></ruby></span>
      </button>
      
      <button @click="showZhuyin = !showZhuyin" class="tool-btn" title="切換注音顯示">
        <span v-if="!showZhuyin">🔤 <ruby>顯<rt>ㄒㄧㄢˇ</rt>示<rt>ㄕˋ</rt>注<rt>ㄓㄨˋ</rt>音<rt>ㄧㄣ</rt></ruby></span>
        <span v-else>🚫 <ruby>隱<rt>ㄧㄣˇ</rt>藏<rt>ㄘㄤˊ</rt>注<rt>ㄓㄨˋ</rt>音<rt>ㄧㄣ</rt></ruby></span>
      </button>
      
      <button @click="isDarkMode = !isDarkMode" class="tool-btn" title="切換深淺色模式">
        <span v-if="!isDarkMode">🌙 <ruby>深<rt>ㄕㄣ</rt>色<rt>ㄙㄜˋ</rt>模<rt>ㄇㄛˊ</rt>式<rt>ㄕˋ</rt></ruby></span>
        <span v-else>☀️ <ruby>淺<rt>ㄑㄧㄢˇ</rt>色<rt>ㄙㄜˋ</rt>模<rt>ㄇㄛˊ</rt>式<rt>ㄕˋ</rt></ruby></span>
      </button>
    </div>

    <!-- 時間顯示區 -->
    <div class="display-screen">
      <div class="time-label">
        <template v-if="isCountdownMode">
          <span v-if="countdownState === 'waiting'">⏳ <ruby>即<rt>ㄐㄧˊ</rt>將<rt>ㄐㄧㄤ</rt>開<rt>ㄎㄞ</rt>始<rt>ㄕˇ</rt>考<rt>ㄎㄠˇ</rt>試<rt>ㄕˋ</rt></ruby></span>
          <span v-else-if="countdownState === 'running'">⏳ <ruby>考<rt>ㄎㄠˇ</rt>試<rt>ㄕˋ</rt>剩<rt>ㄕㄥˋ</rt>餘<rt>ㄩˊ</rt>時<rt>ㄕˊ</rt>間<rt>ㄐㄧㄢ</rt></ruby></span>
          <span v-else-if="countdownState === 'ended'">🛑 <ruby>考<rt>ㄎㄠˇ</rt>試<rt>ㄕˋ</rt>已<rt>ㄧˇ</rt>結<rt>ㄐㄧㄝˊ</rt>束<rt>ㄕㄨˋ</rt></ruby></span>
          <span v-else>⏳ <ruby>考<rt>ㄎㄠˇ</rt>試<rt>ㄕˋ</rt>倒<rt>ㄉㄠˋ</rt>數<rt>ㄕㄨˇ</rt></ruby></span>
        </template>
        <template v-else>
          <span>🕒 <ruby>現<rt>ㄒㄧㄢˋ</rt>在<rt>ㄗㄞˋ</rt>時<rt>ㄕˊ</rt>間<rt>ㄐㄧㄢ</rt></ruby></span>
        </template>
      </div>
      <div class="time-value">
        {{ isCountdownMode ? formattedCountdown : currentTime }}
      </div>
    </div>

    <!-- 資料輸入表單區 -->
    <div class="form-card">
      <h2>📝 <ruby>新<rt>ㄒㄧㄣ</rt>增<rt>ㄗㄥ</rt>監<rt>ㄐㄧㄢ</rt>考<rt>ㄎㄠˇ</rt>紀<rt>ㄐㄧˋ</rt>錄<rt>ㄌㄨˋ</rt></ruby></h2>
      
      <div class="input-group">
        <label>⏰ <ruby>考<rt>ㄎㄠˇ</rt>試<rt>ㄕˋ</rt>時<rt>ㄕˊ</rt>間<rt>ㄐㄧㄢ</rt></ruby></label>
        <input type="text" v-model="formData.time" placeholder="10:00-12:00" />
      </div>
      
      <div class="input-group">
        <label>📚 <ruby>考<rt>ㄎㄠˇ</rt>試<rt>ㄕˋ</rt>科<rt>ㄎㄜ</rt>目<rt>ㄇㄨˋ</rt></ruby></label>
        <input type="text" v-model="formData.subject" placeholder="考試科目" />
      </div>
      
      <div class="input-group">
        <label>🧑‍🏫 <ruby>監<rt>ㄐㄧㄢ</rt>考<rt>ㄎㄠˇ</rt>老<rt>ㄌㄠˇ</rt>師<rt>ㄕ</rt></ruby></label>
        <input type="text" v-model="formData.teacher" placeholder="老師姓名" />
      </div>
      
      <button @click="submitData" class="submit-btn">
        ➕ <ruby>送<rt>ㄙㄨㄥˋ</rt>出<rt>ㄔㄨ</rt>紀<rt>ㄐㄧˋ</rt>錄<rt>ㄌㄨˋ</rt></ruby>
      </button>
    </div>

    <!-- 資料呈現區 -->
    <div class="records-container" v-if="records.length > 0">
      <h2>📋 <ruby>監<rt>ㄐㄧㄢ</rt>考<rt>ㄎㄠˇ</rt>紀<rt>ㄐㄧˋ</rt>錄<rt>ㄌㄨˋ</rt>列<rt>ㄌㄧㄝˋ</rt>表<rt>ㄅㄧㄠˇ</rt></ruby></h2>
      <div class="record-list">
        <div v-for="(record, index) in records" :key="index" class="record-item">
          <div class="record-field">⏰ {{ record.time }}</div>
          <div class="record-field">📚 {{ record.subject }}</div>
          <div class="record-field">🧑‍🏫 {{ record.teacher }}</div>
        </div>
      </div>
    </div>
  </div>
</template>

<style scoped>
/* 基本排版與過渡動畫 */
.app-container {
  min-height: 100vh;
  padding: 20px;
  font-family: "Segoe UI", "Microsoft JhengHei", sans-serif;
  transition: all 0.5s ease;
  background: linear-gradient(135deg, #fdfbfb 0%, #ebedee 100%);
  color: #333;
}

/* 暗色模式設定 */
.app-container.dark-mode {
  background: linear-gradient(135deg, #1c1c1c 0%, #2a2a2a 100%);
  color: #f0f0f0;
}
.app-container.dark-mode input {
  background-color: rgba(255, 255, 255, 0.1);
  color: #fff;
  border: 1px solid rgba(255, 255, 255, 0.2);
}
.app-container.dark-mode .form-card,
.app-container.dark-mode .record-item,
.app-container.dark-mode .display-screen {
  background-color: rgba(30, 30, 30, 0.6);
  border-color: rgba(255, 255, 255, 0.1);
  box-shadow: 0 8px 32px rgba(0, 0, 0, 0.3);
}

/* 注音隱藏設定 */
.hide-zhuyin rt {
  display: none;
}
ruby {
  ruby-align: center;
}

/* 頂部標題與按鈕 */
.app-header {
  display: flex;
  align-items: center;
  justify-content: center;
  position: relative;
  margin-bottom: 40px;
  padding: 10px 0;
}
.link-btn {
  position: absolute;
  left: 0;
  background: linear-gradient(45deg, #005a9c, #0088ff);
  color: white;
  text-decoration: none;
  padding: 10px 18px;
  border-radius: 20px;
  font-weight: bold;
  font-size: 1rem;
  box-shadow: 0 4px 15px rgba(0, 90, 156, 0.3);
  transition: transform 0.2s;
}
.link-btn:hover {
  transform: translateY(-2px);
}
.main-title {
  margin: 0;
  font-size: 2rem;
  text-align: center;
  text-shadow: 0 2px 4px rgba(0,0,0,0.1);
}

/* 工具列按鈕區 */
.toolbar {
  display: flex;
  gap: 15px;
  justify-content: center;
  margin-bottom: 40px;
  flex-wrap: wrap;
}
.tool-btn {
  padding: 12px 24px;
  font-size: 1.1rem;
  font-weight: bold;
  cursor: pointer;
  border: none;
  border-radius: 25px;
  background: rgba(255, 255, 255, 0.5);
  color: inherit;
  box-shadow: 0 4px 10px rgba(0, 0, 0, 0.05);
  backdrop-filter: blur(5px);
  transition: all 0.3s ease;
}
.app-container.dark-mode .tool-btn {
  background: rgba(0, 0, 0, 0.3);
}
.tool-btn:hover {
  transform: translateY(-3px);
  box-shadow: 0 6px 15px rgba(0, 0, 0, 0.1);
}

/* 時間顯示區 */
.display-screen {
  text-align: center;
  margin-bottom: 40px;
  padding: 40px;
  border-radius: 24px;
  background: rgba(255, 255, 255, 0.6);
  box-shadow: 0 8px 32px rgba(0, 0, 0, 0.05);
  backdrop-filter: blur(10px);
  border: 1px solid rgba(255, 255, 255, 0.5);
}
.time-label {
  font-size: 1.5rem;
  margin-bottom: 15px;
  opacity: 0.9;
}
.time-value {
  font-size: 5rem;
  font-weight: 900;
  font-family: 'Courier New', monospace;
  line-height: 1;
  background: linear-gradient(45deg, #ff6b6b, #4ecaec);
  -webkit-background-clip: text;
  -webkit-text-fill-color: transparent;
  text-shadow: 2px 2px 10px rgba(0,0,0,0.05);
}

/* 表單區域 */
.form-card {
  max-width: 500px;
  margin: 0 auto 40px auto;
  padding: 30px;
  background: rgba(255, 255, 255, 0.7);
  border-radius: 20px;
  box-shadow: 0 10px 30px rgba(0,0,0,0.05);
  backdrop-filter: blur(10px);
  border: 1px solid rgba(255, 255, 255, 0.5);
}
.form-card h2 {
  margin-top: 0;
  text-align: center;
  margin-bottom: 25px;
  font-size: 1.8rem;
}
.input-group {
  margin-bottom: 20px;
  display: flex;
  flex-direction: column;
  gap: 10px;
}
.input-group label {
  font-weight: bold;
  font-size: 1.1rem;
}
.input-group input {
  padding: 12px 15px;
  border-radius: 12px;
  border: 1px solid rgba(0,0,0,0.1);
  font-size: 1rem;
  background: rgba(255,255,255,0.9);
  transition: all 0.3s;
}
.input-group input:focus {
  outline: none;
  border-color: #4ecaec;
  box-shadow: 0 0 0 3px rgba(78, 202, 236, 0.2);
}
.submit-btn {
  width: 100%;
  padding: 14px;
  margin-top: 15px;
  background: linear-gradient(45deg, #4CAF50, #2E7D32);
  color: white;
  border: none;
  border-radius: 12px;
  font-size: 1.2rem;
  font-weight: bold;
  cursor: pointer;
  box-shadow: 0 4px 15px rgba(76, 175, 80, 0.3);
  transition: transform 0.2s, box-shadow 0.2s;
}
.submit-btn:hover {
  transform: translateY(-2px);
  box-shadow: 0 6px 20px rgba(76, 175, 80, 0.4);
}

/* 紀錄呈現區 */
.records-container {
  max-width: 700px;
  margin: 0 auto;
}
.records-container h2 {
  text-align: center;
  font-size: 1.8rem;
  margin-bottom: 25px;
}
.record-list {
  display: flex;
  flex-direction: column;
  gap: 15px;
}
.record-item {
  display: grid;
  grid-template-columns: 1fr 1fr 1fr;
  gap: 15px;
  padding: 20px;
  background: rgba(255, 255, 255, 0.7);
  border-radius: 15px;
  border: 1px solid rgba(255, 255, 255, 0.5);
  box-shadow: 0 4px 15px rgba(0,0,0,0.03);
  backdrop-filter: blur(10px);
  align-items: center;
}
.record-field {
  font-size: 1.1rem;
}

/* 響應式設計 */
@media (max-width: 768px) {
  .app-header {
    flex-direction: column;
    gap: 15px;
  }
  .link-btn {
    position: static;
  }
  .time-value {
    font-size: 3.5rem;
  }
  .record-item {
    grid-template-columns: 1fr;
    text-align: center;
  }
}
</style>
