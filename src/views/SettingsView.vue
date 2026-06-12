<template>
  <div class="p-6 space-y-8">
    <h1 class="text-xl font-bold text-white">Paramètres</h1>

    <!-- Clients management -->
    <section>
      <div class="flex items-center justify-between mb-3">
        <h2 class="text-sm font-semibold text-gray-300">Clients EBP</h2>
        <button
          @click="showNewClient = !showNewClient"
          class="px-3 py-1.5 bg-indigo-600 hover:bg-indigo-500 text-white text-xs font-medium rounded-lg transition-colors"
        >
          + Nouveau client
        </button>
      </div>

      <!-- New client form -->
      <div v-if="showNewClient" class="bg-gray-900 border border-gray-800 rounded-xl p-4 mb-3 space-y-3">
        <h3 class="text-xs font-semibold text-gray-400 uppercase tracking-wider">Nouveau client</h3>
        <div class="grid grid-cols-2 gap-3">
          <div>
            <label class="text-xs text-gray-500 block mb-1">DBName *</label>
            <input v-model="newClient.db_name" placeholder="T2M" class="input" />
          </div>
          <div>
            <label class="text-xs text-gray-500 block mb-1">Nom d'affichage</label>
            <input v-model="newClient.display_name" placeholder="Nom commercial" class="input" />
          </div>
          <div>
            <label class="text-xs text-gray-500 block mb-1">Fichier EBP</label>
            <input v-model="newClient.ebp_file" placeholder="T2M.ebp" class="input" />
          </div>
          <div>
            <label class="text-xs text-gray-500 block mb-1">Email expéditeur</label>
            <input v-model="newClient.sender_email" placeholder="contact@client.fr" type="email" class="input" />
          </div>
          <div>
            <label class="text-xs text-gray-500 block mb-1">Seuil silence (heures)</label>
            <input v-model.number="newClient.expected_log_hours" type="number" min="1" max="168" class="input" />
          </div>
        </div>
        <div class="flex gap-2">
          <button @click="createClient" :disabled="!newClient.db_name"
            class="px-3 py-1.5 bg-green-600 hover:bg-green-500 disabled:opacity-50 text-white text-xs rounded-lg transition-colors">
            Créer
          </button>
          <button @click="showNewClient = false"
            class="px-3 py-1.5 bg-gray-700 hover:bg-gray-600 text-gray-300 text-xs rounded-lg transition-colors">
            Annuler
          </button>
        </div>
        <p v-if="clientError" class="text-xs text-red-400">{{ clientError }}</p>
      </div>

      <div class="bg-gray-900 border border-gray-800 rounded-xl overflow-hidden">
        <table class="w-full text-xs">
          <thead class="border-b border-gray-800">
            <tr>
              <th class="px-4 py-2.5 text-left font-medium text-gray-500">DBName</th>
              <th class="px-4 py-2.5 text-left font-medium text-gray-500">Nom affiché</th>
              <th class="px-4 py-2.5 text-left font-medium text-gray-500">Fichier EBP</th>
              <th class="px-4 py-2.5 text-left font-medium text-gray-500">Email</th>
              <th class="px-4 py-2.5 text-right font-medium text-gray-500">Action</th>
            </tr>
          </thead>
          <tbody>
            <tr v-if="!clients.length">
              <td colspan="5" class="px-4 py-6 text-center text-gray-500">Aucun client</td>
            </tr>
            <tr v-for="c in clients" :key="c.id" class="border-b border-gray-800/50">
              <td class="px-4 py-2.5 font-mono text-gray-300">{{ c.db_name }}</td>
              <td class="px-4 py-2.5 text-gray-400">{{ c.display_name || '—' }}</td>
              <td class="px-4 py-2.5 text-gray-500">{{ c.ebp_file || '—' }}</td>
              <td class="px-4 py-2.5 text-gray-500">{{ c.sender_email || '—' }}</td>
              <td class="px-4 py-2.5 text-right">
                <button
                  @click="deleteClient(c)"
                  class="text-xs text-red-500 hover:text-red-400 transition-colors"
                >
                  Supprimer
                </button>
              </td>
            </tr>
          </tbody>
        </table>
      </div>
    </section>

    <!-- API Keys -->
    <section>
      <div class="flex items-center justify-between mb-3">
        <h2 class="text-sm font-semibold text-gray-300">Clés API</h2>
        <button
          @click="showNewKey = !showNewKey"
          class="px-3 py-1.5 bg-indigo-600 hover:bg-indigo-500 text-white text-xs font-medium rounded-lg transition-colors"
        >
          + Nouvelle clé
        </button>
      </div>

      <!-- New key form -->
      <div v-if="showNewKey" class="bg-gray-900 border border-gray-800 rounded-xl p-4 mb-3 space-y-3">
        <h3 class="text-xs font-semibold text-gray-400 uppercase tracking-wider">Nouvelle clé API</h3>
        <div class="grid grid-cols-2 gap-3">
          <div>
            <label class="text-xs text-gray-500 block mb-1">Client *</label>
            <select v-model="newKey.client_id" class="input">
              <option value="">Choisir un client…</option>
              <option v-for="c in clients" :key="c.id" :value="c.id">{{ c.display_name || c.db_name }}</option>
            </select>
          </div>
          <div>
            <label class="text-xs text-gray-500 block mb-1">Nom de la clé *</label>
            <input v-model="newKey.name" placeholder="Clé prod serveur 1" class="input" />
          </div>
        </div>
        <div class="flex gap-2">
          <button @click="createApiKey" :disabled="!newKey.client_id || !newKey.name"
            class="px-3 py-1.5 bg-green-600 hover:bg-green-500 disabled:opacity-50 text-white text-xs rounded-lg transition-colors">
            Générer
          </button>
          <button @click="showNewKey = false"
            class="px-3 py-1.5 bg-gray-700 hover:bg-gray-600 text-gray-300 text-xs rounded-lg transition-colors">
            Annuler
          </button>
        </div>
        <p v-if="keyError" class="text-xs text-red-400">{{ keyError }}</p>
      </div>

      <!-- Created key display -->
      <div v-if="createdKey" class="bg-green-900/20 border border-green-800/40 rounded-xl p-4 mb-3">
        <p class="text-xs text-green-400 font-semibold mb-2">✓ Clé créée — copiez-la maintenant, elle ne sera plus affichée :</p>
        <div class="flex items-center gap-2">
          <code class="flex-1 text-xs font-mono bg-gray-900 border border-gray-700 rounded px-3 py-2 text-green-300 select-all break-all">
            {{ createdKey }}
          </code>
          <button @click="copyKey" class="px-2.5 py-2 bg-gray-700 hover:bg-gray-600 rounded text-xs text-gray-300 transition-colors shrink-0">
            {{ copied ? '✓' : '📋' }}
          </button>
        </div>
        <button @click="createdKey = null" class="mt-2 text-xs text-gray-500 hover:text-gray-400">Fermer</button>
      </div>

      <div class="bg-gray-900 border border-gray-800 rounded-xl overflow-hidden">
        <table class="w-full text-xs">
          <thead class="border-b border-gray-800">
            <tr>
              <th class="px-4 py-2.5 text-left font-medium text-gray-500">Nom</th>
              <th class="px-4 py-2.5 text-left font-medium text-gray-500">Client</th>
              <th class="px-4 py-2.5 text-left font-medium text-gray-500">Créée le</th>
              <th class="px-4 py-2.5 text-left font-medium text-gray-500">Dernière utilisation</th>
              <th class="px-4 py-2.5 text-right font-medium text-gray-500">Action</th>
            </tr>
          </thead>
          <tbody>
            <tr v-if="!apiKeys.length">
              <td colspan="5" class="px-4 py-6 text-center text-gray-500">Aucune clé API</td>
            </tr>
            <tr v-for="key in apiKeys" :key="key.id" class="border-b border-gray-800/50">
              <td class="px-4 py-2.5 text-gray-300">{{ key.name }}</td>
              <td class="px-4 py-2.5 text-gray-500">{{ clientMap[key.client_id] || key.client_id }}</td>
              <td class="px-4 py-2.5 text-gray-500">{{ formatDate(key.created_at) }}</td>
              <td class="px-4 py-2.5 text-gray-500">{{ key.last_used_at ? formatDate(key.last_used_at) : '—' }}</td>
              <td class="px-4 py-2.5 text-right">
                <button
                  @click="deleteApiKey(key.id)"
                  class="text-xs text-red-500 hover:text-red-400 transition-colors"
                >
                  Supprimer
                </button>
              </td>
            </tr>
          </tbody>
        </table>
      </div>
    </section>

    <!-- Users -->
    <section>
      <div class="flex items-center justify-between mb-3">
        <h2 class="text-sm font-semibold text-gray-300">Utilisateurs</h2>
        <button
          @click="showNewUser = !showNewUser"
          class="px-3 py-1.5 bg-indigo-600 hover:bg-indigo-500 text-white text-xs font-medium rounded-lg transition-colors"
        >
          + Nouvel utilisateur
        </button>
      </div>

      <div v-if="showNewUser" class="bg-gray-900 border border-gray-800 rounded-xl p-4 mb-3 space-y-3">
        <h3 class="text-xs font-semibold text-gray-400 uppercase tracking-wider">Nouvel utilisateur Limpid IT</h3>
        <div class="grid grid-cols-3 gap-3">
          <div>
            <label class="text-xs text-gray-500 block mb-1">Email *</label>
            <input v-model="newUser.email" type="email" placeholder="prenom.nom@limpidit.com" class="input" />
          </div>
          <div>
            <label class="text-xs text-gray-500 block mb-1">Nom *</label>
            <input v-model="newUser.name" placeholder="Prénom Nom" class="input" />
          </div>
          <div>
            <label class="text-xs text-gray-500 block mb-1">Mot de passe *</label>
            <input v-model="newUser.password" type="password" class="input" />
          </div>
        </div>
        <div class="flex gap-2">
          <button @click="createUser" :disabled="!newUser.email || !newUser.name || !newUser.password"
            class="px-3 py-1.5 bg-green-600 hover:bg-green-500 disabled:opacity-50 text-white text-xs rounded-lg transition-colors">
            Créer
          </button>
          <button @click="showNewUser = false"
            class="px-3 py-1.5 bg-gray-700 hover:bg-gray-600 text-gray-300 text-xs rounded-lg transition-colors">
            Annuler
          </button>
        </div>
        <p v-if="userError" class="text-xs text-red-400">{{ userError }}</p>
      </div>

      <div class="bg-gray-900 border border-gray-800 rounded-xl overflow-hidden">
        <table class="w-full text-xs">
          <thead class="border-b border-gray-800">
            <tr>
              <th class="px-4 py-2.5 text-left font-medium text-gray-500">Nom</th>
              <th class="px-4 py-2.5 text-left font-medium text-gray-500">Email</th>
              <th class="px-4 py-2.5 text-left font-medium text-gray-500">Créé le</th>
            </tr>
          </thead>
          <tbody>
            <tr v-if="!users.length">
              <td colspan="3" class="px-4 py-6 text-center text-gray-500">Aucun utilisateur</td>
            </tr>
            <tr v-for="u in users" :key="u.id" class="border-b border-gray-800/50">
              <td class="px-4 py-2.5 text-gray-300">{{ u.name }}</td>
              <td class="px-4 py-2.5 text-gray-500">{{ u.email }}</td>
              <td class="px-4 py-2.5 text-gray-500">{{ formatDate(u.created_at) }}</td>
            </tr>
          </tbody>
        </table>
      </div>
    </section>
  </div>
</template>

<script setup>
import { ref, reactive, computed, onMounted } from 'vue'
import api from '@/api/client'

const clients = ref([])
const apiKeys = ref([])
const users = ref([])

const showNewClient = ref(false)
const showNewKey = ref(false)
const showNewUser = ref(false)
const createdKey = ref(null)
const copied = ref(false)

const newClient = reactive({ db_name: '', display_name: '', ebp_file: '', sender_email: '', expected_log_hours: 24 })
const newKey = reactive({ client_id: '', name: '' })
const newUser = reactive({ email: '', name: '', password: '' })

const clientError = ref('')
const keyError = ref('')
const userError = ref('')

const clientMap = computed(() => {
  const m = {}
  for (const c of clients.value) m[c.id] = c.display_name || c.db_name
  return m
})

async function load() {
  const [c, k, u] = await Promise.all([
    api.get('/admin/clients').catch(() => ({ data: [] })),
    api.get('/admin/api-keys').catch(() => ({ data: [] })),
    api.get('/admin/users').catch(e => { console.error('users', e); return { data: [] } }),
  ])
  clients.value = c.data
  apiKeys.value = k.data
  users.value = u.data
}

async function createClient() {
  clientError.value = ''
  try {
    await api.post('/admin/clients', newClient)
    Object.assign(newClient, { db_name: '', display_name: '', ebp_file: '', sender_email: '', expected_log_hours: 24 })
    showNewClient.value = false
    await load()
  } catch (e) {
    clientError.value = e.response?.data?.detail || 'Erreur'
  }
}

async function createApiKey() {
  keyError.value = ''
  try {
    const { data } = await api.post('/admin/api-keys', newKey)
    createdKey.value = data.key
    Object.assign(newKey, { client_id: '', name: '' })
    showNewKey.value = false
    await load()
  } catch (e) {
    keyError.value = e.response?.data?.detail || 'Erreur'
  }
}

async function deleteClient(c) {
  if (!confirm(`Supprimer le client "${c.display_name || c.db_name}" ?\n\nCela supprimera TOUTES ses sessions et logs. Cette action est irréversible.`)) return
  try {
    await api.delete(`/admin/clients/${c.id}`)
    await load()
  } catch (e) {
    alert(e.response?.data?.detail || 'Erreur lors de la suppression')
  }
}

async function deleteApiKey(id) {
  if (!confirm('Supprimer cette clé API ?')) return
  await api.delete(`/admin/api-keys/${id}`)
  await load()
}

async function createUser() {
  userError.value = ''
  try {
    await api.post('/admin/users', newUser)
    Object.assign(newUser, { email: '', name: '', password: '' })
    showNewUser.value = false
    await load()
  } catch (e) {
    userError.value = e.response?.data?.detail || 'Erreur'
  }
}

function copyKey() {
  navigator.clipboard.writeText(createdKey.value)
  copied.value = true
  setTimeout(() => (copied.value = false), 2000)
}

function formatDate(iso) {
  if (!iso) return '—'
  return new Date(iso).toLocaleString('fr-FR', { day: '2-digit', month: '2-digit', year: '2-digit', hour: '2-digit', minute: '2-digit' })
}

onMounted(load)
</script>

<style scoped>
@reference "tailwindcss";
.input {
  @apply w-full px-3 py-2 bg-gray-800 border border-gray-700 rounded-lg text-xs text-gray-200 focus:outline-none focus:border-indigo-500;
}
</style>
