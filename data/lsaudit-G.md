# `asD.sol`

## Linked Compound docs demonstrates, that there's no need to declare redundant `returnCode` variable.
Cheking returnCode can be done directly in `require`, e.g.: 

```
https://docs.compound.finance/v2/ctokens/#redeem-underlying

require(cToken.redeemUnderlying(50) == 0, "something went wrong");
```

Remove `returnCode` from `mint()`, `burn()`, `withdrawCarry()`:

```
require(cNoteToken.mint(_amount) == 0, "Error when minting");
(...)
require(cNoteToken.redeemUnderlying(_amount) == 0, "Error when redeeming");
(...)
require( CErc20Interface(cNote).redeemUnderlying(_amount) == 0, "Error when redeeming");
```

## Redundant variables `note` and `exchangeRate` in `withdrawCarry()`

There's no need to declare those variables, since they are used only once, code can be changed to:

```
        uint256 maximumWithdrawable = (CTokenInterface(cNote).balanceOf(address(this)) * CTokenInterface(cNote).exchangeRateCurrent()) /
            1e28 -
            totalSupply();
```

```
SafeERC20.safeTransfer(IERC20(CErc20Interface(cNote).underlying()), msg.sender, _amount);
```


# `LinearBondingCurve.sol`

## Cache `shareCount + amount` in variable, instead of calculating it on every loop iteration in `getPriceAndFee()`:

```
uint256 shareCoundPlusAmount = shareCount + amount;
for (uint256 i = shareCount; i < shareCoundPlusAmount; i++) {
```


## Do-while loops use less gas than for-loop

`for` loop in `getPriceAndFee()` can be changed to do-while

## Move `tokenPrice` variable declaration outside the loop in `getPriceAndFee()`

A quick test in Remix-IDE was made:

```
  function AAA() public view {
    uint gas = gasleft();

        for (uint i = 0; i< 100; i++){
        uint x = i;
    }
        console.log(gas - gasleft());
    }

    function BBB() public view {
        uint gas = gasleft();
        uint x;
        for (uint i = 0; i< 100; i++){
         x = i;
    }
        console.log(gas - gasleft());
    }
```


`AAA()` uses 6841 gas. `BBB()` uses 6451 gas. This implies, that it's better to declare variable outside the loop:

```
 uint256 tokenPrice;
 for (uint256 i = shareCount; i < shareCount + amount; i++) {
            tokenPrice  = priceIncrease * i;
            price += tokenPrice;
            fee += (getFee(i) * tokenPrice) / 1e18;
        }
```

## `getFee()` can be rewritten

Function can return earlier, there's also no need to declare `divisor` variable. We can reduce the number of divisions (when `divisor == 1`, there's no need to perform `1e17 / divisor`):


```
function getFee(uint256 shareCount) public pure override returns (uint256) {
    if (shareCount > 1) {
        return 1e17 / log2(shareCount);

    } else {
        return 1e17;
    }
    
    
}
```


# Market.sol

## Optimize `onlyShareCreator()` modifier

This modifier checks if share creation is enabled, and if not - it allows to create share only to whitelisted addresses or the owner:
```
whitelistedShareCreators[msg.sender] || msg.sender == owner()
```

Consider, in `constructor()`, adding `owner()` to `whitelistedShareCreators`. If `owner()` will be whitelisted, there'll be no need to perform that additional OR condition: `|| msg.sender == owner()`.

## `++shareCount;` can be unchecked in `createNewShare()`

It's impossible that number of shares reach max uint256 value. Thus it can be unchecked.


## Redundant `bondigCurve` variable in `getBuyPrice()`,  `getSellPrice()`, `getNFTMintingPrice()`

`bondingCurve` is used only once, so we don't need to declare it at all. Code can be changed to:

```
(price, fee) = IBondingCurve(shareData[_id].bondingCurve).getPriceAndFee(shareData[_id].tokenCount + 1, _amount);
(...)
(price, fee) = IBondingCurve(shareData[_id].bondingCurve).getPriceAndFee(shareData[_id].tokenCount - _amount + 1, _amount);
(...)
(uint256 priceForOne, ) = IBondingCurve(shareData[_id].bondingCurve).getPriceAndFee(shareData[_id].tokenCount, 1);
```

## `claimCreatorFee()` and `claimPlatformFee()` - perform `safeTransfer()` only when amount > 0.
There's missing check for `amount > 0` in `claimCreatorFee()` and `claimPlatformFee()`.
Whenever `amount` is 0, it's a waste of gas to perform `safeTransfer()`. 
It's better to verify if `amount > 0` before performing transfer - the same way how it's done in `claimHolderFee()`.


## Reorganize lines in `sell()`

```
184:	tokensByAddress[_id][msg.sender] -= _amount; // Would underflow if user did not have enough tokens
```

Whenever `amount > tokensByAddress[_id][msg.sender]` above line will revert. This revert protects from selling more than `msg.sender` has.
It would be good idea to move this line higher. Whenever `sell()` is called with `amount` greater than `tokensByAddress[_id][msg.sender]`, function `sell()` will revert earlier, without wasting gas on other operations
