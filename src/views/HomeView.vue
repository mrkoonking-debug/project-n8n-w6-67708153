<template>
    <div class="apple-container py-5">
        <div class="row justify-content-center m-0">
            <div class="col-12 col-md-8 col-lg-5">

                <!-- Apple Style Card -->
                <div class="apple-card">

                    <!-- Header -->
                    <div class="text-center mb-4">
                        <div class="apple-icon mb-3">ลงทะเบียน</div>
                        <p class="apple-subtitle">กรอกข้อมูลของคุณเพื่อดำเนินการต่อ</p>
                    </div>

                    <!-- Body -->
                    <div>
                        <!-- Smooth Alert -->
                        <transition name="fade">
                            <div v-if="status.message" 
                                 class="apple-alert mb-4" 
                                 :class="status.type === 'success' ? 'alert-success' : 'alert-error'">
                                {{ status.message }}
                            </div>
                        </transition>

                        <!-- Form -->
                        <form @submit.prevent="submitForm">

                            <div class="mb-3">
                                <label class="apple-label">รหัสนักศึกษา</label>
                                <input type="text" class="apple-input" v-model="data.id" placeholder="Ex. 6401234567" required>
                            </div>

                            <div class="mb-3">
                                <label class="apple-label">ชื่อ-นามสกุล</label>
                                <input type="text" class="apple-input" v-model="data.fullname" placeholder="ชื่อ นามสกุล" required>
                            </div>

                            <div class="mb-4">   
                                <label class="apple-label">คณะ / แผนก</label>
                                <select class="apple-input apple-select" v-model="data.department" required>
                                    <option value="" disabled selected>เลือกคณะของคุณ</option>
                                    <option value="บริหารธุรกิจ">บริหารธุรกิจ</option>
                                    <option value="บัญชี">บัญชี</option>
                                    <option value="เทคโนโลยีสารสนเทศ">เทคโนโลยีสารสนเทศ</option>
                                </select>
                            </div>

                            <button class="apple-btn" :disabled="loading">
                                <span v-if="loading" class="spinner-border spinner-border-sm me-2"></span>
                                {{ loading ? 'กำลังดำเนินการ...' : 'ลงทะเบียน' }}
                            </button>

                        </form>
                    </div>

                </div>

            </div>
        </div>
    </div>
</template>

<script setup>
import { reactive, ref } from "vue";

const data = reactive({
    id: "",
    fullname: "",
    department: ""
});

const loading = ref(false);

const status = reactive({
    message: "",
    type: ""
});

const submitForm = async () => {
    loading.value = true;
    status.message = "";

    try {
        const response = await fetch("http://localhost:5678/webhook-test/register", {
            method: "POST",
            headers: {
                "Content-Type": "application/json"
            },
            body: JSON.stringify({
                ...data,
                // แนะนำ: ส่งเป็น ISOString ตัวเต็มไปเลย แล้วให้ฝั่ง n8n จัดการ Format 
                // เพราะใช้ split('T')[0] ระวังเจอปัญหา Timezone offset ของไทย 
                tdate: new Date().toISOString() 
            })
        });

        if (!response.ok) throw new Error("Network response was not ok");

        status.message = "ลงทะเบียนเสร็จสมบูรณ์";
        status.type = "success";

        // reset form
        data.id = "";
        data.fullname = "";
        data.department = "";

        // ซ่อนข้อความอัตโนมัติหลังผ่านไป 3 วินาที (ให้ฟีลเหมือน Toast ของ iOS)
        setTimeout(() => {
            status.message = "";
        }, 3000);

    } catch (error) {
        console.error(error);
        status.message = "ไม่สามารถลงทะเบียนได้ในขณะนี้ โปรดลองอีกครั้ง";
        status.type = "danger";
    } finally {
        loading.value = false;
    }
};
</script>

<style scoped>
/* 
  Apple Design System (Cupertino)
  เน้นความเรียบง่าย พื้นที่ว่าง และฟอนต์ที่สะอาดตา
*/

.apple-container {
    background-color: #f5f5f7; /* สีพื้นหลังเอกลักษณ์ของ Apple */
    min-height: 100vh;
    font-family: -apple-system, BlinkMacSystemFont, "SF Pro Display", "Segoe UI", Roboto, Helvetica, Arial, sans-serif;
}

.apple-card {
    background: #ffffff;
    border-radius: 20px;
    padding: 40px;
    box-shadow: 0 4px 24px rgba(0, 0, 0, 0.04); /* เงาฟุ้งๆ บางๆ */
}

.apple-icon {
    font-size: 2.5rem;
    color: #1d1d1f;
    line-height: 1;
}

.apple-title {
    font-size: 1.75rem;
    font-weight: 600;
    color: #1d1d1f;
    letter-spacing: -0.02em;
    margin-bottom: 0.5rem;
}

.apple-subtitle {
    font-size: 0.95rem;
    color: #86868b;
}

/* Typography สำหรับ Label */
.apple-label {
    display: block;
    font-size: 0.85rem;
    font-weight: 500;
    color: #1d1d1f;
    margin-bottom: 0.5rem;
}

/* Inputs สไตล์ Apple */
.apple-input {
    width: 100%;
    background-color: #f5f5f7; /* พื้นเทาอ่อน ไม่มีขอบ */
    border: 1px solid transparent;
    border-radius: 12px;
    padding: 14px 16px;
    font-size: 1rem;
    color: #1d1d1f;
    transition: all 0.2s ease;
}

.apple-input::placeholder {
    color: #86868b;
}

.apple-input:focus {
    outline: none;
    background-color: #ffffff;
    border-color: #0071e3;
    box-shadow: 0 0 0 4px rgba(0, 113, 227, 0.15); /* Focus ring สีฟ้า */
}

/* Select Box ทำให้ดูเนียนไปกับ Input */
.apple-select {
    appearance: none;
    background-image: url("data:image/svg+xml;charset=UTF-8,%3csvg xmlns='http://www.w3.org/2000/svg' viewBox='0 0 24 24' fill='none' stroke='%2386868b' stroke-width='2' stroke-linecap='round' stroke-linejoin='round'%3e%3cpolyline points='6 9 12 15 18 9'%3e%3c/polyline%3e%3c/svg%3e");
    background-repeat: no-repeat;
    background-position: right 16px center;
    background-size: 16px;
}

/* ปุ่มกดสไตล์ Apple Blue */
.apple-btn {
    width: 100%;
    background-color: #0071e3;
    color: #ffffff;
    border: none;
    border-radius: 12px;
    padding: 14px;
    font-size: 1rem;
    font-weight: 600;
    cursor: pointer;
    transition: background-color 0.2s ease, transform 0.1s ease;
}

.apple-btn:hover:not(:disabled) {
    background-color: #0077ed;
}

.apple-btn:active:not(:disabled) {
    transform: scale(0.98); /* เวลากดให้ยุบลงนิดนึง ฟีลลิ่งปุ่มจริง */
}

.apple-btn:disabled {
    background-color: #0071e3;
    opacity: 0.5;
    cursor: not-allowed;
}

/* Inline Alert */
.apple-alert {
    padding: 12px 16px;
    border-radius: 10px;
    font-size: 0.9rem;
    font-weight: 500;
    text-align: center;
}
.alert-success {
    background-color: #e5f5ea;
    color: #008a38;
}
.alert-error {
    background-color: #fcebeb;
    color: #cc292b;
}

/* Vue Transitions */
.fade-enter-active, .fade-leave-active {
    transition: opacity 0.3s ease, transform 0.3s ease;
}
.fade-enter-from, .fade-leave-to {
    opacity: 0;
    transform: translateY(-10px);
}
</style>