# 1. Principe central

Tester une hypothèse =

```
Hypothèse → Expérience contrôlée → Mesure → Conclusion
```

Ton système doit garantir que **chaque étape est propre**.

---
# 2. Ce que ton infra doit absolument avoir

## 1. Reproductibilité stricte

```
même input + même config → même output
```

Donc :

- dataset versionné
- paramètres figés
- seed contrôlée (si stochasticité)

👉 sinon : résultats non fiables

---
## 2. Isolation des expériences

Tu dois pouvoir dire :

```
“ce run = cette hypothèse uniquement”
```

Donc :

- pas d’état partagé implicite
- pas de contamination entre runs
- reset complet entre expériences

---
## 3. Traçabilité complète

Pour chaque résultat :

```
tu peux remonter :→ dataset utilisé→ paramètres→ version du code→ outputs intermédiaires
```

👉 sinon : impossible d’expliquer un résultat

---
## 4. Séparation données / logique

Critique.

```
data ≠ code ≠ résultats
```

Sinon :

- biais cachés
- résultats non généralisables

---
## 5. Gestion du biais (très important)

Ton système doit empêcher :

```
- lookahead bias- survivorship bias- data leakage
```

Minimum :

- séparation train / test
- respect du temps (no future access)

---
## 6. Mesures robustes

Tu dois produire autre chose que :

```
“ça monte”
```

Minimum :

```
- PnL
- drawdown
- Sharpe / variance
- distribution des returns
```

---
## 7. Comparaison d’expériences

Tu dois pouvoir faire :

```
stratégie A vs stratégie B
```

dans les mêmes conditions.

Donc :

```
même data
même période
même règles d’exécution
```

---
## 8. Expérimentation paramétrique

Ton système doit permettre :

```
varier un paramètre → observer impact
```

Ex :

```
window SMA = 10 / 20 / 50
```

👉 sinon : pas de recherche, juste du test manuel

---
## 9. Logging scientifique (différent du debug)

Tu dois enregistrer :

```
- résultats agrégés- métriques- paramètres
```

Pas juste des events techniques.

---
## 10. Temps comme contrainte stricte

Règle :

```
aucune donnée future accessible
```

C’est non négociable.

---
## 11. Ce qui manque souvent (et qui te fera sortir du lot)

```
- versioning des expériences
- capacité à rejouer exactement un run
- comparaison automatique entre runs
```

