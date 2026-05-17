
L'objectif : 
- garder la clarté du projet
- tracer les décisions importantes
- permettre l'évolution de l'architecture
- éviter la dériver cognitive

Structure:
```
/research_log
/components
/causal_chains
/adr
/scenarios
```

La documentation doit servir : 
- à penser,
- décider
- rejouer,
- corriger,
- transmettre

---
## /research_log

Journal chronologique R&D.
Très important.

Format :
```
Date
Hypothesis
What was tested
Observed behavior
Breakage
Tradeoff
Next step
```

Le but :

```
tracer l’évolution de la pensée
```

Pas être “propre”.

---
## /components

État courant des composants.

Exemple:
```
MarketStore
FeatureStore
DecisionLayer
```

Avec:
```
Responsibilities
Inputs
Owned state
Outputs
Guarantees
Refusals
Out of scope
```

Le but :

```
stabiliser les boundaries
```

---
## /causal_chains

Très important en systèmes complexes.

Exemple:
```
BarAvailable
→ MarketStore
→ FeatureStore
→ DecisionLayer
→ IntentGenerated | NoOperation
```

Tu documentes :

- ordering ;
- replay ;
- mutations d’état ;
- causalité ;
- outputs.

Le but :

```
rendre les interactions explicites
```

---
## /adr

Architecture Decision Records.

Courts.  
Précis.  
Datés.

```
Context
Decision
Tradeoff
Consequences
Invalidation conditions
```

Le but :

```
conserver la mémoire architecturale
```

---
## /scenarios

Très utile plus tard.

Exemple :
```
nominal_replay
out_of_order_bars
duplicate_timestamp
missing_feature
```

Avec :

```
expected outputs / expected failures
```

Le but :

```
versionner les causalités attendues
```

---
# Et surtout

Ils te diraient probablement :

```
la doc doit servir :
- à penser,
- décider,
- rejouer,
- corriger,
- transmettre.
```

Pas :

```
à produire un document “impressionnant”
```
