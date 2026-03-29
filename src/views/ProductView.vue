<template>
    <div class="apple-container py-5">
        <div class="row justify-content-center m-0">
            <div class="col-12 col-md-10 col-lg-8"> 

                <div class="apple-card">
                    <div class="text-center mb-4">
                        <div class="apple-icon mb-3">📦</div>
                        <h1 class="apple-title">รายการสินค้า</h1>
                        <p class="apple-subtitle">ดึงข้อมูลแบบ Real-time จาก n8n & Google Sheets</p>
                    </div>

                    <transition name="fade">
                        <div v-if="status.message" 
                             class="apple-alert mb-4" 
                             :class="status.type === 'success' ? 'alert-success' : 'alert-error'">
                            {{ status.message }}
                        </div>
                    </transition>

                    <div v-if="loading" class="text-center py-5">
                        <div class="spinner-border text-primary" role="status"></div>
                        <p class="mt-3 apple-subtitle">กำลังโหลดข้อมูลจากระบบ...</p>
                    </div>

                    <div v-else-if="products.length > 0" class="table-responsive">
                        <table class="apple-table">
                            <thead>
                                <tr>
                                    <th>รหัสสินค้า</th>
                                    <th>ชื่อสินค้า</th>
                                    <th>จำนวน</th>
                                    <th>ราคา</th>
                                </tr>
                            </thead>
                            <tbody>
                                <tr v-for="(item, index) in products" :key="index">
                                    <td class="fw-bold">{{ item.id_product }}</td>
                                    <td>{{ item.name_product }}</td>
                                    <td>{{ item.number_product }}</td>
                                    <td>{{ item.price_product }}</td> 
                                </tr>
                            </tbody>
                        </table>
                    </div>

                    <div v-else class="text-center py-5 apple-subtitle">
                        ไม่พบข้อมูลสินค้าในระบบ
                    </div>

                </div>
            </div>
        </div>
    </div>
</template>

<script setup>
import { ref, reactive, onMounted } from 'vue';

const products = ref([]);
const loading = ref(true);
const status = reactive({
    message: "",
    type: ""
});

const fetchProducts = async () => {
    loading.value = true;
    status.message = "";

    try {
        // ลิงก์ตรงนี้ถ้าตอนเทสต์แล้วมันไม่เวิร์ค ให้เช็คเรื่อง Production URL ใน n8n ดีๆ ครับ
        const response = await fetch("http://localhost:5678/webhook-test/get-products");
        
        if (!response.ok) throw new Error("Network response was not ok");
        
        products.value = await response.json();
        
    } catch (error) {
        console.error(error);
        status.message = "ไม่สามารถเชื่อมต่อ API ได้ โปรดเช็ค n8n ว่ารันอยู่หรือไม่";
        status.type = "danger";
    } finally {
        loading.value = false;
    }
};

onMounted(() => {
    fetchProducts();
});
</script>

<style scoped>
.apple-container {
    background-color: #f5f5f7;
    min-height: 100vh;
    font-family: -apple-system, BlinkMacSystemFont, "SF Pro Display", "Segoe UI", Roboto, Helvetica, Arial, sans-serif;
}

.apple-card {
    background: #ffffff;
    border-radius: 20px;
    padding: 40px;
    box-shadow: 0 4px 24px rgba(0, 0, 0, 0.04);
}

.apple-icon { font-size: 2.5rem; line-height: 1; }
.apple-title { font-size: 1.75rem; font-weight: 600; color: #1d1d1f; letter-spacing: -0.02em; margin-bottom: 0.5rem; }
.apple-subtitle { font-size: 0.95rem; color: #86868b; }

.apple-table {
    width: 100%;
    border-collapse: separate;
    border-spacing: 0;
    margin-top: 1rem;
}

.apple-table th {
    background-color: #f5f5f7;
    color: #86868b;
    font-weight: 600;
    font-size: 0.85rem;
    padding: 12px 16px;
    text-align: left;
    border-bottom: none;
}

.apple-table th:first-child { border-top-left-radius: 12px; border-bottom-left-radius: 12px; }
.apple-table th:last-child { border-top-right-radius: 12px; border-bottom-right-radius: 12px; }

.apple-table td {
    padding: 16px;
    font-size: 0.95rem;
    color: #1d1d1f;
    border-bottom: 1px solid #e8e8ed;
    transition: background-color 0.2s ease;
}

.apple-table tbody tr:hover td { background-color: #fcfcfc; }
.apple-table tbody tr:last-child td { border-bottom: none; }

.apple-alert {
    padding: 12px 16px;
    border-radius: 10px;
    font-size: 0.9rem;
    font-weight: 500;
    text-align: center;
}
.alert-error { background-color: #fcebeb; color: #cc292b; }

.fade-enter-active, .fade-leave-active { transition: opacity 0.3s ease, transform 0.3s ease; }
.fade-enter-from, .fade-leave-to { opacity: 0; transform: translateY(-10px); }
</style>