# TaskFlow — dépôt fil rouge CI/CD

TaskFlow est une petite API de gestion de tâches écrite en Python avec FastAPI.
C'est le projet fil rouge du module CI/CD (Mastère DevOps M1, Sup de Vinci) :
pendant trois jours, vous allez construire autour d'elle un pipeline complet
qui teste, construit, sécurise et livre l'application.

## Lancer l'API en local

Prérequis : Python 3.10 ou plus récent.

```bash
python3 -m venv .venv
source .venv/bin/activate          # Windows : .venv\Scripts\activate
pip install -r requirements-dev.txt
uvicorn app.main:app --reload
```

L'API répond sur http://localhost:8000 et sa documentation interactive est sur
http://localhost:8000/docs.

## Vérifier le code

```bash
pytest           # tests automatiques
ruff check .     # lint
ruff format .    # mise en forme
```

## Lancer avec Docker

```bash
docker build -t taskflow .
docker run --rm -p 8000:8000 taskflow
```

## Endpoints

| Méthode | Chemin | Rôle |
| --- | --- | --- |
| GET | `/health` | État de l'API et version |
| GET | `/tasks` | Liste des tâches |
| GET | `/tasks/search?q=...` | Recherche dans les titres |
| POST | `/tasks` | Crée une tâche (`{"title": "..."}`) |
| GET | `/tasks/{id}` | Détail d'une tâche |
| PATCH | `/tasks/{id}/done` | Marque une tâche comme faite |
| DELETE | `/tasks/{id}` | Supprime une tâche (en-tête `X-API-Token` requis) |

## Configuration

| Variable | Rôle | Défaut |
| --- | --- | --- |
| `APP_VERSION` | Version affichée par `/health` | `0.1.0` |
| `DB_PATH` | Fichier SQLite | `taskflow.db` |
| `API_TOKEN` | Jeton exigé pour supprimer une tâche | vide (suppression désactivée) |
| `NOTIFY_WEBHOOK_URL` | Webhook appelé à chaque création de tâche | vide (désactivé) |

## Équipe

- Karim Haddadi ([@KarimHaddadi20](https://github.com/KarimHaddadi20))
- Amine ([@Amine92-cpu](https://github.com/Amine92-cpu))

## Gouvernance du dépôt

Ruleset GitHub **Protection main**, ciblant `main`, enforcement `active`, sans bypass.

| Règle | Pourquoi |
| --- | --- |
| Pull request obligatoire | Aucun commit n'atteint `main` sans revue : on ne peut pas y glisser un workflow en push direct. |
| 1 approbation | Le binôme doit relire avant le merge ; l'auteur ne peut pas s'auto-approuver. |
| Require review from Code Owners | Une modification de `.github/workflows/` doit être approuvée par un propriétaire déclaré dans `CODEOWNERS`. |
| Force push interdit | Empêche de réécrire l'historique (et de masquer un changement de pipeline). |
| Suppression de branche interdite | Empêche d'effacer `main`. |

`CODEOWNERS` limite la revue obligatoire aux workflows :

```
/.github/workflows/ @KarimHaddadi20 @Amine92-cpu
```

### Capture du push direct refusé

Tentative : `git push origin main` après le ruleset.

```
(à capturer après activation du ruleset)
```

## Pipeline CI

Le workflow `.github/workflows/ci.yml` s'exécute à chaque pull request (et sur `main`).
Il lance **deux jobs en parallèle** : aucun n'attend l'autre (`needs` est absent).

| Job | Commande | Ce qu'il vérifie |
| --- | --- | --- |
| `lint` | `ruff check .` | Style, imports inutiles et erreurs Python (règles E, F, W, I). |
| `test` | `pytest` | Le comportement de l'API (santé, CRUD, recherche, suppression protégée). |

Après le merge de `feat/ci`, le ruleset exige que ces deux checks soient **verts** pour fusionner dans `main`. Une PR qui casse un test reste bloquée.

### Capture de la PR bloquée

Branche `fix/casse-test` : un test volontairement cassé. Le job `test` est rouge, le merge est refusé.

```
(à coller la capture GitHub « required status check » / merge bloqué)
```

