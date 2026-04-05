<template>
  <div id="app-container">
    <nav class="navbar">
      <div class="nav-content">
        <router-link to="/" class="nav-brand">
          <span class="brand-name">Project N8N</span>
          <span class="brand-badge">67708153</span>
        </router-link>

        <button class="mobile-toggle" @click="toggleMenu" :class="{ 'is-active': isMenuOpen }">
          <span class="bar"></span><span class="bar"></span><span class="bar"></span>
        </button>

        <div class="nav-links" :class="{ 'nav-active': isMenuOpen }">
          <router-link to="/" class="nav-link" @click="closeMenu">หน้าแรก</router-link>
          <router-link to="/about" class="nav-link" @click="closeMenu">ข้อมูลสมาชิก</router-link>
          <router-link to="/products" class="nav-link" @click="closeMenu">รายการสินค้า</router-link>
          <router-link to="/add-product" class="nav-link" @click="closeMenu">เพิ่มสินค้า</router-link>
          <router-link to="/page" class="nav-link" @click="closeMenu">เบิกอุปกรณ์</router-link>
        </div>
      </div>
    </nav>

    <main class="main-content">
      <router-view v-slot="{ Component }">
        <transition name="page-fade" mode="out-in">
          <component :is="Component" />
        </transition>
      </router-view>
    </main>

    <footer class="app-footer">
      <p>© 2026 Project N8N · 67708153 · Powered by <strong>n8n</strong> + <strong>Vue.js</strong></p>
    </footer>
  </div>
</template>

<script setup>
import { ref } from 'vue'
const isMenuOpen = ref(false)
const toggleMenu = () => { isMenuOpen.value = !isMenuOpen.value }
const closeMenu = () => { isMenuOpen.value = false }
</script>

<style>
@import url('https://fonts.googleapis.com/css2?family=Inter:wght@300;400;500;600;700;800&display=swap');

*, *::before, *::after { margin: 0; padding: 0; box-sizing: border-box; }

body {
  font-family: 'Inter', -apple-system, BlinkMacSystemFont, "SF Pro Text", "Helvetica Neue", sans-serif;
  -webkit-font-smoothing: antialiased;
  background: #ffffff;
  color: #1d1d1f;
}

#app-container { display: flex; flex-direction: column; min-height: 100vh; }

/* ═══ NAVBAR ═══ */
.navbar {
  position: sticky;
  top: 0;
  z-index: 1000;
  height: 52px;
  background: rgba(255,255,255,0.72);
  backdrop-filter: saturate(180%) blur(20px);
  -webkit-backdrop-filter: saturate(180%) blur(20px);
  border-bottom: 1px solid rgba(0,0,0,0.08);
}

.nav-content {
  max-width: 980px;
  margin: 0 auto;
  height: 100%;
  display: flex;
  justify-content: space-between;
  align-items: center;
  padding: 0 22px;
}

.nav-brand { display: flex; align-items: center; gap: 8px; text-decoration: none; }
.brand-name { font-size: 1.05rem; font-weight: 700; color: #1d1d1f; letter-spacing: -0.02em; }
.brand-badge {
  font-size: 0.65rem; font-weight: 600; color: #86868b;
  background: #f5f5f7; padding: 2px 8px; border-radius: 4px;
}

.nav-links { display: flex; gap: 2px; align-items: center; }

.nav-link {
  text-decoration: none; color: #6e6e73; font-size: 0.82rem; font-weight: 500;
  padding: 6px 14px; border-radius: 20px;
  transition: all 0.2s ease;
}

.nav-link:hover { color: #1d1d1f; background: #f5f5f7; }
.nav-links .router-link-exact-active { color: #fff; background: #1d1d1f; }

.mobile-toggle {
  display: none; flex-direction: column; justify-content: space-between;
  width: 18px; height: 14px; background: transparent; border: none; cursor: pointer; padding: 0;
}
.mobile-toggle .bar { height: 1.5px; width: 100%; background: #1d1d1f; border-radius: 1px; transition: all 0.3s ease; }

@media (max-width: 768px) {
  .mobile-toggle { display: flex; }
  .nav-links {
    position: fixed; top: 52px; left: 0; width: 100%;
    background: rgba(255,255,255,0.95); backdrop-filter: blur(20px);
    flex-direction: column; padding: 16px 22px; gap: 4px;
    border-bottom: 1px solid rgba(0,0,0,0.08);
    transform: translateY(-120%); opacity: 0;
    transition: all 0.35s cubic-bezier(0.16,1,0.3,1);
  }
  .nav-links.nav-active { transform: translateY(0); opacity: 1; }
  .nav-link { font-size: 0.95rem; padding: 10px 16px; width: 100%; }
  .mobile-toggle.is-active .bar:nth-child(1) { transform: translateY(6px) rotate(45deg); }
  .mobile-toggle.is-active .bar:nth-child(2) { opacity: 0; }
  .mobile-toggle.is-active .bar:nth-child(3) { transform: translateY(-6px) rotate(-45deg); }
}

.main-content { flex: 1; }

.app-footer {
  border-top: 1px solid rgba(0,0,0,0.06); padding: 20px;
  text-align: center; font-size: 0.75rem; color: #86868b;
}

.page-fade-enter-active, .page-fade-leave-active { transition: opacity 0.25s ease; }
.page-fade-enter-from, .page-fade-leave-to { opacity: 0; }
</style>