Dans le cadre du refactor de mon modèle d'exécution je me suis rendu compte qu'il manquait des identifiants dans les principaux message et également une référence vers le message l'ayant généré.

Messages principaux :

- MarketDataEvent
- Décision
- ExecutionInstruction
- Fill

Du coup je leur ait ajouté un identifiant propre et une référence vers l'identifiant du parent.
Pour les feature je n'ai pas de message formelle étant donné que je met à disposition une vue immuable. Toute fois pour avoir un système déterministe et auditable en backtest et live je dois pouvoir garantir que le jeux de feature de l'étape courante à été calculé avec tel événement, que le déterminisme est garanti par `step_sequence` donc que je ne process pas plusieurs fois la meme sequence et aussi avoir une référence temporelle en tout cas c'est ce qui est ressorti de mes recherches.

Pourquoi avoir 3 attributs qui pourrait vouloir dire la même chose : 
```python
self._last_processed_step_sequence : int = -1  
self._last_processed_timestamp : pd.Timestamp | None = None  
self._last_processed_event_id: str | None = None
```

