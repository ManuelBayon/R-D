# Hypothèses amont

Il y a une couche en amont qui gère la réception des données, éventuellement l'ordering et produit `BarAvailable`.

```python
@dataclass(frozen=True)
class BarAvailable(BaseEvent):
	bar: Bar
```

- Chaque `Bar` possède `timestamp` supérieur au dernier ingéré.
- L'ensemble des `Bar` concernent  un unique `instrument` et une unique `bar_size`.

---
# Minimal causal chain contract

Canonical input : `BarAvailable`

States : 
- `MarketStore` owns `Bars`
- `FeatureStore` owns computed `Features`
- `DecisionLayer` owns no mutable state (pure transformation)

Minimum contracts : 
- MarketStore accepts or refuses a bar
- FeatureStore calculate et each bar the features based on config and MarketStore
- DecisionLayer produit une décision à partir d'uniquement features, market et config.

Output:
- IntentGenerated
- NoOperation

Invariants : 
- Déterminisme : même séquence de bougies + même code et même config donne même features et même décisions.

Tests adverses minimaux :
- bars non ordonnées -> `MarketDataOrderingViolation`
- fenêtre insuffisante pour feature -> `FeatureNotReady`
- decision no-op
- decision intent
- replay identique

---
# Schémas de données minimaux

## `Bar`

```python
@dataclass(frozen=True)  
class Bar:  
    timestamp_utc: datetime
    open: float  
    high: float  
    low: float  
    close: float  
    volume: int = 0
```
## `BarAvailable`

```python
@dataclass(frozen=True)  
class BarAvailable(BaseEvent):  
    run_id: str  
	event_id: str  
	received_at_utc: datetime
    bar: Bar
```
## `IntentGenerated`

```python
@dataclass(frozen=True)
class Intention:
	...

@dataclass(frozen=True)  
class IntentGenerated(BaseEvent):
    run_id: str  
	event_id: str
	bar_timestamp_utc: datetime
	intention: Intention
```
## `NoOperation`

```python
@dataclass(frozen=True)  
class NoOperation(BaseEvent):
    run_id: str  
	event_id: str
	bar_timestamp_utc: datetime
```

---
# Développer les composants minimaux

- `MarketStore`
- `FeatureStore` avec feature trivial SMA
- `DecisionLayer` with trivial rule
- `CausalChainRunner`

---
# Test the architecture by properties

- bars non ordonnées -> `MarketDataOrderingViolation`
- fenêtre insuffisante pour feature -> `FeatureNotReady`
- decision no-op
- decision intent
- replay identique

---
# Success criteria

With only a sequence of Bars + config I reconstruct exactly `MarketStore`, `FeatureStore` and the `Decisions`.

---

Ce qu’un bon environnement te pousserait à faire :

1. **Écrire la chaîne causale minimale**

```
BarAvailable
→ MarketStore.ingest
→ FeatureStore.compute
→ DecisionLayer.evaluate
→ IntentGenerated | NoOperation
```

2. **Définir pour chaque transition**

```
input 
state read
state mutated
output
invariants
failure cases
```

3. **Ensuite seulement écrire les tests**

Tests unitaires :

- `MarketStore` refuse bar non ordonnée ;
- `FeatureStore` retourne not-ready si fenêtre insuffisante ;
- `DecisionLayer` produit intent/no-op de manière déterministe.

Test d’intégration/replay :

- même séquence `BarAvailable`
- même config
- même code  
    ⇒ mêmes features et mêmes décisions.

Donc la réponse stricte :

> La chaîne causale sert à découvrir la structure.  
> Les tests servent à falsifier cette structure.




