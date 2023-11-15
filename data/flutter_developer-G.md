## Summary

### Gas Optimization

no | Issue |Instances||
|-|:-|:-:|:-:|
| [G-01] | | | Main 1155tech contract that is used to buy / sell / create shares |

| [G-02] | 	Linear bonding curve|

| [G-03] | 	Factory for creating application-specific dollars|
 
| [G-04] | 	Application-specific dollar contract| 




## Gas Optimization Description of Codes 


## [G-1] ERC20Votes compatible multi-delegation contract to manage user votings

### Details

```solidity
file: 1155tech-contracts/src/Market.sol
```
  uint256 shareHolderFee = (_fee * HOLDER_CUT_BPS) / 10_000;
        uint256 shareCreatorFee = (_fee * CREATOR_CUT_BPS) / 10_000;
        uint256 platformFee = _fee - shareHolderFee - shareCreatorFee;
        shareData[_id].shareCreatorPool += shareCreatorFee;

```
https://github.com/code-423n4/2023-11-canto/blob/main/1155tech-contracts/src/Market.sol#L285-L288
```




## [G-02] 	Linear bonding curve.

```solidity
file: 1155tech-contracts/src/bonding_curve/LinearBondingCurve.sol
```
   for (uint256 i = shareCount; i < shareCount + amount; i++) {
            uint256 tokenPrice = priceIncrease * i;
            price += tokenPrice;
            fee += (getFee(i) * tokenPrice) / 1e18;
        }
```
https://github.com/code-423n4/2023-11-canto/blob/main/1155tech-contracts/src/bonding_curve/LinearBondingCurve.sol#L20-L24
```


## [G-03] 	Factory for creating application-specific dollars


```solidity
file: asD/src/asDFactory.sol
```

 asD createdToken = new asD(_name, _symbol, msg.sender, cNote, owner());
```
https://github.com/code-423n4/2023-11-canto/blob/main/asD/src/asDFactory.sol#L34
```


### [G-04] 	Application-specific dollar contract

```solidity
file: asD/src/asD.sol
```
   SafeERC20.safeTransferFrom(note, msg.sender, address(this), _amount);

   SafeERC20.safeApprove(note, cNote, _amount);
```
https://github.com/code-423n4/2023-11-canto/blob/main/asD/src/asD.sol#L50-L51
```