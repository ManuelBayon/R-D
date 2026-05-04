## Champs minimaux commun

```python
@dataclass(frozen=True)
class MetaEvent:
	run_id: str
	sequence_id: str
	causation_id: str | None
	event_id: str
```