## Invariants

- chaque `event_id` présent dans le journal est unique
- le journal est append-only (immuable)
- chaque évènement est relié à l'évènement qui l'a causé sauf `MarketDataReceived`.