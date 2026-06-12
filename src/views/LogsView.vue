<template>
  <div class="flex flex-col h-full">
    <!-- Header + filters -->
    <div class="px-6 py-4 border-b border-gray-800 space-y-3 bg-gray-900 shrink-0">
      <div class="flex items-center justify-between">
        <h1 class="text-lg font-bold text-white">Logs</h1>
        <div class="flex items-center gap-2 text-xs text-gray-500">
          <span>{{ total.toLocaleString('fr-FR') }} résultats</span>
          <button
            v-if="hasActiveFilters && total > 0"
            @click="deleteBulk"
            class="px-2.5 py-1 bg-red-900/40 hover:bg-red-900/70 border border-red-800/40 rounded text-red-400 hover:text-red-300 transition-colors"
          >
            🗑 Supprimer les {{ total.toLocaleString('fr-FR') }} résultats
          </button>
          <button
            @click="load"
            class="px-2.5 py-1 bg-gray-800 hover:bg-gray-700 rounded text-gray-400 hover:text-white transition-colors"
          >
            🔄 Rafraîchir
          </button>
        </div>
      </div>

      <!-- Filters row -->
      <div class="flex flex-wrap items-center gap-2">
        <!-- Client filter -->
        <select
          v-model="filters.client_id"
          class="px-2.5 py-1.5 bg-gray-800 border border-gray-700 rounded-lg text-xs text-gray-300 focus:outline-none focus:border-indigo-500"
          @change="resetAndLoad"
        >
          <option value="">Tous les clients</option>
          <option v-for="c in clients" :key="c.id" :value="c.id">{{ c.display_name || c.db_name }}</option>
        </select>

        <!-- Level filter -->
        <select
          v-model="filters.level"
          class="px-2.5 py-1.5 bg-gray-800 border border-gray-700 rounded-lg text-xs text-gray-300 focus:outline-none focus:border-indigo-500"
          @change="resetAndLoad"
        >
          <option value="">Tous les niveaux</option>
          <option value="error">🔴 Erreur</option>
          <option value="warning">🟡 Warning</option>
          <option value="info">🔵 Info</option>
          <option value="system">⚙️ Système</option>
        </select>

        <!-- API code filter -->
        <input
          v-model="filters.api_code"
          type="text"
          placeholder="API (ex: LITPRX)"
          class="px-2.5 py-1.5 bg-gray-800 border border-gray-700 rounded-lg text-xs text-gray-300 w-36 focus:outline-none focus:border-indigo-500"
          @keyup.enter="resetAndLoad"
        />

        <!-- Search -->
        <input
          v-model="filters.search"
          type="text"
          placeholder="Rechercher dans les messages…"
          class="flex-1 min-w-48 px-2.5 py-1.5 bg-gray-800 border border-gray-700 rounded-lg text-xs text-gray-300 focus:outline-none focus:border-indigo-500"
          @keyup.enter="resetAndLoad"
        />

        <!-- Date range -->
        <input
          v-model="filters.date_from"
          type="datetime-local"
          class="px-2.5 py-1.5 bg-gray-800 border border-gray-700 rounded-lg text-xs text-gray-300 focus:outline-none focus:border-indigo-500"
          @change="resetAndLoad"
        />
        <span class="text-gray-600 text-xs">→</span>
        <input
          v-model="filters.date_to"
          type="datetime-local"
          class="px-2.5 py-1.5 bg-gray-800 border border-gray-700 rounded-lg text-xs text-gray-300 focus:outline-none focus:border-indigo-500"
          @change="resetAndLoad"
        />

        <button
          v-if="hasActiveFilters"
          @click="clearFilters"
          class="px-2.5 py-1.5 bg-gray-700 hover:bg-gray-600 rounded-lg text-xs text-gray-400 hover:text-white transition-colors"
        >
          ✕ Effacer
        </button>
      </div>
    </div>

    <!-- Log lines -->
    <div class="flex-1 overflow-y-auto font-mono text-xs">
      <div v-if="loading && !entries.length" class="p-8 text-center text-gray-500">
        Chargement…
      </div>

      <div v-else-if="!entries.length" class="p-8 text-center text-gray-500">
        Aucun résultat
      </div>

      <div v-else>
        <LogLine
          v-for="entry in entries"
          :key="entry.id"
          :entry="entry"
          :client-name="clientMap[entry.client_id]"
          @delete="deleteOne"
        />

        <!-- Load more -->
        <div v-if="entries.length < total" class="p-4 text-center">
          <button
            @click="loadMore"
            :disabled="loading"
            class="px-4 py-2 bg-gray-800 hover:bg-gray-700 rounded-lg text-xs text-gray-400 hover:text-white transition-colors disabled:opacity-50"
          >
            {{ loading ? 'Chargement…' : `Charger plus (${total - entries.length} restants)` }}
          </button>
        </div>
      </div>
    </div>
  </div>
</template>

<script setup>
import { ref, reactive, computed, onMounted } from 'vue'
import { useRoute } from 'vue-router'
import api from '@/api/client'
import LogLine from '@/components/LogLine.vue'

const route = useRoute()

const entries = ref([])
const total = ref(0)
const loading = ref(false)
const clients = ref([])
const page = ref(1)
const PAGE_SIZE = 200

const filters = reactive({
  client_id: route.query.client_id || '',
  level: route.query.level || '',
  api_code: '',
  search: '',
  date_from: '',
  date_to: '',
})

const clientMap = computed(() => {
  const map = {}
  for (const c of clients.value) {
    map[c.id] = c.display_name || c.db_name
  }
  return map
})

const hasActiveFilters = computed(() =>
  filters.client_id || filters.level || filters.api_code || filters.search || filters.date_from || filters.date_to
)

async function load() {
  loading.value = true
  try {
    const params = {
      page: page.value,
      page_size: PAGE_SIZE,
    }
    if (filters.client_id) params.client_id = filters.client_id
    if (filters.level) params.level = filters.level
    if (filters.api_code) params.api_code = filters.api_code
    if (filters.search) params.search = filters.search
    if (filters.date_from) params.date_from = new Date(filters.date_from).toISOString()
    if (filters.date_to) params.date_to = new Date(filters.date_to).toISOString()

    const { data } = await api.get('/logs', { params })
    if (page.value === 1) {
      entries.value = data.items
    } else {
      entries.value.push(...data.items)
    }
    total.value = data.total
  } finally {
    loading.value = false
  }
}

function resetAndLoad() {
  page.value = 1
  entries.value = []
  load()
}

function loadMore() {
  page.value++
  load()
}

function clearFilters() {
  filters.client_id = ''
  filters.level = ''
  filters.api_code = ''
  filters.search = ''
  filters.date_from = ''
  filters.date_to = ''
  resetAndLoad()
}

async function loadClients() {
  try {
    const { data } = await api.get('/admin/clients')
    clients.value = data
  } catch {}
}

async function deleteOne(id) {
  try {
    await api.delete(`/logs/${id}`)
    entries.value = entries.value.filter(e => e.id !== id)
    total.value = Math.max(0, total.value - 1)
  } catch (e) {
    console.error('deleteOne', e)
  }
}

async function deleteBulk() {
  const msg = `Supprimer ${total.value.toLocaleString('fr-FR')} logs ?\n\nCette action est irréversible.`
  if (!confirm(msg)) return

  const params = {}
  if (filters.client_id) params.client_id = filters.client_id
  if (filters.level) params.level = filters.level
  if (filters.date_from) params.date_from = new Date(filters.date_from).toISOString()
  if (filters.date_to) params.date_to = new Date(filters.date_to).toISOString()

  try {
    const { data } = await api.delete('/logs', { params })
    resetAndLoad()
  } catch (e) {
    alert(e.response?.data?.detail || 'Erreur lors de la suppression')
  }
}

onMounted(() => {
  loadClients()
  load()
})
</script>
