## 3.1 Architecture

- Event-driven propre
- Journal canonique (append-only)
- Replay déterministe complet
- Zéro magie implicite

👉 Chaque état dérive d’événements traçables.

---
## 3.2 Rigueur système

- invariants clairement définis
- gestion explicite des erreurs
- idempotence
- cohérence temporelle (timestamps, ordering)

---
## 3.3 Observabilité

- logs structurés
- métriques
- capacité à rejouer un bug exact

👉 “Explainability du système”

---
## 3.4 Performance (quand nécessaire)

- sait profiler avant d’optimiser
- comprend CPU / mémoire / IO
- introduit C++ seulement là où ça compte

---
## 3.5 Compréhension marché

- ordre → exécution → fill → portfolio
- slippage réaliste
- latence / microstructure (au bon niveau)

---
## 3.6 Math appliquée (juste ce qu’il faut)

- sait détecter un backtest biaisé
- comprend variance / bruit
- évite overfitting