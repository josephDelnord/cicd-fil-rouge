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

<!-- Lab J1 : remplacez par les noms du binôme -->
- Joseph DELNORD
- Yani SADKI

## Gouvernance du dépôt

<!-- Lab J1 : listez les règles activées sur main, pourquoi chacune, et ajoutez la capture du push refusé -->

### Règles activées sur `main` et justifications :

- **Restrict deletions** : empêche la suppression accidentelle ou malveillante de la branche de production `main`.
- **Block force pushes** : préserve l'intégrité et la traçabilité de l'historique Git en interdisant toute réécriture (`git push --force`).
- **Require a pull request before merging (1 approval)** : garantit que tout changement est revu par au moins un pair (le binôme), réduisant ainsi les bugs et facilitant le partage de connaissances.
- **Require review from Code Owners** : exige obligatoirement l'approbation d'un des propriétaires déclarés dans `.github/CODEOWNERS` dès qu'un fichier critique (ici `.github/workflows/`) est modifié.

![Règles de protection de main et approbation requise](img/protect-main-rules.png)

![Règle Require review from Code Owners](img/codeowners-review-required.png)

### Capture du push direct refusé :

![Refus du push direct sur main](img/push-direct-main-refuse.png)

Cela montre que les règles de gouvernance sont bien appliquées et que tout changement doit passer par une Pull Request.

### Processus de Pull Request

1. je créé une Pull Request, je remplis les informations nécessaires et je soumets pour revue.

![Commit signé vérifié sur GitHub](img/commit-signature-verified.png)

![Historique des commits de la branche](img/commits-feature-branch.png)

![Liste des rulesets GitHub](img/ruleset-list.png)

![Formulaire de création de Pull Request](img/pull-request-create.png)

2. Mon coéquipier révise la Pull Request et laisse ses commentaires ou approbations.

![Approbation de la Pull Request par le binôme](img/pr-approved.png)

![Soumission de l'approbation de la Pull Request](img/pr-submit-approval.png)

### Création des CODEOWNERS

1. Je créé les CODEOWNERS pour définir les propriétaires des fichiers critiques.

![Fichier CODEOWNERS dans le dépôt](img/codeowners-file.png)

2. Je coche la case correspondant à l'option "Require review from Code Owners" pour activer l'approbation obligatoire des propriétaires de fichiers critiques.

![Règle Require review from Code Owners](img/codeowners-review-required.png)

3. Je créé la PR et mon coéquipier l'approuve en tant que propriétaire de fichiers critiques.

![Confirmation de fusion de la PR CODEOWNERS](img/codeowners-pr-merge.png)

![Pull Request CODEOWNERS approuvée](img/codeowners-pr-approved.png)

### Bonus : signature des commits

1. Pour renforcer la sécurité et la traçabilité, nous pouvons activer la signature des commits.
2. Chaque commit devra être signé avec la clé GPG configurée, assurant ainsi l'authenticité de l'auteur du commit.
3. Les commits non signés seront rejetés, garantissant que chaque modification est vérifiable et provient d'un contributeur authentique.

![Signature SSH vérifiée](img/ssh-signature-verified.png)

Cette capture montre l'activation de la signature des commits et la vérification de l'authenticité de chaque modification.

## Pipeline CI

Le workflow [ci.yml](.github/workflows/ci.yml) s'exécute sur les Pull Requests à destination de `main` et lors d'un push sur `main`. Il exécute deux jobs indépendants en parallèle :

- **lint** : lance `ruff check .` pour contrôler les règles de style et détecter notamment les erreurs statiques et les imports inutilisés.
- **test** : lance `pytest` pour exécuter les tests automatisés de l'API et détecter les régressions fonctionnelles.

Le ruleset de `main` exige la réussite des checks **lint** et **test** avant la fusion. Une PR dont l'un de ces jobs échoue ne peut donc pas être fusionnée.

![Jobs CI lint et test réussis](img/ci-jobs-success.png)

![Checks lint et test requis par le ruleset](img/ci-required-checks.png)

### Démonstration d'une PR bloquée par un test en échec

Sur la branche `fix/casse-test`, le test `test_health_returns_ok` a été volontairement modifié pour attendre une valeur de statut incorrecte.

![Diff du test volontairement rendu défaillant](img/failing-test-diff.png)

Le job **lint** doit réussir, tandis que le job **test** échoue. Cette démonstration vérifie que le ruleset empêche la fusion lorsque les tests ne passent pas.

![Fusion bloquée par l'échec du check test](img/pr-ci-merge-blocked.png)
