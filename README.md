# solidity-posts

TWAP

Time-weighted average price (TWAP) in Uniswap is a pricing mechanism that calculates the average price of a token pair over a specific time period.

Key aspects of TWAP:

1. Price Accumulator
   
Tracks cumulative prices at each block

Updates with each swap in the pool

Stores price * time since last update

3. Calculation Method
   
Takes two price-time observations

Subtracts earlier from later observation

Divides by time elapsed between observations

Here's a practical example of TWAP for ETH/USDC pair:

Let's say we want to calculate the 24-hour TWAP:

Time Points:
```
T1 (24 hours ago): ETH = $2000
T2 (18 hours ago): ETH = $2200
T3 (12 hours ago): ETH = $1800
T4 (6 hours ago): ETH = $2100
T5 (now): ETH = $2300

TWAP Calculation:

Period 1: $2000 × 6 hours = $12,000
Period 2: $2200 × 6 hours = $13,200
Period 3: $1800 × 6 hours = $10,800
Period 4: $2100 × 6 hours = $12,600

Total = $48,600
TWAP = $48,600 ÷ 24 hours = $2,025
```

This $2,025 represents the true average price weighted by time, making it resistant to short-term price spikes or manipulation.

Real-world uses:

DeFi lending protocols determining collateral values

Options protocols setting strike prices

Stablecoin mechanisms maintaining pegs

TWAP serves two main purposes:

1. Protection Against Price Manipulation
   
Instead of using current spot price which can be manipulated

Uses average price over time (like 24 hours)

Makes it expensive for attackers to manipulate prices

3. Stable Price Reference
   
Smooths out temporary price spikes

Provides reliable price feed for other protocols

Better than instant spot prices for important decisions

How does TWAP protect against manipulation?

TWAP effectively protects against manipulation by requiring significant capital over an extended time period to influence the price!

Here's why it works:

To manipulate a 24-hour TWAP, an attacker would need to:

Maintain artificial prices for many hours

Deploy massive capital

Pay high trading fees

Fight against market forces

Example:

Spot Price: Can be manipulated with one large trade

TWAP: Needs continuous large trades for hours

Cost difference: Spot ($1M) vs TWAP ($24M+ over 24 hours)

Real protection:

Flash loan attacks become ineffective

Price spikes get smoothed out

Short-term manipulation attempts fail

Market stability increases

This makes TWAP an excellent security mechanism for DeFi protocols that need reliable price feeds for critical operations like lending, liquidations, and pricing.
