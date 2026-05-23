<script setup>
import { ref, computed, nextTick, watch } from 'vue'
import Problems      from './components/Promblems.vue'
import InputArea     from './components/InputArea.vue'
import NavBar        from './components/NavBar.vue'
import DFAViz        from './components/DFAVisualization.vue'
import PDAViz        from './components/PDAVisualization.vue'
import CFGViz        from './components/CFGVisualization.vue'
import UserManual    from './components/UserManual.vue'
import Footer        from './components/Footer.vue'

const currentView    = ref('simulator')
const activeAutomata = ref('dfa')
const vizRef         = ref(null)

const problems = ref([
  {
    id: 1,
    label: 'Problem 1',
    regexStr: '(bab+bbb)a*b*(a*+b*)(ba)*(aba)(bab+aba)*bb(a+b)*(bab+aba)(a+b)*'
  },
  {
    id: 2,
    label: 'Problem 2',
    regexStr: '(1+0)*1*0*(101+01+000)(1+0)*(101+00)*(111+00+101)(1+0)*'
  }
])

const selectedProblemIndex = ref(0)
const currentProblem = computed(() => problems.value[selectedProblemIndex.value])
const currentRegex   = computed(() => currentProblem.value?.regexStr || '')

const currentTestString = ref('')
const simulationKey     = ref(0)

watch(selectedProblemIndex, () => {
  currentTestString.value = ''
})

const setView = (v) => { currentView.value = v }

const runSimulation = (str) => {
  currentTestString.value = str
  simulationKey.value++
  nextTick(() => {
    vizRef.value?.scrollIntoView({ behavior: 'smooth', block: 'nearest' })
  })
}

const SIM_TABS = [
  { key: 'dfa', label: 'DFA', full: 'Deterministic Finite Automata' },
  { key: 'cfg', label: 'CFG', full: 'Context-Free Grammar' },
  { key: 'pda', label: 'PDA', full: 'Pushdown Automata' },
]
</script>

<template>
  <NavBar @change-view="setView" />

  <!-- ── Simulator ──────────────────────────────────────────── -->
  <div v-if="currentView === 'simulator'" class="workspace">

    <!-- Left sidebar -->
    <aside class="sidebar">

      <!-- Problem selection -->
      <div class="sidebar-section">
        <div class="section-label">
          <svg width="10" height="10" viewBox="0 0 12 12" fill="none" aria-hidden="true" style="margin-right:5px;vertical-align:middle">
            <rect x="1" y="1" width="10" height="10" rx="2" stroke="#8c959f" stroke-width="1.5"/>
            <path d="M4 6h4M6 4v4" stroke="#8c959f" stroke-width="1.5" stroke-linecap="round"/>
          </svg>
          Select Problem
        </div>
        <Problems :problems="problems" v-model="selectedProblemIndex" />

        <div class="regex-block">
          <div class="regex-block-title">
            <svg width="9" height="9" viewBox="0 0 10 10" fill="none" aria-hidden="true" style="margin-right:4px;vertical-align:middle">
              <circle cx="5" cy="5" r="4" stroke="#8c959f" stroke-width="1.3"/>
              <path d="M5 3v2.5L6.5 7" stroke="#8c959f" stroke-width="1.3" stroke-linecap="round" stroke-linejoin="round"/>
            </svg>
            Regular Expression
          </div>
          <code class="regex-text">{{ currentRegex }}</code>
        </div>
      </div>

      <div class="sidebar-divider"></div>

      <!-- Test inputs -->
      <div class="sidebar-section">
        <div class="section-label">
          <svg width="10" height="10" viewBox="0 0 12 12" fill="none" aria-hidden="true" style="margin-right:5px;vertical-align:middle">
            <path d="M2 6h8M8 4l2 2-2 2" stroke="#8c959f" stroke-width="1.5" stroke-linecap="round" stroke-linejoin="round"/>
          </svg>
          Test Strings
        </div>
        <InputArea
          :regexStr="currentRegex"
          @simulate-string="runSimulation"
        />
      </div>

    </aside>

    <!-- Right visualization panel -->
    <main class="main-panel">

      <div class="viz-header">
        <div class="viz-tabs">
          <button
            v-for="tab in SIM_TABS"
            :key="tab.key"
            :class="['viz-tab-btn', activeAutomata === tab.key ? 'is-active' : '']"
            @click="activeAutomata = tab.key"
          >
            <span class="vt-label">{{ tab.label }}</span>
            <span class="vt-full">{{ tab.full }}</span>
          </button>
        </div>
        <div class="viz-header-right">
          <span class="viz-header-hint">Select a tab · Enter a string · Click Run</span>
        </div>
      </div>

      <div ref="vizRef" class="viz-content">
        <DFAViz
          v-if="activeAutomata === 'dfa'"
          :problemId="currentProblem.id"
          :testString="currentTestString"
          :simKey="simulationKey"
        />
        <CFGViz
          v-else-if="activeAutomata === 'cfg'"
          :problemId="currentProblem.id"
          :testString="currentTestString"
        />
        <PDAViz
          v-else-if="activeAutomata === 'pda'"
          :problemId="currentProblem.id"
          :testString="currentTestString"
          :simKey="simulationKey"
        />
      </div>

    </main>

  </div>

  <!-- ── User Manual ────────────────────────────────────────── -->
  <div v-else-if="currentView === 'manual'">
    <UserManual />
  </div>

  <Footer />
</template>

<style scoped>
/* ── Workspace layout ─────────────────────────────────────── */
.workspace {
  display: flex;
  min-height: calc(100vh - 60px);
  background: #f4f6f8;
}

/* ── Left sidebar ─────────────────────────────────────────── */
.sidebar {
  width: 320px;
  flex-shrink: 0;
  background: #ffffff;
  border-right: 1px solid #e1e4e8;
  display: flex;
  flex-direction: column;
  overflow-y: auto;
  box-shadow: 2px 0 8px rgba(0,0,0,0.04);
}


.sidebar-section {
  padding: 16px 18px;
  display: flex;
  flex-direction: column;
  gap: 11px;
}

.section-label {
  display: flex;
  align-items: center;
  font-size: 10px;
  font-weight: 700;
  letter-spacing: 0.09em;
  text-transform: uppercase;
  color: #8c959f;
}

.regex-block {
  background: #f8fafc;
  border: 1.5px solid #e2e8f0;
  border-left: 3px solid #22c55e;
  border-radius: 8px;
  padding: 10px 12px;
}
.regex-block-title {
  display: flex;
  align-items: center;
  font-size: 9.5px;
  font-weight: 700;
  letter-spacing: 0.07em;
  text-transform: uppercase;
  color: #8c959f;
  margin-bottom: 7px;
}
.regex-text {
  font-family: 'SFMono-Regular', 'Consolas', 'Liberation Mono', monospace;
  font-size: 11px;
  color: #0550ae;
  word-break: break-all;
  line-height: 1.7;
  display: block;
}

.sidebar-divider {
  height: 1px;
  background: linear-gradient(to right, transparent, #e8ecf0, transparent);
  margin: 0 18px;
  flex-shrink: 0;
}

/* ── Right panel ──────────────────────────────────────────── */
.main-panel {
  flex: 1;
  min-width: 0;
  display: flex;
  flex-direction: column;
  background: #f4f6f8;
}

.viz-header {
  display: flex;
  align-items: center;
  justify-content: space-between;
  background: #ffffff;
  border-bottom: 1px solid #e1e4e8;
  padding: 0 20px 0 4px;
  flex-shrink: 0;
  box-shadow: 0 1px 4px rgba(0,0,0,0.04);
}

.viz-tabs {
  display: flex;
  gap: 0;
}

.viz-tab-btn {
  display: flex;
  flex-direction: column;
  align-items: flex-start;
  padding: 13px 20px 11px;
  border: none;
  border-bottom: 2.5px solid transparent;
  background: none;
  cursor: pointer;
  transition: all 0.15s ease;
  color: #6b7280;
  gap: 2px;
  margin-bottom: -1px;
}
.viz-tab-btn:hover { color: #0f172a; background: #f9fafb; }
.viz-tab-btn.is-active {
  color: #16a34a;
  border-bottom-color: #22c55e;
  background: #f0fdf4;
}
.vt-label {
  font-size: 12.5px;
  font-weight: 700;
  line-height: 1;
  letter-spacing: 0.01em;
}
.vt-full {
  font-size: 9.5px;
  color: inherit;
  opacity: 0.65;
  letter-spacing: 0.01em;
}

.viz-header-right {
  display: flex;
  align-items: center;
}
.viz-header-hint {
  font-size: 10.5px;
  color: #b0b8c1;
  letter-spacing: 0.01em;
  white-space: nowrap;
}

.viz-content {
  flex: 1;
  overflow-y: auto;
  scroll-margin-top: 60px;
  padding: 4px;
}

/* ── Responsive ───────────────────────────────────────────── */
@media (max-width: 900px) {
  .workspace { flex-direction: column; }
  .sidebar { width: 100%; border-right: none; border-bottom: 1px solid #e1e4e8; box-shadow: none; }
  .main-panel { min-height: 60vh; }
  .viz-header-hint { display: none; }
}
@media (max-width: 520px) {
  .sidebar-section { padding: 14px 14px; }
}
</style>
