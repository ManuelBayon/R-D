## 1. Commencer par écrire le contrat du composant

> **quelle vérité ce composant garantit-il au reste du système ?**

`FeatureStore` reçoit un ensemble de `FeatureCalculator`, calcule leurs valeurs à chaque step de marché, et stock uniquement les publications valides sous forme de `FeaturePoint(timestamp, value)` indexés par `key`.

# 2. Invariants d'initialisation

### I1 - Unicité des `keys`

Si deux calculators exposent la même `key`, alors `FeatureStore` refuse l’initialisation

### I2 Déclaration complète des keys

Toutes les `key` exposées par les calculators sont présentes dans `_history` dès l’initialisation.

### I3 - Historique initial vide.

Pour toute `key`, `_history[key] == []` après init

# 3. Invariants runtime

## R1 - Si le calculateur renvoir None, pas de publication.

ok.

## R2 - Si le calculateur renvoie un flottant

Un `FeaturePoint` avec Timestamp et valeur est ajouté à l'historique.

## R3 - Le dernier point ajouté et accessible via l'index -1 est le dernier recu

---
# Dernier invartiants à valider le 03/04/2026 ensuite vidéo sur Features.

## R4 - view.is_ready retourne un bool si un une valeur flottant est publiée sinon retourne Faux et historique vide

## R5 - view.require() renvoie la dernière valeur

## R6 - view.lastest_point() renvoi le dernier FeaturePoint






