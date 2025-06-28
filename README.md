# Documentation de déploiement – Projet Front + Back

## 1. Architecture du projet

Le projet est un monorepo avec deux sous-dossiers:

```
repo/
├── back/        
├── front/     
├── .github/workflows/deploy.yml
```

## 2. Objectif

Déploiement automatique sur un VPS via Docker, avec:

* Traefik pour le reverse proxy et HTTPS
* Watchtower pour les mises à jour automatiques
* GitHub Actions pour CI/CD et publication sur GHCR

---

## 3. Structure des dossiers sur le VPS

```
/
├── traefik/
│   ├── docker-compose.yml
│   └── letsencrypt/acme.json
│
├── watchtower/
│   └── docker-compose.yml
│
├── mon-projet/
│   └── docker-compose.prod.yml
```

---

## 4. Contenu des fichiers Docker importants

Contenu de `mon-projet/docker-compose.prod.yml` :



* `back` tiré de GHCR, exposé sur `api.thomaspiet.store`
* `front` tiré de GHCR, exposé sur `frontapi.thomaspiet.store`

Chaque service:

* utilise `restart: unless-stopped`
* est relié au réseau `traefik`
* a les labels Traefik
* est surveillé par Watchtower


---

## 5. Traefik

Contenu du `docker-compose.yml` dans `/traefik/`:

* écoute sur ports 80/443
* génère les certificats Let's Encrypt
* expose le dashboard sur `traefik.thomaspiet.store`

---

## 6. Watchtower

Contenu du `docker-compose.yml` dans `/watchtower/`:

```yaml
command:
  - --interval=86400
  - --cleanup
  - --label-enable
  - --log-level=info
```

Surveille les images `ghcr.io/...` toutes les 24h et supprime les anciennes versions après avoir mis à jour.

---

## 7. GitHub Actions

Fichier `.github/workflows/deploy.yml`:

* Job 1: teste le backend
* Job 2: si succès, build + push `back-latest`
* Job 3: ensuite, build + push `front-latest`

### Secrets à ajouter dans GitHub :

| Nom             | Valeur                                             |
| --------------- | -------------------------------------------------- |
| `GHCR_USERNAME` | `thomaspiet`                                       |
| `GHCR_TOKEN`    | Token GitHub (classic) avec scope `write:packages` |      

---

## 8. GHCR (GitHub Container Registry)

Les images sont publiées sous :

```
$ docker pull ghcr.io/thomaspiet/devops-tp:back-latest
$ docker pull ghcr.io/thomaspiet/devops-tp:front-latest
```

> ⚠️ Rendre les packages publics via l'interface GitHub si le VPS n’est pas authentifié. (Je ne le recommande pas) 

---

## 9. Déploiement

### Première fois:

```bash
mkdir -p /mon-projet
cd /mon-projet
nano docker-compose.prod.yml
docker compose -f docker-compose.prod.yml up -d
```

### Authentifier le VPS à GHCR (si images privées) :

```bash
echo <TON_GHCR_TOKEN> | docker login ghcr.io -u thomaspiet --password-stdin
```

