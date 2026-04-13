# 1. Clarifier ce qu'est une Feature

Pour l'instant ce n'est pas clair. Il faut partir de l'interrogation suivante.

>quel problème réel le système doit résoudre, pour quel consommateur, avec quelles garanties

## Étape 1 : Le besoin

Quel est le besoin réel ?

- qui a besoin de quoi ?
- à quel moment ?
- pour faire quoi ?

Exemple de forme :

> La stratégie a besoin de lire une valeur dérivée du marché, cohérente avec le step courant, seulement si elle est valide.

---
## Étape 2 : Le consommateur

Qui lit ou utilise le résultat ?

- stratégie ?
- risk ?
- exécution ?
- audit ?
- orchestrateur ?

Tant que le consommateur n’est pas clair, la structure reste floue.

---
## Étape 3 — contrat

Que dois-tu garantir au consommateur ?

Par exemple :

- lecture immuable
- cohérence avec le step
- pas de valeur avant warmup
- traçabilité
- déterminisme

Le contrat est central.  
C’est lui qui va dicter la structure.