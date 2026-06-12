<template>
  <div class="p-6 space-y-6">

    <!-- Titre + refresh -->
    <div class="flex items-center justify-between">
      <h1 class="text-xl font-bold text-white">Dashboard</h1>
      <button @click="load" class="text-xs text-gray-500 hover:text-gray-300 transition-colors">🔄 Rafraîchir</button>
    </div>

    <!-- ── Tuiles clients ───────────────────────────────────────── -->
    <div v-if="loadingClients" class="text-sm text-gray-500">Chargement…</div>

    <div v-else-if="clients.length" class="grid grid-cols-2 md:grid-cols-3 lg:grid-cols-4 xl:grid-cols-5 gap-3">
      <RouterLink
        v-for="c in clients"
        :key="c.id"
        :to="`/logs?client_id=${c.id}`"
        class="group relative rounded-2xl p-4 flex flex-col gap-2 transition-all duration-150 hover:scale-[1.03] hover:shadow-lg cursor-pointer select-none"
        :class="cardBg(c.status)"
      >
        <!-- Indicateur lumineux -->
        <span
          class="absolute top-3 right-3 w-2.5 h-2.5 rounded-full"
          :class="[dotColor(c.status), c.status !== 'ok' ? 'animate-pulse' : '']"
        />

        <!-- Nom client -->
        <div>
          <p class="font-bold text-white text-sm leading-tight truncate pr-4">
            {{ c.display_name || c.db_name }}
          </p>
          <p class="text-xs font-mono opacity-50 truncate">{{ c.db_name }}</p>
        </div>

        <!-- Badge statut -->
        <div>
          <span
            class="inline-flex items-center gap-1 text-xs font-semibold px-2 py-0.5 rounded-full"
            :class="badgeStyle(c.status)"
          >
            {{ statusIcon(c.status) }} {{ statusText(c.status) }}
          </span>
        </div>

        <!-- Métriques -->
        <div class="mt-auto pt-2 border-t border-white/10 flex items-center justify-between text-xs opacity-70">
          <span v-if="c.last_session_errors > 0" class="text-red-300 font-semibold">
            {{ c.last_session_errors }} erreur{{ c.last_session_errors > 1 ? 's' : '' }}
          </span>
          <span v-else-if="c.last_session_warnings > 0" class="text-yellow-300 font-semibold">
            {{ c.last_session_warnings }} warning{{ c.last_session_warnings > 1 ? 's' : '' }}
          </span>
          <span v-else class="text-white/50">—</span>

          <span class="text-white/40">
            {{ formatRelative(c.last_session_at) }}
          </span>
        </div>
      </RouterLink>
    </div>

    <!-- ── Compteurs globaux ────────────────────────────────────── -->
    <div class="grid grid-cols-2 lg:grid-cols-4 gap-3">
      <StatCard label="Logs aujourd'hui"    :value="stats?.logs_today ?? '—'"     icon="📋" />
      <StatCard label="Erreurs aujourd'hui" :value="stats?.errors_today ?? '—'"   icon="🔴" :alert="stats?.errors_today > 0" />
      <StatCard label="Warnings aujourd'hui":value="stats?.warnings_today ?? '—'" icon="🟡" />
      <StatCard label="Runs aujourd'hui"    :value="stats?.sessions_today ?? '—'" icon="📬" />
    </div>

    <!-- ── Dernières erreurs ────────────────────────────────────── -->
    <div class="bg-gray-900 border border-gray-800 rounded-xl overflow-hidden">
      <div class="flex items-center justify-between px-5 py-3 border-b border-gray-800">
        <h2 class="text-sm font-semibold text-white">Dernières erreurs</h2>
        <RouterLink to="/logs?level=error" class="text-xs text-indigo-400 hover:text-indigo-300">Voir tout →</RouterLink>
      </div>

      <div v-if="!stats?.recent_errors?.length" class="px-5 py-6 text-sm text-gray-500 text-center">
        ✅ Aucune erreur récente
      </div>

      <table v-else class="w-full text-xs">
        <tbody>
          <tr
            v-for="err in stats.recent_errors"
            :key="err.id"
            class="border-b border-gray-800/50 hover:bg-gray-800/30 transition-colors"
          >
            <td class="px-5 py-2.5 text-gray-300 font-semibold whitespace-nowrap w-28">{{ err.client }}</td>
            <td class="px-3 py-2.5 w-14">
              <span class="font-mono bg-gray-800 px-1.5 py-0.5 rounded text-gray-400">{{ err.api_code }}</span>
            </td>
            <td class="px-3 py-2.5 text-red-400 max-w-xs truncate">{{ err.message }}</td>
            <td class="px-5 py-2.5 text-gray-600 text-right whitespace-nowrap">{{ formatTime(err.logged_at) }}</td>
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
const clients = ref([])
const loadingClients = ref(true)

async function load() {
  const [s, c] = await Promise.all([
    api.get('/dashboard').catch(() => ({ data: null })),
    api.get('/clients').catch(() => ({ data: [] })),
  ])
  stats.value = s.data
  clients.value = c.data
  loadingClients.value = false
}

// ── Statut helpers ────────────────────────────────────────────────

const STATUS = {
  ok:      { text: 'OK',       icon: '✓',  bg: 'bg-green-900/70  border border-green-700/50',   dot: 'bg-green-400',  badge: 'bg-green-800/60 text-green-300' },
  warning: { text: 'Warning',  icon: '⚠',  bg: 'bg-yellow-900/60 border border-yellow-700/50',  dot: 'bg-yellow-400', badge: 'bg-yellow-800/60 text-yellow-300' },
  error:   { text: 'Erreur',   icon: '✕',  bg: 'bg-red-900/70    border border-red-700/50',     dot: 'bg-red-500',    badge: 'bg-red-800/60 text-red-300' },
  silent:  { text: 'Silencieux', icon: '!', bg: 'bg-orange-900/60 border border-orange-700/50', dot: 'bg-orange-500', badge: 'bg-orange-800/60 text-orange-300' },
  unknown: { text: 'Inconnu',  icon: '?',  bg: 'bg-gray-800/80   border border-gray-700/40',    dot: 'bg-gray-500',   badge: 'bg-gray-700/60 text-gray-400' },
}

const s = (status) => STATUS[status] || STATUS.unknown
const cardBg     = (status) => s(status).bg
const dotColor   = (status) => s(status).dot
const badgeStyle = (status) => s(status).badge
const statusIcon = (status) => s(status).icon
const statusText = (status) => s(status).text

// ── Formatage ─────────────────────────────────────────────────────

function formatRelative(iso) {
  if (!iso) return 'jamais'
  const mins = Math.floor((Date.now() - new Date(iso)) / 60000)
  if (mins < 1)   return 'à l\'instant'
  if (mins < 60)  return `${mins}min`
  const h = Math.floor(mins / 60)
  if (h < 24)    return `${h}h`
  return `${Math.floor(h / 24)}j`
}

function formatTime(iso) {
  return new Date(iso).toLocaleString('fr-FR', { day: '2-digit', month: '2-digit', hour: '2-digit', minute: '2-digit' })
}

onMounted(load)
</script>
