<template>
  <div class="page">
    <section class="hero">
      <p class="hero-label">Real-time Data</p>
      <h1 class="hero-title">รายการสินค้า</h1>
      <p class="hero-desc">ดึงข้อมูลจาก Google Sheets ผ่าน n8n Webhook</p>
    </section>

    <div class="container">
      <div class="card">
        <div class="toolbar">
          <div class="toolbar-left">
            <span class="toolbar-title">สินค้าทั้งหมด</span>
            <span class="count-badge" v-if="!loading">{{ products.length }}</span>
          </div>
          <button class="btn-sm" @click="fetchProducts" :disabled="loading">{{ loading ? 'โหลด...' : '↻ รีเฟรช' }}</button>
        </div>

        <div v-if="loading" class="state"><div class="spinner"></div><p>กำลังโหลด...</p></div>
        <div v-else-if="status.message" class="state"><p class="err-text">⚠ {{ status.message }}</p></div>
        <div v-else-if="products.length === 0" class="state"><p>📦 ยังไม่มีสินค้า</p></div>

        <div v-else class="grid">
          <div v-for="(item, i) in products" :key="i" class="product-card">
            <div class="card-top">
              <span class="product-code">{{ item.id_product }}</span>
              <span class="product-index">#{{ i + 1 }}</span>
            </div>
            <h4 class="product-name">{{ item.name_product }}</h4>
            <div class="card-meta">
              <div><span class="meta-label">จำนวน</span><span class="meta-val">{{ item.number_product }}</span></div>
              <div class="meta-sep"></div>
              <div><span class="meta-label">ราคา</span><span class="meta-val price">฿{{ Number(item.price_product).toLocaleString() }}</span></div>
            </div>
          </div>
        </div>
      </div>
    </div>
  </div>
</template>

<script setup>
import { ref, reactive, onMounted } from 'vue';
const products = ref([]); const loading = ref(true);
const status = reactive({ message: "", type: "" });

const fetchProducts = async () => {
  loading.value = true; status.message = "";
  try {
    const res = await fetch("http://localhost:5678/webhook/get-products");
    if (!res.ok) throw new Error();
    products.value = await res.json();
  } catch { status.message = "ไม่สามารถเชื่อมต่อ API ได้"; }
  finally { loading.value = false; }
};

onMounted(() => fetchProducts());
</script>

<style scoped>
.page { min-height: calc(100vh - 52px); background: #fbfbfd; }
.hero { text-align: center; padding: 48px 24px 24px; }
.hero-label { font-size: 0.8rem; font-weight: 600; color: #0071e3; text-transform: uppercase; letter-spacing: 0.04em; margin-bottom: 8px; }
.hero-title { font-size: 2.2rem; font-weight: 700; color: #1d1d1f; letter-spacing: -0.03em; margin-bottom: 8px; }
.hero-desc { font-size: 0.95rem; color: #86868b; }

.container { max-width: 860px; margin: 0 auto; padding: 0 24px 60px; }

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
.err-text { color: #ff3b30; }
.spinner { width: 24px; height: 24px; border: 2.5px solid #e8e8ed; border-top-color: #0071e3; border-radius: 50%; animation: spin 0.7s linear infinite; margin: 0 auto 12px; }
@keyframes spin { to { transform: rotate(360deg); } }

.grid { display: grid; grid-template-columns: repeat(auto-fill, minmax(240px, 1fr)); gap: 16px; padding: 20px; }

.product-card {
  background: #fbfbfd; border: 1px solid #e8e8ed; border-radius: 14px;
  padding: 18px; transition: all 0.2s ease;
}
.product-card:hover { border-color: #d2d2d7; box-shadow: 0 4px 16px rgba(0,0,0,0.06); transform: translateY(-2px); }

.card-top { display: flex; justify-content: space-between; align-items: center; margin-bottom: 10px; }
.product-code { font-size: 0.72rem; font-weight: 700; color: #0071e3; background: #e8f0fe; padding: 3px 8px; border-radius: 6px; font-family: 'SF Mono', ui-monospace, monospace; }
.product-index { font-size: 0.7rem; color: #aeaeb2; font-weight: 600; }
.product-name { font-size: 1rem; font-weight: 700; color: #1d1d1f; margin-bottom: 14px; }

.card-meta { display: flex; align-items: center; gap: 14px; padding-top: 12px; border-top: 1px solid #f0f0f5; }
.meta-label { display: block; font-size: 0.68rem; color: #86868b; font-weight: 600; text-transform: uppercase; letter-spacing: 0.03em; }
.meta-val { font-size: 0.92rem; font-weight: 600; color: #1d1d1f; }
.meta-val.price { color: #34c759; }
.meta-sep { width: 1px; height: 28px; background: #e8e8ed; }

@media (max-width: 600px) { .hero-title { font-size: 1.6rem; } .grid { grid-template-columns: 1fr; padding: 16px; } }
</style>