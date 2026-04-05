<template>
  <div class="page">
    <section class="hero">
      <p class="hero-label">ระบบลงทะเบียน</p>
      <h1 class="hero-title">ลงทะเบียนสมาชิก</h1>
      <p class="hero-desc">กรอกข้อมูลของคุณเพื่อบันทึกลง Google Sheets ผ่าน n8n</p>
    </section>

    <div class="container">
      <div class="card">
        <transition name="fade">
          <div v-if="status.message" class="alert" :class="status.type === 'success' ? 'alert-ok' : 'alert-fail'">
            {{ status.type === 'success' ? '✓' : '✕' }} {{ status.message }}
          </div>
        </transition>

        <form @submit.prevent="submitForm">
          <div class="field">
            <label>รหัสนักศึกษา</label>
            <input v-model="data.id" placeholder="Ex. 6401234567" required />
          </div>
          <div class="field">
            <label>ชื่อ-นามสกุล</label>
            <input v-model="data.fullname" placeholder="ชื่อ นามสกุล" required />
          </div>
          <div class="field">
            <label>คณะ / แผนก</label>
            <select v-model="data.department" required>
              <option value="" disabled selected>เลือกคณะของคุณ</option>
              <option value="บริหารธุรกิจ">บริหารธุรกิจ</option>
              <option value="บัญชี">บัญชี</option>
              <option value="เทคโนโลยีสารสนเทศ">เทคโนโลยีสารสนเทศ</option>
            </select>
          </div>
          <button class="btn" :disabled="loading" type="submit">
            {{ loading ? 'กำลังดำเนินการ...' : 'ลงทะเบียน' }}
          </button>
        </form>
      </div>
    </div>
  </div>
</template>

<script setup>
import { reactive, ref } from "vue";
const data = reactive({ id: "", fullname: "", department: "" });
const loading = ref(false);
const status = reactive({ message: "", type: "" });

const submitForm = async () => {
  loading.value = true; status.message = "";
  try {
    const response = await fetch("http://localhost:5678/webhook/register", {
      method: "POST", headers: { "Content-Type": "application/json" },
      body: JSON.stringify({ ...data, tdate: new Date().toISOString() })
    });
    if (!response.ok) throw new Error("Network error");
    status.message = "ลงทะเบียนเสร็จสมบูรณ์"; status.type = "success";
    data.id = ""; data.fullname = ""; data.department = "";
    setTimeout(() => { status.message = ""; }, 3000);
  } catch (e) {
    status.message = "ไม่สามารถลงทะเบียนได้ โปรดลองอีกครั้ง"; status.type = "danger";
  } finally { loading.value = false; }
};
</script>

<style scoped>
.page { min-height: calc(100vh - 52px); background: #fff; }
.hero { text-align: center; padding: 56px 24px 28px; }
.hero-label { font-size: 0.8rem; font-weight: 600; color: #0071e3; text-transform: uppercase; letter-spacing: 0.04em; margin-bottom: 8px; }
.hero-title { font-size: 2.5rem; font-weight: 700; color: #1d1d1f; letter-spacing: -0.03em; margin-bottom: 8px; }
.hero-desc { font-size: 1rem; color: #86868b; }

.container { max-width: 440px; margin: 0 auto; padding: 0 24px 60px; }

.card {
  background: #fff; border: 1px solid #e8e8ed; border-radius: 18px;
  padding: 32px; box-shadow: 0 2px 12px rgba(0,0,0,0.04);
}

.field { margin-bottom: 20px; }
.field label { display: block; font-size: 0.82rem; font-weight: 600; color: #1d1d1f; margin-bottom: 6px; }

.field input, .field select {
  width: 100%; padding: 12px 14px; font-size: 0.95rem; font-family: 'Inter', sans-serif;
  border: 1px solid #d2d2d7; border-radius: 12px; color: #1d1d1f;
  background: #fff; outline: none; transition: all 0.2s ease;
  -webkit-appearance: none; appearance: none;
}
.field input::placeholder { color: #aeaeb2; }
.field input:focus, .field select:focus { border-color: #0071e3; box-shadow: 0 0 0 4px rgba(0,113,227,0.1); }

.field select {
  background-image: url("data:image/svg+xml;charset=UTF-8,%3csvg xmlns='http://www.w3.org/2000/svg' viewBox='0 0 24 24' fill='none' stroke='%2386868b' stroke-width='2'%3e%3cpolyline points='6 9 12 15 18 9'/%3e%3c/svg%3e");
  background-repeat: no-repeat; background-position: right 12px center; background-size: 16px;
  cursor: pointer;
}

.btn {
  width: 100%; padding: 13px; font-size: 0.95rem; font-weight: 600;
  font-family: 'Inter', sans-serif; border: none; border-radius: 12px;
  background: #0071e3; color: #fff; cursor: pointer;
  transition: all 0.2s ease; margin-top: 4px;
}
.btn:hover:not(:disabled) { background: #0077ED; }
.btn:active:not(:disabled) { transform: scale(0.98); }
.btn:disabled { opacity: 0.5; cursor: not-allowed; }

.alert { padding: 12px 16px; border-radius: 12px; font-size: 0.85rem; font-weight: 500; margin-bottom: 20px; }
.alert-ok { background: #e8f5e9; color: #1b5e20; }
.alert-fail { background: #ffeef0; color: #c62828; }

.fade-enter-active { transition: all 0.3s ease; }
.fade-leave-active { transition: all 0.2s ease; }
.fade-enter-from, .fade-leave-to { opacity: 0; transform: translateY(-8px); }

@media (max-width: 600px) { .hero-title { font-size: 1.8rem; } .card { padding: 24px 20px; } }
</style>