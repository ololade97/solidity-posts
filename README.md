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

Starting Pool State:

ETH: 10

USDC: 20,000

k = 200,000

Buying 1 ETH:
```
# Starting: 10 ETH, 20,000 USDC
new_eth = 9
new_usdc = 200000 / 9  # = 22,222 USDC
cost = 22222 - 20000   # = 2,222 USDC for 1 ETH
```

Buying 2 ETH:
```
# Starting: 10 ETH, 20,000 USDC
new_eth = 8
new_usdc = 200000 / 8  # = 25,000 USDC
cost = 25000 - 20000   # = 5,000 USDC for 2 ETH
# Price per ETH = 2,500 USDC
```

Buying 3 ETH:
```
# Starting: 10 ETH, 20,000 USDC
new_eth = 7
new_usdc = 200000 / 7  # = 28,571 USDC
cost = 28571 - 20000   # = 8,571 USDC for 3 ETH
# Price per ETH = 2,857 USDC
```

Buying 4 ETH:
```
# Starting: 10 ETH, 20,000 USDC
new_eth = 6
new_usdc = 200000 / 6  # = 33,333 USDC
cost = 33333 - 20000   # = 13,333 USDC for 4 ETH
# Price per ETH = 3,333 USDC
```

Another way to get constant k is to add x + y:

```
# Pool with stablecoins
usdc = 100000
usdt = 100000
k = usdc + usdt  # = 200,000

# Trading 10,000 USDC to USDT
new_usdc = 90000
new_usdt = 110000
# Sum remains 200,000
```

----------------------------------------------------------
----------------------------------------------------------

# Uniswap V1 Price Manipulation

To understand price manipulation in V1, you must first understand derivative contracts or trades.

A derivative contract is a financial agreement whose value comes from another asset. Let's use clear examples:

```
Simple Leverage Position:

You think ETH price will go up
Instead of buying 1 ETH ($2000)
You open a 5x leverage position with $2000
(Quick note on what leverage poositin is -

5x Leverage means:

You put in $2000
Protocol lends you an extra $8000
Total position = $10,000 (5 times your money)
If ETH goes up 10%:
Regular profit would be $200 (on $2000)
With 5x leverage: $1000 profit (5 times more)
If ETH drops 20%:
You lose your entire $2000 (liquidation))
Now you control 5 ETH worth of position ($10,000)

Options Contract:
You get the right to buy ETH at $2000
Even if ETH price goes to $3000
You can still buy at $2000
Profit = $1000 per ETH

Futures Contract:
Agreement to buy/sell ETH at $2000
On a future date
Regardless of actual price then

These contracts "derive" their value from ETH price movements, hence the name "derivatives". They're used for:

Trading with leverage
Hedging positions
Speculating on price movements
Risk management
```

Let's break this down with a practical example of price manipulation in Uniswap v1. The below example is price manipulation by moving the price up.

```
Initial Pool State & Price Rise:

Pool starts: 100 ETH and 200,000 DAI
Constant k = 100 × 200,000 = 20,000,000

When attacker buys 50 ETH:

New ETH balance = 50 ETH
New DAI balance = 20,000,000/50 = 400,000 DAI
New price = 400,000/50 = 4000 DAI per ETH

User Short Position:

ETH Price: 2000 DAI
To borrow 5 ETH (worth 10,000 DAI)
Deposit 10,000 DAI as collateral
Sells borrowed 5 ETH for 10,000 DAI

When Price Rises to 4000 DAI:

Position value: 5 ETH × 4000 = 20,000 DAI
Collateral: 10,000 DAI
Loss: 10,000 DAI (collateral wiped out)

Attacker's Opportunity:

Buys 10,000 DAI collateral at 20% discount
Pays 8,000 DAI for the collateral
Gets 10,000 DAI worth of value
Profit per position: 2,000 DAI

Multiple Positions:

If 10 similar positions liquidated
Total profit: 20,000 DAI
Minus gas fees and trading costs
All in one transaction

Key Profit Factors:

Liquidation discount rate
Number of short positions affected
Speed of execution
Market depth
```

Price Manipulation DOWN example:

```
Initial State & Price Drop:
Pool: 100 ETH, 200,000 DAI
When attacker sells 50 ETH:
New ETH = 150 ETH
New DAI = 20,000,000/150 = 133,333 DAI
New price = 133,333/150 = 889 DAI per ETH

Long Position Example:
Trader deposits 10,000 DAI as collateral
Borrows additional 40,000 DAI (5x leverage)
Buys 25 ETH at 2000 DAI each
Position value at 889 DAI: 25 ETH × 889 = 22,225 DAI
Loss: 50,000 - 22,225 = 27,775 DAI
Position liquidated as value falls below collateral

Attacker's Profit Path:
Buys liquidated collateral (10,000 DAI) at 20% discount = 8,000 DAI
Buys back 50 ETH from pool at 889 DAI
Pool returns to original state
Profit per liquidation = 2,000 DAI
Multiple liquidations multiply profit

Final Numbers:
Cost to buy back 50 ETH: 44,450 DAI
Original sale of 50 ETH: 66,667 DAI
Trading profit: 22,217 DAI
Plus liquidation profits from discounted collateral
```

