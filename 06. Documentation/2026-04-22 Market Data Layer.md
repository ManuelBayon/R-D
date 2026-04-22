
*MDL = Market Data Layer*

## Responsibilities

- The MDL ingests a `MarketDataEvent` at every `step` of the backtest.
- The MDL maintains an internal market state that is updated exclusively via `ingest(event)` and is exposed as read-only.
- The MDL exposes the most recently ingested `MarketDataEvent`.
- The MDL exposes a window of the last `n` consecutive `MarketDataEvent`.

---
## Assumptions and Prerequisites
  
- `BacktestEngine` injects a `MarketDataEvent` at each step.
- Each `MarketDataEvent` has a non-empty `event_id` of type `str`.
- Each `MarketDataEvent` has a `timestamp` of type `datetime`.
- Each `MarketDataEvent` has a `bar` field of type `OHLCV`.
- All `MarketDataEvents` pertain to a single `instrument` and a single `bar_size`.

---
## State and Ownership

- The MDL holds the market state, which is represented by a `list[MarketDataEvent]`.
- The only component authorized to call `ingest()` is the `BacktestEngine` orchestrator.
- No write access to `_history` is exposed externally.

---
## Invariants

### I1 - Monotonicity of Timestamps

Equal case:
```python
with pytest.raises(BacktestInvariantError):  
    store.ingest(event_same_timestamp)
```

Decreasing case:
```python
with pytest.raises(BacktestInvariantError):  
    store.ingest(event_timestamp_anterior_to_previous)
```

Ensures a strictly ordered time series. Prevents any implicit “reordering” or temporally inconsistent backtesting.

### I2 - Preservation of Ingestion Order

```python
store.ingest(event_1)  
store.ingest(event_2)  
assert store.window(2)[0] == event_1  
assert store.window(2)[1] == event_2
```

Guarantees that the history preserves the exact order in which events were ingested. The view returned by `window(n)` remains consistent with the sequence actually ingested.

## I3 - Immutability of the snapshot returned by `window(n)`

```python
events = store.window(2)  
assert isinstance(events, tuple)  
with pytest.raises(TypeError):  
    events[0] = make_event(  
        event_id="event_03",  
        timestamp=datetime(2020, 1, 3),  
    )
```

Ensures that the structure returned by `window(n)` cannot be modified by the caller. Prevents any accidental alteration of the returned view.

---
## API Specification

- `latest()` returns the most recent `MarketDataEvent` ingested.
- `window(n)` returns the last `n` `MarketDataEvent`s ingested, in the order they were ingested.
- `latest()` == `window(1)[0]`.

---
## Errors

- `MarketStore.ingest(event)` raises a `BacktestInvariantError` if `event.timestamp <= last.timestamp`.
- `latest()` raises a `ContextNotInitializedError` if no events have been ingested.
- `window(n)` raises `ValueError` if `n <= 0`.
- `window(n)` raises `ValueError` if `n > len(history)`.

---
## Out of Scope

- It does not load data from an external source
- It does not sort or reorder market events and assumes that their order is guaranteed by upstream systems
- It does not generate decisions or execute trades.
