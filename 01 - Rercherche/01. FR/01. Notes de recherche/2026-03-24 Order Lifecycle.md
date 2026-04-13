Objectif : 
> garantir que chaque ordre est toujours dans u état valide et traçable.

# 1. États 
## 1.1 Définition d'un état dans un système event-driven

>La condition actuelle d’un objet (agrégat) dérivée de l’historique des events.
### Pourquoi on ne dit pas juste “objet”

Parce qu’un “objet” est trop vague.

Un **agrégat** implique :
- des **invariants métier**
- un **cycle de vie**
- une **frontière claire**
- une **cohérence interne garantie**

---
## 1.1 États actifs

- `CREATED` : L'ordre existe dans le système et est stocké dans `OrderStore`.
- `VALIDATED` : 
- `ACTIVATED`
- `PARTIALLYFILLED`
- `TRIGGERED` (optionnel pour ordres stop lorsque condition atteinte mais pas encore exécuté)
## 1.2 États terminaux 

- `FILLED`
- `CANCELLED`
- `EXPIRED`
- `REJECTED`
# 2. Table des transitions valides

>[!note] a compléter






