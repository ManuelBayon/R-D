Most systems become unmanageable because state and responsibilities are not clearly defined.
# 1. Layers of the v1.2 Architecture

## 1.1 Market Layer

- Role: Maintain the current market state and history
- Consumes: `MarketDataEvent`
- Produces: `MarketView`
- Interacts with: `MarketStore`
## 1.2 Indicators Layer  

- Role: Calculate features based on the market
- Consumes: `MarketView`
- Produces: `FeatureView`
- Updates: `FeatureStore`
## 1.3 Decision Layer

- Role: Transform the context into a trading intention
- Consumes: `BacktestView`
- output: `OrderRequest`
- mutes: none
## 1.4 Order Construction Layer

- role: transforms trading intent into explicit, structured orders
- consumes: `OrderRequest`, `BacktestView`
- output: `OrderBatch`
- mutes: none
## 1.5 Pre-trade Risk Layer

- Role: Validate orders before they are activated by applying risk and consistency rules
- Consumes: `OrderBatch`, `PortfolioView`, `MarketView`, possibly `OrderStoreView`
- Produces: `ValidatedBatchOrder`, `RejectedBatchOrder`
- Mutes: None
## 1.6 Order Lifecycle Layer

- Role: Record validated orders, manage their lifecycle, and expose active orders to the engine
- Consumes: `ValidatedOrderBatch` or `RejectedBatchOrder`, `OrderStoreView`
- Produces: `OrderStoreView`, `OrderStatusUpdate`, `ActiveOrderSet`
- Mutes: `OrderStore`
## 1.7 Execution Layer (simulated or real)

- role: simulate the execution of active orders against the market
- consumes: `ActiveOrderSet`, `MarketView`
- produces: `FillEvent`, `OrderStatusUpdate`
- mutes: none
## 1.8 Accounting Layer

- role: update positions, cash, P&L
- consumes: `FillEvent`
- produces: `PortfolioTransition`, `PortfolioView`
- mutes: `PortfolioStore`
## 1.9 Orchestration layer

- role: orchestrate the `step()`
- consumes: everything necessary
- produces: `StepRecord`
- mutes: nothing directly; it calls the layers that own the state

---
# 2. Ownership of mutable state

- `MarketStore` owns and mutes the market state
- `FeatureStore` owns and mutes the feature state
- `OrderStore` owns and mutes the order state
- `PortfolioStore` owns and mutes the portfolio state
  
- `Strategy` has no global mutable state
- `OrderConstruction` has no mutable state
- `Risk` has no mutable state
- `Execution` has no global mutable state; it produces execution events
- `BacktestEngine` orchestrates the flow but does not hold business state

---
# 3. Views

- `MarketStoreView` holds and updates the market state
- `FeatureStoreView` holds and updates the feature state
- `OrderStoreView` holds and updates the order state
- `PortfolioStoreView` holds and updates the portfolio state