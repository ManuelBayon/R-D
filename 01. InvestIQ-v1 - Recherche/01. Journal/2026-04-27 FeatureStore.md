## Responsabilité

- La responsabilité du `FeatureStore` est d'appeler séquentiellement les `FeatureCalculator` initialisé à chaque `step`.
- `FeatureStore` maintient un état cohérent de l'état via une mutation atomique de son état interne.
- `FeatureStore` renvoi à chaque `step` un `FeatureComputedEvent` afin de connaître et pouvoir rejouer et tracer toutes les valeurs renvoyées par les `FeatureCalculator`.

---
## Hypothesis et pre-conditions

- `FeaturStore` a été initialisé avec des `FeatureCalculator` ayant chacun un identifiant unique.
- `BacktestEngine` injecte à chaque `step` un `StepContext` cohérent (voir la section Invariants et Garanties)
- `BacktestEngine` injecte à chaque `step` une `market_view`.
- `market_view` est ordonnée
- `market_view[-1]` représente l’événement courant
- instrument / bar_size / OHLCV cohérents

---
## Mutation and State

- `FeatureStore` possède l'état des features `_history: dict[str, list[FeaturePoint]]`.
- La mutation de l'historique se fait de manière atomique, si un `FeatureCalculator` échoue aucune valeur n'est publiée au `step` courant. Cela permet d'éviter de se retrouver dans un état incohérent avec une partie des valeurs publiées.
- Un `FeatureCalculator` effectue une transformation pure
- `FeatureStore` créé un `FeaturePoint` avec `calculator_id`, `timestamp` et `value`.
- La source de vérité du `timestamp` est `StepContext.timestamp`.

---
## Invariants et Garanties

### Initialisation

##### I1 : Unicité des `calculator_id`

### Runtime 

##### R1 : `market_view` non vide

Si `market_view` vide `FeatureStore.update()` lève `ContextNotInitializedError`.

##### R2 : Cohérence des `timestamp`

- `context.timestamp == market_view[-1].timestamp`
- `context.timestamp > self._last_processed_timestamp`

##### R3 : Cohérence des `event_id`

- `context.event_id == market_view[-1].event_id:`
- `context.event_id != self._last_processed_event_id`

##### R4 : Cohérence de la séquence

` context.step_sequence == expected_step_sequence:`

---
## API contract

##### `def update( context: StepContext, market_view: tuple[MarketDataEvent, ...]) -> FeatureComputedEvent:`

- Renvoie à chaque `step` un `FeatureComputedEvent` :
```python
@dataclass(frozen=True)  
class FeatureComputationResult:  
    calculator_id: str  
    timestamp: datetime  
    value: float | None

@dataclass(frozen=True)  
class FeatureComputedEvent:  
    run_id: str  
    event_id: str  
    timestamp: datetime  
    computations: dict[str, FeatureComputationResult]
```

##### `FeatureStore.contains(key: str) -> bool`

- `True` si la `key` existe dans l'historique.
- `Faux` si la `key` n'existe PAS dans l'historique.

##### `FeatureStore.has_data(key: str, quantity: int) -> bool`

- `ValueError` si quantité demandé <= 0.
- `KeyError` si la `key` n'existe pas dans l'historique.
- `True` si la quantité demandé pour `key` est disponible.
- `False` si `quantity > historique` pour la feature demandé.

##### `FeatureStore.latest(key: str) -> FeaturePoint`

- lève `KeyError` si la `key` n'existe pas dans l'historique.
- lève `ContextNotInitialized` si aucune valeur n'a été publiée.
- renvoie la dernière valeur (`float`)  de la feature concernée.

##### `FeatureStore.window(key: str, quantity: int) -> tuple[FeaturePoint,...]`

-  lève `ValueError` si `quantity <= 0`.
- lève `KeyError` si la `key` n'existe pas dans l'historique.
- lève `ContextNotInitialized` si la quantité demandée est supérieure à l'historique disponible.
- renvoie les `n` dernières valeurs consécutives de la feature concernée. 

---
## Failure mode

### Type : `Fail-Fast`

### Sources d'erreurs
#### Violation d'un invariant

- Initialisation invalide
- `update(step_context, market_view) invalide
#### Erreur Runtime `FeatureCalculator`

### Politique de gestion des erreurs

`FeatureStore.update()` applique une politique `fail-fast`.

`FeatureStore` ne masque pas les erreurs localement.

Si un invariant est violé ou qu'un calculateur lève une erreur, le `step` courant est considéré comme invalide. 
### Garanties

`FeatureStore.update()` garantit :  
  
- **succès complet** : état muté de façon atomique ;  
- **échec** : aucune mutation de `_history` ni des métadonnées `last_processed_*` ;  

Les erreurs sont propagées au composant d'orchestration `BacktestEngine`.

---
## Out of scope

- `FeatureStore` ne gère pas les données de marchés ni ne maintient d'état du marché.
- `FeatureStore` ne prends pas de décisions stratégiques
- `FeatureStore` ne gère pas le risque
- `FeatureStore` ne s'occupe pas de la partie exécution
- `FeatureStore` n'a pas d'effet de bord, il expose une lecture de son état par son API.
