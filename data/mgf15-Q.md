

## **NC-1** | Outdated Solidity Version

### **Description** :-

The current Solidity version used in the contract is outdated. Consider using a more recent version for improved features and security.

#### <ins>**Proof Of Concept**</ins>

```solidity
File:2023-11-canto/asD/src/asD.sol
```
```solidity
2:pragma solidity >=0.8.0
```
#### **Context**:-
[asD.sol#L2](https://github.com/code-423n4/2023-10-ethena/blob/main/asD/src/asD.sol#L2)

## **NC-2** | Unused Mapping
### **Description** :-

Consider removing unused mapping or moving it to a higher level contract.

#### <ins>Proof Of Concept</ins>


```solidity
File:2023-11-canto/1155tech-contracts/src/Market.sol
```

```solidity
46:    mapping(uint256 => address) public shareBondingCurves;
```
#### **Context**:-
[Market.sol#L46](https://github.com/code-423n4/2023-10-ethena/blob/main/1155tech-contracts/src/Market.sol#L46)

## **NC-3** | Division Before Multiplication
### **Description** :-

In Solidity division can result in rounding down errors, hence to minimize any rounding errors we always want to perform multiplication before division.

#### <ins>Proof Of Concept</ins>


```solidity
File:2023-11-canto/blob/main/1155tech-contracts/src/bonding_curve/LinearBondingCurve.sol
```

```solidity
23:                fee += (getFee(i) * tokenPrice) / 1e18; //@audit getFee(i) = 1e17 / i;
```

[LinearBondingCurve.sol#23](https://github.com/code-423n4/2023-11-canto/blob/main/1155tech-contracts/src/bonding_curve/LinearBondingCurve.sol#L23)