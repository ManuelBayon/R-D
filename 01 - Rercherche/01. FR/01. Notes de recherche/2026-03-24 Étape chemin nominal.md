## 1. Orchestration

- append `StepStarted` (EVENT) au journal des évènements
## 2. Marché

- reçoit et append `MarketDataEvent` (EVENT) au journal des évènements
- mute `MarketStore`
- expose `MarketView`
## 3. Indicateurs

- lit `MarketView`
- mute `FeatureStore`
- append `FeaturesComputed` (EVENT) au journal des évènements
- expose `FeatureView`

## 4. Décision

- lit `BacktestView` (composé de `MarketView`, `FeatureView`, `ActiveOrderView` et `PortfolioView`)
- produit `OrderRequest` (MESSAGE)
- append `OrderRequested` (EVENT) au journal des évènements.
## 5. Construction ordres

- lit `OrderRequest` et `BacktestView`
- produit `BatchOrder` (MESSAGE)
- append `BatchOrderCreated` (EVENT) au journal des évènements
## 6. Risque pré-trade

- lit `OrderBatch` et `RiskContextView` composé de (`MarketView`, `ActiveOrdersView`, `PortfolioView`)
- produit `ValidatedBatchOrder` (MESSAGE)
- append `BatchOrderValidated` (EVENT) au journal des évènements
## 7. Gestion des ordres

- reçoit `ValidatedBatchOrder`
- mute `OrderStore`
- append `OrderActivated` (EVENT) au journal des évènements
- expose `ActiveOrdersView`
## 8. Exécution (réelle ou simulée)

- lit  `ActiveOrdersView` + `MarketView`
- produit `FillEvent` (MESSAGE)
- append `FillEventReceived` (EVENT) au journal des évènements
## 9. comptabilité

- reçoit `FillEvent` 
- mute `PortfolioStore`
- append `PortfolioTransition` (EVENT) au journal des évènements
- expose `PortfolioView`
## 10. Orchestration 

- append `StepCompleted` (EVENT) au journal des évènements 

---
# 2. Principes d'architecture

## 1.1 Event-centric (vérité)

>Le système enregistre des faits (events).
>L'état est dérivé.

- journal canonique = source de vérité
- append-only
- replay possible
- chaque mutation = conséquence d'un évènement

## 1.2 Aggregate-centric (cohérence)

>Les règles métiers sont centralisées dans l'agrégat.

- `OrderAggregate` décide : 
	- ce qui est valide
	- les transitions autorisées
- produit des évènements
- protège les invariants

---
# 3. Pseudo code happy path

```
MarketDataEvent  
→ StepStarted  
→ Market updated  
→ FeaturesComputed  
→ Strategy decides  
→ OrderRequest  
→ OrderSpec built  
→ OrderAggregate.handle_create(...)  
→ OrderCreated  
→ OrderAggregate.handle_validate(...)  
→ OrderValidated  
→ OrderAggregate.handle_activate(...)  
→ OrderActivated  
→ Execution simulates fill  
→ OrderAggregate.handle_fill(...)  
→ OrderFilled  
→ Portfolio updated  
→ StepCompleted
```

```python
def process_market_event(market_data_event):
    step_id = generate_step_id()

    # ------------------------------------------------------------------
    # 1) STEP START
    # ------------------------------------------------------------------
    step_started = StepStarted(
        step_id=step_id,
        caused_by=market_data_event.event_id,
    )
    record_and_apply([step_started])

    # ------------------------------------------------------------------
    # 2) MARKET LAYER
    # ------------------------------------------------------------------
    market_received = MarketDataReceived(
        step_id=step_id,
        instrument=market_data_event.instrument,
        timestamp=market_data_event.timestamp,
        payload=market_data_event.payload,
    )
    record_and_apply([market_received])

    # ------------------------------------------------------------------
    # 3) FEATURES LAYER
    # ------------------------------------------------------------------
    market_view = market_store.view()

    features = feature_engine.compute(market_view)

    features_computed = FeaturesComputed(
        step_id=step_id,
        instrument=market_view.instrument,
        features=features,
        derived_from=market_received.event_id,
    )
    record_and_apply([features_computed])

    # ------------------------------------------------------------------
    # 4) STRATEGY / DECISION
    # ------------------------------------------------------------------
    decision_context = build_decision_context(
        market_view=market_store.view(),
        feature_view=feature_store.view(),
        portfolio_view=portfolio_store.view(),
        order_view=order_store.view(),
    )

    order_request = strategy.decide(decision_context)

    if order_request is None:
        step_completed = StepCompleted(
            step_id=step_id,
            status="SUCCESS",
            reason="NO_DECISION",
        )
        record_and_apply([step_completed])
        return

    order_requested = OrderRequested(
        step_id=step_id,
        request_id=order_request.request_id,
        payload=order_request,
    )
    record_and_apply([order_requested])

    # ------------------------------------------------------------------
    # 5) ORDER CONSTRUCTION
    # ------------------------------------------------------------------
    order_spec = order_construction.build(order_request)

    batch_order_created = BatchOrderCreated(
        step_id=step_id,
        request_id=order_request.request_id,
        batch_spec=order_spec,
    )
    record_and_apply([batch_order_created])

    # ------------------------------------------------------------------
    # 6) ORDER CREATION (AGGREGATE)
    # ------------------------------------------------------------------
    order = OrderAggregate.new()

    created_events = order.handle_create(order_spec)
    record_and_apply(created_events)

    # On récupère l'order_id depuis l'event produit
    order_id = extract_order_id(created_events)

    # ------------------------------------------------------------------
    # 7) PRE-TRADE RISK
    # ------------------------------------------------------------------
    # Rehydrate order aggregate from journal / repository
    order = order_repository.load(order_id)

    risk_result = risk_engine.validate(
        order_view=order_store.get(order_id),
        portfolio_view=portfolio_store.view(),
        market_view=market_store.view(),
    )

    if not risk_result.is_valid:
        rejected_events = order.handle_reject(risk_result.reason)
        record_and_apply(rejected_events)

        step_completed = StepCompleted(
            step_id=step_id,
            status="SUCCESS",
            reason="ORDER_REJECTED",
        )
        record_and_apply([step_completed])
        return

    validated_events = order.handle_validate(risk_result)
    record_and_apply(validated_events)

    # ------------------------------------------------------------------
    # 8) ORDER ACTIVATION
    # ------------------------------------------------------------------
    order = order_repository.load(order_id)

    activated_events = order.handle_activate()
    record_and_apply(activated_events)

    # ------------------------------------------------------------------
    # 9) EXECUTION
    # ------------------------------------------------------------------
    active_order = order_store.get(order_id)
    market_view = market_store.view()

    simulated_fill = execution_engine.try_execute(
        order=active_order,
        market_view=market_view,
    )

    if simulated_fill is not None:
        order = order_repository.load(order_id)

        fill_events = order.handle_fill(simulated_fill)
        record_and_apply(fill_events)

        # --------------------------------------------------------------
        # 10) PORTFOLIO UPDATE
        # --------------------------------------------------------------
        portfolio_events = portfolio_service.handle_fill(
            fill=simulated_fill,
            portfolio_view=portfolio_store.view(),
        )
        record_and_apply(portfolio_events)

    # ------------------------------------------------------------------
    # 11) STEP COMPLETE
    # ------------------------------------------------------------------
    step_completed = StepCompleted(
        step_id=step_id,
        status="SUCCESS",
    )
    record_and_apply([step_completed])
```



