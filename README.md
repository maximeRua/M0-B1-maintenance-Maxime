# M0-B1 — Squelette repo (maintenance prédictive · API REST)

> **Repo template GitHub.** Clique sur **« Use this template »** en haut à
> droite de cette page → **Create a new repository** → nomme-le
> `M0-B1-maintenance-<prenom>` sur **ton** compte GitHub personnel
> (`<prenom>` : ton prénom en minuscules, sans accent). C'est ce nouveau repo que tu cloneras pour travailler.

Brief **M0-B1 « Intégrer un service IA de maintenance prédictive via API REST »**
— **individuel**, 5 h.
L'énoncé complet est publié sur **Simplonline** : c'est lui qui fait foi pour
les livrables et les critères. Ce README est ton tableau de bord au quotidien.

---

## 🚀 Démarrage (4 étapes)

```bash
# 0. Clone ton repo perso fraîchement créé
git clone git@github.com:<ton-user>/M0-B1-maintenance-<prenom>.git
# (sans clé SSH : git clone https://github.com/<ton-user>/M0-B1-maintenance-<prenom>.git)
cd M0-B1-maintenance-<prenom>

# 1. Environnement virtuel + dépendances
python -m venv .venv && source .venv/bin/activate     # Linux/macOS
# .venv\Scripts\activate                              # Windows
pip install -r requirements.txt

# 2. Lancer l'API (rechargement auto) — dans un premier terminal
uvicorn app.main:app --reload

# 3. Lancer les tests — dans un second terminal (réactive d'abord l'environnement)
source .venv/bin/activate
pytest
```

À l'étape 2, ouvre <http://localhost:8000/docs> (Swagger auto). L'endpoint
`/health` doit déjà répondre `{"status": "ok", "model_loaded": true}`.
Si ces commandes marchent, ton poste est prêt.

---

## 📁 Structure du repo

```
M0-B1-maintenance-<prenom>/
├── app/
│   ├── __init__.py
│   ├── main.py             ← FastAPI : /health (✅) + /predict (à compléter)
│   └── schemas.py          ← Pydantic : MachineInput, PredictionResponse
├── data/
│   ├── generate_dataset.py ← régénération du dataset (déjà exécuté)
│   └── maintenance_data.csv ← dataset synthétique 6 500 lignes
├── model/
│   ├── train_baseline.py   ← entraînement (déjà exécuté)
│   └── model.joblib        ← modèle pré-entraîné ~5,7 Mo (chargé au démarrage)
├── tests/
│   ├── __init__.py
│   └── test_health.py      ← test pytest fonctionnel au clone (✅)
├── ressources/             ← 📚 mini-cours d'appui (lecture juste-à-temps)
│   ├── 01_FastAPI_essentiel.md
│   ├── 02_Docker_essentiel.md
│   ├── 03_Loguru_essentiel.md
│   ├── 04_Pytest_API_essentiel.md
│   ├── liens_officiels.md
│   └── README.md           ← ordre de mobilisation + objectifs
├── Dockerfile              ← squelette commenté à compléter
├── requirements.txt        ← dépendances figées
├── .gitignore
└── README.md (ce fichier — à compléter avec ta doc de service)
```

---

## ✏️ Ce que tu dois compléter

| Fichier | Tâche |
|---|---|
| `app/main.py` | Implémenter l'endpoint **POST `/predict`** (TODO marqué dans le code) |
| `app/main.py` | Ajouter du **logging Loguru** sur chaque requête (entrée, sortie, durée) |
| `tests/` | Ajouter **≥ 3 tests** pour `/predict` (cas valide, erreur 422, test paramétré sur les 5 types de machine) |
| `Dockerfile` | Compléter le squelette commenté (image qui build) |
| `README.md` | Documenter le lancement du service (un dev externe en 5 min) |

---

## 🧭 Démarche attendue (ordre suggéré)

| # | Étape | Mini-cours | Durée |
|---|---|---|---|
| 1 | Prise en main du squelette (clone + install + run) | — | 30 min |
| 2 | Analyse de l'existant (`main.py`, `schemas.py`, `train_baseline.py`) | — | 30 min |
| 3 | Implémenter `/predict` | [`01_FastAPI_essentiel.md`](./ressources/01_FastAPI_essentiel.md) | 1 h 30 |
| 4 | Logging Loguru | [`03_Loguru_essentiel.md`](./ressources/03_Loguru_essentiel.md) | 30 min |
| 5 | Tests pytest `/predict` | [`04_Pytest_API_essentiel.md`](./ressources/04_Pytest_API_essentiel.md) | 45 min |
| 6 | Conteneurisation Docker | [`02_Docker_essentiel.md`](./ressources/02_Docker_essentiel.md) | 45 min |
| 7 | README (colle-y les 3 preuves : `pytest -v`, `curl /health`, `docker images`) + push final + lien du repo déposé sur Simplonline | — | 30 min |

Cf. [`./ressources/README.md`](./ressources/README.md) pour le détail. Les
mini-cours se lisent **au moment où tu en as besoin** (~15 min chacun).

---

## 🎯 Ce qui compte vraiment

1. **Une API qui tourne**, pas une API parfaite. Un `/predict` simple qui
   fonctionne vaut mieux qu'une archi sur-engineered qui plante.
2. **Des tests qui passent** : ≥ 3 tests pour `/predict` en PASSED.
3. **Un README** qui permet à un collègue de lancer le service en 5 minutes.
4. **Un Dockerfile qui build** et une image qui répond sur `/health`.

→ Compétence visée : **C6 — imiter** (intégrer un modèle livré dans un service API).

---

## 🔁 Régénérer dataset / modèle (optionnel)

Le dataset et le modèle sont déjà fournis — **pas besoin** de les régénérer.
Si tu veux le faire (déterministe, `random_state=42`) :

```bash
python data/generate_dataset.py     # régénère le CSV
python model/train_baseline.py      # réentraîne le modèle (~30 s)
```

---

## ✅ Conventions de code

- Python 3.11+
- Type hints sur toutes les signatures publiques
- Pas de `print` — utiliser **Loguru**
- `pathlib.Path` pour les chemins (pas de `os.path`)
- `random_state=42` partout où il y a de l'aléa

---

## 🆘 Bloqué·e ?

| Symptôme | Cause probable | Solution |
|---|---|---|
| `ModuleNotFoundError` au lancement | env virtuel pas activé / deps absentes | `source .venv/bin/activate` puis `pip install -r requirements.txt` |
| `Modèle introuvable` au démarrage uvicorn | `model.joblib` absent ou mal placé | `python model/train_baseline.py` |
| `pytest` échoue au clone | env virtuel actif ? deps installées ? | idem ligne 1 |
| Port 8000 déjà utilisé | un autre service tourne dessus | `uvicorn app.main:app --reload --port 8001` |

1. Relis l'**exercice guidé** du mini-cours concerné (chacun a sa solution).
2. Ouvre **Swagger** (`/docs`) pour débugger une route sans curl.
3. Lis les **logs Loguru** dans la console pour repérer les exceptions.
4. **Demande de l'aide**, à l'oral ou par écrit dans le fil Discord du brief :
   les deux se valent. N'attends pas 30 min sur le même problème. Par écrit,
   colle la commande lancée et le message d'erreur complet.
