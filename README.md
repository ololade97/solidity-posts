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

2. Stable Price Reference
   
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



-----------------------------------------------
-----------------------------------------------

Constant k in AMM

x * y = k

What the above mean is that the two pair tokens of a pool must always be equal to k. 

Regardless of the money taken out or sent to a pool, the k must always be equal to the initial amount the pool was started with or the total balance of liquidity supplied to a pool.

For example:

Assume there is ETH/USDC pool

```
# Initial state
initial_usdc = 20000
initial_eth = 10
initial_price = 2000  # (20000/10) USDC per ETH

# After buying 1 ETH
final_usdc = 22222
final_eth = 9
# Price for this specific trade:
trade_price = 2222  # (22222 - 20000) USDC for 1 ETH - That is, amount paid by user who bought 1 ETH

# New pool price
new_price = 2469  # (22222/9) USDC per ETH
```

Proof that k remains 200000:

```
# Initial K
initial_k = 10 * 20000  # = 200,000

# After Trade
new_eth = 9
new_usdc = 200000 / 9  # = 22,222
new_k = 9 * 22222      # = 200,000

# K remains constant: 200,000 = 200,000
```

