<template>
  <div id="app-container">
    
    <!-- Apple Style Navbar -->
    <nav class="apple-navbar">
      <div class="nav-content">
        
        <!-- Brand / Logo Area -->
        <div class="nav-brand">
          <span class="brand-text fw-bold">Project</span>
          <span class="brand-text fw-light">N8N</span>
          <span class="brand-badge">67708153</span>
        </div>

        <!-- Hamburger Button (Mobile Only) -->
        <button class="mobile-toggle" @click="toggleMenu" :class="{ 'is-active': isMenuOpen }">
          <span class="bar"></span>
          <span class="bar"></span>
          <span class="bar"></span>
        </button>

        <!-- Navigation Links -->
        <div class="nav-links" :class="{ 'nav-active': isMenuOpen }">
          <router-link to="/" class="apple-link" @click="closeMenu">หน้าแรก</router-link>
          <router-link to="/about" class="apple-link" @click="closeMenu">เกี่ยวกับ</router-link>
          <!-- ลิงก์ไปหน้าฟอร์ม / Dashboard -->
        </div>

      </div>
    </nav>

    <!-- Router View Container with Smooth Transition -->
    <main class="main-content">
      <router-view v-slot="{ Component }">
        <transition name="page-fade" mode="out-in">
          <component :is="Component" />
        </transition>
      </router-view>
    </main>

  </div>
</template>

<script setup>
import { ref } from 'vue'

const isMenuOpen = ref(false)

const toggleMenu = () => {
  isMenuOpen.value = !isMenuOpen.value
}

// ปิดเมนูอัตโนมัติเวลากดเลือกลิงก์บนมือถือ
const closeMenu = () => {
  isMenuOpen.value = false
}
</script>

<style>
/* Global Reset & Font */
body {
  margin: 0;
  padding: 0;
  background-color: #f5f5f7;
  font-family: -apple-system, BlinkMacSystemFont, "SF Pro Text", "Helvetica Neue", Helvetica, Arial, sans-serif;
  -webkit-font-smoothing: antialiased;
  -moz-osx-font-smoothing: grayscale;
  color: #1d1d1f;
  overflow-x: hidden;
}

#app-container {
  display: flex;
  flex-direction: column;
  min-height: 100vh;
}

/* Frosted Glass Navbar */
.apple-navbar {
  position: sticky;
  top: 0;
  z-index: 1000;
  width: 100%;
  height: 54px;
  background-color: rgba(255, 255, 255, 0.72);
  backdrop-filter: saturate(180%) blur(20px);
  -webkit-backdrop-filter: saturate(180%) blur(20px);
  border-bottom: 1px solid rgba(0, 0, 0, 0.08);
}

.nav-content {
  max-width: 980px;
  margin: 0 auto;
  height: 100%;
  display: flex;
  justify-content: space-between;
  align-items: center;
  padding: 0 20px;
}

/* Typography Redesign สำหรับโลโก้ */
.nav-brand {
  display: flex;
  align-items: center;
  gap: 4px;
  cursor: default;
}

.brand-text {
  font-size: 1.15rem;
  letter-spacing: -0.02em;
  color: #1d1d1f;
}
.brand-text.fw-bold { font-weight: 700; }
.brand-text.fw-light { font-weight: 400; opacity: 0.8; }

.brand-badge {
  background: #e8e8ed;
  color: #86868b;
  font-size: 0.7rem;
  font-weight: 600;
  padding: 2px 6px;
  border-radius: 6px;
  margin-left: 6px;
  font-family: ui-monospace, SFMono-Regular, Menlo, Monaco, Consolas, monospace;
}

/* Links Styling */
.nav-links {
  display: flex;
  gap: 8px;
  align-items: center;
}

.apple-link {
  text-decoration: none;
  color: #1d1d1f;
  font-size: 0.85rem;
  font-weight: 400;
  padding: 6px 14px;
  border-radius: 16px;
  transition: all 0.2s ease;
  letter-spacing: -0.01em;
}

.apple-link:hover {
  background-color: rgba(0, 0, 0, 0.05);
}

.nav-links .router-link-exact-active {
  background-color: #1d1d1f;
  color: #ffffff;
  font-weight: 500;
}

/* Hamburger Menu Toggle (ซ่อนใน Desktop) */
.mobile-toggle {
  display: none;
  flex-direction: column;
  justify-content: space-between;
  width: 24px;
  height: 16px;
  background: transparent;
  border: none;
  cursor: pointer;
  padding: 0;
  z-index: 1001;
}

.mobile-toggle .bar {
  height: 2px;
  width: 100%;
  background-color: #1d1d1f;
  border-radius: 2px;
  transition: all 0.3s ease;
}

/* Mobile Responsive */
@media (max-width: 768px) {
  .mobile-toggle {
    display: flex;
  }

  .nav-links {
    position: fixed;
    top: 54px;
    left: 0;
    width: 100%;
    background-color: rgba(255, 255, 255, 0.95);
    backdrop-filter: blur(20px);
    -webkit-backdrop-filter: blur(20px);
    flex-direction: column;
    padding: 20px 0;
    gap: 15px;
    box-shadow: 0 10px 30px rgba(0, 0, 0, 0.1);
    
    /* Animation ซ่อน/โชว์ เมนู */
    transform: translateY(-150%);
    opacity: 0;
    transition: all 0.4s cubic-bezier(0.16, 1, 0.3, 1);
  }

  .nav-links.nav-active {
    transform: translateY(0);
    opacity: 1;
  }

  .apple-link {
    font-size: 1.1rem;
    padding: 10px 20px;
    width: 80%;
    text-align: center;
  }

  /* Hamburger Animation เวลาเปิดเมนู */
  .mobile-toggle.is-active .bar:nth-child(1) {
    transform: translateY(7px) rotate(45deg);
  }
  .mobile-toggle.is-active .bar:nth-child(2) {
    opacity: 0;
  }
  .mobile-toggle.is-active .bar:nth-child(3) {
    transform: translateY(-7px) rotate(-45deg);
  }
}

/* Content Area & Page Transitions */
.main-content {
  flex: 1;
  padding-top: 20px;
}

/* Vue Router View Transition Classes */
.page-fade-enter-active,
.page-fade-leave-active {
  transition: opacity 0.3s ease, transform 0.3s ease;
}

.page-fade-enter-from {
  opacity: 0;
  transform: translateY(10px);
}

.page-fade-leave-to {
  opacity: 0;
  transform: translateY(-10px);
}
</style>