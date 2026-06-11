<template>
  <div class="p-6 space-y-4">
    <div class="flex items-center justify-between">
      <h1 class="text-xl font-bold text-white">Clients EBP</h1>
      <button @click="load" class="text-xs text-gray-500 hover:text-gray-300 transition-colors">
        🔄 Rafraîchir
      </button>
    </div>

    <div v-if="loading" class="text-center py-12 text-gray-500 text-sm">Chargement…</div>

    <div v-else-if="!clients.length" class="text-center py-12 text-gray-500 text-sm">
      Aucun client enregistré
    </div>

    <div v-else class="bg-gray-900 border border-gray-800 rounded-xl overflow-hidden">
      <table class="w-full text-sm">
        <thead>
          <tr class="border-b border-gray-800 text-xs text-gray-500 uppercase tracking-wider">
            <th class="px-5 py-3 text-left w-8">Statut</th>
            <th class="px-4 py-3 text-left">Client</th>
            <th class="px-4 py-3 text-left">Dernier log</th>
            <th class="px-4 py-3 text-center">Erreurs</th>
            <th class="px-4 py-3 text-center">Warnings</th>
            <th class="px-4 py-3 text-center">Sessions</th>
            <th class="px-4 py-3 text-right"></th>
          </tr>
        </thead>
        <tbody>
          <tr
            v-for="client in clients"
            :key="client.id"
            class="border-b border-gray-800/60 hover:bg-gray-800/30 transition-colors"
          >
            <!-- Statut -->
            <td class="px-5 py-3.5">
              <span
                class="inline-block w-2.5 h-2.5 rounded-full"
                :class="statusDot(client)"
                :title="statusLabel(client)"
              />
            </td>

            <!-- Nom -->
            <td class="px-4 py-3.5">
              <p class="font-semibold text-white text-sm">{{ client.display_name || client.db_name }}</p>
              <p class="text-xs text-gray-500 font-mono">{{ client.db_name }}</p>
            </td>

            <!-- Dernier log -->
            <td class="px-4 py-3.5">
              <p v-if="client.last_session_at" class="text-sm text-gray-300">
                {{ formatRelative(client.last_session_at) }}
              </p>
              <p v-if="client.last_session_at" class="text-xs text-gray-600">
                {{ formatDate(client.last_session_at) }}
              </p>
              <p v-else class="text-xs text-gray-600">—</p>
            </td>

            <!-- Erreurs dernière session -->
            <td class="px-4 py-3.5 text-center">
              <span
                v-if="client.last_session_errors > 0"
                class="inline-flex items-center gap-1 px-2 py-0.5 bg-red-900/40 border border-red-800/50 text-red-400 text-xs font-semibold rounded-full"
              >
                {{ client.last_session_errors }}
              </span>
              <span v-else class="text-gray-600 text-xs">—</span>
            </td>

            <!-- Warnings dernière session -->
            <td class="px-4 py-3.5 text-center">
              <span
                v-if="client.last_session_warnings > 0"
                class="inline-flex items-center gap-1 px-2 py-0.5 bg-yellow-900/30 border border-yellow-800/40 text-yellow-400 text-xs font-semibold rounded-full"
              >
                {{ client.last_session_warnings }}
              </span>
              <span v-else class="text-gray-600 text-xs">—</span>
            </td>

            <!-- Total sessions -->
            <td class="px-4 py-3.5 text-center text-gray-500 text-xs">
              {{ client.session_count }}
            </td>

            <!-- Actions -->
            <td class="px-4 py-3.5 text-right">
              <div class="flex items-center justify-end gap-3">
                <RouterLink
                  v-if="client.last_session_errors > 0"
                  :to="`/logs?client_id=${client.id}&level=error`"
                  class="text-xs text-red-400 hover:text-red-300 transition-colors whitespace-nowrap"
                >
                  Voir erreurs →
                </RouterLink>
                <RouterLink
                  :to="`/logs?client_id=${client.id}`"
                  class="text-xs text-indigo-400 hover:text-indigo-300 transition-colors whitespace-nowrap"
                >
                  Tous les logs →
                </RouterLink>
              </div>
            </td>
          </tr>
        </tbody>
      </table>
    </div>
  </div>
</template>

<script setup>
import { ref, onMounted } from 'vue'
import api from '@/api/client'

const clients = ref([])
const loading = ref(true)

async function load() {
  loading.value = true
  try {
    const { data } = await api.get('/clients')
    clients.value = data
  } finally {
    loading.value = false
  }
}

// 🔴 erreurs  🟡 warnings  🟢 ok  ⚫ jamais reçu
function statusDot(client) {
  if (!client.last_session_at) return 'bg-gray-600'
  if (client.last_session_errors > 0) return 'bg-red-500'
  if (client.last_session_warnings > 0) return 'bg-yellow-400'
  return 'bg-green-500'
}

function statusLabel(client) {
  if (!client.last_session_at) return 'Aucun log reçu'
  if (client.last_session_errors > 0) return `${client.last_session_errors} erreur(s) lors du dernier run`
  if (client.last_session_warnings > 0) return `${client.last_session_warnings} warning(s) lors du dernier run`
  return 'Dernier run sans erreur'
}

function formatDate(iso) {
  if (!iso) return '—'
  return new Date(iso).toLocaleString('fr-FR', {
    day: '2-digit', month: '2-digit', year: '2-digit',
    hour: '2-digit', minute: '2-digit',
  })
}

function formatRelative(iso) {
  const diff = Date.now() - new Date(iso).getTime()
  const mins = Math.floor(diff / 60000)
  if (mins < 1)  return 'À l\'instant'
  if (mins < 60) return `il y a ${mins} min`
  const hours = Math.floor(mins / 60)
  if (hours < 24) return `il y a ${hours}h`
  const days = Math.floor(hours / 24)
  return `il y a ${days}j`
}

onMounted(load)
</script>
