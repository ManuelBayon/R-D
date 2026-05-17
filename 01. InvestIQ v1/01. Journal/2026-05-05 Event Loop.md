## Définition simple

Une **event loop** =

```
prendre un événement
→ le traiter
→ éventuellement produire d’autres événements
→ continuer
```

Rien de plus.

---

## Forme mentale

```
queue = [events initiaux]

tant que queue non vide :
    prendre 1 event
    appliquer une règle    
    produire 0..n nouveaux events    
    les ajouter à la queue
```

---
