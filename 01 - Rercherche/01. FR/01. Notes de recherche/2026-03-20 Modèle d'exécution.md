# 1. Types d’ordres supportés

- Market
- Limit
- Stop

Le modèle suppose une exécution sans contrainte de liquidité et peut introduire un biais optimiste sur les ordres limites.

---
# 2. Modèle de prix et règle de déclenchement

## 2.1 Market : NBO (next bar open)

- **condition** : ordre actif au step courant
- **fill price** : `open` de la bougie suivante
## 2.2 Limit : Touch

- **condition :**
    - buy : low ≤ limit
    - sell : high ≥ limit
- **fill** : limit price
## 2.3 Stop : Gap-aware

- **trigger :**
    - buy : high ≥ stop
    - sell : low ≤ stop
- **fill :**
    - si open franchit le stop → open
    - sinon → stop price

## 2.4 Cas ambigus

- SL & TP même bougie
- plusieurs ordres simultanés

---
# 3. Timing 

Les ordres sont créés au step `t` et exécuté au step `t+1`.