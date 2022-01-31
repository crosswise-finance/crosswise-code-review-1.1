# Source Core Review - CrossWise Router contract

As the router contract extends the proven Pancakeswap Router contracts, we focus on the extension.

<p align="center">
  <img src=".\_images\router-extension-model.PNG" width="1280" title="hover text">
</p>

## 0. General

### 0.1 The position of router, in general
- The adding/removing liquidity to/from liquidity pools and swapping one token for another, necessarily introduce the concepts of pair, pair factory, and router.
- The conceptual structure of pair, pair factory, and router was proven, and we don't need to create another structure.
- According to the pair-factory-router architecture:
  - Factory is responsible for creating, authentication of, and tracking inventory of token pairs, which is a liquidity pool.
  - Pair is responsible for 
    - managing token reserves via receiving and sending, 
    - ensuring the consistency of liquidity constant K, 
    - maintaining the price oracle,
  - Router is responsible for
    - Routing the requests of addLiquidity, swap, and their variations, to proper pairs
    - controlling token input amounts to pairs

### 0.2 The position that the CrossWise router takes
The CrossWise router extended the conventional router with the following functions:
- setWhitelistToken
- pausePriceGuard
- setMaxSpreadTolerance
- setAntiWhale

These functions belong to the category of input data to pairs, conforming to the architecture.
So, architecture has by and large no problem.


## 1. Finance

### 1.1 admin and priceConsumer
- Logic: The admin and priceConsumer is set when the contract is deployed.
  - the admin has the privilege to setWhilelistToken(.)
  - The priceConsumer contract is used to getLatestPrice(), maybe from the off-chain side.

### 1.2 setwhitelistToken(.) function
- Logic: The admin can whitelist a token. Only whitelisted tokens can attend to a pair.
- Comment: Do we need this regulation?

### 1.3 pausePriceGuard(.) function
- Logic: The creator of a pair can pause or resume the price guard on the pair.
- Comment: This could, alternatively, be implemented by the price tolerance.

### 1.4 setMaxSpreadTolerance(.) function
- Logic: The creator of a pair can change the price tolerance of the pair.
- Comment:

### 1.5 setAntiWhale(.) funciton
- Logic: The creator of a pair can change if the anti-whale check will be active or not.
- Comment:

### 1.6 The antiWhale and verifyPrice checks
- Logic: All and only swap calls are subject to the antiWhale and veryfyPrice checks before every swap action.
- Comment: antiWhale checks the amount of transfer while verifyPrice checks the price.

### 1.7 The antiWhale check
- Logic: ...
- **Issue**: There is a **terrible error** in this code. The coder does **NOT** understand how to use a pair contract.
  The code fragment comes below:
  ```math
    function antiWhale(address[] memory path, uint amountIn) internal view {
        for (uint256 i = 0; i < path.length - 1; i++) {
            ICrosswisePair pair = ICrosswisePair(
                CrosswiseLibrary.pairFor(factory, path[i], path[i + 1])
            );

            if (antiWhalePerLp[address(pair)]) {
                (uint reserve0, uint reserve1,) = pair.getReserves();
                (reserve0, reserve1) =
                    path[i] == pair.token0() ?
                    (reserve0, reserve1) :
                    (reserve1, reserve0);
                uint maxTransferAmount = (reserve0 * maxTransferAmountRate) / maxShare;
                require(amountIn <= maxTransferAmount, "CrssRouter.antiWhale: Transfer amount exceeds the maxTransferAmount");
                // The coder thought 'reserve0' was the input side of a pair
                // The coder thought 'amountIn' entered every pair on the path
            }
        }
    }
  ```

- **Comment**: This error can be hidden only if the value scale of tokens are similar in a pair.
  eg: if the crss and busd have the similar price, then this error does not surface.

### 1.8 The verifyPrice check
- Logic: ... 

## 2. Security


## 3. Programming