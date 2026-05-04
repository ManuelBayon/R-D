Réflexion sur le point de canonisation de mon moteur et l'unification des différents mode d'utilisation backtest / live / replay.

---
# A ce jour

**Backtest :** 
- Au lancement de mon programme, je passe les données historiques récupérées via l'API de mon broker à un composant qui se charge de les transformer en `Iterator`.
- `BacktestEngine.run(bt_input)` itère sur les données sous forme `Iterator` en appelant pour chaque évènement la méthode `BacktestEngine.step(event)`.
- `BacktestEngine.step(event)` est le point de canonisation de mon moteur. Elle prend un évènement de marché en entrée et orchestre séquentiellement l'appel aux différentes couches de mon pipeline (`Market`, `Feature`, `Decision`, `Risk`, `Planning`, `Execution`, `Accounting`) et retourne un `StepRecord`.

**Live :** Rien
**Replay :** Rien

---

A priori ce qui est indépendant du mode d'exécution c'est bien la méthode `step` même si celle ci peut etre orchestrer par un BacktestEngine ou LiveEngine ou ReplayEngine avec chacun leur source de données.

Sources de données : 
backtest : historique data feed
live : live data feed
replay : Canonical journal


