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

---
Deja qu'appelle t-on point de canonisation ?

Reponse : Le **point de canonisation** est la frontière où une donnée externe/raw devient un **event interne standard**, validé, typé, rejouable.

---
Ce que je voulais dire c'est que la méthode step doit etre strictement commune aux différents mode dexecution backtest-live-replay.

Mais la couche execution (EMS) est différent dans chaque mode. 

Je me demande si il ne faut pas faire des méthodes pour chaque partie :
- on_data_event qui gère market, feature, decision 
- on_decision qui gère risk-plannification 
- on_request qui gere la partie execution (avec broker si live) 
- on_fill qui gere les message recu du broker, màj portfolio etc.

---
Changement de paradigme avant je pensais en `step` mais ce modèle semble résister quand il s'agit d'unifier proprement les différents modes (live-backtest-replay).

Il faudrait passer d'un modèle statique et séquentiel à un modèle piloté par évènement. 

MAIS ATTENTION : rester mono-thread, séquentiel et déterministe !

---
Ancienne architecture : 

```
step(bar) -> market -> feature -> decision -> risk -> planning -> execution -> accounting
```

Ca casse quand je veux brancher le système en réel parce que : 
- un `FillEvent` ne passe PAS par market/feature/décision/risk/planning
- un `OrderAck` ne vient pas d'une bar
- le live recoit des évènements non synchrones
- le broker peut répondre entre 2 market events
- le replay doit rejouer exactement ces interleavings (ordre d’entrelacement des événements provenant de sources différentes).

Donc actuellement on a un truc du genre : 

```
input -> toutes les couches -> output
```

---
Nouvelle architecture : 

- Le moteur n'exécute pas un pipeline.
- Le moteur applique une fonction de transition sur une séquence ordonnée d'évènements.

```
(event, state) → (new_state, emitted_events, effects)
```

Chaque event :

- touche UNE partie du state
- produit éventuellement d’autres events

---

