<template>
  <div
    class="bg-gray-900 border rounded-xl px-4 py-4"
    :class="alert ? 'border-red-800/60' : 'border-gray-800'"
  >
    <div class="flex items-start justify-between">
      <p class="text-xs text-gray-500 leading-tight">{{ label }}</p>
      <span class="text-lg">{{ icon }}</span>
    </div>
    <p
      class="text-2xl font-bold mt-2"
      :class="alert ? 'text-red-400' : 'text-white'"
    >{{ value }}</p>
  </div>
</template>

<script setup>
defineProps({
  label: String,
  value: [String, Number],
  icon: String,
  color: { type: String, default: 'indigo' },
  alert: { type: Boolean, default: false },
})
</script>
