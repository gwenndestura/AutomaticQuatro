<script setup>
import { ref, computed, watch, onMounted, nextTick } from 'vue'

const props = defineProps({
  problemId: { type: Number, required: true },
  testString: { type: String, default: '' }
})

const CFG_DATA = {
  1: {
    startSymbol: 'S',
    productions: [
      { lhs: 'S', alts: ['ABCDEFGHIJI'], note: '(bab+bbb)·a*·b*·(a*+b*)·(ba)*·aba·(bab+aba)*·bb·(a+b)*·(bab+aba)·(a+b)*' },
      { lhs: 'A', alts: ['bab', 'bbb'],        note: '(bab + bbb)' },
      { lhs: 'B', alts: ['λ', 'aB'],           note: 'a*' },
      { lhs: 'C', alts: ['λ', 'bC'],           note: 'b*' },
      { lhs: 'D', alts: ['B', 'C'],            note: '(a* + b*)' },
      { lhs: 'E', alts: ['λ', 'baE'],          note: '(ba)*' },
      { lhs: 'F', alts: ['aba'],               note: 'aba' },
      { lhs: 'G', alts: ['λ', 'babG', 'abaG'],note: '(bab + aba)*' },
      { lhs: 'H', alts: ['bb'],               note: 'bb' },
      { lhs: 'I', alts: ['λ', 'aI', 'bI'],    note: '(a+b)*' },
      { lhs: 'J', alts: ['bab', 'aba'],        note: '(bab + aba)' },
    ],
    terminals: ['a', 'b'],
    nonTerminals: ['S','A','B','C','D','E','F','G','H','I','J']
  },
  2: {
    startSymbol: 'S',
    productions: [
      { lhs: 'S', alts: ['ABCDEFGH'], note: '(1+0)*·1*·0*·(101+01+000)·(1+0)*·(101+00)*·(111+00+101)·(1+0)*' },
      { lhs: 'A', alts: ['λ', '0A', '1A'],      note: '(1+0)*' },
      { lhs: 'B', alts: ['λ', '1B'],            note: '1*' },
      { lhs: 'C', alts: ['λ', '0C'],            note: '0*' },
      { lhs: 'D', alts: ['101', '01', '000'],   note: '(101 + 01 + 000)' },
      { lhs: 'E', alts: ['λ', '0E', '1E'],      note: '(1+0)*' },
      { lhs: 'F', alts: ['λ', '101F', '00F'],   note: '(101 + 00)*' },
      { lhs: 'G', alts: ['111', '00', '101'],   note: '(111 + 00 + 101)' },
      { lhs: 'H', alts: ['λ', '0H', '1H'],      note: '(1+0)*' },
    ],
    terminals: ['0', '1'],
    nonTerminals: ['S','A','B','C','D','E','F','G','H']
  }
}

const REGEX_MAP = {
  1: '(bab+bbb)a*b*(a*+b*)(ba)*(aba)(bab+aba)*bb(a+b)*(bab+aba)(a+b)*',
  2: '(1+0)*1*0*(101+01+000)(1+0)*(101+00)*(111+00+101)(1+0)*'
}

const cfg         = computed(() => CFG_DATA[props.problemId] || CFG_DATA[1])
const problemRegex = computed(() => REGEX_MAP[props.problemId])
const hoveredRow  = ref(null)
const derivation  = ref(null)
const simError    = ref('')
const simStatus   = ref('')   // 'ok' | 'fail' | ''
const simCardRef  = ref(null)

// Tokenise an alt string character-by-character
const tokenizeAlt = (alt) => {
  if (alt === 'λ') return [{ ch: 'λ', isNT: false }]
  return [...alt].map(ch => ({ ch, isNT: cfg.value.nonTerminals.includes(ch) }))
}

// Tokenise a sentential-form step string for the simulation display
const tokenizeStep = (step) => {
  if (!step || step === 'λ') return [{ ch: 'λ', isNT: false, isLambda: true }]
  return [...step].map(ch => ({
    ch,
    isNT: cfg.value.nonTerminals.includes(ch),
    isLambda: ch === 'λ'
  }))
}

// ── Leftmost derivation (backtracking) ──────────────────────────────────────
function getDerivation(cfgData, input) {
  const ntSet = new Set(cfgData.nonTerminals)
  const MAX   = 300
  const steps = []

  const sentStr = (syms) => {
    const s = syms.filter(x => x !== 'λ').join('')
    return s || 'λ'
  }

  function tryDerive(syms) {
    if (steps.length >= MAX) return false

    const ntIdx = syms.findIndex(s => ntSet.has(s))

    // No non-terminals left — check if string matches
    if (ntIdx === -1) {
      return syms.filter(x => x !== 'λ').join('') === input
    }

    // Prune: terminal prefix must be a prefix of input
    const prefix = syms.slice(0, ntIdx).filter(x => x !== 'λ').join('')
    if (!input.startsWith(prefix)) return false

    // Prune: total terminals already derived can't exceed input length
    const termCount = syms.filter(x => !ntSet.has(x) && x !== 'λ').length
    if (termCount > input.length) return false

    const nt   = syms[ntIdx]
    const prod = cfgData.productions.find(p => p.lhs === nt)
    if (!prod) return false

    for (const alt of prod.alts) {
      const altSyms = alt === 'λ' ? ['λ'] : [...alt]
      const newSyms = [...syms.slice(0, ntIdx), ...altSyms, ...syms.slice(ntIdx + 1)]

      if (alt === 'λ') {
        // Show the λ explicitly before collapsing it away
        const prefix     = syms.slice(0, ntIdx).filter(x => x !== 'λ').join('')
        const suffix     = syms.slice(ntIdx + 1).filter(x => x !== 'λ').join('')
        const lambdaStep = prefix + 'λ' + suffix
        const finalStep  = sentStr(newSyms)
        const twoSteps   = lambdaStep !== finalStep
        steps.push(lambdaStep)
        if (twoSteps) steps.push(finalStep)
        if (tryDerive(newSyms)) return true
        if (twoSteps) steps.pop()
        steps.pop()
      } else {
        steps.push(sentStr(newSyms))
        if (tryDerive(newSyms)) return true
        steps.pop()
      }
    }
    return false
  }

  steps.push(cfgData.startSymbol)
  const ok = tryDerive([cfgData.startSymbol])
  return ok ? steps : null
}

function runSimulation(scroll = false) {
  simError.value   = ''
  derivation.value = null
  simStatus.value  = ''
  const input = props.testString.trim()
  if (!input) return

  const steps = getDerivation(cfg.value, input)
  if (steps) {
    derivation.value = steps
    simStatus.value  = 'ok'
    if (scroll) {
      nextTick(() => setTimeout(() => simCardRef.value?.scrollIntoView({ behavior: 'smooth', block: 'start' }), 120))
    }
  } else {
    simStatus.value = 'fail'
    simError.value  = `"${input}" is not accepted by this grammar.`
  }
}

// scroll=true only when Run is clicked (testString prop changes)
watch(() => props.testString, (val) => { if (val) runSimulation(true) })
watch(() => props.problemId, () => {
  derivation.value = null; simError.value = ''; simStatus.value = ''
})
// on tab switch, just re-derive without scrolling
onMounted(() => { if (props.testString) runSimulation(false) })
</script>

<template>
  <div class="cfg-wrap">

    <!-- Header -->
    <div class="cfg-header">
      <div class="header-left">
        <span class="badge">CFG</span>
        <span class="title">Problem {{ problemId }}</span>
      </div>
      <div class="header-right">
        <span class="dot nt-dot"></span><span class="leg">Non-terminal</span>
        <span class="dot t-dot"></span><span class="leg">Terminal</span>
      </div>
    </div>

    <!-- Regex -->
    <div class="regex-wrap">
      <span class="regex-label">Regex</span>
      <code class="regex-code">{{ problemRegex }}</code>
    </div>

    <!-- Productions -->
    <div class="rule-card">
      <div class="rule-card-head">
        <span>Production Rules</span>
        <span class="rule-count">{{ cfg.productions.length }} rules</span>
      </div>
      <div class="rule-list">
        <div
          v-for="(prod, idx) in cfg.productions"
          :key="idx"
          class="rule-row"
          :class="{ hovered: hoveredRow === idx }"
          @mouseenter="hoveredRow = idx"
          @mouseleave="hoveredRow = null"
        >
          <span class="lhs">{{ prod.lhs }}</span>
          <span class="arrow">→</span>
          <span class="rhs-group">
            <template v-for="(alt, ai) in prod.alts" :key="ai">
              <span v-if="ai > 0" class="pipe">|</span>
              <span
                v-for="(tok, ti) in tokenizeAlt(alt)"
                :key="ti"
                :class="['tok', tok.isNT ? 'nt' : 't']"
              >{{ tok.ch }}</span>
            </template>
          </span>
          <span class="rule-note">{{ prod.note }}</span>
        </div>
      </div>
    </div>

    <!-- Simulation -->
    <div class="sim-card" ref="simCardRef">
      <div class="sim-head">String Simulation</div>
      <div class="sim-body">
        <!-- No result yet -->
        <div v-if="!derivation && !simError" class="sim-hint">
          Run a test string from the left panel to see the leftmost derivation here.
        </div>

        <!-- Error -->
        <div v-if="simError" class="sim-error">{{ simError }}</div>

        <!-- Derivation steps -->
        <div v-if="derivation" class="deriv-box">
          <div class="deriv-title">
            Leftmost Derivation of
            <code class="deriv-input-label">{{ testString.trim() }}</code>
            <span class="deriv-steps-count">{{ derivation.length - 1 }} step{{ derivation.length !== 2 ? 's' : '' }}</span>
          </div>

          <div class="deriv-lines">
            <div
              v-for="(step, si) in derivation.slice(1)"
              :key="si"
              class="deriv-line"
            >
              <span class="deriv-lhs">
                <span
                  v-if="si === 0"
                  v-for="(tok, ti) in tokenizeStep(derivation[0])"
                  :key="ti"
                  :class="['dtok', tok.isNT ? 'dnt' : 'dt']"
                >{{ tok.ch }}</span>
              </span>
              <span class="deriv-sep">→</span>
              <span class="deriv-form">
                <span
                  v-for="(tok, ti) in tokenizeStep(step)"
                  :key="ti"
                  :class="['dtok', tok.isLambda ? 'dlambda' : tok.isNT ? 'dnt' : 'dt']"
                >{{ tok.ch }}</span>
              </span>
            </div>
          </div>
        </div>
      </div>
    </div>

  </div>
</template>

<style scoped>
.cfg-wrap {
  display: flex;
  flex-direction: column;
  gap: 0.75rem;
  max-width: 900px;
  margin: 20px auto;
  padding: 1.2rem;
  background: #ffffff;
  border-radius: 12px;
  box-shadow: 0 8px 20px rgba(0,0,0,0.08);
  font-family: 'Inter', 'Segoe UI', sans-serif;
}

/* Header */
.cfg-header { display:flex; align-items:center; justify-content:space-between; flex-wrap:wrap; gap:0.5rem; }
.header-left { display:flex; align-items:center; gap:0.6rem; }
.badge { background:#1e1e2e; color:#cdd6f4; font-size:11px; font-weight:600; letter-spacing:0.08em; padding:3px 8px; border-radius:5px; }
.title { font-size:18px; font-weight:600; color:#1e1e2e; }
.header-right { display:flex; align-items:center; gap:0.4rem; }
.dot { width:8px; height:8px; border-radius:50%; display:inline-block; }
.nt-dot { background:#f59e0b; }
.t-dot  { background:#10b981; margin-left:0.6rem; }
.leg { font-size:12px; color:#6b7280; }

/* Regex */
.regex-wrap { display:flex; align-items:flex-start; gap:0.6rem; background:#f8fafc; border:1px solid #e2e8f0; border-radius:8px; padding:0.6rem 0.9rem; }
.regex-label { font-size:11px; font-weight:600; color:#94a3b8; letter-spacing:0.06em; text-transform:uppercase; padding-top:2px; white-space:nowrap; }
.regex-code { font-family:'Courier New',monospace; font-size:13px; color:#334155; word-break:break-all; line-height:1.6; }

/* Rule card */
.rule-card { border:1px solid #e2e8f0; border-radius:10px; overflow:hidden; background:#fff; }
.rule-card-head { display:flex; align-items:center; justify-content:space-between; padding:0.55rem 1rem; background:#f8fafc; border-bottom:1px solid #e2e8f0; font-size:12px; font-weight:600; color:#64748b; letter-spacing:0.04em; text-transform:uppercase; }
.rule-count { font-size:11px; font-weight:500; color:#94a3b8; background:#f1f5f9; padding:2px 8px; border-radius:20px; border:1px solid #e2e8f0; }
.rule-list { display:flex; flex-direction:column; }
.rule-row { display:flex; align-items:center; gap:0; padding:0.45rem 1rem; border-bottom:1px solid #f1f5f9; font-family:'Courier New',monospace; font-size:13.5px; transition:background 0.15s; cursor:default; }
.rule-row:last-child { border-bottom:none; }
.rule-row.hovered { background:#fafafa; }
.lhs { font-weight:700; color:#f59e0b; min-width:18px; }
.rule-row.hovered .lhs { color:#d97706; }
.arrow { color:#cbd5e1; margin:0 0.65rem; font-size:15px; }
.rhs-group { display:flex; flex-wrap:wrap; align-items:center; gap:1px; flex:1; }
.pipe { color:#cbd5e1; margin:0 0.35rem; }
.tok { font-family:'Courier New',monospace; font-size:13.5px; }
.tok.nt { color:#f59e0b; }
.tok.t  { color:#10b981; }
.rule-note { font-size:10.5px; color:#94a3b8; margin-left:auto; padding-left:12px; font-family:'Inter',sans-serif; white-space:nowrap; overflow:hidden; text-overflow:ellipsis; max-width:260px; }
.rule-row.hovered .rule-note { color:#64748b; }

/* Simulation card */
.sim-card { border:1px solid #e2e8f0; border-radius:10px; overflow:hidden; background:#fff; }
.sim-head { padding:0.55rem 1rem; background:#f8fafc; border-bottom:1px solid #e2e8f0; font-size:12px; font-weight:600; color:#64748b; letter-spacing:0.04em; text-transform:uppercase; }
.sim-body { padding:1rem; display:flex; flex-direction:column; gap:0.75rem; }

.sim-hint { font-size:12.5px; color:#94a3b8; font-style:italic; padding:4px 0; }

.sim-error { font-size:12.5px; color:#ef4444; padding:6px 10px; background:#fef2f2; border:1px solid #fecaca; border-radius:6px; }

/* Derivation display */
.deriv-box { background:#f8fafc; border:1px solid #e2e8f0; border-radius:8px; overflow:hidden; }
.deriv-title {
  display:flex; align-items:center; gap:8px; flex-wrap:wrap;
  padding:8px 14px; border-bottom:1px solid #e2e8f0;
  font-size:11.5px; font-weight:600; color:#64748b;
}
.deriv-input-label { font-family:'Courier New',monospace; color:#0f172a; background:#e2e8f0; padding:1px 6px; border-radius:4px; }
.deriv-steps-count { margin-left:auto; font-size:10.5px; color:#94a3b8; font-weight:500; }

.deriv-lines { padding:12px 14px; display:flex; flex-direction:column; gap:3px; max-height:440px; overflow-y:auto; }

.deriv-line  { display:flex; align-items:baseline; gap:6px; font-family:'Courier New',monospace; font-size:13px; }
.deriv-lhs   { min-width:1ch; flex-shrink:0; display:flex; }
.deriv-sep   { color:#94a3b8; font-size:14px; flex-shrink:0; }
.deriv-form  { display:flex; flex-wrap:wrap; gap:0; }
.dtok.dnt    { color:#f59e0b; font-weight:700; }
.dtok.dt     { color:#10b981; }
.dtok.dlambda{ color:#94a3b8; font-style:italic; }
</style>
