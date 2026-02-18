# 🚀 Guide de Déploiement — Evolution API sur VPS

Ce guide décrit comment déployer Evolution API sur le VPS `164.68.103.8` pour un agent conversationnel WhatsApp connecté à n8n.

## Prérequis

- Docker & Docker Compose installés
- Traefik configuré avec Let's Encrypt
- PostgreSQL accessible sur le réseau `postgres_network`
- n8n accessible sur `https://n8n.videalys.net`
- DNS `evolution.videalys.net` pointant vers `164.68.103.8`

## 1. Cloner le dépôt

```bash
ssh jean_yves@164.68.103.8
cd /opt  # ou le répertoire de votre choix
git clone https://github.com/Keziah2/evolution-api.git
cd evolution-api
```

## 2. Créer le fichier `.env`

```bash
cp env.production .env
```

Puis éditez le fichier `.env` pour ajuster les valeurs suivantes :

```bash
nano .env
```

**Valeurs à modifier obligatoirement :**

| Variable | Description |
|---|---|
| `DATABASE_CONNECTION_URI` | URI complète de votre PostgreSQL (vérifiez le mot de passe) |
| `AUTHENTICATION_API_KEY` | Clé API sécurisée de votre choix |

## 3. Lancer les services

```bash
docker compose up -d
```

Vérifiez que tout fonctionne :

```bash
docker compose ps
docker compose logs -f api
```

## 4. Accéder à Evolution API

- **API** : `https://evolution.videalys.net`
- **Documentation** : `https://evolution.videalys.net/docs`

## 5. Créer une instance WhatsApp

```bash
curl -X POST https://evolution.videalys.net/instance/create \
  -H "apikey: VOTRE_CLE_API" \
  -H "Content-Type: application/json" \
  -d '{
    "instanceName": "whatsapp-agent",
    "integration": "WHATSAPP-BAILEYS",
    "qrcode": true
  }'
```

Scannez le QR code retourné avec votre WhatsApp.

## 6. Connecter à n8n

Dans n8n (`https://n8n.videalys.net`), créez un workflow :

1. **Trigger** : Nœud `Webhook` — notez l'URL du webhook (ex: `https://n8n.videalys.net/webhook/xxxxx`)
2. **Configurer le webhook dans Evolution** :

```bash
curl -X POST https://evolution.videalys.net/webhook/set/whatsapp-agent \
  -H "apikey: VOTRE_CLE_API" \
  -H "Content-Type: application/json" \
  -d '{
    "webhook": {
      "enabled": true,
      "url": "https://n8n.videalys.net/webhook/VOTRE_WEBHOOK_ID",
      "webhookByEvents": false,
      "events": ["MESSAGES_UPSERT"]
    }
  }'
```

3. Dans n8n, traitez les messages reçus et répondez via l'API Evolution :

```bash
POST https://evolution.videalys.net/message/sendText/whatsapp-agent
Headers: apikey: VOTRE_CLE_API
Body: {
  "number": "NUMERO_DESTINATAIRE",
  "text": "Votre réponse ici"
}
```

## Commandes utiles

```bash
# Voir les logs
docker compose logs -f api

# Redémarrer
docker compose restart api

# Mettre à jour
docker compose pull
docker compose up -d

# Arrêter
docker compose down
```
