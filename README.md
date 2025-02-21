# solidity-posts

Useful intro to UniswapV2 - https://ethereum.org/en/developers/tutorials/uniswap-v2-annotated-code/#introduction

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

To get the average price between two timestamps:

Take two snapshots of the cumulative price
Subtract the earlier value from the later value
Divide by the time elapsed


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

-------------------------------------------------------------
-------------------------------------------------------------

# Uniswap V2 Bugs You should look out for

1. Is token prices recorded for the two token pairs

   For instance, if the price at block 1 is 100 USD/ETH (B being USD, A being ETH) and at block 2 is 300 USD/ETH,
   the average price is 200 USD/ETH, but the average price of ETH/USD would be 1/150 ETH/USD. Since the contract
   cannot know which token users will use as the pricing unit, Uniswap v2 records prices for both tokens.

   See: https://github.com/adshao/publications/blob/master/uniswap/dive-into-uniswap-v2-whitepaper/README.md

2. Users can directly send tokens to the pair contract (changing the token balances and affecting the price) without    a transaction, bypassing the oracle update mechanism

   If the contract simply checks its balances and uses the current balance to update the oracle price, an attacker      could manipulate the oracle price by sending tokens to the contract just before the first transaction of a           block.

   For example:
   
   ```
   Imagine a shop that records the average price of apples throughout the day. They calculate this by looking at how    many apples and dollars they have in their register.

   Normal Scenario:

   Shop has 100 apples and $200
   This means apples cost $2 each
   This price should be recorded for the whole time period until the next real sale

   Attack Scenario:

   Shop has 100 apples and $200 (price is $2)
   Someone sneakily adds 100 more apples to the shop's inventory without buying them
   Now there are 200 apples and $200
   This makes it look like apples cost $1 each
   The shop wrongly records this $1 price for the whole time period
   ```

   To prevent this, the core Uniswap V2 contract caches the token balances after each interaction, updating the         oracle price using the cached (not real-time) balances.

   3. USDT and BNB don't return a value after transfer
      
      The ERC-20 standard requires transfer() and transferFrom() to return a boolean indicating success. However,          some tokens, like USDT and BNB, do not return a value for these methods (or one of them). Uniswap v1 treated         the absence of a return value as a failure, leading to transaction reverts and failures.

      Uniswap v2 addresses non-standard ERC-20 implementations differently. If a transfer() method does not return a       value, Uniswap v2 assumes it indicates success, not failure. This change does not affect tokens adhering to          the standard ERC-20 (as their transfer() methods return a value).

      Also, Uniswap v1 assumed transfer() and transferFrom() could not trigger reentrant calls into the pair               contract. This assumption conflicts with some ERC-20 tokens, including those supporting ERC-777 standard             hooks. To fully support these tokens, Uniswap v2 introduces a "lock" mechanism to solve reentrancy issues for        all publicly mutable state methods, also addressing custom callback reentrancy in flash loans.

      The lock is essentially a Solidity modifier, controlled by an unlock variable, to manage synchronous locking.




------------------------------------------------------------------
------------------------------------------------------------------

# How Flashloan works

```
User wants to swap ETH for USDC
Uniswap sends USDC to user's callback contract first
Callback contract uses USDC however it wants
Callback contract must send ETH back to Uniswap as payment
If ETH payment is correct, swap succeeds; if not, everything reverts
```
A better way to understand flashloan is to think that the receiver of the swap amount is the callback contract - the contract the user is sending the swapped USDC. The callback contract receives USDC on behalf of the user.

Note that the "k" constant must be correct AFTER deducting fees for the service.

Note that aside the example above, users can return the original tokens without performing a swap. This feature enables anyone to flash borrow any amount of tokens from Uniswap pools (flash loan fees are the same as transaction fees, both 0.30%).


# Using Geometric mean in Uniswap V2

What's Geometric mean?

Geometric mean in Uniswap is calculated by multiplying two numbers and taking their square root.

Let's break down the example:

First deposit is:
2 ABC tokens
800 XYZ tokens

Geometric mean = √(2 * 800)
2 * 800 = 1600
√1600 = 40 liquidity tokens

This method is valuable because:

- It captures the true relationship between two different token quantities

- It ensures liquidity providers get fair representation of their deposits

The value can only go up through:

- Trading fees being added to the pool

- Direct token donations to the pool

How can the value only go up through trading fees added to the pool and direct token donations to the pool?

The value of liquidity tokens can only increase through these mechanisms because:

Trading Fees:

- Every trade adds a 0.3% fee to the pool

- These fees accumulate in the pool
  
- The total pool value grows while the number of liquidity tokens stays the same
  
- Therefore, each liquidity token represents a larger share of a growing pool

Direct Token Donations:

- When tokens are sent directly to the pool without minting new liquidity tokens

- The pool's total value increases

- The same number of liquidity tokens now represent more underlying tokens

- Each liquidity token becomes more valuable

Note the word "value". The amount of liquidity tokens shows the percentage of your shares in a liquidity of a pool. Even if a liquidity pool increases, it doesn't change the percentage you hold, that is, your liquidity tokens.

**What determines the amount of liquidity to be provided by liquidity providers to a pool?**

The amount of liquidity to be provided by later liquidity providers is determined by the current ratio of tokens in the pool.

For example: If a pool has:

- 10 ETH
- 20,000 USDC
- Price ratio is 1 ETH : 2000 USDC

A new liquidity provider must maintain this exact ratio. They could deposit:

- 1 ETH + 2000 USDC (small position)
- 5 ETH + 10,000 USDC (medium position)
- 20 ETH + 40,000 USDC (large position)

Note that the worth of LP tokens determines the minimum amount of liquidity to be provided. See below.


# First deposit attack on Uniswap V2 (Although prevented)

Note that the smallest unit of Liquidity Pool (LP) token you can get is 1 wei. Now imagine that to get the 1 wei LP token, you have to provide a very large liquidity to the pool. This therefore denies others from proividing liquidity.

Here's a practical example:

```
Step 1: Attacker's Initial Minimum Deposit

Deposits 1 wei ABC and 1 wei XYZ (smallest possible units)
Gets 1 wei of liquidity tokens (√(1 * 1) = 1)
Pool ratio is 1 ABC : 1 XYZ
Step 2: Attack Execution (same transaction)

Transfers 1 million ABC + 1 million XYZ directly to pool
Calls sync() to update balances
Pool now has: 1,000,000.000000000000001 ABC and 1,000,000.000000000000001 XYZ
Still only 1 wei of liquidity tokens exists
Step 3: Impact

That 1 wei liquidity token now represents claim to:
1 million ABC + 1 wei ABC
1 million XYZ + 1 wei XYZ
New providers must deposit proportionally to get even 1 wei of LP token
Example: To get 1 wei of LP token, need to deposit:
1 million ABC + 1 million XYZ
```

The worth of LP tokens determines the minimum amount of liquidity to be provided.

In the attack scenario:

1 wei LP token is worth:

- 1 million ABC + 1 wei ABC
- 1 million XYZ + 1 wei XYZ
- New LPs must provide liquidity proportional to this value to receive even 1 wei of LP token:

- Need to deposit approximately 1 million of each token
- This maintains the pool's ratio and value per LP token



