<template>
  <div class="container">
    <!-- Header -->
    <header class="header">
      <h1>Samruddh Bharat Technologies</h1> <br />
    </header>
    <div class="date"><span class="tiffin-date">Tiffin Date: </span> {{ todayDisplay }}</div>

    <!-- Card -->
    <div class="card">
      <div v-if="isLoading" class="page-loader">
        <div class="spinner big"></div>
        <p>Loading tiffin counts...</p>
      </div>
      <div v-else>
        <p v-if="!canSubmitNow" style="color:red; margin-top:10px">
          ⏰ Submission: 9:00 AM – 1:00 PM. Timing matters 🙂
        </p>
        <h2>🍱 Select Your Tiffin</h2>

        <div
          class="option"
          :class="{ active: selected === 'Regular Tiffin', disabled: isDisabled }"
          @click="!isDisabled && selectOption('Regular Tiffin')"
        >
          Regular Tiffin
        </div>

        <div
          class="option"
          :class="{ active: selected === '1 Bhakari', disabled: isDisabled }"
          @click="!isDisabled && selectOption('1 Bhakari')"
        >
          1 Bhakari
        </div>

        <div
          class="option"
          :class="{ active: selected === '2 Bhakari', disabled: isDisabled }"
          @click="!isDisabled && selectOption('2 Bhakari')"
        >
          2 Bhakari
        </div>

        <div
          class="option"
          :class="{ active: selected === 'Sabudana Khichadi', disabled: isDisabled }"
          @click="!isDisabled && selectOption('Sabudana Khichadi')"
        >
          Sabudana Khichadi
        </div> 

        <button :disabled="isDisabled || isSubmitting || !canSubmitNow" @click="submit">
          <span v-if="isSubmitting" class="spinner"></span>
          <span v-else>
            {{ isDisabled ? 'Already Submitted' : 'Submit' }}
          </span>
        </button>

        <p v-if="submitted" class="success">
          ✅ Submitted successfully
        </p>

        <div class="counts">
          <p>Regular: {{ counts.regular }}</p>
          <p>1 Bhakari: {{ counts.one }}</p>
          <p>2 Bhakari: {{ counts.two }}</p>
          <p>Sabudana khichdi: {{ counts.khichadi }}</p>
          <p class="total">Total Tiffins: {{ totalTiffins }}</p>
        </div>
      </div>
    </div>
  </div>
</template>

<script setup>
import { ref, onMounted, computed } from 'vue'

/* ================= CONFIG ================= */
const API_URL =
  'https://script.google.com/macros/s/AKfycbymWn16bZ5vDjEJ4zt53Uxd5JTnTYUkcI0CuVAgjT9J3vCMyS0GuMlKi0uQfa8YpSrKOg/exec'

/* ================= DATE ================= */
const todayKey = new Date().toISOString().split('T')[0]
const todayDisplay = new Date().toLocaleDateString('en-IN', {
  day: '2-digit',
  month: 'short',
  year: 'numeric'
})

const storageKey = `tiffin_submitted_${todayKey}`

/* ================= STATE ================= */
const selected = ref('')
const submitted = ref(false)
const isDisabled = ref(false)
const isSubmitting = ref(false)   // submit button loader
const isLoading = ref(true)       // initial page loader

const counts = ref({
  regular: 0,
  one: 0,
  two: 0,
  khichadi: 0
})

/* ================= METHODS ================= */
function selectOption(value) {
  selected.value = value
}

const totalTiffins = computed(() => {
  return counts.value.regular + counts.value.one + counts.value.two + counts.value.khichadi
})

async function submit() {
  if (isDisabled.value || isSubmitting.value) return

  if (!selected.value) {
    alert('Please select one option')
    return
  }

  try {
    isSubmitting.value = true

    await fetch(API_URL, {
      method: 'POST',
      body: JSON.stringify({
        selection: selected.value
      })
    })

    localStorage.setItem(storageKey, 'true')
    localStorage.setItem('last_submit_date', todayKey)
    submitted.value = true
    isDisabled.value = true

    await loadCounts()
  } finally {
    isSubmitting.value = false
  }
}

const canSubmitNow = computed(() => {
  const h = new Date().getHours()
  return h >= 9 && h < 13
})


async function loadCounts() {
  isLoading.value = true
  try {
    const res = await fetch(API_URL, {
      method: 'POST'
    })
    const data = await res.json()

    counts.value = {
      regular: data.regular || 0,
      one: data.one || 0,
      two: data.two || 0,
      khichadi: data.khichadi || 0
    }
  } catch (e) {
    console.error('Count load failed', e)
  } finally {
    isLoading.value = false
  }
}


/* ================= INIT ================= */
onMounted(() => {
  const lastSubmitDate = localStorage.getItem('last_submit_date')
  if (lastSubmitDate === todayKey) {
    isDisabled.value = true
    submitted.value = true
  } else {
    localStorage.removeItem('last_submit_date')
    isDisabled.value = false
    submitted.value = false
  }
  loadCounts()
})

</script>
<style scoped>
.container {
  min-height: 100vh;
  background: linear-gradient(135deg, #0f172a, #1e293b);
  font-family: 'Segoe UI', Tahoma, Geneva, Verdana, sans-serif;
  padding: 1px 3px 10px 3px;
}

/* Header */
.header {
  padding: 15px 1px 0px 0px;
  background: linear-gradient(90deg, #c55622, #f1f3f2, #16a34a);
  color: #0a4fe4;
  text-align: center;
  box-shadow: 0 4px 15px rgba(0, 0, 0, 0.2);
}

.header h1 {
  font-size: 22px;
  margin: 0;
  letter-spacing: 0.5px;
}

.date {
  text-align: center;
  color: #e5e7eb;
  margin-top: 12px;
  font-size: 20px;
}

/* Card */
.card {
  background: rgba(255, 255, 255, 0.95);
  margin: 20px auto;
  padding: 30px;
  width: 360px;
  border-radius: 20px;
  box-shadow: 0 20px 40px rgba(0, 0, 0, 0.25);
  text-align: center;
  animation: fadeIn 0.6s ease;
}

.card h2 {
  margin-bottom: 20px;
  color: #111827;
}

/* Options */
.option {
  border: 2px solid #e5e7eb;
  border-radius: 14px;
  padding: 16px;
  margin-bottom: 14px;
  cursor: pointer;
  font-size: 16px;
  font-weight: 500;
  background: #f9fafb;
  transition: all 0.25s ease;
}

.option:hover {
  transform: scale(1.02);
  background: #ecfdf5;
}

.option.active {
  border-color: #16a34a;
  background: linear-gradient(135deg, #dcfce7, #bbf7d0);
  color: #065f46;
  font-weight: 600;
}

.option.disabled {
  opacity: 0.6;
  cursor: not-allowed;
}

/* Button */
button {
  width: 100%;
  padding: 14px;
  margin-top: 10px;
  background: linear-gradient(90deg, #22c55e, #16a34a);
  color: white;
  border: none;
  border-radius: 14px;
  font-size: 16px;
  font-weight: 600;
  cursor: pointer;
  transition: all 0.3s ease;
}

button:hover:not(:disabled) {
  transform: translateY(-1px);
  box-shadow: 0 6px 15px rgba(34, 197, 94, 0.4);
}

button:disabled {
  background: #9ca3af;
  cursor: not-allowed;
}

/* Success */
.success {
  margin-top: 14px;
  color: #15803d;
  font-weight: 600;
  background: #ecfdf5;
  padding: 10px;
  border-radius: 10px;
}

/* Counts Dashboard */
.counts {
  margin-top: 22px;
  display: grid;
  grid-template-columns: repeat(3, 1fr);
  gap: 10px;
}

.counts p {
  background: #f1f5f9;
  padding: 12px;
  border-radius: 12px;
  font-size: 14px;
  font-weight: 600;
  color: #1f2937;
  box-shadow: inset 0 0 5px rgba(0, 0, 0, 0.05);
}

/* Animation */
@keyframes fadeIn {
  from {
    opacity: 0;
    transform: translateY(10px);
  }
  to {
    opacity: 1;
    transform: translateY(0);
  }
}
.counts .total {
  grid-column: span 3; /* full width */
  background: #dcfce7;
  color: #065f46;
  font-weight: 700;
  text-align: center;
  padding: 12px;
  border-radius: 12px;
  margin-top: 10px;
  box-shadow: inset 0 0 5px rgba(0,0,0,0.05);
}
/* Spinner */
.spinner {
  width: 18px;
  height: 18px;
  border: 3px solid rgba(255,255,255,0.4);
  border-top: 3px solid #ffffff;
  border-radius: 50%;
  animation: spin 0.8s linear infinite;
  display: inline-block;
}

.spinner.big {
  width: 40px;
  height: 40px;
  border-width: 4px;
  border-top-color: #16a34a;
}

/* Page loader */
.page-loader {
  display: flex;
  flex-direction: column;
  align-items: center;
  gap: 12px;
  padding: 40px 0;
  color: #374151;
  font-weight: 600;
}

@keyframes spin {
  to {
    transform: rotate(360deg);
  }
}


</style>
