
M.D.L = Market Data Layer

---
## Responsabilité

- La MDL ingère un `MarketDataEvent` à  chaque `step` du backtest.
- La MDL maintient un état de marché interne, muté exclusivement via `ingest(event)`, exposé en lecture seule.
- La MDL expose le dernier `MarketDataEvent` ingéré.
- La MDL expose une fenêtre des `n` derniers `MarketDataEvent` consécutifs.

---
## Hypothèses et préconditions
  
- `BacktestEngine` injecte un `MarketDataEvent` à chaque `step`.
- Chaque `MarketDataEvent` possède un  `event_id` de type `str` non vide.
- Chaque `MarketDataEvent` possède `timestamp` de type `datetime`.
- Chaque `MarketDataEvent` possède un champ `bar` de type `OHLCV`.
- L'ensemble des `MarketDataEvent` concernent  un unique `instrument` et une unique `bar_size`.

---
## Mutation et état

- La MDL possède l’état du marché qui se matérialise par une `list[MarketDataEvent]`.
- Le seul composant habilité à appeler `ingest()` est l'orchestrateur `BacktestEngine`.
- Aucun accès en écriture à `_history` n’est exposé à l’extérieur.

---
## Invariants

### I1 - Monotonie des timestamps

Cas égal:
```python
with pytest.raises(BacktestInvariantError):  
    store.ingest(event_same_timestamp)
```

Cas décroissant:
```python
with pytest.raises(BacktestInvariantError):  
    store.ingest(event_timestamp_anterior_to_previous)
```

Garantie un flux temporel strictement ordonné. Empêche tout "reorder" implicite, tout backtest temporellement incohérent.
### I2 - Conservation de l'ordre d'ingestion

```python
store.ingest(event_1)  
store.ingest(event_2)  
assert store.window(2)[0] == event_1  
assert store.window(2)[1] == event_2
```

Garantie que l'historique conserve l'ordre exacte d'ingestion des évènements. La vue retournée par `window(n)` reste cohérente avec la séquence effectivement ingérée.
## I3 - Immutabilité du snapshot retourné par `window(n)`

```python
events = store.window(2)  
assert isinstance(events, tuple)  
with pytest.raises(TypeError):  
    events[0] = make_event(  
        event_id="event_03",  
        timestamp=datetime(2020, 1, 3),  
    )
```

Garantit que la structure retournée par `window(n)` ne peut pas être mutée par l’appelant. Empêche toute altération accidentelle de la vue renvoyée.

---
## Contrat API

- `latest()` retourne le dernier `MarketDataEvent` ingéré.
- `window(n)` retourne les `n` derniers `MarketDataEvent` ingérés, dans l’ordre d’ingestion.
- `latest()` == `window(1)[0]`.

---
## Erreurs

- `MarketStore.ingest(event)` lève `BacktestInvariantError` si `event.timestamp <= last.timestamp`.
- `latest()` lève `ContextNotInitializedError` si aucun événement n’a été ingéré.
- `window(n)` lève `ValueError` si `n <= 0`.
- `window(n)` lève `ValueError` si `n > len(history)`.

---
## Hors périmètre

- Elle ne charge pas les données depuis une source externe
- Elle ne trie ni réordonne les évènement de marché et suppose que leur ordre est garantie par l'amont
- Elle ne produit ni décision ni exécution.