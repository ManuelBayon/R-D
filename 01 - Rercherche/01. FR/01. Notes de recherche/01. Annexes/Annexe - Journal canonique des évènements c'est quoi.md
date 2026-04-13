# 1. Pourquoi il est indispensable

Sans ce journal, mon moteur reste dépendant de l’état courant :

- `FeatureStore`
- `OrderStore`
- `PortfolioStore`
- éventuellement des résumés comme `StepRecord`

Le problème est que l’état courant répond à la question :

> “où en est le système ?”

mais pas à :

> “comment exactement est-il arrivé là ?”

Or pour une infra de trading utile, il faut pouvoir répondre précisément à :

- pourquoi cet ordre a existé
- pourquoi il a été rejeté ou activé
- pourquoi ce fill a été produit
- pourquoi le portefuille a été muté
- ce qui s'est passé entre deux steps
- si un bug bient de la stratégie, du risque, de gestionnaire d'ordres, de l'exécution ou de l'orchestrateur?

Le journal canonique répond à celà.

---
# 2. Ce qu'est ce journal 
 
 Ce journal est une infrastructure transverse : `DomainEventLog`

- append-only
- ordonné
- immutable
- rejouable
- auditable

C'est lui qui donne une mémoire fiable au moteur.

Ma structure actuelle dit déjà en substance : 

- `MarketStore` possède et mute l’état du marché
- `FeatureStore` possède et mute l’état des features
- `OrderStore` possède et mute l’état des ordres
- `PortfolioStore` possède et mute l’état du portefeuille

- `Strategy` ne possède aucun état mutable global
- `OrderConstruction` ne possède aucun état mutable
- `Risk` ne possède aucun état mutable
- `Execution` ne possède aucun état mutable global ; elle produit des événements d’exécution
- `BacktestEngine` orchestre le flux mais ne possède pas l’état métier

Le journal vient compléter cela avec un règle fondamentale : 

>**toute mutation métier "importante" doit être justifiable par un évènement canonique appendé dans le journal.**

Autrement dit :

- si un ordre existe, il doit y avoir la trace de son acceptation / activation / modification de statut ;
- si une position change, il doit y avoir un `FillEvent` puis une `PortfolioTransition` ;
- si un step s’est déroulé, il doit y avoir au moins un début et une fin de step.

---
# 3. Propriétés inférés par le journal 

## 1. Re jouabilité

je peux rejouer un backtest ou un step exact à partir des événements.
## 2. Auditabilité 

Je peux remonter la chaîne causale d’une décision à une exécution et à son impact portefeuille.
## 3. Débogage propre

Au lieu d’inspecter un état final opaque, j'inspecte la séquence causale.
# 4. Testabilité

Je peux tester une couche en disant :

- entrée donnée,
- événement attendu,
- mutation attendue.



