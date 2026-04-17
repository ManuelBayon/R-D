
# 1. Vue simplifié du nouveau modèle d'exécution

```
1. Execution Instruction Resolver : list[AtomicAction] + FIFOReader -> list[ExecutionInstruction] 
   
2. Fill Simulator : list[ExecutionInstruction] + MarketView -> list[Fill]

3. Portfolio : apply_fills(...) -> Update FIFO queues / Portfolio
```

---
# 3. Fill Simulator

```
Fill Simulator : list[ExecutionInstruction] + MarketView -> list[Fill]
```

## ExecutionInstruction



## MarketView

```python
@dataclass(frozen=True)  
class MarketView:  
    reader: MarketHistoryReader  
    @property  
    def event_id(self) -> str:  
        return self.reader.latest().event_id  
    @property  
    def timestamp(self) -> pd.Timestamp:  
        return self.reader.latest().timestamp  
    @property  
    def bar(self) -> OHLCV:  
        return self.reader.latest().bar
```
## Fill

```python
@dataclass(frozen=True)
class Fill:
	instruction_id: str
	fill_id: str
	inventory_entry_id : str | None
	action : ActionType
	decision_timestamp : pd.Timestamp
	execution_timestamp : pd.Timestamp
	execution_price : float
	quantity : float
	
	def __post_init__(self) -> None:  
		if self.quantity <= 0:  
			raise ValueError("quantity must be > 0")  
		if self.price <= 0:  
			raise ValueError("execution_price must be > 0")  
		if self.action == ActionType.OPEN and self.entry_id is not None:  
			raise ValueError("OPEN fill must not reference an entry_id")  
		if self.action == ActionType.CLOSE and self.entry_id is None:  
			raise ValueError("CLOSE fill must reference an entry_id")
```
## Fill Simulator

Entrée : 
- `list[ExecutionInstruction]`
- `MarketView`

Sortie :
- `list[Fill]`

```python
class FillSimulator:
	
	def simulate(
		instructions: list[ExecutionInstruction],
		market_view : MarketView,
	) -> list[Fill]
		
		fills : list[Fill]
		
		for instruction in instructions:
			...
```