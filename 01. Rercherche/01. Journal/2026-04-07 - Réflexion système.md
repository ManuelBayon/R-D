Question:
La gestion de l'indopotence du timestamp est géré par qui dans la couche FeatureStore ou dans la couche market ou orchestration ?

Le Step context est construit par l'orchestrateur et doit matcher avec l'état interne soit du step doit de chaque couche a voir. On garde `last_processed_sequence` dans le FeatureStore mais à challenger.