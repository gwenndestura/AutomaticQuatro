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
      Problem {{ prob.id }}
    </button>
  </div>
</template>

<style scoped>
.prob-tabs {
  display: flex;
  gap: 6px;
}

.prob-btn {
  flex: 1;
  padding: 8px 12px;
  font-size: 12px;
  font-weight: 600;
  border: 1px solid #e1e4e8;
  border-radius: 6px;
  background: #f6f8fa;
  color: #57606a;
  cursor: pointer;
  transition: all 0.15s ease;
  letter-spacing: 0.01em;
}
.prob-btn:hover {
  background: #eaeef2;
  color: #0f172a;
  border-color: #d0d7de;
}
.prob-btn.prob-btn--active {
  background: #f0fff4;
  border-color: #aceebb;
  color: #1a7f37;
  font-weight: 700;
}
</style>
