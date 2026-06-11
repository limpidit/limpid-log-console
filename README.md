# Limpid IT — Log Console (Frontend)

Interface web de consultation des logs EBP. Réservée aux utilisateurs Limpid IT.

## Démarrage local

```bash
npm install
# Créer .env.local avec VITE_API_URL=http://localhost:8000/api
npm run dev
```

## Build production

```bash
VITE_API_URL=https://limpid-log-api.onrender.com/api npm run build
```

## Pages

- **Dashboard** `/dashboard` — stats du jour, dernières erreurs
- **Logs** `/logs` — vue log style terminal, filtres client/niveau/API/recherche/dates
- **Clients** `/clients` — liste des clients avec stats d'erreurs
- **Paramètres** `/settings` — gestion clients, clés API, utilisateurs
