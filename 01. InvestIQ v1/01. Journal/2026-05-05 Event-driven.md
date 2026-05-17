## Séparation Event / Request  

#### Event = résultat observé

```
BarEvent
FillEvent
OrderAcceptedEvent
OrderRejectedEvent
TimerEvent
```

#### Effect = tentative

```
SendOrderEffect
CancelOrderEffect
WriteJournalEffect
NotifyEffect
```

---
## Responsabilités

```
Engine
= décide quoi demander
= produit SendOrderEffect

Runner
= transporte les effects vers les bons gateways

ExecutionGateway
= parle au broker ou au simulateur
= transforme réponse externe en events canoniques
```

---

 