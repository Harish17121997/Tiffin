<template>
  <div class="page">
    <!-- Header -->
    <header class="top-bar">
      <h1>Samruddh Bharat Technologies</h1>
    </header>
    <div class="date-text">
      Tiffin Date: {{ todayDisplay }}
    </div>

    <div class="card">
      <div v-if="isLoading" class="page-loader">
        <div class="spinner big"></div>
        <p>Loading tiffin counts...</p>
      </div>
      <div v-else>
        <p v-if="!canSubmitNow" class="time-warning" style="color:red; margin-top:1px; text-align: center;">
          ⏰ Submission: 9:00 AM – 11:00 AM. Timing matters 🙂
        </p>
        <h2 class="title">🍱 Tiffin Selection</h2>
        <!-- Name -->
        <div>
          <label for="name" class="form-label">Name:<span class="required">*</span></label>
          <input class="field" type="text" placeholder="Enter your name" v-model="name"
            :disabled="isDisabled || !canSubmitNow" oninput="this.value = this.value.toUpperCase()" />
        </div>
        <!-- Select -->
        <div class="select-wrapper">
          <select class="field select" v-model="foodType" :disabled="isDisabled || !canSubmitNow">
            <option disabled value="">Select Tiffin Option</option>
            <option value="Regular Tiffin">Regular Tiffin</option>
            <option value="1 Poli">1 Poli</option>
            <option value="1 Bhakari">1 Bhakari</option>
            <option value="2 Bhakari">2 Bhakari</option>
            <option value="Sabudana Khichadi">Sabudana Khichadi</option>
          </select>

          <!-- custom arrow -->
          <span class="select-arrow">⌄</span>
        </div>


        <!-- Submit -->
        <button class="submit-btn"
          :disabled="isDisabled || isSubmitting || !canSubmitNow || foodType === '' || name === ''" @click="submit">
          <span v-if="isSubmitting" class="spinner"></span>
          <span v-else>
            {{ isDisabled ? 'Already Submitted' : 'Submit' }}
          </span>
        </button>

        <p v-if="submitted" class="success-msg">
          ✅ Submitted successfully
        </p>

        <div class="counts-section">
          <button class="camera-btn" @click="takeScreenshot">📸</button>
          <div class="counts-wrapper" ref="countsRef">
            <div class="screenshot-date"> 📅 {{ todayDisplay }}</div>
            <div class="counts">
              <div class="count-box">Regular: {{ counts.regular }}</div>
              <div class="count-box">1 Poli: {{ counts.poli }}</div>
              <div class="count-box">1 Bhakari: {{ counts.one }}</div>
              <div class="count-box">2 Bhakari: {{ counts.two }}</div>
              <div class="count-box">Khichadi: {{ counts.khichadi }}</div>
              <div class="count-box total">
                Total Tiffins: {{ totalTiffins }}
              </div>
            </div>
          </div>

          <!-- 👇 HERE (inside card) -->
          <div class="order-footer">
            <h3>Today's Orders <span style="color: red; padding-left: 10px;"> {{ totalTiffins }}</span></h3>

            <div v-if="orderNames.length === 0" class="no-orders">
              No orders yet
            </div>

            <ul v-else class="orders-list">
              <li v-for="(n, i) in orderNames" :key="i">
                {{ n }}
              </li>
            </ul>
          </div>
        </div>

      </div>
    </div>
  </div>
</template>

<script setup>
import { ref, computed, onMounted } from 'vue'
import html2canvas from 'html2canvas'

const API_URL =
  'https://script.google.com/macros/s/AKfycbymWn16bZ5vDjEJ4zt53Uxd5JTnTYUkcI0CuVAgjT9J3vCMyS0GuMlKi0uQfa8YpSrKOg/exec'

const todayDisplay = new Date().toLocaleDateString('en-IN', {
  day: '2-digit',
  month: 'short',
  year: 'numeric'
})

const name = ref('')
const foodType = ref('')
const isDisabled = ref(false)
const isSubmitting = ref(false)
const submitted = ref(false)
const isLoading = ref(true)
const nameError = ref('')
const orderNames = ref([])
const todayKey = new Date().toISOString().slice(0, 10)

const counts = ref({
  regular: 0,
  poli: 0,
  one: 0,
  two: 0,
  khichadi: 0
})

const countsRef = ref(null)

const totalTiffins = computed(() =>
  Object.values(counts.value).reduce((a, b) => a + b, 0)
)

const canSubmitNow = computed(() => {
  const h = new Date().getHours()
  return h >= 9 && h < 11
})

async function loadCounts() {
  isLoading.value = true
  try {
    const res = await fetch(API_URL, { method: 'POST' })
    const data = await res.json()

    counts.value = {
      regular: data.regular || 0,
      poli: data.poli || 0,
      one: data.one || 0,
      two: data.two || 0,
      khichadi: data.khichadi || 0
    }
    orderNames.value = data.names || []
  } finally {
    isLoading.value = false
  }
}


async function submit() {
  // validation first
  if (!name.value) {
    nameError.value = 'Name is required'
    return
  }
  isSubmitting.value = true

  try {
    await fetch(API_URL, {
      method: 'POST',
      body: JSON.stringify({
        name: name.value,
        selection: foodType.value
      })
    })

    submitted.value = true
    isDisabled.value = true
    localStorage.setItem(
      'tiffinSubmission',
      JSON.stringify({
        date: todayKey,
        name: name.value.trim().toUpperCase()
      })
    )

    // reload counts after submit
    await loadCounts()
  } catch (e) {
    console.error('Something went wrong:', e)
  } finally {
    isSubmitting.value = false
    nameError.value = ''
    foodType.value = ''
    name.value = ''
  }
}


async function takeScreenshot() {
  const canvas = await html2canvas(countsRef.value, {
    backgroundColor: '#ffffff',
    scale: 2
  })

  const link = document.createElement('a')
  link.download = 'Todays-tiffin-counts.png'
  link.href = canvas.toDataURL()
  link.click()
}

onMounted(async () => {
  const saved = localStorage.getItem('tiffinSubmission')
  if (saved) {
    const data = JSON.parse(saved)
    if (data.date === todayKey) {
      isDisabled.value = true
      submitted.value = true
    } else {
      localStorage.removeItem('tiffinSubmission')
    }
  }
  await loadCounts()
})
</script>
<style scoped>
/* ================= RESET ================= */
* {
  box-sizing: border-box;
  margin: 0;
  padding: 0;
}

/* ================= PAGE ================= */
.page {
  min-height: 100vh;
  background: linear-gradient(135deg, #0f172a, #1e293b);
  font-family: 'Segoe UI', Tahoma, Geneva, Verdana, sans-serif;
  padding: 1px 3px 10px;
}

/* ================= HEADER ================= */
.top-bar {
  padding: 15px 0;
  background: linear-gradient(90deg, #c55622, #f1f3f2, #16a34a);
  color: #0a4fe4;
  text-align: center;
  box-shadow: 0 4px 15px rgba(0, 0, 0, 0.2);
}

.top-bar h1 {
  font-size: 22px;
  margin: 0;
  letter-spacing: 0.5px;
}

/* ================= DATE ================= */
.date-text {
  text-align: center;
  color: #e5e7eb;
  margin-top: 12px;
  font-size: 20px;
}

/* ================= CARD ================= */
.card {
  background: linear-gradient(180deg, #ffffff, #f8fafc);
  box-shadow:
    0 10px 25px rgba(0, 0, 0, 0.12),
    inset 0 1px 0 rgba(255, 255, 255, 0.6);
  max-width: 360px;
  margin: 20px auto;
  padding: 30px;
  border-radius: 24px;
}

.title {
  text-align: center;
  margin-bottom: 18px;
}

/* ================= FORM FIELDS ================= */
.field {
  width: 100%;
  padding: 14px;
  margin-bottom: 14px;
  border-radius: 14px;
  border: 1.5px solid #d1d5db;
  font-size: 15px;
  transition: all 0.25s ease;
}

.field:focus {
  outline: none;
  border-color: #22b2c5;
  box-shadow: 0 0 0 3px rgba(34, 64, 197, 0.25);

}

/* ================= SELECT ================= */
.select-wrapper {
  position: relative;
  width: 100%;
  margin-bottom: 14px;
}

.select {
  appearance: none;
  -webkit-appearance: none;
  -moz-appearance: none;
  background: #e1e2fb;
  border-color: #22b2c5;
  padding-right: 42px;
  cursor: pointer;
}

/* Custom arrow */
.select-arrow {
  position: absolute;
  top: 35%;
  right: 16px;
  transform: translateY(-50%);
  pointer-events: none;
  font-size: 20px;
  color: #252525;
  font-weight: 700;
}

/* ================= BUTTON ================= */
.submit-btn {
  width: 100%;
  padding: 14px;
  border-radius: 15px;
  border: none;
  background: linear-gradient(120deg, #665a54, #1b0448);
  color: #ffffff;
  font-size: 16px;
  font-weight: 600;
  cursor: pointer;
  transition: all 0.3s ease;
}

.submit-btn:hover:not(:disabled) {
  transform: translateY(-1px);
  box-shadow: 0 15px 20px rgba(228, 50, 50, 0.35);
}

.submit-btn:disabled {
  background: linear-gradient(120deg, #926b58, #1b0448);
  cursor: not-allowed;
}

/* ================= SUCCESS MESSAGE ================= */
.success-msg {
  margin-top: 12px;
  text-align: center;
  color: #15803d;
  background: #ecfdf5;
  padding: 10px;
  border-radius: 10px;
}

/* ================= COUNTS ================= */
.counts-section {
  margin-top: 22px;
  position: relative;
}

.camera-btn {
  position: absolute;
  top: -14px;
  right: -14px;
  width: 34px;
  height: 34px;
  border-radius: 50%;
  border: none;
  background: #16a34a;
  color: #fff;
  font-size: 16px;
  cursor: pointer;
}

.counts-wrapper {
  background: #ffffff;
  padding: 14px;
  border-radius: 18px;
  border: 1px dashed #22c55e;
}

.screenshot-date {
  text-align: center;
  font-weight: 700;
  margin-bottom: 12px;
  color: #065f46;
  background: #dcfce7;
  padding: 8px;
  border-radius: 10px;
}

.counts {
  display: grid;
  grid-template-columns: repeat(2, 1fr);
  gap: 10px;
}

.count-box {
  background: #f1f5f9;
  padding: 12px;
  border-radius: 12px;
  text-align: center;
  font-weight: 600;
}

.total {
  grid-column: span 2;
  background: #dcfce7;
  color: #065f46;
}

.form-group {
  margin-bottom: 16px;
}

/* ===== LABEL ===== */
.form-label {
  display: block;
  margin-bottom: 6px;
  font-size: 14px;
  font-weight: 600;
  color: #374151;
}

.required {
  color: #dc2626;
  margin-left: 2px;
}

/* ===== ERROR TEXT ===== */
.error-text {
  margin-top: 6px;
  font-size: 12.5px;
  color: #dc2626;
  font-weight: 500;
}

/* ===== ERROR INPUT ===== */
.field.error {
  border-color: #dc2626;
  background: #fef2f2;
  box-shadow: 0 0 0 3px rgba(220, 38, 38, 0.18);
}

.page-loader {
  display: flex;
  flex-direction: column;
  align-items: center;
  padding: 30px 0;
  color: #374151;
  font-weight: 600;
}

.spinner.big {
  width: 45px;
  height: 45px;
  border-width: 4px;
  border-top-color: #3e81b8;
  margin-bottom: 20px;
}

.spinner {
  width: 18px;
  height: 18px;
  border: 3px solid rgba(255, 255, 255, 0.4);
  border-top: 3px solid #ffffff;
  border-radius: 50%;
  animation: spin 0.7s linear infinite;
  display: inline-block;
}

@keyframes spin {
  to {
    transform: rotate(360deg);
  }
}

/*  ===== ORDER FOOTER ===== */
.order-footer {
  position: fixed;
  right: 15px;
  bottom: 15px;
  width: 250px;
  background: #ffffff;
  border-radius: 16px;
  padding: 14px;
  border: 2px solid #22c55e;
  box-shadow: 0 18px 40px rgba(0, 0, 0, 0.25);
  z-index: 999;

}

/* heading */
.order-footer h3 {
  text-align: center;
  font-size: 14px;
  margin-bottom: 8px;
  color: #065f46;
  font-weight: 700;
}

/* list */
.orders-list {
  list-style: none;
  padding: 0;
  margin: 0;
  max-height: 230px;
  overflow-y: auto;
  scrollbar-width: thin;
}

/* each name row */
.orders-list li {
  padding: 6px 8px;
  font-size: 13px;
  border-bottom: 1px dashed #cbd5e1;
}

/* empty text */
.no-orders {
  text-align: center;
  color: #64748b;
  font-size: 13px;
}

/* Mobile view */
@media (max-width: 768px) {

  .order-footer {
    position: static;
    width: 100%;
    margin-top: 16px;
    right: auto;
    top: auto;
  }

}
</style>
