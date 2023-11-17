# Surround with unchecked
## Instances
[Market.sol #290](https://github.com/code-423n4/2023-11-canto/blob/main/1155tech-contracts/src/Market.sol#L290)

# Loops can be implemented more efficiently 
## Instances
[LinearBondingCurve.sol #20](https://github.com/code-423n4/2023-11-canto/blob/main/1155tech-contracts/src/bonding_curve/LinearBondingCurve.sol#L20)
## Mitigation
Use the following loop format to save gas
```
uint length = arr.length;

for (uint i; i < length; i++) {

//Operations not effecting the length of the array.

}
```
