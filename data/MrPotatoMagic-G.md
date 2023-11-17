# Gas Optimizations Report

| Gas Optimizations | Issues                                                                    | Instances |
|-------------------|---------------------------------------------------------------------------|-----------|
| [G-01]            | Remove else block to save gas                                             | 1         |
| [G-02]            | Remove redundant mapping shareBondingCurves to save gas                   | 1         |
| [G-03]            | Parameter can be made calldata to save gas                                | 1         |
| [G-04]            | Remove redundant check `shareData[_id].creator != msg.sender` to save gas | 1         |

### Total Deployment cost: 31813 gas saved
### Total Function execution cost: 624 gas saved (per all calls)
### Total Gas saved: 32437 gas

## [G-01] Remove else block to save gas

[Link to instance](https://github.com/code-423n4/2023-11-canto/blob/486d0723d686964a6dacc93ae0e1876705b4aa6b/1155tech-contracts/src/bonding_curve/LinearBondingCurve.sol#L31C1-L33C10)

**Function execution cost: 860 - 848 = 12 gas saved (per call)**

Instead of this:
```solidity
File: LinearBondingCurve.sol
27: 
28:     function getFee(uint256 shareCount) public pure override returns (uint256) {
29:         uint256 divisor;
30:         if (shareCount > 1) {
31:             divisor = log2(shareCount);
32:         } else {
33:             divisor = 1;
34:         } 
35:         // 0.1 / log2(shareCount)
36:         return 1e17 / divisor;
37:     }
```
Use this:
```solidity
File: LinearBondingCurve.sol
27: 
28:     function getFee(uint256 shareCount) public pure override returns (uint256) {
29:         uint256 divisor = 1;
30:         if (shareCount > 1) {
31:             divisor = log2(shareCount);
32:         }
33:         // 0.1 / log2(shareCount)
34:         return 1e17 / divisor;
35:     }
```

## [G-02] Remove redundant mapping shareBondingCurves to save gas

[Link to instance](https://github.com/code-423n4/2023-11-canto/blob/335930cd53cf9a137504a57f1215be52c6d67cb3/1155tech-contracts/src/Market.sol#L46)

The mapping shareBondingCurves is not used anywhere in the contract and can thus be removed.

**Deployment cost: 2499354 - 2487364 = 11990 gas saved**

Remove the mapping shareBondingCurves below:
```solidity
File: Market.sol
45:     /// @notice Stores the bonding curve per share
46:     mapping(uint256 => address) public shareBondingCurves;
```

## [G-03] Parameter can be made calldata to save gas

[Link to instance](https://github.com/code-423n4/2023-11-canto/blob/335930cd53cf9a137504a57f1215be52c6d67cb3/1155tech-contracts/src/Market.sol#L115C1-L117C35)

Parameters that are only read can be marked calldata.

**Function execution cost: 10457 - 10068 = 389 gas saved (per call)**

Instead of this:
```solidity
File: Market.sol
115:     function createNewShare(
116:         string memory _shareName,
117:         address _bondingCurve,
118:         string memory _metadataURI
```
Use this:
```solidity
File: Market.sol
115:     function createNewShare(
116:         string calldata _shareName,
117:         address _bondingCurve,
118:         string calldata _metadataURI
```

## [G-04] Remove redundant check `shareData[_id].creator != msg.sender` to save gas

[Link to instance](https://github.com/code-423n4/2023-11-canto/blob/335930cd53cf9a137504a57f1215be52c6d67cb3/1155tech-contracts/src/Market.sol#L151)

The check `require(shareData[_id].creator != msg.sender, "Creator cannot buy");` on Line 151 can be circumvented by the creator by using another address as mentioned in the README's publicly known issues section. This check has no use if it can be circumvented and should be removed.

**Deployment cost: 2499354 - 2479531 = 19823 gas saved**

**Function execution cost: 157773 - 157550 = 223 gas saved (per call)**

Remove the check shown below:
```solidity
File: Market.sol
151:  require(shareData[_id].creator != msg.sender, "Creator cannot buy");
```