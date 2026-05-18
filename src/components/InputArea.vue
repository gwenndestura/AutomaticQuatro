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
        <input
          v-model="userInputs[idx]"
          :placeholder="`String ${idx + 1}`"
          class="str-input"
          spellcheck="false"
          autocomplete="off"
        />
        <span
          v-if="userInputs[idx] !== '' && inputResults[idx]"
          :class="['validity-text', inputResults[idx]]"
        >{{ inputResults[idx] }}</span>
      </div>
      <button
        class="run-btn"
        title="Simulate this string"
        @click="$emit('simulate-string', userInputs[idx])"
      >
        Run
      </button>
    </div>
  </div>
</template>

<style scoped>
.input-list {
  display: flex;
  flex-direction: column;
  gap: 8px;
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
  border: 1px solid #e1e4e8;
  border-left: 3px solid #d0d7de;
  border-radius: 6px;
  background: #ffffff;
  overflow: hidden;
  transition: border-color 0.15s;
}
.input-row-wrap:focus-within {
  border-color: #0969da;
  border-left-color: #0969da;
  box-shadow: 0 0 0 3px rgba(9,105,218,0.08);
}
.input-row-wrap.is-valid   {
  border-left-color: #1a7f37;
}
.input-row-wrap.is-invalid {
  border-left-color: #cf222e;
}

.str-input {
  flex: 1;
  min-width: 0;
  padding: 7px 9px;
  border: none;
  outline: none;
  font-family: 'SFMono-Regular', 'Consolas', 'Liberation Mono', monospace;
  font-size: 12.5px;
  color: #0f172a;
  background: transparent;
}
.str-input::placeholder { color: #8c959f; }

.validity-text {
  font-size: 10px;
  font-weight: 700;
  letter-spacing: 0.04em;
  text-transform: uppercase;
  padding: 0 8px 0 4px;
  flex-shrink: 0;
}
.validity-text.valid   { color: #1a7f37; }
.validity-text.invalid { color: #cf222e; }

.run-btn {
  padding: 7px 12px;
  font-size: 12px;
  font-weight: 600;
  color: #57606a;
  background: #f6f8fa;
  border: 1px solid #e1e4e8;
  border-radius: 6px;
  cursor: pointer;
  transition: all 0.15s ease;
  flex-shrink: 0;
  letter-spacing: 0.01em;
}
.run-btn:hover {
  background: #1a7f37;
  border-color: #1a7f37;
  color: #ffffff;
}
.run-btn:active { transform: scale(0.96); }
</style>
