## Responsibilities

- The `FeatureStore` is responsible for sequentially calling the `FeatureCalculators` initialized at each `step`.
- The `FeatureStore` maintains a consistent state through atomic mutations of its internal state.
- `FeatureStore` returns a `FeatureComputedEvent` at each `step` in order to track and be able to replay all values returned by the `FeatureCalculators`.

---
## Assumptions and Preconditions

- `FeatureStore` has been initialized with `FeatureCalculators`, each having a unique identifier.
- `BacktestEngine` injects a consistent `StepContext` at each `step` (see the Invariants and Guarantees section)
- `BacktestEngine` injects a `market_view` at each `step`.
- `market_view` is ordered
- `market_view[-1]` represents the current event
- Consistent instrument / bar_size / OHLCV

---
## State Ownership and Mutation

- `FeatureStore` holds the state of the features `_history: dict[str, list[FeaturePoint]]`.
- Mutations to the history are atomic; if a `FeatureCalculator` fails, no values are published to the current `step`. This prevents ending up in an inconsistent state with some published values.
- A `FeatureCalculator` performs a pure transformation
- `FeatureStore` creates a `FeaturePoint` with `calculator_id`, `timestamp`, and `value`.
- The source of truth for the `timestamp` is `StepContext.timestamp`.

---
## Invariants and Guarantees

### Initialization

##### I1: Uniqueness of `calculator_id`

### Runtime

##### R1: `market_view` is not empty

If `market_view` is empty, `FeatureStore.update()` raises a `ContextNotInitializedError`.

##### R2: Consistency of `timestamp`

- `context.timestamp == market_view[-1].timestamp`
- `context.timestamp > self._last_processed_timestamp`

##### R3: Consistency of `event_id`

- `context.event_id == market_view[-1].event_id:`
- `context.event_id != self._last_processed_event_id`

##### R4: Sequence consistency

```
expected_step_sequence = last_processed_step_sequence + 1
context.step_sequence == expected_step_sequence:
```

### Global 

##### G1 : After each sucessful update : 

- `history` is strictly ordered by `timestamps`
- `last_processed_timestamp` and `last_processed_event_id` and `last_processed_step_sequence` reflects the last ingested event.

##### G2 : `None` indicates insufficient data or undefined computation

---
## API contract

##### `def update( context: StepContext, market_view: tuple[MarketDataEvent, ...]) -> FeatureComputedEvent:`

- Returns a `FeatureComputedEvent` for each `step`:
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

- `True` if the `key` exists in the history.
- `False` if the `key` does NOT exist in the history.

##### `FeatureStore.has_data(key: str, quantity: int) -> bool`

- `ValueError` if the requested quantity is <= 0.
- `KeyError` if the `key` does not exist in the history.
- `True` if the requested quantity for `key` is available.
- `False` if `quantity > history` for the requested feature.

##### `FeatureStore.latest(key: str) -> FeaturePoint`

- Raises `KeyError` if the `key` does not exist in the history.
- Raises `ContextNotInitialized` if no values have been published.
- Returns the latest `FeaturePoint` of the feature in question.

##### `FeatureStore.window(key: str, quantity: int) -> tuple[FeaturePoint,...]`

-  raises `ValueError` if `quantity <= 0`.
- raises `KeyError` if the `key` does not exist in the history.
- raises `ContextNotInitialized` if the requested quantity exceeds the available history.
- returns the last `n` consecutive `FeaturePoint` of the feature in question.

---
## Failure Semantics

### Type: `Fail-Fast`

### Sources of Errors
#### Invariant Violation

- Invalid initialization
- Invalid `update(step_context, market_view)`
#### `FeatureCalculator` Runtime Error

### Error Handling Policy

`FeatureStore.update()` applies a `fail-fast` policy.

`FeatureStore` does not mask errors locally.

If an invariant is violated or a calculator raises an error, the current `step` is considered invalid.
### Guarantees

`FeatureStore.update()` guarantees:  
  
- **complete success**: state updated atomically;  
- **failure**: no mutation of :
	- history  
	- last_processed_timestamp  
	- last_processed_event_id  
	- last_processed_step_sequence  

Errors are propagated to the `BacktestEngine` orchestration component.

---
## Out of scope

- `FeatureStore` does not handle market data or maintain market state.
- `FeatureStore` does not make strategic decisions
- `FeatureStore` does not manage risk
- `FeatureStore` does not handle execution
- `FeatureStore` has no external side effects; it exposes a read of its state via its API.
