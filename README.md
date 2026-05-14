# Portfolio — React + Node + MongoDB + Docker + Jenkins

## Stack
- **Frontend** : React (Nginx en prod)
- **Backend**  : Node.js / Express sur le port 5000
- **Base de données** : MongoDB 7
- **CI/CD** : Jenkins
- **Conteneurisation** : Docker + Docker Compose

---

## Lancer le projet en local

### Développement
```bash
# Backend
cd portfolio-api
cp .env.example .env
npm install
npm start

# Frontend
cd React
npm install
npm start
```

### Production (Docker)
```bash
cp .env.example .env      # remplir MONGO_USER et MONGO_PASS
docker compose up -d
```

Accès :
- Frontend → http://localhost:3000
- Backend  → http://localhost:5000
- API      → http://localhost:5000/health

---

## Structure du projet

```
Dockerisation/
├── portfolio-api/
│   └── Dockerfile
├── React/
│   ├── Dockerfile
│   ├── nginx.conf
│   ├── .env.development
│   └── .env.production
├── docker-compose.yml
├── Jenkinsfile
├── .env.example
├── .gitignore
└── README.md
```

---

## CI/CD Jenkins

Le pipeline se déclenche automatiquement sur push.

| Stage | Description |
|---|---|
| Lint | Vérification du code |
| Test | Tests backend avec MongoDB temporaire |
| Build | Build React |
| Docker build | Construction des images |
| Push | Push sur Docker Hub (branche main uniquement) |
| Deploy | Redémarrage des conteneurs |
| Health check | Vérification que les services répondent |

### Credentials à configurer dans Jenkins
- `dockerhub-credentials` — username + token Docker Hub
- SMTP configuré pour les notifications email

---

## Variables d'environnement

Copier `.env.example` en `.env` et remplir :

```
MONGO_USER=admin
MONGO_PASS=changeme
```

> ⚠️ Ne jamais commiter `.env` — il est dans `.gitignore`