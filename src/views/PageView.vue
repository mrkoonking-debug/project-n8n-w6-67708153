<template>
  <div class="page">
    <section class="hero">
      <p class="hero-label">ข้อมูลเรียลไทม์</p>
      <h1 class="hero-title">รายการเบิกอุปกรณ์</h1>
      <p class="hero-desc">ดึงข้อมูลจาก Google Sheets ผ่าน n8n Webhook API</p>
    </section>

    <div class="container">
      <div class="card">
        <div class="toolbar">
          <div class="toolbar-left">
            <span class="toolbar-title">รายการเบิกอุปกรณ์</span>
            <span class="count-badge">{{ items.length }}</span>
          </div>
          <button class="btn-sm" @click="fetchData" :disabled="loading">{{ loading ? 'โหลด...' : '↻ อัปเดต' }}</button>
        </div>

        <div v-if="loading" class="state"><div class="spinner"></div><p>กำลังโหลด...</p></div>
        <div v-else-if="error" class="state"><p class="err-text">⚠ {{ error }}</p><button class="btn-sm" @click="fetchData">ลองอีกครั้ง</button></div>
        <div v-else-if="items.length === 0" class="state"><p>📋 ยังไม่มีข้อมูลการเบิก</p></div>

        <div v-else class="table-wrap">
          <table>
            <thead>
              <tr>
                <th>#</th>
                <th>ชื่อ-นามสกุล</th>
                <th>แผนก</th>
                <th>รายการอุปกรณ์</th>
                <th>จำนวน</th>
                <th class="r">วันที่เบิก</th>
              </tr>
            </thead>
            <tbody>
              <tr v-for="(item, i) in items" :key="i">
                <td class="muted">{{ i + 1 }}</td>
                <td class="bold">{{ item.fullname }}</td>
                <td><span class="chip" :class="getDeptClass(item.department)">{{ item.department }}</span></td>
                <td><span class="equip-tag">{{ item.equipment }}</span></td>
                <td><span class="qty-badge">{{ item.quantity }}</span></td>
                <td class="r muted small">{{ formatDate(item.date) }}</td>
              </tr>
            </tbody>
          </table>
        </div>
      </div>
    </div>
  </div>
</template>

<script setup>
import { ref, onMounted } from 'vue'

const items = ref([])
const loading = ref(false)
const error = ref(null)

const formatDate = (d) => {
  if (!d) return '-'
  const date = new Date(d)
  return isNaN(date.getTime()) ? d : date.toLocaleDateString('th-TH', { year:'numeric', month:'short', day:'numeric', hour:'2-digit', minute:'2-digit' })
}

const getDeptClass = (dept) => {
  if (!dept) return ''
  if (dept.includes('บุคคล')) return 'chip-purple'
  if (dept.includes('บัญชี')) return 'chip-green'
  if (dept.includes('ไอที')) return 'chip-blue'
  if (dept.includes('การตลาด')) return 'chip-orange'
  if (dept.includes('ประชาสัมพันธ์')) return 'chip-pink'
  return ''
}

const fetchData = async () => {
  loading.value = true; error.value = null
  try {
    const res = await fetch('http://localhost:5678/webhook/equipment')
    if (!res.ok) throw new Error()
    items.value = await res.json()
  } catch { error.value = 'ตรวจสอบว่า Workflow ใน n8n เปิดใช้งานอยู่หรือไม่' }
  finally { loading.value = false }
}

onMounted(() => fetchData())
</script>

<style scoped>
.page { min-height: calc(100vh - 52px); background: #fbfbfd; }
.hero { text-align: center; padding: 48px 24px 24px; }
.hero-label { font-size: 0.8rem; font-weight: 600; color: #0071e3; text-transform: uppercase; letter-spacing: 0.04em; margin-bottom: 8px; }
.hero-title { font-size: 2.2rem; font-weight: 700; color: #1d1d1f; letter-spacing: -0.03em; margin-bottom: 8px; }
.hero-desc { font-size: 0.95rem; color: #86868b; }

.container { max-width: 920px; margin: 0 auto; padding: 0 24px 60px; }

.card {
  background: #fff; border: 1px solid #e8e8ed; border-radius: 18px;
  overflow: hidden; box-shadow: 0 2px 12px rgba(0,0,0,0.04);
}

.toolbar { display: flex; justify-content: space-between; align-items: center; padding: 18px 24px; border-bottom: 1px solid #f0f0f5; }
.toolbar-left { display: flex; align-items: center; gap: 10px; }
.toolbar-title { font-size: 0.9rem; font-weight: 700; color: #1d1d1f; }
.count-badge { font-size: 0.72rem; font-weight: 700; color: #0071e3; background: #e8f0fe; padding: 2px 10px; border-radius: 10px; }

.btn-sm {
  font-size: 0.8rem; font-weight: 600; font-family: 'Inter', sans-serif;
  padding: 6px 14px; border-radius: 8px; border: 1px solid #d2d2d7;
  background: #fff; color: #1d1d1f; cursor: pointer; transition: all 0.15s;
}
.btn-sm:hover:not(:disabled) { background: #f5f5f7; }
.btn-sm:disabled { opacity: 0.4; }

.state { padding: 60px 24px; text-align: center; color: #86868b; font-size: 0.9rem; }
.err-text { color: #ff3b30; margin-bottom: 12px; }
.spinner { width: 24px; height: 24px; border: 2.5px solid #e8e8ed; border-top-color: #0071e3; border-radius: 50%; animation: spin 0.7s linear infinite; margin: 0 auto 12px; }
@keyframes spin { to { transform: rotate(360deg); } }

.table-wrap { overflow-x: auto; }
table { width: 100%; border-collapse: collapse; }
th { text-align: left; padding: 12px 24px; font-size: 0.72rem; font-weight: 700; color: #86868b; text-transform: uppercase; letter-spacing: 0.04em; background: #fafafa; border-bottom: 1px solid #f0f0f5; }
td { padding: 14px 24px; font-size: 0.88rem; border-bottom: 1px solid #f5f5f7; }
tr:hover { background: #fafbfc; }
.r { text-align: right; }
.muted { color: #86868b; }
.small { font-size: 0.78rem; }
.bold { font-weight: 600; color: #1d1d1f; }

.equip-tag { font-size: 0.82rem; font-weight: 600; color: #0071e3; background: #e8f0fe; padding: 3px 10px; border-radius: 6px; }
.qty-badge { font-size: 0.82rem; font-weight: 700; color: #ff9500; background: #fff3e0; padding: 3px 10px; border-radius: 6px; font-family: 'SF Mono', ui-monospace, monospace; }

.chip { padding: 3px 10px; border-radius: 6px; font-size: 0.78rem; font-weight: 600; }
.chip-purple { background: #f3e8fd; color: #7b2ff2; }
.chip-green { background: #e6f4ea; color: #137333; }
.chip-blue { background: #e8f0fe; color: #0055b3; }
.chip-orange { background: #fff3e0; color: #e65100; }
.chip-pink { background: #fce4ec; color: #c2185b; }

@media (max-width: 768px) {
  .hero-title { font-size: 1.6rem; }
  th, td { padding: 10px 16px; font-size: 0.82rem; }
}
</style>
