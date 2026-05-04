#### Phase 1 : Verrouiller le moteur minimal

Objectif : **un backtest rejouable de bout en bout**.

Pas toutes les features. Pas le risk layer. Pas C++.

Définition of done :

- même input journal → même output journal
- MarketDataLayer + FeatureLayer + DecisionLayer + Portfolio minimal intégrés
- tests unitaires par couche
- tests d’intégration par paire
- test replay global
- README avec garanties + limites

---
#### Phase 2 :  Ajouter Risk Layer

Objectif : montrer que les décisions ne vont pas directement au marché.

Invariant clé :

> aucune intention de trading ne devient ordre exécutable sans passage explicite par le risque.

---
#### Phase 3 : Pipeline unifié

Objectif :

> même logique métier en backtest et live, seules les sources/sinks changent.

C’est un gros signal infra.

---
#### Phase 4 : Profiling avant C++

Très important : **pas de C++ avant mesure**.

Règle :

> aucun composant ne migre en C++ sans profil prouvant qu’il est dominant dans le coût total.

Sinon c’est de l’over-engineering.