<script setup>
import { ref, computed, watch } from 'vue'

const props = defineProps({
    regexStr: { type: String, required: true }
})

const emit = defineEmits(['inputs-updated', 'simulate-string'])

const userInputs = ref(['', '', '', '', ''])

watch(userInputs, (val) => emit('inputs-updated', [...val]), { deep: true })

const inputResults = computed(() => {
    if (!props.regexStr) return userInputs.value.map(() => null)
    try {
        const jsRegex = new RegExp('^(?:' + props.regexStr.split('+').join('|') + ')$')
        return userInputs.value.map(val => {
            if (val === null || val === undefined) return null
            return jsRegex.test(val) ? 'valid' : 'invalid'
        })
    } catch {
        return userInputs.value.map(() => null)
    }
})

const rowClass = (idx) => {
    const r = inputResults.value[idx]
    if (userInputs.value[idx] === '') return ''
    return r === 'valid' ? 'is-valid' : 'is-invalid'
}
</script>

<template>
  <div class="input-list">
    <div
      v-for="(_, idx) in userInputs"
      :key="idx"
      class="input-row"
    >
      <div :class="['input-row-wrap', rowClass(idx)]">
        <span class="input-index">{{ idx + 1 }}</span>
        <input
          v-model="userInputs[idx]"
          :placeholder="`Enter string…`"
          class="str-input"
          spellcheck="false"
          autocomplete="off"
        />
        <span
          v-if="userInputs[idx] !== '' && inputResults[idx]"
          :class="['validity-chip', inputResults[idx]]"
        >
          <svg v-if="inputResults[idx] === 'valid'" width="9" height="9" viewBox="0 0 10 10" fill="none">
            <path d="M1.5 5l2.5 2.5L8.5 2" stroke="currentColor" stroke-width="1.8" stroke-linecap="round" stroke-linejoin="round"/>
          </svg>
          <svg v-else width="9" height="9" viewBox="0 0 10 10" fill="none">
            <path d="M2 2l6 6M8 2l-6 6" stroke="currentColor" stroke-width="1.8" stroke-linecap="round"/>
          </svg>
          {{ inputResults[idx] }}
        </span>
      </div>
      <button
        class="run-btn"
        title="Simulate this string"
        @click="$emit('simulate-string', userInputs[idx])"
      >
        <svg width="11" height="11" viewBox="0 0 12 12" fill="none" aria-hidden="true">
          <polygon points="2,1 11,6 2,11" fill="currentColor"/>
        </svg>
        Run
      </button>
    </div>
  </div>
</template>

<style scoped>
.input-list {
  display: flex;
  flex-direction: column;
  gap: 7px;
}

.input-row {
  display: flex;
  align-items: center;
  gap: 7px;
}

.input-row-wrap {
  flex: 1;
  display: flex;
  align-items: center;
  border: 1.5px solid #e1e4e8;
  border-radius: 8px;
  background: #ffffff;
  overflow: hidden;
  transition: border-color 0.15s, box-shadow 0.15s;
  min-width: 0;
}
.input-row-wrap:focus-within {
  border-color: #0969da;
  box-shadow: 0 0 0 3px rgba(9,105,218,0.09);
}
.input-row-wrap.is-valid   { border-color: #1a7f37; }
.input-row-wrap.is-invalid { border-color: #cf222e; }

.input-index {
  display: inline-flex;
  align-items: center;
  justify-content: center;
  width: 28px;
  height: 100%;
  font-size: 10px;
  font-weight: 700;
  color: #b0b8c1;
  background: #f6f8fa;
  border-right: 1.5px solid #e1e4e8;
  flex-shrink: 0;
  padding: 8px 0;
  letter-spacing: 0;
}

.str-input {
  flex: 1;
  min-width: 0;
  padding: 7px 8px;
  border: none;
  outline: none;
  font-family: 'SFMono-Regular', 'Consolas', 'Liberation Mono', monospace;
  font-size: 12.5px;
  color: #0f172a;
  background: transparent;
}
.str-input::placeholder { color: #b0b8c1; }

.validity-chip {
  display: inline-flex;
  align-items: center;
  gap: 3px;
  font-size: 9.5px;
  font-weight: 700;
  letter-spacing: 0.05em;
  text-transform: uppercase;
  padding: 2px 7px 2px 5px;
  border-radius: 20px;
  margin-right: 6px;
  flex-shrink: 0;
}
.validity-chip.valid   { background: #f0fff4; color: #1a7f37; }
.validity-chip.invalid { background: #fff0f0; color: #cf222e; }

.run-btn {
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
.run-btn:hover {
  background: #1a7f37;
  border-color: #1a7f37;
  color: #ffffff;
  box-shadow: 0 2px 8px rgba(26,127,55,0.25);
  transform: translateY(-1px);
}
.run-btn:active { transform: scale(0.96) translateY(0); }
</style>
