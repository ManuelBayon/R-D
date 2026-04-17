
M.D.L = Market Data Layer
# Préconditions / Inputs
  
- Le `BacktestEngine` injecte un `MarketDataEvent` à chaque step.  
- Les événements concernent un unique instrument et une unique granularité (bar size).  
- La séquence d’événements est strictement croissante selon leur timestamp.
# Responsabilité / Rôle

- La MDL ingère un `MarketDataEvent` à  chaque `step` du backtest.
- La MDL transforme un flux ordonné d'évènements de marché ordonné en un état de marché (interne) exposé en lecture seule.
- La MDL expose le dernier évènement de marché et une fenêtre de taille n évènements de marché.

# Hors périmètre

- Elle ne charge pas les données depuis une source externe
- Elle ne trie ni réordonne les évènement de marché et suppose que leur ordre est garantie par l'amont
- Elle ne calcule pas de feature
- Ne produit pas de décision de trading
- Ne produit pas d'instruction d'exécution
- Ne mute aucun autre état que le sien.
# Postconditions / Output

- Après chaque step, l'historique contient tous les `MarketDataEvent` ingéré depuis le début, dans l'ordre d'ingestion.
- Le dernier élément de l’historique correspond toujours au `MarketDataEvent` du step courant.
- Toute fenêtre de taille n retournée par la MDL correspond aux n derniers événements consécutifs de l’historique.












