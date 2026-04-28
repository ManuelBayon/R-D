## Responsabilité

### Initialisation :

- `FeatureStore` reçoit une séquence `Calculator` lors de son initialisation.
- `FeatureStore` créé un historique vie pour chacun d'entre eux.
### Runtime

- `FeatureStore.update()` appelle `calculate(market_view)` pour chaque calculateur initialisé.
- `FeatureStore.update()` mute l'état du store `_history: dict[str, list[float]]`.
- `FeatureStore` expose une API de lecture des features via `latest(key)` et `window(key, quantity)`.
- `FeatureStore` expose une méthode utilitaire `has_data(key, quantity)` pour savoir si l'historique contient la quantité demandé.
- `FeatureStore.update()` retourne un `FeatureComputedEvent` avec la valeur renvoyé par chaque calculateur au `step` courant.

```python
@dataclass(frozen=True)
FeatureComputedEvent:
	run_id: str
	event_id: str
	computations: dict[str, tuple[FeaturePoint, ...]]
```

---
## Hypothesis et pre-conditions

- A chaque `step`, `BacktestEngine` injecte une vue de marché `market_view` via la méthode `update(market_view) -> None`.
- `market_view` a un flux temporel strictement croissant selon les timestamps.
- L'ensemble des `MarketDataEvent` de la `market_view` sont cohérents (instrument, bar_size, OHLC).

---
## Mutation and State

- `FeatureStore` possède l'état des features `_history: dict[str, list[float]]`.
- Chaque `FeatureCalculator` initialisé possède son entrée dans l'historique.
- `FeatureCalculator` effectue une transformation pure, `FeatureStore` créé un `FeaturePoint` avec `calculator_id`, `timestamp` et `value`.
- La source de vérité du `timestamp` est `StepContext.timestamp`.

---
## Invariants 

### initialisation

#### I1 : Unicité des `calculator_id`

#### I2 : Initialisation d'une liste vide pour chaque `FeatureCalculator`

### Runtime 

#### `.update(market_view, step_context)`
##### R1 : `market_view` non vide

`market_view` est vide lève `ContextNotInitializedError`.
##### R2 : Cohérence des `timestamp`

`context.timestamp == market_view[-1].timestamp`
##### R3 : Cohérence des `event_id`

`context.event_id == market_view[-1].event_id`

---
## Edge cases

	...

---
## API contract

```
def update(
        context: StepContext,  
        market_view: tuple[MarketDataEvent, ...]  
) -> FeatureComputedEvent:
```



##### `FeatureStore.contains(key: str) -> bool`

- `True` si la `key` existe dans l'historique.
- `Faux` si la `key` n'existe PAS dans l'historique.

##### `FeatureStore.has_data(key: str, quantity: int) -> bool`

- `KeyError` si la `key` n'existe pas dans l'historique.
- `True` si la quantité demandé pour `key` est disponible.
- `False` si `quantity` > historique pour la  feature demandé.

##### `FeatureStore.latest(key: str) -> float`

- lève `KeyError` si la `key` n'existe pas dans l'historique.
- lève `ContextNotInitialized` si aucune valeur n'a été publiée.
- renvoie la dernière valeur (`float`)  de la feature concernée.

##### `FeatureStore.window(key: str, quantity: int) -> list[float]` 

-  lève `ValueError` si `quantity <= 0`.
- lève `KeyError` si la `key` n'existe pas dans l'historique.
- lève `ContextNotInitialized` si la quantité demandée est supérieure à l'historique disponible.
- renvoie les `n` dernières valeurs consécutives de la feature concernée. 

---
## Failure modes

	...

---
## Out of scope

	...