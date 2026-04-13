Métriques exactes à implémenter dans ton moteur pour détecter les problemes entre PnL théorique et PnL réel. 

---
# 1) PnL théorique vs PnL exécuté

C’est la base.

## A. PnL théorique

Hypothèse idéale :

- fill parfait
- pas de latence
- pas de slippage
- éventuellement pas de fees

## B. PnL simulé / exécuté

Avec :

- ton vrai fill model
- gaps
- règles d’exécution
- fees

## Ce que tu mesures

- `pnl_theoretical`
- `pnl_executed`
- `execution_drag = pnl_theoretical - pnl_executed`

👉 si l’écart est grand, tu sais déjà que **l’exécution détruit une partie de l’edge**

---

# 2) Slippage

Indispensable.

## À mesurer par ordre / par fill

- `expected_price`
- `fill_price`
- `slippage_abs = fill_price - expected_price`
- `slippage_bps`

Selon le side :

- buy : plus haut = défavorable
- sell : plus bas = défavorable

## Agrégations utiles

- slippage moyen
- slippage médian
- slippage p95
- slippage par type d’ordre :
    - market
    - limit
    - stop

👉 ça te permet de dire :

> “le signal est bon, mais le coût d’exécution le tue”

---

# 3) Fill rate

Très important pour éviter l’illusion.

## Mesures

- `orders_sent`
- `orders_filled`
- `orders_partially_filled`
- `fill_rate = filled / sent`

Et par type d’ordre :

- market fill rate
- limit fill rate
- stop trigger rate

👉 un alpha peut sembler excellent si tu supposes que tout est exécuté, alors qu’en vrai **les limits ne passent presque jamais**

---

# 4) Temps de vie des ordres

Très utile.

## Mesures

- temps entre `OrderCreated` et `OrderActivated`
- temps entre `OrderActivated` et premier fill
- temps entre `OrderActivated` et terminal state (`FILLED`, `CANCELLED`, `EXPIRED`)

Variables :

- `time_to_activate`
- `time_to_first_fill`
- `time_to_terminal`

👉 ça t’aide à voir :

- ordres trop lents
- ordres qui traînent
- stratégie trop sensible au timing

---

# 5) Taux de rejet / d’annulation / d’expiration

Il faut mesurer les pertes “avant marché”.

## Mesures

- `rejection_rate`
- `cancel_rate`
- `expiry_rate`

Et les raisons :

- risque
- invalid spec
- non déclenché
- pas touché

👉 ça te permet de dire :

> “le problème n’est pas le modèle, le problème est qu’une partie trop grande des ordres n’arrive jamais au marché”

---

# 6) Conversion funnel du signal

Très puissant.

Tu suis la chaîne :

signal  
→ order request  
→ order created  
→ order validated  
→ order activated  
→ order filled  
→ pnl

## Compteurs

- `signals_generated`
- `order_requests_generated`
- `orders_created`
- `orders_validated`
- `orders_activated`
- `orders_filled`

## Ratios

- signal → request
- request → created
- created → validated
- validated → activated
- activated → filled

👉 c’est probablement une des métriques les plus importantes pour toi

---

# 7) Qualité du signal avant / après exécution

Là tu commences à connecter modèle et réel.

## Mesures

- performance du signal en hypothèse idéale
- performance du signal avec exécution simulée
- performance par régime de marché

Par exemple :

- `signal_edge_raw`
- `signal_edge_after_execution`

👉 si l’écart est énorme :

> le modèle n’est peut-être pas faux, mais il n’est pas exploitable

---

# 8) Sensibilité aux coûts

Très desk-like.

Tu testes la robustesse du modèle à :

- fees
- slippage
- délai

## Exemple

Rejouer la stratégie avec :

- 0 bps
- 1 bp
- 3 bps
- 5 bps
- 10 bps

Et voir :

- Sharpe
- PnL
- drawdown

👉 ça permet de dire :

> “l’alpha existe uniquement dans un monde sans friction”  
> ou  
> “l’edge survit jusqu’à 5 bps”

---

# 9) Latence simulée

Même en bar-based, tu peux introduire une forme de délai.

## Mesures

- délai décision → activation
- délai activation → exécution possible
- impact du délai sur le PnL

Variables :

- `decision_latency_steps`
- `activation_latency_steps`

👉 si 1 step de retard détruit tout :

> ton signal est fragile

---

# 10) Cohérence et déterminisme

Très important pour ton architecture.

## Mesures / checks

- même journal → même état final
- même journal → même PnL
- aucune divergence de replay

Tu peux exposer :

- `replay_consistent = True/False`
- nombre de divergences
- premier event divergent

👉 ça ne parle pas directement d’alpha, mais ça parle de **fiabilité de la machine**

---

# 11) Métriques portefeuille utiles

Pour relier exécution et résultat.

## Mesures

- PnL net
- max drawdown
- average trade
- expectancy
- profit factor
- turnover

## Plus utile encore

- PnL expliqué par :
    - alpha
    - coûts
    - timing
    - stop / take profit behavior

---

# 12) Le tableau minimal à implémenter maintenant

Si tu veux rester simple et pro, commence par ça :

## Must-have

- `pnl_theoretical`
- `pnl_executed`
- `execution_drag`
- `avg_slippage_bps`
- `fill_rate`
- `rejection_rate`
- `cancel_rate`
- `time_to_fill`
- `replay_consistent`

👉 avec seulement ça, tu peux déjà apprendre énormément

---

# 13) Comment le brancher dans ton système

Ton architecture s’y prête très bien.

## Via le journal

À partir des events :

- `OrderCreated`
- `OrderValidated`
- `OrderActivated`
- `OrderFilled`
- `OrderRejected`
- `PortfolioTransition`

Tu peux dériver :

- durées
- ratios
- funnel
- slippage
- impact PnL

👉 c’est exactement la force de ton architecture event-centric

---

# 14) Ce que tu pourras dire plus tard

Avec ces métriques, tu pourras dire proprement :

- “le modèle a un edge brut, mais il est mangé par 4 bps de slippage”
- “la moitié des ordres limites ne sont jamais fill”
- “un step de délai détruit la stratégie”
- “le problème vient du funnel request → fill, pas du signal”

Là, tu ne récites plus.  
Tu diagnoses.

---
# 15) Ordre de mise en place conseillé

## Phase 1

- PnL théorique / exécuté
- slippage
- fill rate

## Phase 2

- funnel complet
- temps de vie des ordres
- reject / cancel / expiry

## Phase 3

- sensibilité aux coûts
- latence simulée
- décomposition de la performance