<script setup>
import { ref, computed, watch } from 'vue'

const props = defineProps({
  regexStr: { type: String, required: true }
})

const emit = defineEmits(['simulate-string'])

const inputVal  = ref('')
const history   = ref([])

// ── Suggestions per problem regex signature ──────────────────
const P1_SUGGESTIONS = ['bab', 'bababa', 'bbbababbbab', 'bbabaaba', 'baabbaabb']
const P2_SUGGESTIONS = ['101', '1010', '000101', '0001010', '111001']

const suggestions = computed(() => {
  if (!props.regexStr) return []
  return props.regexStr.includes('bab') ? P1_SUGGESTIONS : P2_SUGGESTIONS
})

// ── Real-time validity ────────────────────────────────────────
const validity = computed(() => {
  if (!inputVal.value || !props.regexStr) return null
  try {
    const jsRegex = new RegExp('^(?:' + props.regexStr.split('+').join('|') + ')$')
    return jsRegex.test(inputVal.value) ? 'valid' : 'invalid'
  } catch {
    return null
  }
})

// ── Char preview ──────────────────────────────────────────────
const chars = computed(() => inputVal.value === '' ? [] : inputVal.value.split(''))

// ── Run ───────────────────────────────────────────────────────
const run = (str) => {
  const s = str ?? inputVal.value
  if (s === '') return
  emit('simulate-string', s)
  if (!history.value.find(h => h.str === s)) {
    history.value.unshift({ str: s, id: Date.now() })
    if (history.value.length > 10) history.value.pop()
  }
}

const onEnter = () => run()

const useSuggestion = (s) => {
  inputVal.value = s
  run(s)
}

const clearHistory = () => { history.value = [] }

watch(() => props.regexStr, () => {
  inputVal.value = ''
  history.value  = []
})
</script>

<template>
  <div class="ia-root">

    <!-- Input row -->
    <div class="ia-input-row">
      <div :class="['ia-input-wrap', validity ? 'ia-wrap-' + validity : '']">
        <input
          v-model="inputVal"
          class="ia-input"
          placeholder="Enter string…"
          spellcheck="false"
          autocomplete="off"
          @keydown.enter="onEnter"
        />
        <span v-if="validity" :class="['ia-validity-text', validity]">{{ validity.toUpperCase() }}</span>
      </div>
      <button class="ia-run-btn" title="Simulate (Enter)" @click="run()">
        <svg width="10" height="10" viewBox="0 0 12 12" fill="none" aria-hidden="true">
          <polygon points="2,1 11,6 2,11" fill="currentColor"/>
        </svg>
        Run
      </button>
    </div>

    <!-- Char tape -->
    <div v-if="chars.length" class="ia-chars">
      <span
        v-for="(c, i) in chars"
        :key="i"
        :class="['ia-char', validity === 'invalid' ? 'ia-char-invalid' : '']"
      >{{ c }}</span>
      <span class="ia-char-count">{{ chars.length }} char{{ chars.length !== 1 ? 's' : '' }}</span>
    </div>

    <!-- Suggestions -->
    <div class="ia-suggest-label">Suggestions</div>
    <div class="ia-chips">
      <button
        v-for="s in suggestions"
        :key="s"
        class="ia-chip"
        @click="useSuggestion(s)"
      >{{ s }}</button>
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
          <button class="ia-rerun-btn" title="Re-run" @click="run(item.str)">
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

/* Input row */
.ia-input-row {
  display: flex;
  align-items: stretch;
  gap: 7px;
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
.ia-suggest-label {
  font-size: 9.5px;
  font-weight: 700;
  letter-spacing: 0.08em;
  text-transform: uppercase;
  color: #8c959f;
}
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
  color: #0550ae;
  cursor: pointer;
  transition: all 0.13s;
  line-height: 1;
}
.ia-chip:hover {
  border-color: #22c55e;
  background: #f0fdf4;
  color: #16a34a;
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
