<template>
    <div class="apple-container py-5">
        <div class="row justify-content-center m-0">
            <div class="col-12 col-md-8 col-lg-5">

                <div class="apple-card">

                    <div class="text-center mb-4">
                        <div class="apple-icon mb-3">➕</div>
                        <h1 class="apple-title">เพิ่มสินค้าใหม่</h1>
                        <p class="apple-subtitle">บันทึกข้อมูลลง Google Sheets ผ่าน n8n</p>
                    </div>

                    <div>
                        <transition name="fade">
                            <div v-if="status.message" 
                                 class="apple-alert mb-4" 
                                 :class="status.type === 'success' ? 'alert-success' : 'alert-error'">
                                {{ status.message }}
                            </div>
                        </transition>

                        <form @submit.prevent="submitProduct">

                            <div class="mb-3">
                                <label class="apple-label">รหัสสินค้า</label>
                                <input type="text" class="apple-input" v-model="product.id_product" placeholder="Ex. P006" required>
                            </div>

                            <div class="mb-3">
                                <label class="apple-label">ชื่อสินค้า</label>
                                <input type="text" class="apple-input" v-model="product.name_product" placeholder="Ex. ปากกาสีแดง" required>
                            </div>

                            <div class="row">
                                <div class="col-6 mb-4">
                                    <label class="apple-label">จำนวน</label>
                                    <input type="number" class="apple-input" v-model="product.number_product" placeholder="0" min="0" required>
                                </div>
                                <div class="col-6 mb-4">
                                    <label class="apple-label">ราคา (บาท)</label>
                                    <input type="number" class="apple-input" v-model="product.price_product" placeholder="0.00" min="0" required>
                                </div>
                            </div>

                            <button type="submit" class="apple-btn" :disabled="loading">
                                <span v-if="loading" class="spinner-border spinner-border-sm me-2"></span>
                                {{ loading ? 'กำลังบันทึกข้อมูล...' : 'บันทึกสินค้า' }}
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

// ตรงนี้คือตัวแปรที่ n8n ของพี่กำลังดักรออยู่ (id_product, name_product, ฯลฯ)
const product = reactive({
    id_product: "",
    name_product: "",
    number_product: "",
    price_product: ""
});

const loading = ref(false);

const status = reactive({
    message: "",
    type: ""
});

const submitProduct = async () => {
    loading.value = true;
    status.message = "";

    try {
        // 🔥 ข้อควรระวัง: ตอนนี้ใช้ webhook-test อยู่ ไว้สำหรับทดสอบตอนที่กด Listen ใน n8n
        // ถ้าจะใช้จริงอย่าลืมเปลี่ยนเป็น /webhook/newproducts นะครับ
        const response = await fetch("http://localhost:5678/webhook-test/newproducts", {
            method: "POST", // บังคับว่าเป็น POST ตามที่ n8n ต้องการ
            headers: {
                "Content-Type": "application/json"
            },
            body: JSON.stringify({
                id_product: product.id_product,
                name_product: product.name_product,
                // แปลงเป็นตัวเลขก่อนส่งไป Sheets จะได้คำนวณต่อได้
                number_product: Number(product.number_product),
                price_product: Number(product.price_product)
            })
        });

        if (!response.ok) throw new Error("Network response was not ok");

        status.message = "เพิ่มสินค้าลง Google Sheets สำเร็จ!";
        status.type = "success";

        // เคลียร์ฟอร์มให้ว่างหลังบันทึกเสร็จ
        product.id_product = "";
        product.name_product = "";
        product.number_product = "";
        product.price_product = "";

        setTimeout(() => {
            status.message = "";
        }, 3000);

    } catch (error) {
        console.error("API Error:", error);
        status.message = "ไม่สามารถบันทึกได้ โปรดเช็คว่ากด Listen for test event ใน n8n หรือยัง";
        status.type = "danger";
    } finally {
        loading.value = false;
    }
};
</script>

<style scoped>
/* ดีไซน์เดิมของพี่คิงเลยครับ ก๊อปมาใช้ได้เลย ความคลีนระดับ Apple */
.apple-container { background-color: #f5f5f7; min-height: 100vh; font-family: -apple-system, BlinkMacSystemFont, "SF Pro Display", "Segoe UI", Roboto, Helvetica, Arial, sans-serif; }
.apple-card { background: #ffffff; border-radius: 20px; padding: 40px; box-shadow: 0 4px 24px rgba(0, 0, 0, 0.04); }
.apple-icon { font-size: 2.5rem; line-height: 1; }
.apple-title { font-size: 1.75rem; font-weight: 600; color: #1d1d1f; letter-spacing: -0.02em; margin-bottom: 0.5rem; }
.apple-subtitle { font-size: 0.95rem; color: #86868b; }
.apple-label { display: block; font-size: 0.85rem; font-weight: 500; color: #1d1d1f; margin-bottom: 0.5rem; }
.apple-input { width: 100%; background-color: #f5f5f7; border: 1px solid transparent; border-radius: 12px; padding: 14px 16px; font-size: 1rem; color: #1d1d1f; transition: all 0.2s ease; }
.apple-input::placeholder { color: #86868b; }
.apple-input:focus { outline: none; background-color: #ffffff; border-color: #0071e3; box-shadow: 0 0 0 4px rgba(0, 113, 227, 0.15); }
.apple-btn { width: 100%; background-color: #0071e3; color: #ffffff; border: none; border-radius: 12px; padding: 14px; font-size: 1rem; font-weight: 600; cursor: pointer; transition: background-color 0.2s ease, transform 0.1s ease; }
.apple-btn:hover:not(:disabled) { background-color: #0077ed; }
.apple-btn:active:not(:disabled) { transform: scale(0.98); }
.apple-btn:disabled { background-color: #0071e3; opacity: 0.5; cursor: not-allowed; }
.apple-alert { padding: 12px 16px; border-radius: 10px; font-size: 0.9rem; font-weight: 500; text-align: center; }
.alert-success { background-color: #e5f5ea; color: #008a38; }
.alert-error { background-color: #fcebeb; color: #cc292b; }
.fade-enter-active, .fade-leave-active { transition: opacity 0.3s ease, transform 0.3s ease; }
.fade-enter-from, .fade-leave-to { opacity: 0; transform: translateY(-10px); }
</style>