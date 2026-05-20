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

      <div class="sidebar-brand">
        <span class="sidebar-brand-name">Automata Simulator</span>
        <span class="sidebar-brand-sub">Theory of Formal Languages</span>
      </div>

      <!-- Problem selection -->
      <div class="sidebar-section">
        <div class="section-label">Problem</div>
        <Problems :problems="problems" v-model="selectedProblemIndex" />

        <div class="regex-block">
          <div class="regex-block-title">Regular Expression</div>
          <code class="regex-text">{{ currentRegex }}</code>
        </div>
      </div>

      <div class="sidebar-divider"></div>

      <!-- Test inputs -->
      <div class="sidebar-section">
        <div class="section-label">Test Strings</div>
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
}

/* ── Left sidebar ─────────────────────────────────────────── */
.sidebar {
  width: 340px;
  flex-shrink: 0;
  background: #ffffff;
  border-right: 1px solid #e1e4e8;
  display: flex;
  flex-direction: column;
  overflow-y: auto;
}

.sidebar-brand {
  padding: 20px 20px 16px;
  border-bottom: 1px solid #f0f2f4;
  flex-shrink: 0;
}
.sidebar-brand-name {
  display: block;
  font-size: 13px;
  font-weight: 700;
  color: #0f172a;
  letter-spacing: -0.01em;
}
.sidebar-brand-sub {
  display: block;
  font-size: 11px;
  color: #8c959f;
  margin-top: 2px;
  letter-spacing: 0.02em;
}

.sidebar-section {
  padding: 16px 20px;
  display: flex;
  flex-direction: column;
  gap: 12px;
}

.section-label {
  font-size: 10px;
  font-weight: 700;
  letter-spacing: 0.08em;
  text-transform: uppercase;
  color: #8c959f;
}

.regex-block {
  background: #f6f8fa;
  border: 1px solid #e1e4e8;
  border-radius: 6px;
  padding: 10px 12px;
}
.regex-block-title {
  font-size: 10px;
  font-weight: 600;
  letter-spacing: 0.06em;
  text-transform: uppercase;
  color: #8c959f;
  margin-bottom: 6px;
}
.regex-text {
  font-family: 'SFMono-Regular', 'Consolas', 'Liberation Mono', monospace;
  font-size: 11.5px;
  color: #0550ae;
  word-break: break-all;
  line-height: 1.65;
  display: block;
}

.sidebar-divider {
  height: 1px;
  background: #f0f2f4;
  flex-shrink: 0;
}

/* ── Right panel ──────────────────────────────────────────── */
.main-panel {
  flex: 1;
  min-width: 0;
  display: flex;
  flex-direction: column;
  background: #f6f8fa;
}

.viz-header {
  background: #ffffff;
  border-bottom: 1px solid #e1e4e8;
  padding: 0 24px;
  flex-shrink: 0;
}

.viz-tabs {
  display: flex;
  gap: 0;
}

.viz-tab-btn {
  display: flex;
  flex-direction: column;
  align-items: flex-start;
  padding: 14px 20px 12px;
  border: none;
  border-bottom: 2px solid transparent;
  background: none;
  cursor: pointer;
  transition: all 0.15s ease;
  color: #57606a;
  gap: 1px;
  margin-bottom: -1px;
}
.viz-tab-btn:hover { color: #0f172a; }
.viz-tab-btn.is-active {
  color: #1a7f37;
  border-bottom-color: #1a7f37;
}
.vt-label {
  font-size: 13px;
  font-weight: 700;
  line-height: 1;
}
.vt-full {
  font-size: 10px;
  color: inherit;
  opacity: 0.7;
}

.viz-content {
  flex: 1;
  overflow-y: auto;
  scroll-margin-top: 60px;
}

/* ── Responsive ───────────────────────────────────────────── */
@media (max-width: 900px) {
  .workspace { flex-direction: column; }
  .sidebar { width: 100%; border-right: none; border-bottom: 1px solid #e1e4e8; }
  .main-panel { min-height: 60vh; }
}
</style>
