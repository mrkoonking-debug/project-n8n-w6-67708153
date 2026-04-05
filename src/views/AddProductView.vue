<template>
  <div class="page">
    <section class="hero">
      <p class="hero-label">เพิ่มข้อมูล</p>
      <h1 class="hero-title">เพิ่มสินค้าใหม่</h1>
      <p class="hero-desc">บันทึกข้อมูลลง Google Sheets ผ่าน n8n Webhook</p>
    </section>

    <div class="container">
      <div class="card">
        <transition name="fade">
          <div v-if="status.message" class="alert" :class="status.type === 'success' ? 'alert-ok' : 'alert-fail'">
            {{ status.type === 'success' ? '✓' : '✕' }} {{ status.message }}
          </div>
        </transition>

        <form @submit.prevent="submitProduct">
          <div class="field">
            <label>รหัสสินค้า</label>
            <input v-model="product.id_product" placeholder="Ex. P006" required />
          </div>
          <div class="field">
            <label>ชื่อสินค้า</label>
            <input v-model="product.name_product" placeholder="Ex. ปากกาสีแดง" required />
          </div>
          <div class="field-row">
            <div class="field">
              <label>จำนวน</label>
              <input type="number" v-model="product.number_product" placeholder="0" min="0" required />
            </div>
            <div class="field">
              <label>ราคา (บาท)</label>
              <input type="number" v-model="product.price_product" placeholder="0.00" min="0" step="0.01" required />
            </div>
          </div>
          <button class="btn" :disabled="loading" type="submit">
            {{ loading ? 'กำลังบันทึก...' : 'บันทึกสินค้า' }}
          </button>
        </form>
      </div>
    </div>
  </div>
</template>

<script setup>
import { reactive, ref } from "vue";
const product = reactive({ id_product: "", name_product: "", number_product: "", price_product: "" });
const loading = ref(false);
const status = reactive({ message: "", type: "" });

const submitProduct = async () => {
  loading.value = true; status.message = "";
  try {
    const res = await fetch("http://localhost:5678/webhook-test/newproducts", {
      method: "POST", headers: { "Content-Type": "application/json" },
      body: JSON.stringify({ id_product: product.id_product, name_product: product.name_product, number_product: Number(product.number_product), price_product: Number(product.price_product) })
    });
    if (!res.ok) throw new Error();
    status.message = "เพิ่มสินค้าสำเร็จ!"; status.type = "success";
    product.id_product = ""; product.name_product = ""; product.number_product = ""; product.price_product = "";
    setTimeout(() => { status.message = ""; }, 3000);
  } catch { status.message = "ไม่สามารถบันทึกได้"; status.type = "danger"; }
  finally { loading.value = false; }
};
</script>

<style scoped>
.page { min-height: calc(100vh - 52px); background: #fff; }
.hero { text-align: center; padding: 56px 24px 28px; }
.hero-label { font-size: 0.8rem; font-weight: 600; color: #0071e3; text-transform: uppercase; letter-spacing: 0.04em; margin-bottom: 8px; }
.hero-title { font-size: 2.5rem; font-weight: 700; color: #1d1d1f; letter-spacing: -0.03em; margin-bottom: 8px; }
.hero-desc { font-size: 1rem; color: #86868b; }

.container { max-width: 480px; margin: 0 auto; padding: 0 24px 60px; }

.card {
  background: #fff; border: 1px solid #e8e8ed; border-radius: 18px;
  padding: 32px; box-shadow: 0 2px 12px rgba(0,0,0,0.04);
}

.field { margin-bottom: 20px; }
.field label { display: block; font-size: 0.82rem; font-weight: 600; color: #1d1d1f; margin-bottom: 6px; }

.field-row { display: grid; grid-template-columns: 1fr 1fr; gap: 14px; }

.field input {
  width: 100%; padding: 12px 14px; font-size: 0.95rem; font-family: 'Inter', sans-serif;
  border: 1px solid #d2d2d7; border-radius: 12px; color: #1d1d1f;
  background: #fff; outline: none; transition: all 0.2s ease;
}
.field input::placeholder { color: #aeaeb2; }
.field input:focus { border-color: #0071e3; box-shadow: 0 0 0 4px rgba(0,113,227,0.1); }

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

@media (max-width: 600px) { .hero-title { font-size: 1.8rem; } .card { padding: 24px 20px; } .field-row { grid-template-columns: 1fr; } }
</style>