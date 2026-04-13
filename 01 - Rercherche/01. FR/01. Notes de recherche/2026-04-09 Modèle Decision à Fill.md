# 1. Modèle actuel (mauvais)

## 1.1 Step du backtest engine

```python
def step(self, event: MarketDataEvent) -> StepRecord:  
  
    # -> atomic_actions   
  
    fills: list[FIFOOperation] = resolve_fifo(  
        actions=list(atomic_actions),  
        position_book=view_before.portfolio_view.fifo_book,  
        execution_price=decision.execution_price,  
    )  
  
    # . Portfolio accounting  
    self._portfolio.apply_fills(fills)          
  
    # . Return step record  
    view_after = self._build_view()  
    return StepRecord(  
        timestamp=view_before.market_view.timestamp,  
        event=event,  
        view_before=view_before,  
        decision=decision,  
        fills=fills,  
        view_after=view_after,  
        diagnostics={  
            "decision":decision.diagnostics,  
        },  
    )
```

## 1.2 Appel du "FIFOResolver"

Le resolver FIFO ne fait pas de simulation d’exécution.  
Il fait de la **lot allocation** / **close matching**.

Donc ses inputs naturels sont :
- `AtomicAction`
- `PositionBookReader`

et ses outputs doivent être des objets du genre :

- `ExecutionIntent`
- `CloseAllocation`
- `OpenInstruction`
- `ExecutionInstruction`

mais **pas encore des Fills**, et pas besoin de prix.
```python
class FIFOResolver:
    """
    Orchestrator: AtomicAction -> FIFOOperation(s).
    Delegates per-action resolution to registered FIFO resolve strategies.
    """

    def __init__(self) -> None:
        self._factory = FIFOResolveFactory()

    def resolve_action(
        self,
        action: AtomicAction,
        position_book: PositionBookReader,
        execution_price: float,
    ) -> list[FIFOOperation]:
        strategy = self._factory.create(action_type=action.type)
        return strategy.resolve(
            action=action,
            position_book=position_book,
            execution_price=execution_price
        )

    def resolve(
        self,
        actions: list[AtomicAction],
        position_book: PositionBookReader,
        execution_price: float,
    ) -> list[FIFOOperation]:
        ops: list[FIFOOperation] = []
        for a in actions:
            ops.extend(
                self.resolve_action(
                    action=a,
                    position_book=position_book,
                    execution_price=execution_price
                )
            )
        return ops
```

## 1.3 PositionBookReader

C'est un wrapper autour des `FIFO` Queues.

```python
from investiq.api.enums import FIFOSide  
from investiq.api.portfolio import PositionBookReader  
from investiq.api.types import FIFOPosition  
  
  
class InMemoryPositionBookReader(PositionBookReader):  
    """  
    Read-only façade over internal FIFO queues.    """  
    def __init__(self, fifo_queues: dict[FIFOSide, list[FIFOPosition]]):  
        self._fifo_queues = fifo_queues  
  
    def active(self, side: FIFOSide) -> tuple[FIFOPosition, ...]:  
        return tuple(p for p in self._fifo_queues.get(side, []) if p.is_active)  
  
    def all(self, side: FIFOSide) -> tuple[FIFOPosition, ...]:  
        return tuple(p for p in self._fifo_queues.get(side, []))  
  
    def count_active(self, side: FIFOSide) -> float:  
        return sum(  
            p.quantity for p in self._fifo_queues.get(side, []) if p.is_active  
        )
```

## 1.4 "FIFOResolve Strategies"

On appelle la stratégie correspondant au type d'action (atomique). Ces stratégies sont censée identifier quoi faire sur le FIFO mais ne sont pas des fills c'est plutot un algo de matching.b   
```python
from typing import ClassVar, Final  
  
from investiq.api.enums import AtomicActionType, FIFOOperationType, FIFOSide  
from investiq.api.fifo import FIFOResolveStrategy  
from investiq.api.portfolio import PositionBookReader  
from investiq.api.types import AtomicAction, FIFOOperation  
  
_EPS: Final[float] = 0.0  
  
def _require(cond: bool, msg: str) -> None:  
    if not cond:  
        raise ValueError(msg)  
  
def _require_price(price: float, name: str) -> None:  
    _require(price > 0.0, f"[{name}] execution_price must be > 0, got {price}")  
  
def _require_qty(qty: float, name: str) -> None:  
    _require(qty > _EPS, f"[{name}] quantity must be > 0, got {qty}")  
  
  
@register_fifo_resolve_strategy(AtomicActionType.OPEN_LONG)  
class OpenLongFIFO(FIFOResolveStrategy):  
    NAME: ClassVar[str] = "OpenLongFIFO"  
    ACTION: ClassVar[AtomicActionType] = AtomicActionType.OPEN_LONG  
    def resolve(  
        self,  
        action: AtomicAction,  
        position_book: PositionBookReader, # Not used for opening a new position.  
        execution_price: float,  
    ) -> list[FIFOOperation]:  
  
        _require(action.type == self.ACTION, f"[{self.NAME}] unexpected action.type={action.type}")  
        _require_price(execution_price, self.NAME)  
        _require_qty(action.quantity, self.NAME)  
  
        return [  
            FIFOOperation(  
                id=FIFOOperation.next_id(),  
                timestamp=action.timestamp,  
                type=FIFOOperationType.OPEN,  
                side=FIFOSide.LONG,  
                quantity=action.quantity,  
                execution_price=execution_price,  
                linked_position_id=None,  
            )  
        ]  
  
  
@register_fifo_resolve_strategy(AtomicActionType.OPEN_SHORT)  
class OpenShortFIFO(FIFOResolveStrategy):  
    NAME: ClassVar[str] = "OpenShortFIFO"  
    ACTION: ClassVar[AtomicActionType] = AtomicActionType.OPEN_SHORT  
    def resolve(  
        self,  
        action: AtomicAction,  
        position_book: PositionBookReader, # Not used for opening a new position.  
        execution_price: float,  
    ) -> list[FIFOOperation]:  
        _require(action.type == self.ACTION, f"[{self.NAME}] unexpected action.type={action.type}")  
        _require_price(execution_price, self.NAME)  
        _require_qty(action.quantity, self.NAME)  
  
        return [  
            FIFOOperation(  
                id=FIFOOperation.next_id(),  
                timestamp=action.timestamp,  
                type=FIFOOperationType.OPEN,  
                side=FIFOSide.SHORT,  
                quantity=action.quantity,  
                execution_price=execution_price,  
                linked_position_id=None,  
            )  
        ]  
  
def _close_from_fifo(  
    *,  
    name: str,  
    side: FIFOSide,  
    action: AtomicAction,  
    position_book: PositionBookReader,  
    execution_price: float,  
) -> list[FIFOOperation]:  
    _require_price(execution_price, name)  
    _require_qty(action.quantity, name)  
  
    fifo = position_book.active(side=side)  
    remaining = action.quantity  
    ops: list[FIFOOperation] = []  
  
    for pos in fifo:  
        if not pos.is_active:  
            continue  
        if pos.quantity <= 0:  
            continue  
  
        close_qty = min(remaining, pos.quantity)  
        ops.append(  
            FIFOOperation(  
                id=FIFOOperation.next_id(),  
                timestamp=action.timestamp,  
                type=FIFOOperationType.CLOSE,  
                side=side,  
                quantity=close_qty,  
                execution_price=execution_price,  
                linked_position_id=pos.id,  
            )  
        )  
        remaining -= close_qty  
        if remaining <= 0:  
            break  
  
    _require(remaining <= 0, f"[{name}] insufficient FIFO capacity: missing={remaining}")  
    return ops  
  
  
@register_fifo_resolve_strategy(AtomicActionType.CLOSE_LONG)  
class CloseLongFIFO(FIFOResolveStrategy):  
    NAME: ClassVar[str] = "CloseLongFIFO"  
    ACTION: ClassVar[AtomicActionType] = AtomicActionType.CLOSE_LONG  
    def resolve(  
        self,  
        action: AtomicAction,  
        position_book: PositionBookReader,  
        execution_price: float,  
    ) -> list[FIFOOperation]:  
        _require(action.type == self.ACTION, f"[{self.NAME}] unexpected action.type={action.type}")  
        return _close_from_fifo(  
            name=self.NAME,  
            side=FIFOSide.LONG,  
            action=action,  
            position_book=position_book,  
            execution_price=execution_price,  
        )  
  
  
@register_fifo_resolve_strategy(AtomicActionType.CLOSE_SHORT)  
class CloseShortFIFO(FIFOResolveStrategy):  
    NAME: ClassVar[str] = "CloseShortFIFO"  
    ACTION: ClassVar[AtomicActionType] = AtomicActionType.CLOSE_SHORT  
    def resolve(  
        self,  
        action: AtomicAction,  
        position_book: PositionBookReader,  
        execution_price: float,  
    ) -> list[FIFOOperation]:  
        _require(action.type == self.ACTION, f"[{self.NAME}] unexpected action.type={action.type}")  
        return _close_from_fifo(  
            name=self.NAME,  
            side=FIFOSide.SHORT,  
            action=action,  
            position_book=position_book,  
            execution_price=execution_price,  
        )
```

Par contre je pense qu'il ne faut pas passer le prix car il ne s'agit pas là de simuler une exécution (Fill). Il faut faire un mapping action <-> stratégie et passer a la stratégie, l'action et une `PositionBookReader` pour identifier quelles positions ouvrir / cloturer et passer les identifiants au module dexécution avec simulation slippage, latence, liquidité etc, ce module produisante des Fills, Cancels etc. tout ce qui est de la responsabilité de l'exécution (simulée et réel).

Ensuite les Fills et autre messages produits par la couche exécution sont appliquées au portefeuille.

Ceci est juste pour corriger la version, pour la suivante j'aurait un modele aggregate et event centric mais je dois deja corriger les problemes structurels de cette version.

---
# 2. Correction du modèle 

```
list[AtomicAction] + FIFOReader -> list[ExecutionInstruction] 
list[ExecutionInstruction] + MarketView -> list[Fill]
Portfolio.apply_fills(...) -> Update FIFO queues / Portfolio
```

## 2.1 AtomicAction

```python
@dataclass(frozen=True)
class AtomicAction:
	type : ActionType
	side : FIFOSide
	decision_timestamp: pd.Timestamp
	quantity: float
	
	def __post_init__(self):
		if self.quantity <= 0:
			raise ValueError("Quantity must be > 0")
```
## 2.2 Fill

```python
@dataclass(frozen=True)
class Fill:
	entry_id : str | None
	action : ActionType
	side : FIFOSide
	decision_timestamp : pd.Timestamp
	execution_timestamp : pd.Timestamp
	quantity : float
	price : float
	
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

## 2.3 MarketView

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

## 2.4 FIFOReader and FIFOView

```python
class FIFOReader:
	
	def __init__(self, inventory: dict[FIFOSide, list[InventoryEntry]]):
		self._inventory = inventory
	
	def active_entries(self, side: FIFOSide) -> tuple[InventoryEntry, ...]:
		...
	
	def available_quantity(self, side: FIFOSide) -> float:
		...
	
	def require_capacity(self, side: FIFOSide, quantity: float) -> None:
		...
		
class FIFOView:
	reader: FIFOReader
	
	def next_closable_entries(side, quantity) -> tuple[InventoryEntry, ...]
		...
```
## 2.5 Fill Simulator

Entrée : 
- `list[AtomicAction]`
- `MarketView`
- `FIFOView`

Sortie :
- `list[Fill]`

```python
class FillSimulator:
	
	def __init__(self):
		self._next_entry_id = 0
	
	def simulate(
		actions: list[AtomicAction],
		market_view : MarketView,
		FIFOView : FIFOView,
	) -> list[Fill]
		
		fills : list[Fill]
		
		
		for action in actions:
			strategy = ...[action.type]
```