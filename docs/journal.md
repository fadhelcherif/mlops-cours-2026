# Journal de bord du binôme

<!--
Mode d'emploi (à garder en tête du fichier) :
- Une entrée par séance (et par session de travail importante hors séance), la plus récente en bas.
- 3 à 5 lignes : qui a piloté, ce qui a été fait, ce qui a bloqué ou ce qui a été appris,
  la prochaine étape. Pas de roman : c'est un outil de pilotage, pas un rapport.
- Titre d'entrée au format exact « ## S<n> — AAAA-MM-JJ — <sujet> » (date réelle) :
  le pré-contrôle de rendu (make check-m10) compte les entrées datées.
- Liez les PR, issues et ADR (#12, docs/adr/0003-...). Les problèmes non résolus restent
  visibles : c'est précisément ce qui aide l'enseignant à vous débloquer.
- L'entrée d'exemple ci-dessous (date AAAA-MM-JJ) n'est pas comptée ; supprimez-la
  quand vous aurez écrit la vôtre.
-->

**Binôme :** @fadhelcherif (Fadhel) et @MahdiGH10 (Mahdi Gharbi)
**Repo :** https://github.com/fadhelcherif/mlops-cours-2026
**Dataset :** german-credit — [Statlog German Credit Data (UCI)](https://archive.ics.uci.edu/dataset/144/statlog+german+credit+data)
**Extensions :** à définir en S8

## S1 — 2026-10-19 — Lab 0 et 1 rejoués

- **Pilote / copilote :** @MahdiGH10 / @fadhelcherif ; Mahdi a rejoué les labs sur son propre cluster.
- **Fait :** cluster kind à 3 nœuds, SeaweedFS et MLflow opérationnels ; Lab 0 validé (19/19) ; pipeline DVC poussé, 22 runs MLflow, `@champion` promu, Feast matérialisé ; Lab 1 validé le 2026-10-20 (18/18).
- **Bloqué / appris :** Docker Desktop a manqué de mémoire virtuelle avec Oracle actif ; le grid search a remplacé la sortie locale DVC, restaurée depuis le cache ; le téléchargement MLflow utilise le proxy via une option locale dans `.env`.
- **Prochaine étape :** conserver les deux captures des checks, faire relire cette PR par Fadhel et signaler le problème de `make pf` dans le repo du cours.
