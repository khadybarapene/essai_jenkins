# ─── Étape 1 : build React ────────────────────────────────────────────────────
FROM node:20-alpine AS builder
WORKDIR /app
COPY package*.json ./
RUN npm ci
COPY . .
RUN npm run build

# ─── Étape 2 : servir avec Nginx ─────────────────────────────────────────────
FROM nginx:1.25-alpine
WORKDIR /usr/share/nginx/html

# Supprimer la config nginx par défaut
RUN rm -rf ./*

# Copier le build depuis l'étape précédente
COPY --from=builder /app/build .

# Config nginx personnalisée (React Router + gzip + sécurité)
COPY nginx.conf /etc/nginx/conf.d/default.conf

EXPOSE 80

HEALTHCHECK --interval=30s --timeout=5s --retries=3 \
  CMD wget -qO- http://localhost:80 || exit 1

CMD ["nginx", "-g", "daemon off;"]
