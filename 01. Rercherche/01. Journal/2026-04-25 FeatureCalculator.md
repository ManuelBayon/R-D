## 1 Responsabilité

- Transformer une vue de marché en une mesure scalaire (`float`), ou `None` pendant la phase de warm-up.
## 2 Hypothèses et préconditions

- `FeatureStore` injecte `market_view: tuple[MarketDataEvent, ...]` à chaque `step`.
- La `Market_view` est fournie par la `Market Data Layer`.
- La `Market_view` est injectée par le `BacktestEngine` dans le `FeatureStore`.
- Le `FeatureStore` a la responsabilité de vérifier la cohérence de la `market_view` avant de l'injecter dans les `FeatureCalculator`.
## 3 Mutation et état

- Le `FeatureCalculator` ne possède pas d'état, il représente une transformation pure : `market_view: tuple[MarketDataEvent, ...] -> float| None
## 4. Contrat API

`FeatureCalculator.calculate(market_view) -> float | None`
## 5. Hors périmètre

- `FeatureCalculator` ne possède pas de `timestamp` courant. La publication horodaté est la responsabilité du `FeatureStore`.
- `FeatureCalculator` ne vérifie PAS la cohérence de la `market_view`.
- `FeatureCalculator` n'est pas supposé stocker son historique, ce dernier est maintenu par le `FeatureStore`.
- `FeatureCalculator` ne produit pas ni d'intentions de trading.
- `FeatureCalculator` ne produit pas d'évènements liés à l'exécution.
- `FeatureCaculator` n'est pas censé avoir un effet de bord.