# 1. Message vs Évènement
## 1. Message

Commande interne adressée à une couche précise.
- orienté destinataire
- pas forcément journalisé tel quel
- sert à faire avancer le flux
## 2. Event

Fait métier constaté et appendé au journal canonique.
- orienté histoire du système
- immutable
- rejouable
- daté, ordonné, identifiable

>les couches se parlent en messages, le système se souvient en events.

---

==Template==

**Inputs :** 
**Contexte:** 
**Trigger** : 
**Producteur** : 
**Evènement** : 
**Outputs** : 
- 
**State transitions** : 
- 

---
## . Couche orchestration

**Inputs :** Aucun
**Contexte:** Aucun
**Trigger** : Début de l'étape d'orchestration de réception d'un évènement de marché `MarketDataEvent`
**Producteur** : Orchestrateur
**Evènement** : `StepStarted`
**Outputs** : 
- ajout de l'évènement `StepStarted` au journal
**State transitions** : 
- Aucune

## . Couche Marché

**Inputs :** 
- backtest : données historiques 
- live : flux live
**Trigger** : Arrivée d’une nouvelle donnée de marché (bougie, tick)
**Producteur** : 
- backtest : `DataFeed` 
- live : `MarketDataSource`
**Evènement (exogène)** : `MarketDataEvent`
**Outputs** : 
- `MarketStore` mis à jour
- `MarketView` exposée aux autres composants
- `MarketDataEvent` ajouté au journal canonique
**State transitions** : Aucune (réception d'un évènement marché)

## . Couche Indicateurs

**Inputs :** Aucun
**Contexte:** Lit `MarketView`
**Trigger** : Suite à l'ingestion du `MarketDataEvent`, c'est au tour des indicateurs d'être potentiellement recalculé.
**Producteur** : La couche des indicateurs
**Evènement** : `FeaturesComputed`
**Outputs** : 
- mute `FeatureStore`
- expose `FeatureView`
- ajoute `FeaturesComputed` au journal 
**State transitions** : 
- Aucune

## . Couche Décision

**Inputs :** 
**Contexte:** 
**Trigger** : 
**Producteur** : 
**Evènement** : 
**Outputs** : 
- 
**State transitions** : 

## . Couche construction des ordres 

## . Couche risque pré-trade

## . Couche gestion des ordres

## . Couche exécution (réelle ou simulée)

## . Couche comptabilité

## . Couche orchestration

**Inputs :** `ValidatedBatchOrder`
**Contexte:** `OrderStoreView`
**Trigger** :  réception d'un `ValidatedBatchOrder`
**Producteur** : `OrderManager`
**Evènement** : `OrderCreated`
**Outputs** : 
- Append `OrderCreated` dans le journal canonique des évènements.
- `OrderStore` mis à jour (nouvel ordre persisté)
- `OrderStoreView` mise à jour (dérivée du store)
**State transitions** : 
- Initialisation de l'ordre dans l'état `CREATED`
- Pas de transition depuis un état précédent

**Inputs :** `RejectedOrderBatch`
**Contexte:** `OrderStoreView`
**Trigger** : réception d'un `RejectedOrderBatch` décision de rejet issue de la validation pré-trade
**Producteur** : `OrderManager`
**Evènement** : `OrderRejected`
**Outputs** : 
- Append `OrderRejected` dans le journal canonique des évènements.
- exposition d’un `OrderStoreView` cohérent
**State transitions** : 


---
## Gestion des ordres

- `OrderActivated`
- `OrderCancelled`
- `OrderExpired`
## Exécution

- `OrderFilled`
- `OrderPartiallyFilled` (en option)
## Portefeuille

- `PortfolioTransition`

## Orchestration

- `StepStarted`
- `StepCompleted`



