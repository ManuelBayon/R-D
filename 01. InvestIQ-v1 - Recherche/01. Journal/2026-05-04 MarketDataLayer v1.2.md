


---
## Hypothèses


## Entrée

```python
@dataclass(frozen=True)
class Bar:
	timestamp: datetime
	open: float
	high: float
	low: float
	close: float
	volume: int
```
## Sortie

- `MarketDataReceived`
- `MarketStoreUpdated`
- Exposition de la vue de marché via API.

```python
@dataclass(frozen=True)
class MarketDataReceived:
	meta: MetaEvent
	event_id: str
	payload: OHLCV

@dataclass(frozen=True)
class MarketStoreUpdated:
	meta: MetaEvent
	event_id: str
	causation_id: str
```

## Invariants minimaux

- Historique append-only
- Historique strictement croissant selon les timestamp
- `MarketStoreUpdated.causation_id` == `MarketDataReceived.event_id`
- Le journal vérifié ses propres invariants notamment unicité des `event_id` etc.

