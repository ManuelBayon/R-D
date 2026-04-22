# 1. Video summary

## 1.1 What is and how to create a Feature in InvestIQ.

- A feature is a calculator
- Exposes a UNIQUE key
- reads from `MarketView`
- Returns :
	- None during warmup 
	- A float when enought data is available
- features are stateless, they are pure functions.
- All state is manager by the FeatureStore
## 1.2 How does the calculator enter the backtest

- Features are explicitly declared in the backtest configuration. There is no dynamic discovery.
- At intialisation the `FeatureStore` 
	- Receives the list of calculators
	- Enforces key uniqueness
	- Initializes an empty history for each declared feature
## 1.3 What happens at each step

At each market step (timestamp t) :
1. The `FeatureStore` receives a `MarketView(t)`
2. It iterates over all the calculators
For each calculator :
- Ensures the feature is not published twice at the same step
- computes value
Then:
- if value is None -> no publication (warm-up)
- if value is float:
	- Create a `FeaturePoint (timestamp=t, value=value)`
	- append to the feature history
Finally:
- the timestamp is recorded
- re-processing raises a Value Error.
## 1.4 How the strategy reads features

- The strategy does not access the FeatureStore directly. It receives a `FeatureView`, which is immutable.
- Through this view, the strategy can:
	- check if a feature is ready (`is_ready`)
	- access the latest value (`require`)
	- access the latest point (value + timestamp)
- This enforces a strict separation:
	- FeatureStore = mutation + state ownership
	- FeatureView = read-only access

## 1.5 Core invariants

- Keys are unique
- A timestamp is processed at most once
- A feature may no publish at every step so the timestamp is given with the value.
- Strategies cannot mutate feature state
- All features at a given step are computed from the same market snapshot.

