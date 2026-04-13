# 1. Les couches de l'architecture v1.2

## 1.1 Couche marché

- rôle : maintenir l’état marché courant et l’historique
- consomme : `MarketDataEvent`
- produit : `MarketView`
- mute : `MarketStore`
## 1.2 Couche indicateurs

- rôle : calculer les features à partir du marché
- consomme : `MarketView`
- produit : `FeatureView`
- mute : `FeatureStore`
## 1.3 Couche décision

- rôle : transformer le contexte en intention de trading
- consomme : `BacktestView`
- produit : `OrderRequest`
- mute : rien
## 1.4 Couche construction des ordres

- rôle : transformer l’intention de trading en ordres explicites et structurés
- consomme : `OrderRequest`, `BacktestView`
- produit : `OrderBatch`
- mute : rien
## 1.5 Couche risque pré-trade

- rôle : valider les ordres avant leur activation en appliquant les règles de risque et de cohérence
- consomme : `OrderBatch`, `PortfolioView`, `MarketView`, éventuellement `OrderStoreView`
- produit : `ValidatedBatchOrder`, `RejectedBatchOrder`
- mute : rien

## 1.6 Couche cycle de vie des ordres

- rôle : enregistrer les ordres validés, maintenir leur cycle de vie et exposer les ordres actifs au moteur
- consomme : `ValidatedOrderBatch` ou `RejectedBatchOrder`, `OrderStoreView`
- produit : `OrderStoreView`, `OrderStatusUpdate`, `ActiveOrderSet`
- mute : `OrderStore`
## 1.7 Couche exécution (simulée ou réelle)

- rôle : simuler l’exécution des ordres actifs contre le marché
- consomme : `ActiveOrderSet`, `MarketView`
- produit : `FillEvent`, `OrderStatusUpdate`
- mute : rien
## 1.6 Couche comptabilité

- rôle : mettre à jour positions, cash, pnl
- consomme : `FillEvent`
- produit : `PortfolioTransition`, `PortfolioView`
- mute : `PortfolioStore`
## 1.8 Couche orchestration

- rôle : orchestrer le `step()`
- consomme : tout ce qui est nécessaire
- produit : `StepRecord`
- mute : rien directement, il appelle les couches propriétaires du state

---
# 2. Appartenance de l'état mutable

- `MarketStore` possède et mute l’état du marché
- `FeatureStore` possède et mute l’état des features
- `OrderStore` possède et mute l’état des ordres
- `PortfolioStore` possède et mute l’état du portefeuille

- `Strategy` ne possède aucun état mutable global
- `OrderConstruction` ne possède aucun état mutable
- `Risk` ne possède aucun état mutable
- `Execution` ne possède aucun état mutable global ; elle produit des événements d’exécution
- `BacktestEngine` orchestre le flux mais ne possède pas l’état métier

---
# 3. Les vues

- `MarketStoreView` représente l’état du marché
- `FeatureStoreView` représente l’état des features
- `OrderStoreView` représente l’état des ordres
- `PortfolioStoreView` représente l’état du portefeuille



