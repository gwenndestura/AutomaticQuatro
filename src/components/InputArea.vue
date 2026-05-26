<script setup>
import { ref, computed, watch } from 'vue'

const props = defineProps({
  regexStr: { type: String, required: true }
})

const emit = defineEmits(['simulate-string'])

const ROW_COUNT = 5
const inputVals = ref(Array(ROW_COUNT).fill(''))
const history   = ref([])
const activeIdx = ref(null)

// ── Suggestions per problem regex signature ──────────────────
const P1_SUGGESTIONS = {
  valid:   ['bbbababbbab', 'babababbbab', 'bbbababbaba'],
  invalid: ['bab', 'bababa', 'bbabaaba', 'baabbaabb']
}
const P2_SUGGESTIONS = {
  valid:   ['000101', '0001010', '101111'],
  invalid: ['101', '1010', '111001']
}

const suggestions = computed(() => {
  if (!props.regexStr) return { valid: [], invalid: [] }
  return props.regexStr.includes('bab') ? P1_SUGGESTIONS : P2_SUGGESTIONS
})

// ── Real-time validity per row ────────────────────────────────
const checkValidity = (val) => {
  if (!val || !props.regexStr) return null
  try {
    const jsRegex = new RegExp('^(?:' + props.regexStr.split('+').join('|') + ')$')
    return jsRegex.test(val) ? 'valid' : 'invalid'
  } catch {
    return null
  }
}

const validities = computed(() => inputVals.value.map(checkValidity))

// ── Char preview for focused row ──────────────────────────────
const activeChars = computed(() =>
  activeIdx.value !== null ? inputVals.value[activeIdx.value].split('') : []
)

// ── Run ───────────────────────────────────────────────────────
const run = (idx, str) => {
  const s = str ?? inputVals.value[idx]
  if (s === '') return
  emit('simulate-string', s)
  if (!history.value.find(h => h.str === s)) {
    history.value.unshift({ str: s, id: Date.now() })
    if (history.value.length > 10) history.value.pop()
  }
}

const useSuggestion = (s) => {
  const emptyIdx = inputVals.value.findIndex(v => v === '')
  const target = emptyIdx !== -1 ? emptyIdx : 0
  inputVals.value[target] = s
  run(target, s)
}

const clearHistory = () => { history.value = [] }

watch(() => props.regexStr, () => {
  inputVals.value = Array(ROW_COUNT).fill('')
  history.value   = []
})
</script>

<template>
  <div class="ia-root">

    <!-- 5 Input rows -->
    <div class="ia-rows">
      <div
        v-for="(_, i) in inputVals"
        :key="i"
        class="ia-input-row"
      >
        <span class="ia-row-num">{{ i + 1 }}</span>
        <div :class="['ia-input-wrap', validities[i] ? 'ia-wrap-' + validities[i] : '']">
          <input
            v-model="inputVals[i]"
            class="ia-input"
            placeholder="Enter string…"
            spellcheck="false"
            autocomplete="off"
            @focus="activeIdx = i"
            @blur="activeIdx = null"
            @keydown.enter="run(i)"
          />
          <span v-if="validities[i]" :class="['ia-validity-text', validities[i]]">{{ validities[i].toUpperCase() }}</span>
        </div>
        <button class="ia-run-btn" title="Simulate (Enter)" @click="run(i)">
          <svg width="10" height="10" viewBox="0 0 12 12" fill="none" aria-hidden="true">
            <polygon points="2,1 11,6 2,11" fill="currentColor"/>
          </svg>
          Run
        </button>
      </div>
    </div>

    <!-- Char tape for focused row -->
    <div v-if="activeChars.length" class="ia-chars">
      <span
        v-for="(c, i) in activeChars"
        :key="i"
        :class="['ia-char', validities[activeIdx] === 'invalid' ? 'ia-char-invalid' : '']"
      >{{ c }}</span>
      <span class="ia-char-count">{{ activeChars.length }} char{{ activeChars.length !== 1 ? 's' : '' }}</span>
    </div>

    <!-- Suggestions -->
    <div class="ia-suggest-section">
      <div class="ia-suggest-label ia-suggest-valid-label">Valid Examples</div>
      <div class="ia-chips">
        <button
          v-for="s in suggestions.valid"
          :key="s"
          class="ia-chip ia-chip-valid"
          @click="useSuggestion(s)"
        >{{ s }}</button>
      </div>
    </div>
    <div class="ia-suggest-section">
      <div class="ia-suggest-label ia-suggest-invalid-label">Invalid Examples</div>
      <div class="ia-chips">
        <button
          v-for="s in suggestions.invalid"
          :key="s"
          class="ia-chip ia-chip-invalid"
          @click="useSuggestion(s)"
        >{{ s }}</button>
      </div>
    </div>

    <!-- History -->
    <template v-if="history.length">
      <div class="ia-hist-header">
        <span class="ia-hist-label">Recent</span>
        <button class="ia-clear-btn" @click="clearHistory">Clear</button>
      </div>
      <div class="ia-history">
        <div
          v-for="item in history"
          :key="item.id"
          class="ia-hist-row"
        >
          <code class="ia-hist-str">{{ item.str }}</code>
          <button class="ia-rerun-btn" title="Re-run" @click="run(0, item.str)">
            <svg width="9" height="9" viewBox="0 0 12 12" fill="none" aria-hidden="true">
              <polygon points="2,1 11,6 2,11" fill="currentColor"/>
            </svg>
          </button>
        </div>
      </div>
    </template>

  </div>
</template>

<style scoped>
.ia-root {
  display: flex;
  flex-direction: column;
  gap: 10px;
}

/* Input rows */
.ia-rows {
  display: flex;
  flex-direction: column;
  gap: 6px;
}
.ia-input-row {
  display: flex;
  align-items: stretch;
  gap: 7px;
}
.ia-row-num {
  display: flex;
  align-items: center;
  justify-content: center;
  min-width: 18px;
  font-size: 10px;
  font-weight: 700;
  color: #b0b8c1;
  flex-shrink: 0;
  user-select: none;
}
.ia-input-wrap {
  flex: 1;
  display: flex;
  align-items: center;
  border: 1.5px solid #e1e4e8;
  border-radius: 8px;
  overflow: hidden;
  background: #fff;
  transition: border-color 0.15s, box-shadow 0.15s;
}
.ia-input-wrap:focus-within {
  border-color: #22c55e;
  box-shadow: 0 0 0 3px rgba(34,197,94,0.12);
}
.ia-wrap-valid   { border-color: #1a7f37 !important; box-shadow: 0 0 0 3px rgba(26,127,55,0.10) !important; }
.ia-wrap-invalid { border-color: #cf222e !important; box-shadow: 0 0 0 3px rgba(207,34,46,0.10) !important; }

.ia-validity-text {
  font-size: 9.5px;
  font-weight: 700;
  letter-spacing: 0.07em;
  padding: 0 9px 0 4px;
  flex-shrink: 0;
  white-space: nowrap;
}
.ia-validity-text.valid   { color: #1a7f37; }
.ia-validity-text.invalid { color: #cf222e; }

.ia-input {
  flex: 1;
  min-width: 0;
  padding: 8px 10px;
  border: none;
  outline: none;
  font-family: 'SFMono-Regular', 'Consolas', 'Liberation Mono', monospace;
  font-size: 12.5px;
  color: #0f172a;
  background: transparent;
}
.ia-input::placeholder { color: #b0b8c1; }

.ia-run-btn {
  display: inline-flex;
  align-items: center;
  gap: 5px;
  padding: 7px 12px;
  font-size: 12px;
  font-weight: 600;
  color: #57606a;
  background: #f6f8fa;
  border: 1.5px solid #e1e4e8;
  border-radius: 8px;
  cursor: pointer;
  transition: all 0.15s ease;
  flex-shrink: 0;
  letter-spacing: 0.01em;
  white-space: nowrap;
}
.ia-run-btn:hover {
  background: #1a7f37;
  border-color: #1a7f37;
  color: #ffffff;
  box-shadow: 0 2px 8px rgba(26,127,55,0.25);
  transform: translateY(-1px);
}
.ia-run-btn:active { transform: scale(0.96) translateY(0); }

/* Char preview */
.ia-chars {
  display: flex;
  flex-wrap: wrap;
  gap: 3px;
  align-items: center;
}
.ia-char {
  display: inline-flex;
  align-items: center;
  justify-content: center;
  min-width: 22px;
  height: 22px;
  padding: 0 5px;
  border: 1px solid #d1fae5;
  border-radius: 4px;
  background: #f0fdf4;
  font-family: 'SFMono-Regular', 'Consolas', monospace;
  font-size: 11px;
  font-weight: 700;
  color: #16a34a;
}
.ia-char.ia-char-invalid {
  border-color: #fecaca;
  background: #fef2f2;
  color: #dc2626;
}
.ia-char-count {
  font-size: 9.5px;
  color: #b0b8c1;
  margin-left: 2px;
}

/* Suggestions */
.ia-suggest-section {
  display: flex;
  flex-direction: column;
  gap: 5px;
}
.ia-suggest-label {
  font-size: 9.5px;
  font-weight: 700;
  letter-spacing: 0.08em;
  text-transform: uppercase;
}
.ia-suggest-valid-label   { color: #16a34a; }
.ia-suggest-invalid-label { color: #b91c1c; }
.ia-chips {
  display: flex;
  flex-wrap: wrap;
  gap: 5px;
}
.ia-chip {
  padding: 4px 9px;
  border: 1.5px solid #e1e4e8;
  border-radius: 20px;
  background: #f6f8fa;
  font-family: 'SFMono-Regular', 'Consolas', monospace;
  font-size: 11px;
  cursor: pointer;
  transition: all 0.13s;
  line-height: 1;
}
.ia-chip-valid {
  color: #166534;
  border-color: #bbf7d0;
  background: #f0fdf4;
}
.ia-chip-valid:hover {
  border-color: #16a34a;
  background: #dcfce7;
  color: #14532d;
}
.ia-chip-invalid {
  color: #991b1b;
  border-color: #fecaca;
  background: #fef2f2;
}
.ia-chip-invalid:hover {
  border-color: #dc2626;
  background: #fee2e2;
  color: #7f1d1d;
}

/* History */
.ia-hist-header {
  display: flex;
  align-items: center;
  justify-content: space-between;
}
.ia-hist-label {
  font-size: 9.5px;
  font-weight: 700;
  letter-spacing: 0.08em;
  text-transform: uppercase;
  color: #8c959f;
}
.ia-clear-btn {
  font-size: 9.5px;
  color: #b0b8c1;
  background: none;
  border: none;
  cursor: pointer;
  padding: 0;
  transition: color 0.13s;
}
.ia-clear-btn:hover { color: #cf222e; }

.ia-history {
  display: flex;
  flex-direction: column;
  gap: 4px;
}
.ia-hist-row {
  display: flex;
  align-items: center;
  gap: 6px;
  background: #f6f8fa;
  border: 1px solid #e1e4e8;
  border-radius: 6px;
  padding: 5px 8px;
}
.ia-hist-str {
  flex: 1;
  min-width: 0;
  font-family: 'SFMono-Regular', 'Consolas', monospace;
  font-size: 11.5px;
  color: #0f172a;
  overflow: hidden;
  text-overflow: ellipsis;
  white-space: nowrap;
}
.ia-rerun-btn {
  display: inline-flex;
  align-items: center;
  justify-content: center;
  width: 22px;
  height: 22px;
  border: 1px solid #e1e4e8;
  border-radius: 4px;
  background: #fff;
  color: #57606a;
  cursor: pointer;
  transition: all 0.13s;
  flex-shrink: 0;
}
.ia-rerun-btn:hover {
  background: #16a34a;
  border-color: #16a34a;
  color: #fff;
}
</style>
