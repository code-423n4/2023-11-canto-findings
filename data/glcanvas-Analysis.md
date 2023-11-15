# Approach taken in evaluating the codebase
To evaluate codebase was used:
1) Reference to ERC1155, to ensure with corresponding to protocol;
2) Next, was considered Compound token. to ensure, that `asD` token implemented correctly.

Then, I started with evaluation asD folder. Because it's cornerstone of this protocol.
After understanding asD protocol was considered Market protocol.

# Architecture recommendations

Current protocol has straightforward architecture. It's clear and simple.

We have `asDFactory`, which creates `asD` tokens.

There `asD` tokens used in Market to buy ERC1155 tokens.

In this topic I haven't any recommendations.

# Codebase quality analysis

Core is clear, and well documented.
All questionable places marked with commends, like here:
```solidity
// The amount of cNOTE the contract has to hold (based on the current exchange rate which is always increasing) such that it is always possible to receive 1 NOTE when burning 1 asD
        uint256 maximumWithdrawable = (CTokenInterface(cNote).balanceOf(address(this)) * exchangeRate) /
            1e28 -
            totalSupply();
```

However, better to use names constants instead of just numbers. This will be clearer for users to understand what's going on.

# Centralization risks

Current protocol hasn't any restrictions. Owner or admin can't stop whole protocol. It is completely decentralized.

# Mechanism review

Let's start with asD part.

`asDFactory` responsible for creating `asD` tokens. 

These `asD` tokens are pegged to CANTO 1:1. 
Anyone can buy these `asD` tokens use cCANTO (Compound's token). When user buy `asD` he invest his cNOTE to `asD` creator. From this moment user won't make any profit on this `asD`, token creator will receive all rewards.

Then, let's consider 1155 protocol.

Here we have `LinearBondingCurve` which calculates price and fee for buying use formula: `price = priceIncrease * i`, `fee = `1 / log(i) * priceIncrease`.

Market contract is ERC1155 contract.

Market created with one of `asD` tokens.

Market gives possibility to buy and sell tokens.

When user buy or sell tokens he pays a fees (like in usual exchanges, where user pay to trade order independently on order side).

Price amount for token calculated with linear increased equation (LinearBondingCurve).

When user buy or sell token, fees he pay distributed proportional to participants:
1) 33% to Token holders;
2) 33% to Current ERC20 owner;
3) 34% to Protocol owner.

Each participant can withdraw their amount of tokens without any restrictions.

Tokens which user bought he can exchange to NFT. he can call `mintNFT` or `burnNFT` to mint and burn them. 
These NFT can be used on second markets.

# Systemic risks

To work with protocol need to have a bit of cNOTE. These cNOTE will be available till stakeholders has enough liquidity to support whole CANTO blockchain.

So, possible failure point is cNOTE, but it's unlikely. Because if this happens, whole blockchain can be unavailable. 


  


### Time spent:
10 hours