# Architecture Scalable avec Traefik

Cette configuration permet de scaler horizontalement les instances FrankenPHP (avec worker mode) tout en conservant un hub Mercure centralisé.

## Architecture

```
┌─────────────────────┐
│      Traefik        │  Load Balancer
│   (ports 80/443)    │  + TLS automatique
└──────────┬──────────┘
           │
           ├──────────────────────┬──────────────────────┐
           │                      │                      │
    ┌──────▼──────┐        ┌──────▼──────┐      ┌──────▼──────┐
    │ FrankenPHP  │        │ FrankenPHP  │      │ FrankenPHP  │
    │ Instance 1  │        │ Instance 2  │      │ Instance N  │
    │ (worker)    │        │ (worker)    │      │ (worker)    │
    └──────┬──────┘        └──────┬──────┘      └──────┬──────┘
           │                      │                      │
           └──────────────────────┴──────────────────────┘
                                  │
                          ┌───────▼────────┐
                          │  Mercure Hub   │  Hub centralisé
                          │   (unique)     │  pour tous les workers
                          └────────────────┘
```

## Composants

### 1. Traefik (Reverse Proxy & Load Balancer)
- Distribution automatique des requêtes HTTP
- Health checks sur les instances FrankenPHP
- TLS automatique avec Let's Encrypt (production)
- Dashboard de monitoring (développement uniquement)

### 2. FrankenPHP (Application Symfony)
- **Worker mode activé** : Application Symfony gardée en mémoire
- **Scalable horizontalement** : 2-5 instances selon la charge
- Chaque instance traite les requêtes HTTP de manière indépendante
- Publication vers le hub Mercure centralisé

### 3. Mercure (Hub de messaging temps réel)
- **Hub unique et partagé** entre toutes les instances
- Garantit la cohérence des messages temps réel
- Tous les clients reçoivent les messages, peu importe l'instance source

## Utilisation

### Développement

```bash
# Démarrer l'environnement de dev (1 instance par défaut)
docker compose up --wait

# Accéder à l'application
open http://app.localhost

# Accéder au dashboard Traefik
open http://localhost:8080

# Scaler à 3 instances pour tester
docker compose up --scale php=3 -d

# Vérifier les instances actives
docker compose ps
```

### Production

```bash
# Créer un fichier .env.prod avec vos secrets
cat > .env.prod << EOF
SERVER_NAME=votre-domaine.com
APP_SECRET=votre-secret-tres-long
CADDY_MERCURE_JWT_SECRET=votre-jwt-secret-tres-long
ACME_EMAIL=votre-email@example.com
CORS_ORIGINS=https://votre-domaine.com
EOF

# Démarrer en production avec 3 instances
docker compose -f compose.yaml -f compose.prod.yaml --env-file .env.prod up --scale php=3 -d

# Surveiller les logs
docker compose -f compose.yaml -f compose.prod.yaml logs -f
```

### Scaling dynamique

```bash
# Scaler à 5 instances
docker compose up --scale php=5 -d

# Réduire à 2 instances
docker compose up --scale php=2 -d

# Voir la distribution de charge dans Traefik
curl http://localhost:8080/api/http/services
```

## Configuration

### Variables d'environnement importantes

| Variable | Défaut | Description |
|----------|--------|-------------|
| `SERVER_NAME` | `app.localhost` | Nom de domaine de l'application |
| `HTTP_PORT` | `80` | Port HTTP exposé |
| `HTTPS_PORT` | `443` | Port HTTPS exposé |
| `APP_SECRET` | - | **Production uniquement** : Secret Symfony |
| `CADDY_MERCURE_JWT_SECRET` | `!ChangeThisMercureHubJWTSecretKey!` | Secret JWT pour Mercure |
| `ACME_EMAIL` | - | **Production uniquement** : Email pour Let's Encrypt |
| `CORS_ORIGINS` | `https://${SERVER_NAME}` | Origines CORS autorisées pour Mercure |

### Endpoints

- **Application Symfony** : `http://app.localhost/` (dev) ou `https://votre-domaine.com/` (prod)
- **Mercure Hub** : `http://app.localhost/.well-known/mercure` (dev)
- **Traefik Dashboard** : `http://localhost:8080` (dev uniquement)
- **Mercure UI** : `http://app.localhost/.well-known/mercure/ui` (dev uniquement)

## Avantages de cette architecture

### ✅ Performance
- **Worker mode FrankenPHP** : Pas de bootstrap PHP à chaque requête
- **Latence minimale** : Pas de proxy supplémentaire entre l'application et le load balancer
- **Mise en cache optimale** : Caddy gère la compression et les headers

### ✅ Scalabilité
- **Horizontal scaling** : De 1 à N instances selon la charge
- **Load balancing intelligent** : Traefik distribue selon les health checks
- **Mercure centralisé** : Tous les clients reçoivent tous les messages

### ✅ Fiabilité
- **Health checks** : Traefik retire automatiquement les instances défaillantes
- **Zero-downtime deployment** : Possibilité de rolling updates
- **Isolation des services** : Mercure ne partage pas les ressources avec PHP

### ✅ Simplicité opérationnelle
- **TLS automatique** : Let's Encrypt via Traefik (production)
- **Configuration déclarative** : Tout via labels Docker
- **Monitoring intégré** : Dashboard Traefik pour visualiser la charge

## Monitoring

### Health checks

Les health checks sont configurés automatiquement :
- **FrankenPHP** : Vérifié toutes les 10 secondes sur `/`
- **Traefik** : Retire automatiquement les instances non-responsives

### Logs

```bash
# Logs de toutes les instances PHP
docker compose logs -f php

# Logs de Traefik (access logs et routing)
docker compose logs -f traefik

# Logs de Mercure
docker compose logs -f mercure
```

### Métriques Traefik

```bash
# API Traefik (dev uniquement)
curl http://localhost:8080/api/overview

# Voir tous les services actifs
curl http://localhost:8080/api/http/services | jq

# Voir tous les routers
curl http://localhost:8080/api/http/routers | jq
```

## Déploiement en production

### Prérequis
1. Serveur avec Docker et Docker Compose
2. Nom de domaine pointant vers le serveur
3. Ports 80 et 443 ouverts

### Étapes

1. **Cloner le repository**
   ```bash
   git clone https://github.com/bertrand-leroux/dunglas-symfony-docker.git
   cd dunglas-symfony-docker
   ```

2. **Configurer les variables d'environnement**
   ```bash
   cp .env.example .env.prod
   # Éditer .env.prod avec vos valeurs
   ```

3. **Générer des secrets forts**
   ```bash
   # APP_SECRET (64 caractères)
   openssl rand -hex 32

   # CADDY_MERCURE_JWT_SECRET (64 caractères minimum)
   openssl rand -base64 64
   ```

4. **Démarrer avec le nombre d'instances souhaité**
   ```bash
   docker compose -f compose.yaml -f compose.prod.yaml --env-file .env.prod up --scale php=3 -d
   ```

5. **Vérifier que Let's Encrypt a généré les certificats**
   ```bash
   docker compose logs traefik | grep -i "certificate"
   ```

6. **Tester l'application**
   ```bash
   curl -I https://votre-domaine.com
   ```

## Troubleshooting

### Les instances ne reçoivent pas de trafic
```bash
# Vérifier que les labels Traefik sont bien appliqués
docker inspect <container_id> | grep -i traefik

# Vérifier que Traefik voit les services
docker compose logs traefik | grep -i "provider"
```

### Mercure ne fonctionne pas
```bash
# Vérifier que le routage Mercure est actif
curl http://localhost:8080/api/http/routers | jq '.[] | select(.name | contains("mercure"))'

# Tester la connexion au hub
curl -X POST http://app.localhost/.well-known/mercure
```

### Let's Encrypt échoue (production)
```bash
# Vérifier les logs ACME
docker compose logs traefik | grep -i acme

# Vérifier que le port 80 est accessible depuis Internet
curl -I http://votre-domaine.com
```

## Migration depuis l'ancienne architecture

Si vous migrez depuis la version précédente avec Mercure intégré :

1. **Les données Mercure** : L'ancien volume `caddy_data` contient `mercure.db`. Ce fichier n'est plus utilisé car Mercure a maintenant son propre volume `mercure_data`.

2. **Variables d'environnement** : Les URLs Mercure ont changé :
   - `MERCURE_URL` : `http://mercure/.well-known/mercure` (nom de service Docker)
   - `MERCURE_PUBLIC_URL` : `http://app.localhost/.well-known/mercure` (URL publique)

3. **Ports** : Les services PHP n'exposent plus de ports directement. Tout passe par Traefik.

## Ressources

- [Documentation Traefik](https://doc.traefik.io/traefik/)
- [Documentation Mercure](https://mercure.rocks/)
- [Documentation FrankenPHP](https://frankenphp.dev/)
- [Symfony + Mercure](https://symfony.com/doc/current/mercure.html)
