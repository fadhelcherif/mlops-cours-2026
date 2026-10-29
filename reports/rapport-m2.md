# Rapport comparatif — Kubeflow Pipelines vs Airflow (Lab 2)

**Binôme :** Fadhel Cherif · **Date :** 29/10/2026 · **Machine :** AMD Ryzen 5 5600H (6 cœurs / 12 threads), Windows + WSL2 Ubuntu 26.04, 12 Go de RAM alloués à WSL ·
**Dataset :** german-credit

## 1. Contexte

Le pipeline de credit scoring du Lab 1 (dataset german-credit) est orchestré deux fois, en six étapes : ingestion, validation des données, préparation, entraînement, évaluation et enregistrement du modèle dans le Model Registry MLflow. Les deux orchestrateurs exécutent la même image `localhost:5001/scoring-pipeline:1.0.0` dans le cluster kind `mlops-lab` : KFP via Argo Workflows (namespace `kubeflow`), Airflow, installé en Docker Compose hors du cluster, via le KubernetesPodOperator (namespace `airflow`). Le code métier (`src/`), le contrat `dataset.yaml`, le serveur MLflow et le stockage S3 (SeaweedFS) sont partagés : seule la couche d'orchestration change.

## 2. Résultats quantitatifs

| Mesure | KFP | Airflow |
|---|---|---|
| Installation de zéro (min) | env. 20 | env. 5 (`make airflow`) |
| Temps jusqu'au premier run réussi (min) | env. 45 | env. 15 (dont une erreur de dossier `dags/`) |
| Durée d'un run complet (min) | 4,1 | env. 3 |
| Relance avec les mêmes paramètres (min) | 2,2 (cache) | env. 3 (pas de cache natif) |
| Hot-reload : modifier un step et voir le résultat (min) | env. 5 (rebuild, recompilation, nouvelle version) | env. 1 (DAG relu en 30 s ; env. 4 si l'image change) |
| Lignes de code propres à l'orchestrateur | 280 (`components.py` 192 + `scoring_pipeline.py` 88) | 71 (`scoring_dag.py`) |
| Mémoire consommée par l'outil (Go, `docker stats`) | env. 3,1 (3 nœuds kind, MLflow et SeaweedFS inclus) | env. 4 (7 conteneurs Compose) |

## 3. Analyse qualitative

### Developer experience

Airflow est plus concis : 71 lignes contre 280 pour KFP, et le DAG est un simple fichier Python lu automatiquement, sans compilation ni upload. Mais Airflow ne transporte aucun fichier : chaque task relit et réécrit son dossier de travail sur S3, et seules de petites valeurs passent par XCom (l'identifiant du run MLflow). KFP demande plus de cérémonie (composants typés, compilation en YAML, upload d'une nouvelle version), en échange d'artefacts typés, d'une lignée visible dans l'UI et d'un lien direct avec MLflow (`kfp_run_id`). Piège rencontré : un DAG placé dans `infra/airflow/dags` n'est pas lu, car l'override monte le dossier `dags/` à la racine du repo.

### Scaling

Les deux délèguent l'exécution à Kubernetes (une étape = un pod), avec retries, limites mémoire et timeouts. KFP est natif Kubernetes ; Airflow ajoute une infrastructure séparée (scheduler, worker Celery, Redis, Postgres) d'environ 4 Go. Sur notre poste, le cluster plus Airflow pendant l'exécution du DAG ont saturé les 9,6 Go alloués par défaut à WSL et fait planter Ubuntu ; il a fallu passer à 12 Go plus 8 Go de swap. Le coût fixe d'Airflow pèse lourd sur une petite machine.

### Debugging

KFP affiche le graphe, les logs de chaque pod et les artefacts intermédiaires directement dans l'UI, et le cache est visible. Airflow donne aussi les logs par task et permet de relancer une seule task ; de plus, `pipelines/stage.py` permet de rejouer une étape en local avec un dossier de travail local au lieu de S3. Dans les deux cas, l'entraînement dure environ 3 secondes : les temps mesurés sont dominés par le démarrage des pods.

## 4. Recommandation

KFP convient à une équipe déjà sur Kubernetes qui fait surtout de l'expérimentation ML : artefacts typés, lignée, cache et intégration MLflow justifient le code supplémentaire. Airflow est préférable si l'organisation l'utilise déjà pour ses pipelines de données et veut orchestrer le ML à côté de l'ETL, avec une planification calendaire et de nombreux connecteurs, à condition de faire passer les données par S3 (jamais par XCom) et de disposer de ressources suffisantes.

## 5. Limites de la comparaison

Les mesures viennent d'un seul poste sous WSL, avec peu de runs (deux pour KFP, un pour Airflow) ; les temps d'installation et de hot-reload sont estimés. La mémoire KFP inclut MLflow et SeaweedFS. Le dataset est petit (1 000 lignes), donc le lab ne dit rien du comportement à grande échelle, en multi-utilisateurs ou avec du parallélisme. Enfin, Airflow tourne en Docker Compose et non sur Kubernetes, ce qui n'est pas représentatif d'une production.
