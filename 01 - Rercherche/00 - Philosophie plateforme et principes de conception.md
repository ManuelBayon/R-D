# 0. Objet du présent document

Ce document définit la philosophie de recherche et développement d'InvestIQ v2.

Il sert :
- De contrat de conception pour tous les développement futurs.
- De filtre décisionnel pour l'architecture, les fonctionnalités et la refactorisations.
- De référence pour les contributeurs, les réviseurs et les futurs responsables de la maintenance.

Le système n'est pas conçu comme un simple moteur de backtest mais comme une plateforme scientifique et opérationnelle permettant de transformer des idées de trading en systèmes de trading déployables, vérifiables et évolutifs.

---
# 1. Vision fondatrice

InvestIQ v2 est conçu comme une infrastructure unifiée de recherche et de trading, pilotée par évènements, dont l'objectif central est le suivant : 

>Transformer les observations de marché en décisions, en exécutions et en résultats financiers d'une manière reproductible, explicable et extensible à travers le temps, les classes d'actifs, les familles de stratégies ainsi que l'échelle organisationnelle.

Cela implique que chaque exécution est traitée comme une expérience scientifique et chaque session en conditions réelles comme un processus opérationnel traçable.

---
# 2. Critères scientifiques

Ce bloc définit si le système produit de la **connaissance exploitable et fiable**, ou seulement des résultats numériques sans valeur expérimentale durable.

## 2.1 Reproductibilité 

**Intention**
Une expérience n'a de valeur scientifique que si elle peut être reproduite dans le temps par un tiers indépendant.

**Sens**
Un résultat n'est pas seulement un chiffre final, mais la conséquence déterministe d'une ensemble de conditions initiales : données, code, configuration et environnement d'exécution. Si l'un de ces éléments n'est pas maitriser, le résultat devient une observation isolée, non vérifiable.

**Exigence pour le système**
Le système doit permettre de reconstruire une exécution passée comme on rejoue une expérience en laboratoire.

## 2.2 Absence de biais méthodologiques

**Intention**  
Garantir que la stratégie ne bénéficie d’aucun avantage artificiel ou informationnel qui n’existerait pas en conditions réelles de marché.

**Sens**  
Un backtest peut produire d’excellentes performances tout en étant scientifiquement invalide s’il repose sur :

- des données futures accessibles implicitement,
- des hypothèses irréalistes d’exécution,
- une sélection ex post des paramètres ou des périodes.

Ce critère vise à séparer la **performance apparente** de la **validité expérimentale**.

**Exigence pour le système**  
L’infrastructure doit imposer des contraintes structurelles sur :

- l’ordre temporel des événements,
- l’accès à l’information,
- les modèles d’exécution et de latence,  
    afin que toute décision soit prise uniquement à partir de ce qui était réellement observable au moment où elle est censée l’avoir été.

## 2.3 Comparabilité expérimentale

**Intention**  
Pouvoir comparer deux stratégies comme on compare deux expériences scientifiques, et non comme deux résultats isolés.

**Sens**  
Comparer des performances n’a de sens que si les conditions expérimentales sont contrôlées.  
Une différence de résultat doit pouvoir être attribuée à une différence de modèle, de logique de décision ou de paramétrisation — pas à une différence cachée dans les données, l’environnement ou les hypothèses d’exécution.

**Exigence pour le système**  
Le système doit fournir un cadre commun d’expérimentation, dans lequel :
- les stratégies partagent les mêmes données,
- les mêmes règles d’exécution,
- les mêmes métriques d’évaluation,  
    afin que les écarts observés aient une interprétation causale claire.

## 2.4 Traçabilité causale

**Intention**  
Ne pas seulement savoir _ce que_ le système a fait, mais _pourquoi_ il l’a fait.

**Sens**  
Une décision de trading est le résultat d’une chaîne causale :  
données → signaux → transformation → règle de décision → ordre → exécution → impact portefeuille.  
Sans visibilité sur cette chaîne, le système devient une boîte noire impossible à auditer, corriger ou améliorer de manière rationnelle.

**Exigence pour le système**  
Chaque action doit pouvoir être reliée explicitement à :
- ses entrées informationnelles,
- ses règles de transformation,
- son contexte de marché et de portefeuille,  
    de façon à reconstruire a posteriori le raisonnement opérationnel du système.

---
# 3. Critères architecturaux

Ce bloc définit si le système constitue une **plateforme structurée et évolutive**, ou simplement un assemblage de composants fonctionnels sans cohérence architecturale globale.
## 3.1 Pipeline unifié

**Intention**  
Garantir que la recherche, la production et l’audit reposent sur un même moteur conceptuel et opérationnel.

**Sens**  
Un système devient fragile lorsque le backtest, le trading en conditions réelles et l’analyse a posteriori utilisent des logiques différentes.  
Dans ce cas, les résultats observés en recherche ne sont plus un modèle fiable de ce qui se produit en production.

Un pipeline unifié impose que chaque décision, qu’elle soit simulée ou réelle, traverse la même chaîne de transformation et d’exécution.

**Exigence pour le système**  
Le cœur de décision et d’orchestration doit être identique pour :

- les exécutions hors ligne (backtests),
- les sessions en temps réel,
- les processus d’audit et de relecture,  
    avec uniquement des adaptateurs en entrée (sources de données) et en sortie (couches d’exécution, persistance, visualisation).

## 3.2 Séparation des responsabilités

**Intention**  
Permettre l’évolution indépendante des dimensions scientifiques, techniques et opérationnelles du système.

**Sens**  
Une stratégie (alpha) exprime une logique de décision, tandis que l’exécution exprime une interaction avec le marché.  
Si ces deux dimensions sont couplées, toute amélioration technique ou opérationnelle devient un risque pour la validité scientifique de la stratégie.

La séparation des responsabilités transforme le système en un ensemble de modules composables plutôt qu’en un flux monolithique.

**Exigence pour le système**  
Les couches suivantes doivent être découplées par des interfaces formelles :

- génération de signaux et logique de décision,
- gestion du portefeuille et du risque,
- modélisation de l’exécution et connectivité marché,
- persistance, monitoring et visualisation.

## 3.3 Typage des artefacts

**Intention**  
Faire des objets du système des représentations formelles de faits, et non de simples structures de données manipulables arbitrairement.

**Sens**  
Un prix observé, un signal, un ordre ou un état de portefeuille n’est pas une variable : c’est un **événement ou un fait économique daté et contextualisé**.  
Sans typage clair et sans contraintes d’immutabilité, ces objets peuvent être modifiés, réinterprétés ou réutilisés hors de leur contexte, ce qui détruit la traçabilité et la validité des résultats.

**Exigence pour le système**  
Les artefacts centraux doivent être :

- explicitement typés,
- porteurs de métadonnées temporelles et contextuelles,
- immuables après création,  
    et ne pouvoir être transformés que par des opérateurs ou des modules identifiés et traçables.

## 3.4 Dépendances dirigées

**Intention**  
Maintenir une structure du système compréhensible, analysable et maîtrisable à grande échelle.

**Sens**  
Un graphe de dépendances cyclique rend le comportement du système difficile à raisonner, à tester et à faire évoluer.  
Une architecture saine impose une direction claire des flux : des données vers la décision, de la décision vers l’exécution, et de l’exécution vers l’observation et l’audit.

**Exigence pour le système**  
Les modules doivent être organisés selon un graphe orienté acyclique, dans lequel :

- les couches basses ne dépendent jamais des couches hautes,
- chaque dépendance est explicite, justifiée et documentée,
- le chemin d’une information ou d’une décision peut être suivi sans ambiguïté.

## 3.5 Extensibilité structurelle

**Intention**  
Permettre au système de croître en complexité et en diversité sans remise en cause de son noyau.

**Sens**  
Ajouter un nouvel actif, une nouvelle famille de stratégies ou une nouvelle couche d’exécution ne devrait pas nécessiter de modifier le moteur central.  
Si chaque extension impose un refactoring global, le système n’est pas une plateforme, mais une implémentation figée.

L’extensibilité structurelle repose sur des abstractions stables et des points d’extension formels.

**Exigence pour le système**  
Le noyau doit définir :
- des interfaces pour les sources de données,
- des contrats pour les stratégies,
- des adaptateurs pour l’exécution et la persistance,  
    de telle sorte que toute nouvelle composante puisse être intégrée par composition plutôt que par modification.

---
# 4. Critères opérationnels

Ce bloc définit si le système peut fonctionner de manière fiable, contrôlée et gouvernable dans un environnement soumis à des contraintes de temps réel, de risque financier et de responsabilité réglementaire.
## 4.1 Observabilité en temps réel

**Intention**  
Rendre l’état interne du système visible pendant qu’il agit, et non seulement après coup.

**Sens**  
Un système de trading qui ne peut être observé qu’a posteriori est, en pratique, une boîte noire en production.  
En conditions réelles, les décisions, les latences, les rejets d’ordres et les dérives de comportement doivent être détectés pendant qu’ils se produisent, pas lorsqu’ils ont déjà un impact financier.

L’observabilité ne concerne pas seulement les métriques de performance, mais la **dynamique décisionnelle** du système.

**Exigence pour le système**  
Le système doit exposer en temps réel :

- l’état des flux de données,
- les signaux et décisions en cours,
- les ordres émis, acceptés, rejetés ou en attente,
- l’état du portefeuille et des contraintes de risque,  
    via des interfaces de monitoring, de journalisation structurée et de visualisation temps réel.

## 4.2 Auditabilité

**Intention**  
Permettre à un tiers qualifié de reconstruire et de valider une action passée sans dépendre de l’interprétation de l’opérateur.

**Sens**  
Dans un contexte professionnel ou réglementé, une décision de trading doit pouvoir être expliquée formellement :  
quelles données étaient disponibles, quelles règles ont été appliquées, quels contrôles de risque ont été franchis, et quelles actions ont été déclenchées.

Sans cette capacité, le système est opérationnellement inacceptable, même s’il est techniquement performant.

**Exigence pour le système**  
Chaque trade et chaque transition d’état doivent être associés à :
- une trace temporelle complète,
- les entrées informationnelles correspondantes,
- les règles ou modules décisionnels impliqués,
- les validations et rejets des couches de contrôle et de risque,  
    dans un format persistant, consultable et non modifiable.

## 4.3 Gestion des incidents

**Intention**  
Assurer que les défaillances ne se transforment pas en pertes incontrôlées ou en comportements indéterminés.

**Sens**  
En production, les pannes ne sont pas des anomalies exceptionnelles, mais des événements normaux :  
déconnexion de flux de données, latence réseau, rejet d’ordres, corruption d’état, surcharge système.

La qualité opérationnelle d’une infrastructure se mesure à la manière dont elle **dégrade son comportement** face à ces événements.

**Exigence pour le système**  
Le système doit définir explicitement :
- des états de fonctionnement dégradé,
- des mécanismes de suspension ou de repli (fail-safe),
- des alertes et escalades automatiques,  
    de sorte qu’aucune décision critique ne soit prise dans un état non maîtrisé ou non observable.

## 4.4 Sécurité des runs

**Intention**  
Garantir l’isolation entre les expériences, les stratégies et les sessions opérationnelles.

**Sens**  
Deux exécutions ne doivent jamais partager d’état implicite :  
ni données intermédiaires, ni configurations, ni ressources critiques.  
Sans cette isolation, les résultats deviennent non fiables, et un incident dans une stratégie peut contaminer l’ensemble du système.

Cette propriété est aussi bien scientifique (validité des expériences) qu’opérationnelle (sécurité et stabilité).

**Exigence pour le système**  
Chaque run doit disposer :
- de son espace de configuration propre,
- de ses journaux et de sa persistance isolés,
- de ses ressources d’exécution contrôlées,  
    avec des frontières explicites empêchant toute interaction non intentionnelle entre exécutions parallèles ou successives.

---
# 5. Critères de scalabilité

Ce bloc définit si le système conserve ses propriétés scientifiques, architecturales et opérationnelles lorsque son périmètre, son volume et sa complexité sont multipliés par un ordre de grandeur.
## 5.1 Scalabilité des données

**Intention**  
Permettre au système d’absorber une croissance massive des volumes et des fréquences de données sans dégrader la validité des expériences ni la stabilité opérationnelle.

**Sens**  
Une infrastructure qui fonctionne sur quelques actifs en données journalières peut devenir inopérante lorsqu’elle est exposée à :

- des flux multi-marchés,
- des données haute fréquence,
- des historiques longs et multi-sources.

La scalabilité des données ne concerne pas seulement le stockage ou la bande passante, mais la capacité du système à **maintenir des garanties de traçabilité, de reproductibilité et de causalité** sous charge.

**Exigence pour le système**  
Le système doit supporter :

- des mécanismes de partitionnement et de versionnement des données,
- des empreintes et identifiants de jeux de données,
- des pipelines d’ingestion et de relecture parallélisables,  
    sans compromettre l’ordre temporel des événements ni l’isolement des runs.

## 5.2 Scalabilité de la recherche

**Intention**  
Permettre à plusieurs chercheurs ou équipes d’explorer, tester et comparer des stratégies en parallèle sans interférence ni dérive méthodologique.

**Sens**  
Quand la recherche passe d’un individu à une équipe, les problèmes ne sont plus seulement techniques, mais organisationnels :

- duplication d’expériences,
- résultats non comparables,
- divergences de conventions et d’hypothèses.

La scalabilité de la recherche consiste à transformer un effort individuel en un **processus collectif structuré**.

**Exigence pour le système**  
Le système doit fournir :
- des conventions formelles pour la définition des stratégies et des expériences,
- des mécanismes de gestion de versions et de validation des runs,
- des référentiels communs de métriques et de protocoles expérimentaux,  
    afin que les résultats produits par différentes personnes soient directement comparables et cumulables.

## 5.3 Scalabilité des marchés

**Intention**  
Étendre le périmètre du système à de nouvelles classes d’actifs, de nouveaux marchés et de nouveaux microstructures sans remettre en cause le noyau décisionnel.

**Sens**  
Actions, futures, options, crypto, FX ou produits exotiques obéissent à des règles d’exécution, de liquidité et de contraintes différentes.  
Un système non scalable sur cet axe tend à “figer” ses abstractions autour d’un seul type de marché, ce qui limite sa valeur stratégique à long terme.

**Exigence pour le système**  
Le noyau doit reposer sur :
- des abstractions génériques d’instruments et d’ordres,
- des modèles d’exécution interchangeables,
- des adaptateurs spécifiques par marché ou par venue,  
    de sorte que l’ajout d’un nouveau marché soit une opération d’intégration, et non une refonte architecturale.

## 5.4 Scalabilité des familles de stratégies

**Intention**  
Permettre la coexistence et l’orchestration de stratégies de natures fondamentalement différentes au sein d’un même cadre expérimental et opérationnel.

**Sens**  
Une infrastructure conçue uniquement pour des stratégies directionnelles ne s’adapte pas naturellement à :

- du market making,
- de l’arbitrage statistique ou structurel,
- des stratégies de couverture ou de portage,
- des approches multi-actifs ou multi-horizons.

La valeur d’une plateforme apparaît lorsque ces familles peuvent être développées, testées et déployées sans créer des “sous-systèmes” incompatibles entre eux.

**Exigence pour le système**  
Le système doit définir :

- un contrat abstrait pour la logique de décision,
- des interfaces pour la gestion des positions et du risque multi-stratégies,
- des mécanismes de coordination et de priorisation des ordres,  
    permettant l’intégration de nouvelles familles de stratégies par implémentation d’interfaces, et non par spécialisation du moteur.

---
# 6. Critères organisationnels

Ce bloc définit si le système est capable de structurer et de soutenir une activité collective durable, plutôt que de dépendre des connaissances implicites ou de l’expertise tacite d’un individu.
## 6.1 Onboarding

**Intention**  
Réduire le temps nécessaire pour qu’un nouveau contributeur devienne productif sans compromettre la validité scientifique ou la qualité technique des résultats.

**Sens**  
Un système qui ne peut être utilisé que par ses concepteurs est, par nature, non transmissible.  
L’onboarding mesure la capacité de l’infrastructure à **rendre explicites ses conventions, ses abstractions et ses invariants**, plutôt que de les laisser enfouis dans la tête de ses développeurs ou dans du code non documenté.

**Exigence pour le système**  
Le système doit fournir :
- une documentation normative des concepts, des flux et des interfaces,
- des exemples de runs et de stratégies de référence,
- des outils d’initialisation et de validation automatique des environnements,  
    permettant à un nouveau quant ou ingénieur de produire une expérience reproductible en un temps borné et prévisible.

## 6.2 Standardisation

**Intention**  
Assurer l’homogénéité des pratiques et la comparabilité des résultats à l’échelle de plusieurs équipes.

**Sens**  
Sans standards partagés, chaque équipe développe ses propres conventions de données, de métriques et de validation, ce qui rend les résultats difficiles à agréger, à auditer ou à arbitrer au niveau de la direction ou du risk management.

La standardisation transforme une collection de projets en un **portefeuille de recherche cohérent**.

**Exigence pour le système**  
Le système doit imposer :
- des formats communs pour les artefacts expérimentaux et opérationnels,
- des métriques de performance et de risque normalisées,
- des protocoles de validation et de revue des stratégies,  
    afin que les productions de différentes équipes puissent être comparées sur une base méthodologique identique.
## 6.3 Gouvernance technique 

**Intention**  
Encadrer l’évolution du système pour éviter que les changements techniques ne dégradent la stabilité scientifique, opérationnelle ou organisationnelle.

**Sens**  
Dans une plateforme partagée, une modification d’interface, de contrat ou de noyau peut invalider des stratégies, des expériences ou des processus de production existants.  
Sans mécanisme de décision clair, l’architecture devient le produit d’initiatives locales plutôt que d’une vision collective.

La gouvernance technique formalise la **responsabilité sur l’évolution du système**.

**Exigence pour le système**  
Le système doit définir :
- des processus de proposition et de validation des changements d’interface,
- des mécanismes de versionnement et de dépréciation contrôlée,
- des rôles explicites de responsabilité architecturale,  
    de sorte que toute évolution majeure soit traçable, justifiée et compatible avec les invariants fondamentaux.

---
# 7. Critères business / capital

Ce bloc définit si le système peut être considéré comme un **actif économique stratégique**, et non simplement comme un outil technique.
## 7.1 Confiance

**Intention**  
Fonder la prise de décision financière sur des résultats dont la validité et l’origine sont établies de manière formelle.

**Sens**  
Un PM ou un investisseur ne s’engage pas sur des chiffres, mais sur le **processus qui les produit**.  
La confiance ne vient pas de la performance passée, mais de la certitude que :

- les résultats ne sont pas biaisés,
- les hypothèses sont explicites,
- les risques sont mesurés et contrôlés,
- les erreurs peuvent être détectées et expliquées.

Sans cette base, le capital alloué repose sur une croyance, non sur une évaluation rationnelle.

**Exigence pour le système**  
Le système doit fournir :
- une traçabilité complète des performances et des risques,
- des rapports explicables et auditables,
- des indicateurs de qualité des données et des décisions,  
    permettant à un décideur non technique d’évaluer la solidité du processus, pas seulement ses résultats.

## 7.2 Avantage compétitif

**Intention**  
Transformer la plateforme en un levier stratégique plutôt qu’en une commodité technique.

**Sens**  
Dans un environnement où les données et les modèles sont de plus en plus accessibles, l’avantage ne vient pas seulement de l’idée de stratégie, mais de la **vitesse et de la qualité avec lesquelles elle peut être testée, validée et déployée**.

Un système supérieur permet :

- d’explorer plus d’hypothèses,
- de réduire le temps entre recherche et production,
- de réagir plus vite aux changements de marché,  
    tout en maintenant des garanties de rigueur et de contrôle.

**Exigence pour le système**  
Le système doit offrir :
- des cycles de recherche-déploiement courts et maîtrisés,
- des outils d’automatisation des tests et de la validation,
- une capacité à opérer plusieurs stratégies et marchés en parallèle,  
    sans multiplier la complexité opérationnelle.

## 7.3 Protection des résultats

**Intention**  
Préserver la valeur créée par la recherche et l’ingénierie face aux risques internes et externes.

**Sens**  
Les résultats d’un desk ne sont pas seulement des PnL, mais :

- des modèles,
- des données enrichies,
- des processus,
- des connaissances organisationnelles.

Sans mécanismes de protection, ces actifs peuvent être :

- perdus par erreur ou incident,
- dégradés par des changements non contrôlés,
- ou captés par des tiers.

La protection des résultats vise à transformer la performance en **capital durable**.

**Exigence pour le système**  
Le système doit intégrer :
- des contrôles d’accès et des mécanismes d’autorisation,
- des politiques de sauvegarde et de restauration,
- des procédures de validation avant mise en production,  
    garantissant que les résultats scientifiques et financiers restent intègres, traçables et récupérables dans le temps.

---
# 8. Le test ultime (desk-grade)

Un head of quant ou CTO demanderait :
>**Si je perds toute mon équipe demain, est ce que cette infrastructure me permet de reconstruire l'activité de trading en 6 mois ?**

Si la réponse est **oui** alors mon infra est **capital-grade**.

En une phrase : 
>**Une infrastructure mérite du capital réel si elle peut transformer des idées en décisions financières de manière reproductible, explicable, scalable et indépendante des personnes qui l’ont construite.**




















