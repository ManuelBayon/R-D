Hyopthèse : 
1. Cette méthode dupplique les reponsabilités. 
2. Le nom fifo_operations devrait s'appeler `Fills` et on applique ces dernier au portefeuille.

 ```python
 def process(  
        self,  
        decision: Decision,  
        portfolio_view: PortfolioView,  
) -> list[FIFOOperation]:  
  
    # 1. Build context  
    key = compute_key(  
        current_position=portfolio_view.current_position,  
        target_position=decision.target_position  
    )  
    # 2. Get the transition_engine rule and resolve transition_engine  
    rule: TransitionRule = self._transition_rule_factory.create(key=key)  
    transition_type: TransitionType = rule.classify(  
        current_position=portfolio_view.current_position,  
        target_position=decision.target_position  
    )  
    # 3. Get the transition_engine strategy and resolve the atomic actions  
    strategy : TransitionStrategy = self._transition_strategy_factory.create(  
        transition_type=transition_type  
    )  
    atomic_actions: list[AtomicAction] = strategy.resolve(  
        current_position=portfolio_view.current_position,  
        target_position=decision.target_position,  
        timestamp=decision.timestamp  
    )  
    # 4. Resolve fifo operations  
    fifo_operations: list[FIFOOperation] = self._fifo_resolver.resolve(  
        actions=atomic_actions,  
        position_book=portfolio_view.fifo_book,  
        execution_price=decision.execution_price
 ```

# 1. Étape 1 : Build context

A l'étape on fait rien d'utile on fait juste un mapping numérique (current, target) -> Enum (state, event) donc inutile.

# 2. Étape 2 : Get transition rule and and get transition type

Potentiellement inutile

# 3. Étape 3 : Get the transition strategy and resolve the atomic actions

Créer des actions atomique important et potentiel point d'entrée du TransitionEngine en passant `current_position` et  `target_position` pour récupérer la stratégie de transition qui produit des actions atomiques.

# 4. Etape 4 : Resolve fifo operations

Avec plus de recul je sais mtn que ce qui se passe la enfaite au niveau métier c'est la production d'un `Fill` simulé par rapport aux FIFO. 






