# Docker — Mini-cours

> Brief associé : M0-B1
> Durée de lecture + pratique : ~45 min
> Pré-requis : Docker Desktop (ou Docker Engine sous Linux) installé et fonctionnel.
> Test : `docker --version` doit afficher une version ≥ 24.

## Pourquoi cette techno ?

> *« Ça marche chez moi. »* — Tout dev, un jour.

Docker répond à ce problème : **emballer ton application avec son environnement
exact** (Python, libs, OS minimum, fichiers de conf) dans un objet portable
appelé **image**. On lance ensuite cette image sur n'importe quelle machine
qui a Docker installé : ton poste, le serveur de prod, le cloud.

Pour un service ML c'est particulièrement utile parce que :

- les versions de NumPy / scikit-learn / Pydantic doivent **matcher exactement**
  celle utilisée pour entraîner le modèle ;
- les data scientists qui livrent le `.joblib` ne contrôlent pas l'OS du
  serveur de prod ;
- l'équipe ops doit pouvoir relancer l'image en 30 secondes en cas d'incident.

**Alternatives à connaître :**

| Outil | Position |
|---|---|
| **Podman** | Compatible Docker, daemon-less, populaire en sécurité. Drop-in replacement. |
| **VM (VirtualBox, VMware)** | Trop lourd pour un service. Préfère Docker pour l'applicatif. |
| **Nix / packages OS** | Reproductible mais courbe d'apprentissage abrupte. Niche. |

Sur ce parcours, **Docker est imposé** (M0 à M5). C'est aussi ce que demande
explicitement le sujet certif janvier 2026.

## Concepts clés

- **Image** : un blob figé qui contient tout pour faire tourner ton appli.
  Construite à partir d'un `Dockerfile`. Identifiée par un nom + tag :
  `fastia-maintenance:0.1.0`.
- **Conteneur** : une instance vivante d'une image. Tu peux en lancer plusieurs
  à partir de la même image. Quand tu fais `docker run`, tu crées un conteneur.
- **`Dockerfile`** : recette de construction de l'image. Une instruction par
  ligne. Chaque instruction crée une **couche** (layer) cachable.
- **Caching** : Docker rejoue uniquement les couches qui ont changé. Astuce :
  copier `requirements.txt` AVANT le code pour que `pip install` reste en cache
  tant que les deps ne bougent pas.
- **`EXPOSE`** : déclaration informative du port utilisé par l'appli (ne le
  publie pas, c'est juste de la doc).
- **`-p 8000:8000`** : à `docker run`, mappe le port 8000 du conteneur au port
  8000 de l'hôte. Sans ça, le service est inaccessible depuis l'extérieur.
- **`.dockerignore`** : équivalent de `.gitignore` côté build. **Indispensable**
  pour ne pas balancer `.venv/`, `__pycache__/`, `data/raw/` (etc.) dans l'image.

## Exemple minimal qui tourne

`Dockerfile` minimaliste pour FastAPI :

```dockerfile
# Versions testées : Docker 24+, image python:3.11-slim
FROM python:3.11-slim

WORKDIR /app

# 1. Dépendances d'abord (cache des couches)
COPY requirements.txt .
RUN pip install --no-cache-dir -r requirements.txt

# 2. Code applicatif
COPY app/ ./app/
COPY model/ ./model/

EXPOSE 8000

CMD ["uvicorn", "app.main:app", "--host", "0.0.0.0", "--port", "8000"]
```

`.dockerignore` à mettre à côté :

```
.venv/
__pycache__/
*.pyc
.pytest_cache/
.git/
tests/
*.md
```

Build + run + test :

```bash
docker build -t demo:0.1 .
docker run --rm -p 8000:8000 demo:0.1
# dans un autre terminal :
curl http://localhost:8000/health
```

Tu dois voir `{"status":"ok","model_loaded":true}`. Bravo, ton service tourne
en conteneur.

## Exercice guidé

Le squelette **M0-B1** contient un `Dockerfile` vide commenté. Complète-le.

**Étapes :**

1. Ouvre `Dockerfile` et lis les indices.
2. Implémente :
   - `FROM python:3.11-slim`
   - `WORKDIR /app`
   - `COPY requirements.txt` → `RUN pip install`
   - `COPY app/ ./app/` puis `COPY model/ ./model/`
   - `EXPOSE 8000`
   - `CMD uvicorn ...`
3. Crée un `.dockerignore` minimal (cf. exemple ci-dessus).
4. Build :
   ```bash
   docker build -t fastia-maintenance:dev .
   ```
5. Vérifie que l'image existe :
   ```bash
   docker images fastia-maintenance:dev
   ```
   Elle pèse environ 800 Mo : c'est normal pour cette stack (scikit-learn, scipy, pandas), il n'y a rien à optimiser ici.
6. Run :
   ```bash
   docker run --rm -p 8000:8000 fastia-maintenance:dev
   ```
7. Test :
   ```bash
   curl http://localhost:8000/health
   curl -X POST http://localhost:8000/predict -H "Content-Type: application/json" -d '{"type_machine":"pompe","age_machine_jours":1200,"derniere_maintenance_jours":90,"temperature_moyenne":85.0,"vibration_moyenne":4.2,"pression_moyenne":6.5,"nb_incidents_3_mois":1}'
   ```

✅ **Solution attendue** : `/health` répond 200 depuis le conteneur.

⭐ **Bonus** : ajouter un `HEALTHCHECK` dans le `Dockerfile` qui interroge
`/health` toutes les 30 secondes. Cherche dans la doc Docker la syntaxe
de `HEALTHCHECK` puis essaie d'écrire ta version.

<details><summary>▶ Voir une solution possible</summary>

```dockerfile
HEALTHCHECK --interval=30s --timeout=3s --retries=3 \
  CMD python -c "import urllib.request; urllib.request.urlopen('http://localhost:8000/health')" || exit 1
```

</details>

## Pièges fréquents

| Piège | Conséquence |
|---|---|
| Lancer uvicorn sans `--host 0.0.0.0` dans le `CMD` | L'API n'écoute que sur 127.0.0.1 **du conteneur**, inaccessible depuis l'hôte. |
| Confondre **image** (figée) et **conteneur** (instance vivante) | `docker run` crée un nouveau conteneur ; `docker start` redémarre un conteneur existant. |
| Oublier `.dockerignore` | Image gonflée par `.venv/`, `__pycache__/`, tests, données brutes (parfois plusieurs Go en trop). |
| Copier le code AVANT `requirements.txt` | Chaque modif de code invalide le cache `pip install` → builds très lents. |
| `docker run` sans `-p 8000:8000` | API démarre mais inaccessible depuis l'hôte (port pas mappé). |
| Modifier le code et oublier de **rebuild l'image** | Le conteneur tourne avec l'ancienne version — *« mais j'ai changé le fichier pourtant »*. |

**Symptôme → cause probable**

| Symptôme | Cause probable |
|---|---|
| Conteneur démarré, `curl localhost:8000` répond `Connection refused` | Soit `--host 0.0.0.0` oublié dans `CMD`, soit `-p 8000:8000` oublié dans `docker run` |
| Image fait 2 Go au lieu de ~800 Mo | `.dockerignore` manquant (un `.venv` local recopié dans l'image) |
| Rebuild prend 5 min à chaque modif minuscule | `COPY app/` placé avant `COPY requirements.txt` → cache pip invalide |
| Modif de code non prise en compte au run | Image pas rebuilt — relancer `docker build` |
| `EXPOSE 8000` mais port toujours inaccessible | `EXPOSE` est informatif ; le mapping réel se fait avec `-p` au `docker run` |

## Pour aller plus loin

- **Doc officielle** : <https://docs.docker.com/>
- **Déployer FastAPI dans Docker (doc officielle)** :
  <https://fastapi.tiangolo.com/deployment/docker/>
- **Dockerfile reference (toutes les directives)** :
  <https://docs.docker.com/reference/dockerfile/>
- **Debug** : `docker logs <container>`, `docker exec -it <container> bash`.

⭐ Pour M0-B2, tu utiliseras `docker-compose.yml` pour orchestrer 2 services
(modèle + UI). C'est une couche au-dessus, pas un autre outil.

## Vérification (checklist apprenant)

- [ ] `docker --version` affiche bien une version ≥ 24.
- [ ] Mon `Dockerfile` build sans erreur (`docker build .`).
- [ ] Mon `.dockerignore` exclut `.venv`, `__pycache__`, `tests`, `.git`.
- [ ] Mon conteneur démarre, `/health` répond depuis l'extérieur.
- [ ] Je sais expliquer la différence entre **image** et **conteneur** en 1 minute.
- [ ] Je sais pourquoi on copie `requirements.txt` AVANT le code (cache des couches).