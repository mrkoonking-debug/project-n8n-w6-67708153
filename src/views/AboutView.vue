<template>
  <div class="apple-dashboard py-5">
    
    <!-- Header with Subtle Gradient -->
    <div class="text-center mb-5 fade-in">
      <h2 class="apple-title gradient-text">ระบบแสดงรายชื่อผู้ลงทะเบียน</h2>
      <p class="apple-subtitle">ข้อมูลเรียลไทม์จาก n8n Webhook API</p>
    </div>

    <!-- Main Card (Window style) -->
    <div class="container">
      <div class="apple-window">
        
        <!-- Toolbar -->
        <div class="apple-toolbar d-flex flex-column flex-md-row justify-content-between align-items-center">
          <h5 class="mb-0 fw-semibold text-dark d-flex align-items-center gap-2">
            รายการข้อมูล 
            <span class="apple-badge badge-blue">{{ users.length }}</span>
          </h5>
          <button class="apple-btn d-flex align-items-center gap-2 mt-3 mt-md-0" @click="fetchData" :disabled="loading">
            <span :class="{'spin-icon': loading}">↻</span> 
            {{ loading ? 'กำลังโหลด...' : 'อัปเดตข้อมูล' }}
          </button>
        </div>

        <!-- Content Area -->
        <div class="apple-content">
          <!-- State: Loading -->
          <transition name="fade" mode="out-in">
            <div v-if="loading" class="state-container text-center">
              <div class="spinner-border text-primary mb-3" role="status" style="width: 2rem; height: 2rem; border-width: 0.2em; color: #0071e3 !important;"></div>
              <h6 class="apple-subtitle fw-medium" style="color: #0071e3;">กำลังซิงค์ข้อมูล...</h6>
            </div>

            <!-- State: Error -->
            <div v-else-if="error" class="state-container text-center">
              <div class="state-icon error-glow mb-3">🚨</div>
              <h5 class="text-danger fw-semibold">ไม่สามารถเชื่อมต่อได้</h5>
              <p class="apple-subtitle">{{ error }}</p>
              <button class="apple-btn-danger btn-sm mt-3" @click="fetchData">ลองอีกครั้ง</button>
            </div>

            <!-- State: Data Ready (Table) -->
            <div v-else-if="users.length > 0" class="table-responsive">
              <table class="apple-table">
                <thead>
                  <tr>
                    <th width="5%">#</th>
                    <th width="20%">รหัสนักศึกษา</th>
                    <th width="30%">ชื่อ-นามสกุล</th>
                    <th width="20%">แผนก</th>
                    <th width="25%" class="text-end">วันที่ลงทะเบียน</th>
                  </tr>
                </thead>
                <transition-group name="list" tag="tbody">
                  <tr v-for="(item, index) in users" :key="item.id || index">
                    <td class="text-muted fw-medium">{{ index + 1 }}</td>
                    <td><span class="apple-tag">{{ item.id }}</span></td>
                    <td class="fw-semibold text-dark">{{ item.fullname }}</td>
                    <td>
                      <!-- Color Pill for Department -->
                      <span class="dept-pill" :class="getDepartmentStyle(item.department)">
                        {{ item.department }}
                      </span>
                    </td>
                    <td class="text-muted text-end" style="font-size: 0.85rem;">{{ formatDate(item.tdate) }}</td>
                  </tr>
                </transition-group>
              </table>
            </div>

            <!-- State: Empty -->
            <div v-else class="state-container text-center">
              <div class="state-icon empty-glow mb-3">✨</div>
              <h5 class="fw-semibold text-dark">ยังไม่มีผู้ลงทะเบียน</h5>
              <p class="apple-subtitle">ข้อมูลจะปรากฏที่นี่เมื่อมีผู้ส่งฟอร์ม</p>
            </div>
          </transition>
        </div>

      </div>
    </div>
  </div>
</template>

<script setup>
import { ref, onMounted } from 'vue'

const users = ref([])
const loading = ref(false)
const error = ref(null)

const formatDate = (dateString) => {
  // 1. ถ้าไม่มีข้อมูลส่งมา ให้โชว์ขีด
  if (!dateString) return '-'

  // 2. ลองให้ JS แปลงเป็น Date Object
  const date = new Date(dateString)

  // 3. ตรวจสอบว่า JS อ่านค่านี้รู้เรื่องไหม?
  if (isNaN(date.getTime())) {
    // ❌ ถ้าอ่านไม่รู้เรื่อง (Invalid Date) ให้ return ค่าดิบๆ ที่ n8n ส่งมาเลย
    // อย่างน้อยผู้ใช้ก็ยังเห็นข้อมูล เช่น "25/10/2567 14:30" 
    return dateString 
  }

  // ✅ ถ้าอ่านรู้เรื่อง ก็จัด Format ให้สวยงามตามสไตล์ Apple
  const options = { 
    year: 'numeric', 
    month: 'short', 
    day: 'numeric', 
    hour: '2-digit', 
    minute: '2-digit' 
  }
  return date.toLocaleDateString('th-TH', options)
}

// Logic กำหนดสีตามแผนก (Color Coding)
const getDepartmentStyle = (dept) => {
  if (!dept) return 'pill-default'
  const d = dept.toLowerCase()
  
  if (d.includes('บริหาร') || d.includes('business')) return 'pill-purple'
  if (d.includes('บัญชี') || d.includes('account')) return 'pill-green'
  if (d.includes('ไอที') || d.includes('เทคโนโลยี') || d.includes('it')) return 'pill-blue'
  if (d.includes('การตลาด') || d.includes('market')) return 'pill-orange'
  
  return 'pill-default'
}

const fetchData = async () => {
  loading.value = true
  error.value = null
  
  try {
    const response = await fetch('http://localhost:5678/webhook-test/data')
    if (!response.ok) throw new Error(`HTTP error! status: ${response.status}`)
    const data = await response.json()
    users.value = data
  } catch (err) {
    console.error('Fetch Error:', err)
    error.value = 'ตรวจสอบ Webhook หรือการเชื่อมต่ออินเทอร์เน็ตของคุณ'
  } finally {
    loading.value = false
  }
}

onMounted(() => {
  fetchData()
})
</script>

<style scoped>
/* พื้นหลังและ Font สไตล์ Apple */
.apple-dashboard {
  min-height: 100vh;
  background-color: #f5f5f7;
  font-family: -apple-system, BlinkMacSystemFont, "SF Pro Text", "Helvetica Neue", Helvetica, Arial, sans-serif;
  -webkit-font-smoothing: antialiased;
}

/* Typography แบบมีสีสัน (Gradient) */
.apple-title {
  font-size: 2.2rem;
  font-weight: 800;
  letter-spacing: -0.02em;
  margin-bottom: 0.5rem;
}

.gradient-text {
  background: linear-gradient(135deg, #0071e3 0%, #34c759 100%);
  -webkit-background-clip: text;
  -webkit-text-fill-color: transparent;
  display: inline-block;
}

.apple-subtitle {
  font-size: 1.05rem;
  color: #86868b;
  font-weight: 500;
}

/* กรอบหน้าต่าง (Window/Card) */
.apple-window {
  background: #ffffff;
  border-radius: 20px;
  box-shadow: 0 10px 40px rgba(0, 0, 0, 0.08), 0 1px 3px rgba(0, 0, 0, 0.03);
  overflow: hidden;
  border: 1px solid rgba(0, 0, 0, 0.04);
}

.apple-toolbar {
  padding: 24px;
  border-bottom: 1px solid #f0f0f5;
  background-color: #ffffff;
}

.apple-content {
  padding: 0;
  min-height: 400px;
}

/* ปุ่มหลัก (Apple System Blue) */
.apple-btn {
  background-color: #0071e3;
  color: #ffffff;
  border: none;
  border-radius: 10px;
  padding: 10px 20px;
  font-size: 0.95rem;
  font-weight: 600;
  cursor: pointer;
  transition: all 0.2s ease;
  box-shadow: 0 2px 8px rgba(0, 113, 227, 0.2);
}
.apple-btn:hover:not(:disabled) {
  background-color: #0077ed;
  transform: translateY(-1px);
  box-shadow: 0 4px 12px rgba(0, 113, 227, 0.3);
}
.apple-btn:active:not(:disabled) {
  transform: translateY(0);
}
.apple-btn:disabled {
  opacity: 0.6;
  box-shadow: none;
  cursor: not-allowed;
}

.apple-btn-danger {
  background-color: #ff3b30;
  color: white;
  border: none;
  border-radius: 8px;
  padding: 8px 16px;
  font-weight: 600;
  transition: all 0.2s;
}
.apple-btn-danger:hover {
  background-color: #ff453a;
  box-shadow: 0 4px 12px rgba(255, 59, 48, 0.3);
}

/* Badge & Tag */
.apple-badge {
  padding: 4px 12px;
  border-radius: 12px;
  font-size: 0.9rem;
  font-weight: 700;
}
.badge-blue {
  background-color: #e5f0ff;
  color: #0071e3;
}

.apple-tag {
  background-color: #f2f2f7;
  padding: 4px 8px;
  border-radius: 6px;
  font-size: 0.85rem;
  color: #1d1d1f;
  font-family: ui-monospace, SFMono-Regular, Menlo, Monaco, Consolas, monospace;
}

/* Department Color Pills */
.dept-pill {
  padding: 6px 12px;
  border-radius: 8px;
  font-size: 0.85rem;
  font-weight: 600;
  display: inline-block;
}
.pill-purple { background-color: #f2e6ff; color: #af52de; }
.pill-green { background-color: #e5f5ea; color: #34c759; }
.pill-blue { background-color: #e5f0ff; color: #0071e3; }
.pill-orange { background-color: #ffe8d6; color: #ff9500; }
.pill-default { background-color: #f2f2f7; color: #86868b; }

/* ตารางแบบ Apple (Clean List) */
.apple-table {
  width: 100%;
  border-collapse: collapse;
}
.apple-table th {
  text-align: left;
  padding: 16px 24px;
  font-size: 0.8rem;
  font-weight: 600;
  color: #86868b;
  text-transform: uppercase;
  letter-spacing: 0.05em;
  border-bottom: 1px solid #f0f0f5;
  background-color: #fafafc;
}
.apple-table td {
  padding: 18px 24px;
  font-size: 0.95rem;
  border-bottom: 1px solid #f0f0f5;
  vertical-align: middle;
}
.apple-table tbody tr {
  transition: all 0.2s ease;
}
/* Hover Effect ที่มีสีสันขึ้น */
.apple-table tbody tr:hover {
  background-color: #f8fbff;
  transform: scale(1.002);
}

/* State & Icons */
.state-container {
  display: flex;
  flex-direction: column;
  justify-content: center;
  align-items: center;
  height: 400px;
}
.state-icon {
  font-size: 3.5rem;
}
.error-glow {
  filter: drop-shadow(0 0 15px rgba(255, 59, 48, 0.4));
}
.empty-glow {
  filter: drop-shadow(0 0 15px rgba(255, 204, 0, 0.4));
}

.spin-icon {
  display: inline-block;
  animation: spin 1s linear infinite;
}
@keyframes spin { 100% { transform: rotate(360deg); } }

/* Transitions */
.fade-enter-active, .fade-leave-active { transition: opacity 0.25s ease; }
.fade-enter-from, .fade-leave-to { opacity: 0; }

.list-enter-active, .list-leave-active { transition: all 0.4s cubic-bezier(0.16, 1, 0.3, 1); }
.list-enter-from { opacity: 0; transform: translateY(15px); }
.list-leave-to { opacity: 0; transform: translateX(-20px); }
</style>