# Image « projet » du pipeline : le code du Lab 1 (src/), le contrat dataset.yaml,
# params.yaml et leurs dépendances figées. TOUS les composants KFP (et les tasks Airflow)
# tournent dans cette image : le code est réutilisé tel quel, jamais recopié.
# Contexte de build = RACINE du repo :
#   docker build -f pipelines/Dockerfile -t localhost:5001/scoring-pipeline:1.0.0 .
FROM python:3.12-slim

WORKDIR /app

COPY pipelines/requirements.txt /tmp/requirements.txt
RUN pip install --no-cache-dir --root-user-action=ignore -r /tmp/requirements.txt \
    && pip install --no-cache-dir --root-user-action=ignore --no-deps kfp==2.17.0

COPY dataset.yaml params.yaml ./
COPY src/ src/
COPY pipelines/__init__.py pipelines/stage.py pipelines/

ENV PYTHONPATH=/app \
    DATASET_CONFIG=/app/dataset.yaml \
    PYTHONUNBUFFERED=1

ARG VERSION=dev
LABEL org.opencontainers.image.title="scoring-pipeline" \
      org.opencontainers.image.version="${VERSION}"
