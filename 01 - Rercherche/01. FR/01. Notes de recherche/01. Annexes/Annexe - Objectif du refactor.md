
Ce refactor vise à : 

- clarifier les couches du système 
- formaliser les messages et évènements produits par ces couches
- introduire un modèle d'ordres plus complet (market, limit, stop, bracket)
- définir le cycle de vie des ordres (aggregate-centric permettant la protection des invariants).
- Préparer pour les version future le pipeline unifié backtest et live.
- Améliorer la couverture de test unitaires et d'intégration des composants critiques.
- Converger vers une architecture event-centric et aggregagte centric (pour les ordres).

