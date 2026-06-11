<template>
  <div class="min-h-screen bg-gray-950 flex items-center justify-center p-4">
    <div class="w-full max-w-sm">
      <!-- Logo -->
      <div class="flex items-center justify-center gap-3 mb-8">
        <div class="w-10 h-10 rounded-xl bg-indigo-600 flex items-center justify-center text-white font-bold text-lg">
          L
        </div>
        <div>
          <p class="font-bold text-white text-lg leading-tight">Log Console</p>
          <p class="text-xs text-indigo-400">Limpid IT</p>
        </div>
      </div>

      <!-- Card -->
      <div class="bg-gray-900 border border-gray-800 rounded-xl p-6">
        <h1 class="text-lg font-semibold text-white mb-6">Connexion</h1>

        <form @submit.prevent="handleLogin" class="space-y-4">
          <div>
            <label class="block text-xs font-medium text-gray-400 mb-1">Email</label>
            <input
              v-model="email"
              type="email"
              autocomplete="email"
              required
              placeholder="prenom.nom@limpidit.com"
              class="w-full px-3 py-2.5 bg-gray-800 border border-gray-700 rounded-lg text-sm text-white placeholder-gray-500 focus:outline-none focus:border-indigo-500 focus:ring-1 focus:ring-indigo-500"
            />
          </div>
          <div>
            <label class="block text-xs font-medium text-gray-400 mb-1">Mot de passe</label>
            <input
              v-model="password"
              type="password"
              autocomplete="current-password"
              required
              class="w-full px-3 py-2.5 bg-gray-800 border border-gray-700 rounded-lg text-sm text-white focus:outline-none focus:border-indigo-500 focus:ring-1 focus:ring-indigo-500"
            />
          </div>

          <p v-if="error" class="text-xs text-red-400 bg-red-900/20 border border-red-900/40 rounded-lg px-3 py-2">
            {{ error }}
          </p>

          <button
            type="submit"
            :disabled="loading"
            class="w-full py-2.5 bg-indigo-600 hover:bg-indigo-500 disabled:opacity-50 disabled:cursor-not-allowed text-white text-sm font-semibold rounded-lg transition-colors"
          >
            {{ loading ? 'Connexion…' : 'Se connecter' }}
          </button>
        </form>
      </div>
    </div>
  </div>
</template>

<script setup>
import { ref } from 'vue'
import { useRouter } from 'vue-router'
import { useAuthStore } from '@/stores/auth'

const auth = useAuthStore()
const router = useRouter()

const email = ref('')
const password = ref('')
const loading = ref(false)
const error = ref('')

async function handleLogin() {
  loading.value = true
  error.value = ''
  try {
    await auth.login(email.value, password.value)
    router.push('/dashboard')
  } catch (err) {
    error.value = err.response?.data?.detail || 'Erreur de connexion'
  } finally {
    loading.value = false
  }
}
</script>
