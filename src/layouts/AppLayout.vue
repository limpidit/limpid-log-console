<template>
  <div class="flex h-screen bg-gray-950 text-gray-100">
    <!-- Sidebar -->
    <aside class="flex flex-col w-56 shrink-0 bg-gray-900 border-r border-gray-800">
      <!-- Logo -->
      <div class="flex items-center gap-3 px-5 py-4 border-b border-gray-800">
        <div class="w-8 h-8 rounded-lg bg-indigo-600 flex items-center justify-center text-white font-bold text-sm">
          L
        </div>
        <div>
          <span class="font-semibold text-white text-sm block leading-tight">Log Console</span>
          <span class="text-xs text-indigo-400">Limpid IT</span>
        </div>
      </div>

      <!-- Nav -->
      <nav class="flex-1 px-3 py-3 space-y-0.5">
        <RouterLink
          v-for="item in navItems"
          :key="item.to"
          :to="item.to"
          class="flex items-center gap-3 px-3 py-2 rounded-lg text-sm font-medium transition-colors"
          :class="isActive(item.to)
            ? 'bg-indigo-600 text-white'
            : 'text-gray-400 hover:bg-gray-800 hover:text-gray-100'"
        >
          <span class="text-base">{{ item.icon }}</span>
          {{ item.label }}
        </RouterLink>
      </nav>

      <!-- User -->
      <div class="px-3 py-3 border-t border-gray-800">
        <div class="flex items-center gap-2 px-3 py-2 mb-1">
          <div class="w-6 h-6 rounded-full bg-indigo-500 flex items-center justify-center text-xs font-semibold shrink-0">
            {{ userInitial }}
          </div>
          <span class="text-xs text-gray-400 truncate">{{ auth.user?.email }}</span>
        </div>
        <button
          @click="handleLogout"
          class="w-full flex items-center gap-2 px-3 py-2 text-xs text-gray-500 rounded-lg hover:bg-red-900/30 hover:text-red-400 transition-colors"
        >
          <span>↪</span>
          Déconnexion
        </button>
      </div>
    </aside>

    <!-- Content -->
    <main class="flex-1 min-w-0 overflow-y-auto">
      <RouterView />
    </main>
  </div>
</template>

<script setup>
import { computed } from 'vue'
import { useRouter, useRoute } from 'vue-router'
import { useAuthStore } from '@/stores/auth'

const auth = useAuthStore()
const router = useRouter()
const route = useRoute()

const navItems = [
  { to: '/dashboard', icon: '📊', label: 'Dashboard' },
  { to: '/logs',      icon: '📋', label: 'Logs' },
  { to: '/clients',   icon: '🏢', label: 'Clients' },
  { to: '/settings',  icon: '⚙️',  label: 'Paramètres' },
]

const userInitial = computed(() =>
  (auth.user?.email || '?')[0].toUpperCase()
)

function isActive(path) {
  if (path === '/') return route.path === '/'
  return route.path.startsWith(path)
}

function handleLogout() {
  auth.logout()
  router.push('/login')
}
</script>
