<template>
  <div class="p-6 space-y-5">
    <div class="flex items-center justify-between">
      <h1 class="text-xl font-bold text-white">Clients EBP</h1>
    </div>

    <div v-if="loading" class="text-center py-12 text-gray-500 text-sm">Chargement…</div>

    <div v-else-if="!clients.length" class="text-center py-12 text-gray-500 text-sm">
      Aucun client enregistré
    </div>

    <div v-else class="grid gap-4">
      <div
        v-for="client in clients"
        :key="client.id"
        class="bg-gray-900 border border-gray-800 rounded-xl p-5 hover:border-gray-700 transition-colors"
      >
        <div class="flex items-start justify-between gap-4">
          <div>
            <div class="flex items-center gap-2 mb-1">
              <h2 class="font-semibold text-white text-sm">
                {{ client.display_name || client.db_name }}
              </h2>
              <span class="font-mono text-xs text-gray-500 bg-gray-800 px-1.5 py-0.5 rounded">
                {{ client.db_name }}
              </span>
            </div>
            <div class="flex items-center gap-4 text-xs text-gray-500">
              <span v-if="client.ebp_file">📁 {{ client.ebp_file }}</span>
              <span v-if="client.db_id" class="font-mono">🔑 {{ client.db_id.slice(0, 8) }}…</span>
            </div>
          </div>

          <!-- Stats -->
          <div class="flex items-center gap-6 shrink-0">
            <div class="text-center">
              <p class="text-lg font-bold text-white">{{ client.session_count }}</p>
              <p class="text-xs text-gray-500">fichiers</p>
            </div>
            <div class="text-center">
              <p class="text-lg font-bold" :class="client.error_count > 0 ? 'text-red-400' : 'text-gray-400'">
                {{ client.error_count }}
              </p>
              <p class="text-xs text-gray-500">erreurs</p>
            </div>
            <div class="text-center">
              <p class="text-sm text-gray-400">{{ formatDate(client.last_received_at) }}</p>
              <p class="text-xs text-gray-500">dernier log</p>
            </div>
          </div>
        </div>

        <!-- View logs link -->
        <div class="mt-3 pt-3 border-t border-gray-800 flex gap-3">
          <RouterLink
            :to="`/logs?client_id=${client.id}`"
            class="text-xs text-indigo-400 hover:text-indigo-300 transition-colors"
          >
            Voir les logs →
          </RouterLink>
          <RouterLink
            :to="`/logs?client_id=${client.id}&level=error`"
            class="text-xs text-red-400 hover:text-red-300 transition-colors"
            v-if="client.error_count > 0"
          >
            Voir les erreurs ({{ client.error_count }}) →
          </RouterLink>
        </div>
      </div>
    </div>
  </div>
</template>

<script setup>
import { ref, onMounted } from 'vue'
import api from '@/api/client'

const clients = ref([])
const loading = ref(true)

async function load() {
  try {
    const { data } = await api.get('/clients')
    clients.value = data
  } finally {
    loading.value = false
  }
}

function formatDate(iso) {
  if (!iso) return '—'
  const d = new Date(iso)
  return d.toLocaleString('fr-FR', { day: '2-digit', month: '2-digit', year: '2-digit', hour: '2-digit', minute: '2-digit' })
}

onMounted(load)
</script>
