# 1. Contrat du resolver

```
Resolver : list[AtomicAction] + FIFOReader -> list[ExecutionInstruction]
```

## AtomicAction

```python
@dataclass(frozen=True)
class AtomicAction:
	decision_id: str
	action_id: str
	action_type : ActionType # OPEN/CLOSE
	inventory_side : InventorySide # LONG/SHORT
	decision_timestamp: pd.Timestamp
	order_type : OrderType = MARKET_ORDER
	quantity: float
	
	def __post_init__(self):
		if self.quantity <= 0:
			raise ValueError("Quantity must be > 0")
```

## FIFOReader

```python
class FIFOReader:
	
	def __init__(self, inventory: dict[FIFOSide, list[InventoryEntry]]):
		self._inventory = inventory
	
	def active_entries(self, inventory_side : InventorySide) -> tuple[InventoryEntry, ...]:
		...
	
	def available_quantity(self, inventory_side : InventorySide) -> float:
		...
	
	def require_capacity(self, inventory_side : InventorySide, quantity: float) -> None:
		...
	
	def next_closable_entries(inventory_side : InventorySide, quantity: float) -> tuple[InventoryEntry, ...]
```

## Exécution Instructions

Exemple `AtomicAction` :
```python
AtomicAction:
	action_id = "ATM_ACT_00123"
	instruction_id = "EXEC_INSTR_00123"
	action_type = OPEN
	inventory_side = LONG
	decision_timestamp= datetime(2025,04,13,10,00,00)
	order_type OrderType = MARKET_ORDER
	quantity= 1
```

**Responsabilité :** 
Une `ExecutionInstruction` représente une instruction élémentaire ciblant exactement une inventory entry existante pour une clôture, ou une ouverture élémentaire sans cible inventory préexistante.

**Champs minimaux :**
- côté  inventaire (long/short)
- type d'action (ouvrir/fermer)
- Identifiant de l'entrée FIFO à clôturer (si clôture)
- Quantité à ouvrir ou fermer sur lentrée en question (une action atomique peut créer plusieurs instructions d'exécution).

**Invariants :**
1. Invariants locaux
	- `quantity > 0`
	- `OPEN -> linked_entry_id is None`
	- `CLOSE -> linked_entry_id is not None`
	- `CLOSE -> linked_entry_id` référence une entrée active
	- `instruction_type`, `fifo_side`, `decision_timestamp` inherited from source action
	- respect strict de l’ordre FIFO (fermetures des premières entrée dans l'ordre).

2. Invariants d'aggrégation
	- -somme des quantités = quantité de l’action
	- toutes les instructions produites partagent la même action source

**Ce que l'objet ne contient surtout pas :**
- Pas de prix d'exécution 
- Pas de timestamp d'exécution
c'est de la responsabilité du Fill simulator avec modèle de slippage, latence, liquidité, etc.

```python
@dataclass(frozen=True)
class ExecutionInstruction:
	action_id:str
	instruction_id: str
	instruction_type : ActionType
	inventory_side : InventorySide
	decision_timestamp : pd.Timestamp
	order_type : OrderType = MARKET_ORDER
	quantity : float
	linked_entry_id : str | None = None
```

## 2.4 Execution Instruction Resolver

```python
class ExecutionInstructionResolver:
	
	def __init__(
		self, 
		fifo_reader: FIFOReader
	):
		self._fifo_reader = fifo_reader
	
	def resolve_close(
		action: AtomicAction,
	) -> list[ExecutionInstruction]:
		entries = self._fifo_reader.next_closable_entries(
				side = action.inventory_side, 
				quantity = action.quantity,
		)
		quantity_to_close = action.quantity
		instructions: list[ExecutionInstruction] = []
		
		for entry in entries
			
			if quantity_to_close <= 0:
				break
			
			elif quantity_to_close > entry.remaining:
				 closed_quantity =  entry.remaining
			
			else:
				closed_quantity =  quantity_to_close
			
			instructions.append(
				 ExecutionInstruction(
					action_id = ,
					instruction_id = ,
					instruction_type = action.action_type,
					inventory_side = action.inventory_side,
					decision_timestamp = action.decision_timestamp,
					order_type = action.order_type,
					quantity = closed_quantity,
					linked_entry_id = entry.id,
				)
			)
			quantity_to_close -= closed_quantity
		
		if not nearly_equal(quantity_to_close, 0.0):
			raise ValueError(...)
		
		return instructions
	
	def resolve_open(action: AtomicAction) -> list[ExecutionInstruction]:
		return [
			ExecutionInstruction(
				action_id = ,
				instruction_id = ,
				instruction_type = action.action_type,
				inventory_side = action.inventory_side,
				decision_timestamp = action.decision_timestamp,
				order_type = action.order_type,
				quantity = action.quantity,
				linked_entry_id = None,
			),
		]
	
	def resolve(
		actions: list[AtomicAction],
		fifo_reader : FIFOReader,
	) -> list[ExecutionInstruction]
		# Initialization
		execution_instructions: list[ExecutionInstruction] = []
		for action in actions:
			if action.action_type is ActionType.CLOSE:
				result = self.resolve_close(action)
			elif action.action_type is ActionType.OPEN:
				result = self.resolve_open(action)
			else:
				raise ValueError(f"ActionType: {action.action_type} not recognized.")
			instructions.extend(result)
```
