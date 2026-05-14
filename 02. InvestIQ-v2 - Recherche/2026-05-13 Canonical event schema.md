# Périmètre

- Single broker
- Single asset
- Pas distribué
- Pas multi processus
- Pas multi stratégies

---
# Enveloppe commune

>[!note] à raffiner

```python
from datetime import datetime
class BaseEvent(ABC):
	run_id: str
	event_id: str
```

---
# `BarReceived `

- Évènement canonique exogène
- Point d'entrée du pipeline unifiée backtest-live-replay.

```python
from investiq... import bar
@dataclass(frozen=True)
class BarReceived(BaseEvent):
	received_at_utc: datetime
	bar: Bar
```

---
# `MarketStore` / `FeatureStore`

- `MarketStore` et `FeatureStore` sont une projection dérivée.
- Il peuvent être entièrement reconstruit en rejouant les événements `BarReceived` journalisés en amont.
- Ils ne produisent pas d'évènements canoniques.

---
# `DecisionLayer`

- La couche de décision produit l'évènement canonique `OrderIntentGenerated` qui est la source de vérité concernant les décisions stratégique. 
- `OrderIntentGenerated` journalise les type d'ordres, les quantité et prix respectifs pour chacun d'entre eux.

```python
@dataclass(frozen=True)
class OrderIntentGenerated(BaseEvent):
	...
```


---
# Risk and Execution venue

Si l'état de Market, feature est rejouer a partir de `BarReceived` et qu'on a `OrderIntentGenerated` comme évènement canonique avec quantité, `side`, etc. `FillReceived` pour reconstruire le portefeuille je dirai que les décision de risque et confirmation d'envoi ne sont pas des évènements canoniques nécessaire au replay.

---
# Portefeuille

- L'état du portefeuille dérive directement des `FillReceived`. 
- L'état peut être reconstitué à partir de son état initial et en appliquant séquentiellement l'ensemble des `Fill` reçus enregistrés dans le journal canonique des évènements.
