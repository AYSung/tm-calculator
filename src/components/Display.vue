<script setup>
defineProps({
  label: {
    type: String,
    required: true
  },
  value: {
    type: Number,
    required: true,
  },
  unit: {
    type: Number,
    required: true,
  },
})
</script>

<template>
  <div class="count">
    <h3>{{ label }}</h3>
    <div style="display: flex; align-items: center; gap: 0.6rem;">
      <h1 class="green">{{ value }}</h1>
      <h3 class="unit">{{ unit }}</h3>
    </div>
  </div>
</template>

<style scoped>
h1 {
  font-weight: 500;
  font-size: 2.6rem;
  position: relative;
  top: -10px;
}

h3 {
  font-size: 1.2rem;
}

h3.unit {
  position: relative;
  top: -2px;
}

.count {
  border-left: 1px solid lightgrey;
  padding-left: 1rem;
}
</style>