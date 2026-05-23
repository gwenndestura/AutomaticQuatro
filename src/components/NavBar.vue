<script setup>
import { ref, onMounted } from 'vue'

const emit = defineEmits(['change-view', 'change-automata'])

const active     = ref('simulator')
const isDarkMode = ref(false)

const changeView = (view) => {
  active.value = view
  emit('change-view', view)
}

const toggleDarkMode = () => {
  isDarkMode.value = !isDarkMode.value
  document.body.classList.toggle('dark-mode', isDarkMode.value)
}

onMounted(() => {
  isDarkMode.value = document.body.classList.contains('dark-mode')
})
</script>

<template>
  <nav class="navbar">

    <!-- Brand -->
    <button class="brand" @click="changeView('simulator')" title="Go to home">
      <img src="../assets/aqlogo.png" alt="AutomaticQuatro" class="brand-logo" />
    </button>

    <!-- Nav tabs -->
    <div class="nav-tabs" role="tablist">
      <button
        role="tab"
        :aria-selected="active === 'simulator'"
        :class="['nav-tab', active === 'simulator' ? 'nav-tab--active' : '']"
        @click="changeView('simulator')"
      >
        <span class="tab-label">Simulations</span>
      </button>
      <button
        role="tab"
        :aria-selected="active === 'manual'"
        :class="['nav-tab', active === 'manual' ? 'nav-tab--active' : '']"
        @click="changeView('manual')"
      >
        <span class="tab-label">User Manual</span>
      </button>
    </div>

    <!-- Dark mode toggle -->
    <button class="dark-btn" :title="isDarkMode ? 'Switch to light mode' : 'Switch to dark mode'" @click="toggleDarkMode">
      <svg v-if="!isDarkMode" width="15" height="15" viewBox="0 0 24 24" fill="none"
        stroke="currentColor" stroke-width="2" stroke-linecap="round" stroke-linejoin="round" aria-hidden="true">
        <path d="M21 12.79A9 9 0 1111.21 3a7 7 0 009.79 9.79z"/>
      </svg>
      <svg v-else width="15" height="15" viewBox="0 0 24 24" fill="none"
        stroke="currentColor" stroke-width="2" stroke-linecap="round" stroke-linejoin="round" aria-hidden="true">
        <circle cx="12" cy="12" r="5"/>
        <line x1="12" y1="1"  x2="12" y2="3"/><line x1="12" y1="21" x2="12" y2="23"/>
        <line x1="4.22" y1="4.22" x2="5.64" y2="5.64"/>
        <line x1="18.36" y1="18.36" x2="19.78" y2="19.78"/>
        <line x1="1" y1="12" x2="3" y2="12"/><line x1="21" y1="12" x2="23" y2="12"/>
        <line x1="4.22" y1="19.78" x2="5.64" y2="18.36"/>
        <line x1="18.36" y1="5.64" x2="19.78" y2="4.22"/>
      </svg>
    </button>

  </nav>
</template>

<style scoped>
.navbar {
  position: sticky;
  top: 0;
  z-index: 100;
  height: 60px;
  display: flex;
  align-items: center;
  justify-content: space-between;
  padding: 0 20px;
  background: #111f14;
  border-bottom: 1px solid #1e3323;
}

/* Brand */
.brand {
  display: flex;
  align-items: center;
  background: none;
  border: none;
  padding: 0;
  cursor: pointer;
}
.brand-logo {
  height: 48px;
  width: auto;
  object-fit: contain;
}

/* Tabs */
.nav-tabs {
  display: flex;
  align-items: center;
  height: 100%;
  gap: 4px;
}

.nav-tab {
  display: inline-flex;
  align-items: center;
  padding: 0 14px;
  height: 100%;
  font-size: 11.5px;
  font-weight: 700;
  letter-spacing: 0.07em;
  text-transform: uppercase;
  color: #5a8060;
  background: none;
  border: none;
  cursor: pointer;
  transition: color 0.15s;
  white-space: nowrap;
}
.nav-tab:hover {
  color: #9dc4a2;
}
.nav-tab--active {
  color: #4caf50;
}

/* Underline sits right under the text */
.tab-label {
  padding-bottom: 3px;
  border-bottom: 2px solid transparent;
  transition: border-color 0.15s;
}
.nav-tab--active .tab-label {
  border-bottom-color: #4caf50;
}
.nav-tab:hover:not(.nav-tab--active) .tab-label {
  border-bottom-color: #3d6645;
}

/* Dark mode toggle */
.dark-btn {
  display: inline-flex;
  align-items: center;
  justify-content: center;
  width: 34px;
  height: 34px;
  color: #5a8060;
  background: none;
  border: 1px solid transparent;
  border-radius: 6px;
  cursor: pointer;
  transition: color 0.15s, background 0.15s, border-color 0.15s;
  flex-shrink: 0;
}
.dark-btn:hover {
  color: #9dc4a2;
  background: #1a2e1d;
  border-color: #1e3323;
}

/* Responsive */
@media (max-width: 600px) {
  .navbar { padding: 0 12px; }
  .brand-logo { height: 36px; }
  .nav-tab { padding: 0 10px; font-size: 11px; }
}
</style>
