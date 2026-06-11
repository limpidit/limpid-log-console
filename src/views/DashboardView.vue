<template>
  <div class="p-6 space-y-6">
    <h1 class="text-xl font-bold text-white">Dashboard</h1>

    <!-- Stats cards -->
    <div class="grid grid-cols-2 lg:grid-cols-5 gap-4">
      <StatCard label="Clients" :value="stats?.clients_count ?? '—'" icon="🏢" color="indigo" />
      <StatCard label="Fichiers reçus aujourd'hui" :value="stats?.sessions_today ?? '—'" icon="📬" color="blue" />
      <StatCard label="Logs aujourd'hui" :value="stats?.logs_today ?? '—'" icon="📋" color="gray" />
      <StatCard label="Erreurs aujourd'hui" :value="stats?.errors_today ?? '—'" icon="🔴" color="red" :alert="stats?.errors_today > 0" />
      <StatCard label="Warnings aujourd'hui" :value="stats?.warnings_today ?? '—'" icon="🟡" color="yellow" :alert="stats?.warnings_today > 0" />
    </div>

    <!-- Recent errors -->
    <div class="bg-gray-900 border border-gray-800 rounded-xl overflow-hidden">
      <div class="flex items-center justify-between px-5 py-3 border-b border-gray-800">
        <h2 class="text-sm font-semibold text-white">Dernières erreurs</h2>
        <RouterLink to="/logs?level=error" class="text-xs text-indigo-400 hover:text-indigo-300">
          Voir tout →
        </RouterLink>
      </div>

      <div v-if="loading" class="px-5 py-8 text-center text-sm text-gray-500">
        Chargement…
      </div>

      <div v-else-if="!stats?.recent_errors?.length" class="px-5 py-8 text-center text-sm text-gray-500">
        ✅ Aucune erreur récente
      </div>

      <table v-else class="w-full text-xs">
        <thead>
          <tr class="border-b border-gray-800">
            <th class="px-5 py-2 text-left font-medium text-gray-500">Client</th>
            <th class="px-3 py-2 text-left font-medium text-gray-500">API</th>
            <th class="px-3 py-2 text-left font-medium text-gray-500">Message</th>
            <th class="px-5 py-2 text-right font-medium text-gray-500">Heure</th>
          </tr>
        </thead>
        <tbody>
          <tr
            v-for="err in stats.recent_errors"
            :key="err.id"
            class="border-b border-gray-800/50 hover:bg-gray-800/30 transition-colors"
          >
            <td class="px-5 py-2.5 text-gray-300 font-medium whitespace-nowrap">{{ err.client }}</td>
            <td class="px-3 py-2.5">
              <span class="px-1.5 py-0.5 bg-gray-800 rounded text-gray-400 font-mono">{{ err.api_code }}</span>
            </td>
            <td class="px-3 py-2.5 text-red-400 max-w-xs truncate">{{ err.message }}</td>
            <td class="px-5 py-2.5 text-gray-500 text-right whitespace-nowrap">{{ formatTime(err.logged_at) }}</td>
          </tr>
        </tbody>
      </table>
    </div>
  </div>
</template>

<script setup>
import { ref, onMounted } from 'vue'
import api from '@/api/client'
import StatCard from '@/components/StatCard.vue'

const stats = ref(null)
const loading = ref(true)

async function load() {
  try {
    const { data } = await api.get('/dashboard')
    stats.value = data
  } finally {
    loading.value = false
  }
}

function formatTime(iso) {
  const d = new Date(iso)
  return d.toLocaleString('fr-FR', { day: '2-digit', month: '2-digit', hour: '2-digit', minute: '2-digit' })
}

onMounted(load)
</script>
