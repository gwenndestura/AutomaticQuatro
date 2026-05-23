<script setup>
defineProps({
    problems:   { type: Array,  required: true },
    modelValue: { type: Number, required: true }
})
defineEmits(['update:modelValue'])
</script>

<template>
  <div class="prob-tabs" role="tablist" aria-label="Problem selection">
    <button
      v-for="(prob, index) in problems"
      :key="prob.id"
      role="tab"
      :aria-selected="modelValue === index"
      :class="['prob-btn', modelValue === index ? 'prob-btn--active' : '']"
      @click="$emit('update:modelValue', index)"
    >
      <span class="prob-num">{{ prob.id }}</span>
      <span class="prob-label">Problem {{ prob.id }}</span>
    </button>
  </div>
</template>

<style scoped>
.prob-tabs {
  display: flex;
  gap: 8px;
}

.prob-btn {
  flex: 1;
  display: flex;
  align-items: center;
  gap: 8px;
  padding: 9px 14px;
  font-size: 12.5px;
  font-weight: 600;
  border: 1.5px solid #e1e4e8;
  border-radius: 8px;
  background: #ffffff;
  color: #57606a;
  cursor: pointer;
  transition: all 0.18s ease;
  letter-spacing: 0.01em;
}
.prob-btn:hover {
  background: #f6f8fa;
  color: #0f172a;
  border-color: #c6cdd5;
  transform: translateY(-1px);
  box-shadow: 0 2px 6px rgba(0,0,0,0.06);
}
.prob-btn.prob-btn--active {
  background: linear-gradient(135deg, #f0fff4, #e8f9ec);
  border-color: #6fcf8c;
  color: #1a7f37;
  font-weight: 700;
  box-shadow: 0 2px 8px rgba(26,127,55,0.12);
}

.prob-num {
  display: inline-flex;
  align-items: center;
  justify-content: center;
  width: 20px;
  height: 20px;
  border-radius: 50%;
  font-size: 10.5px;
  font-weight: 800;
  background: #e1e4e8;
  color: #57606a;
  flex-shrink: 0;
  transition: all 0.18s;
}
.prob-btn--active .prob-num {
  background: #1a7f37;
  color: #fff;
}

.prob-label { flex: 1; }
</style>
