
`FeatureLayer`
`StepContext`
`MarketDataEvent`

---
## Responsabilité

- La `FeatureLayer` a pour responsabilité de mettre à jour l'ensemble des calculateurs configurés.
- La `FeatureLayer` a pour rôle de maintenir un état interne des features disponibles.
- La `FeatureLayer` retourne une `FeatureStepEvent` comprenant un `StepContext` ainsi que le résultat renvoyé par chacun des calculateurs au `step` courant `float | None`.

---
## Hypothèses et pré-conditions

- `BacktestEngine` injecte un `StepContext` et une `MarketView` à chaque `step`.
- `StepContext` possède un `RunId` de type `str != ""`.
- `StepContext` possède un `step_sequence` de type `int > 0` .
- `StepContext` possède un `source_event_id` de type `str != ""`.
- `StepContext` possède un `market_timestamp` de type `datetime`.
- `market_view` est un tuple avec au moins 1 `MarketDataEvent`. 
- Le tuple de `MarketDataEvent` est considéré comme valide.

---
## Mutation et état 

- La `FeatureLayer` possède l'état des `features` qui se matérialise par ` dict[str, list[FeaturePoint]]`.
- 

---
## Invariants



---
## Contrat API



---
## Erreurs



---
## Hors périmètre