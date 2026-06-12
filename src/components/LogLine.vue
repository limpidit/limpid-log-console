<template>
  <div
    class="group flex items-start gap-3 px-4 py-1.5 border-b border-gray-800/40 hover:bg-gray-800/20 transition-colors"
    :class="rowClass"
  >
    <!-- Level badge -->
    <span class="shrink-0 w-2 h-2 rounded-full mt-1.5" :class="dotClass" />

    <!-- Timestamp -->
    <span class="shrink-0 text-gray-500 w-32">{{ formattedTime }}</span>

    <!-- Client -->
    <span class="shrink-0 text-gray-400 w-28 truncate" :title="clientName">{{ clientName }}</span>

    <!-- API code -->
    <span class="shrink-0 font-mono text-gray-500 w-14">{{ entry.api_code }}</span>

    <!-- Level label -->
    <span class="shrink-0 w-16" :class="levelClass">{{ levelLabel }}</span>

    <!-- Message -->
    <span class="flex-1 min-w-0 break-all" :class="messageClass">{{ entry.message }}</span>

    <!-- Delete button (visible on hover) -->
    <button
      @click.stop="$emit('delete', entry.id)"
      class="shrink-0 opacity-0 group-hover:opacity-100 text-gray-600 hover:text-red-500 transition-all text-xs px-1"
      title="Supprimer ce log"
    >
      ✕
    </button>
  </div>
</template>

<script setup>
import { computed } from 'vue'

const props = defineProps({
  entry: Object,
  clientName: String,
})

defineEmits(['delete'])

const levelConfig = {
  error:   { dot: 'bg-red-500',    label: 'ERREUR',   row: 'bg-red-950/10',     msg: 'text-red-400',    lbl: 'text-red-500' },
  warning: { dot: 'bg-yellow-500', label: 'WARNING',  row: 'bg-yellow-950/10',  msg: 'text-yellow-300', lbl: 'text-yellow-500' },
  info:    { dot: 'bg-blue-500',   label: 'info',     row: '',                  msg: 'text-gray-300',   lbl: 'text-blue-500' },
  system:  { dot: 'bg-gray-600',   label: 'sys',      row: '',                  msg: 'text-gray-500',   lbl: 'text-gray-600' },
}

const cfg = computed(() => levelConfig[props.entry.level] || levelConfig.info)

const dotClass     = computed(() => cfg.value.dot)
const levelLabel   = computed(() => cfg.value.label)
const rowClass     = computed(() => cfg.value.row)
const messageClass = computed(() => cfg.value.msg)
const levelClass   = computed(() => cfg.value.lbl)

const formattedTime = computed(() => {
  const d = new Date(props.entry.logged_at)
  return d.toLocaleString('fr-FR', {
    day: '2-digit',
    month: '2-digit',
    hour: '2-digit',
    minute: '2-digit',
    second: '2-digit',
  })
})
</script>
