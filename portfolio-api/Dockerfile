# ─── Étape 1 : dépendances ────────────────────────────────────────────────────
FROM node:20-alpine AS deps
WORKDIR /app
COPY package*.json ./
RUN npm ci --only=production

# ─── Étape 2 : image finale ───────────────────────────────────────────────────
FROM node:20-alpine
WORKDIR /app

# Sécurité : ne pas tourner en root
RUN addgroup -S appgroup && adduser -S appuser -G appgroup

# Copier uniquement les dépendances de prod + le code
COPY --from=deps /app/node_modules ./node_modules
COPY . .

# Donner les droits à l'utilisateur non-root
RUN chown -R appuser:appgroup /app
USER appuser

EXPOSE 5000

# Healthcheck : Jenkins / Docker sait si le conteneur est vraiment prêt
HEALTHCHECK --interval=30s --timeout=5s --start-period=10s --retries=3 \
  CMD wget -qO- http://localhost:5000/health || exit 1

CMD ["node", "server.js"]