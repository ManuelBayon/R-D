
À partir de maintenant, quand tu prends un sujet comme `FeatureStore`, `OrderAggregate`, `ExecutionModel`, tu passes **toujours** par cette séquence :
## Étape 1 : besoin

Quel est le besoin réel ?

Pas “je veux un pipeline”.  
Pas “il faut une couche feature”.

Mais :

- qui a besoin de quoi ?
- à quel moment ?
- pour faire quoi ?

Exemple de forme :

> La stratégie a besoin de lire une valeur dérivée du marché, cohérente avec le step courant, seulement si elle est valide.

---
## Étape 2 : consommateur

Qui lit ou utilise le résultat ?

- stratégie ?
- risk ?
- exécution ?
- audit ?
- orchestrateur ?

Tant que le consommateur n’est pas clair, la structure reste floue.

---
## Étape 3 : contrat

Que dois-tu garantir au consommateur ?

Par exemple :

- lecture immuable
- cohérence avec le step
- pas de valeur avant warmup
- traçabilité
- déterminisme

Le contrat est central.  
C’est lui qui va dicter la structure.

---
## Étape 4 : transformation nécessaire

Quelle transformation doit exister pour passer de l’entrée au résultat ?

Exemple :

- lire le marché
- accumuler un historique minimal
- calculer une moyenne
- publier la valeur
- exposer un statut de validité

Ici tu identifies **la mécanique nécessaire**, pas encore les classes.

---
## Étape 5 : invariants

Qu’est-ce qui doit toujours rester vrai ?

Exemples :

- une feature publiée est cohérente avec l’état courant
- une stratégie ne lit pas de valeur invalide sans le savoir
- le store est l’unique owner de l’état publié
- le replay redonne le même résultat

Les invariants servent de garde-fous.

---
## Étape 6 : frontières

Qui possède quoi ?  
Qui lit ?  
Qui écrit ?  
Qui orchestre ?

C’est ici que tu poses :

- ownership
- mutation vs lecture
- dépendances autorisées

---
## Étape 7 : structure minimale

Seulement maintenant tu choisis :

- faut-il un store ?
- faut-il un calculator ?
- faut-il un snapshot ?
- faut-il un event ?
- faut-il un orchestrateur ?

Pas avant.

---
# 3) La bonne question n’est presque jamais “quel objet créer ?”

La bonne question est plutôt :

> “de quelles responsabilités distinctes ai-je besoin pour satisfaire le contrat le plus simplement possible ?”

C’est très différent.

Parce qu’un objet n’est légitime que s’il porte une responsabilité claire.

---
# 4) La méthode pratique à appliquer

Je te propose un canevas fixe.  
Tu peux l’utiliser pour chaque composant.

```
1. Besoin  
2. Consommateur  
3. Contrat / garanties  
4. Entrées nécessaires  
5. Transformation nécessaire  
6. État nécessaire (si oui, lequel ?)  
7. Invariants  
8. Ownership  
9. Mutations autorisées  
10. Sorties exposées  
11. Structure minimale proposée  
12. Ce que le composant ne doit pas faire
```

Si tu remplis ça sérieusement, la structure émerge beaucoup plus naturellement.