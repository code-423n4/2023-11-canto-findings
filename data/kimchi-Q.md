### LOW: Interface name not compliant with the standard.

### Description:
The Turnstile interface, even though it is an interface, is named exactly like a contract.
```solidity
import {Turnstile} from "../interface/Turnstile.sol";
```

```solidity
// SPDX-License-Identifier: GPL-3.0-only
pragma solidity >=0.8.0;
interface Turnstile {
    function register(address) external returns(uint256);
    function assign(uint256) external returns(uint256);
}
```

### Recommendation:
Change the name of the interface from `Turnstile` to `ITurnstile`.

### Affected lines:
https://github.com/code-423n4/2023-11-canto/blob/main/asD/src/asDFactory.sol#L4
https://github.com/code-423n4/2023-11-canto/blob/main/asD/src/asD.sol#L4
https://github.com/code-423n4/2023-11-canto/blob/main/1155tech-contracts/src/Market.sol#L8

### LOW: Misleading comment about share creation

### Description:
The comment from line L60 suggests that only whitelisted addresses can create shares while `shareCreationRestricted = true;`.
```solidity
    /// @notice If true, only the whitelisted addresses can create shares
    bool public shareCreationRestricted = true;
```
However, according to the implementation of modifier `onlyShareCreator()`, in addition to whitelisted addresses, the owner can also create shares.

```solidity
    modifier onlyShareCreator() {
        require(
            !shareCreationRestricted || whitelistedShareCreators[msg.sender] || msg.sender == owner(),
            "Not allowed"
        );
        _;
    }
```
and `createNewShare` function doesn't have any other restrictions for the owner:
```solidity
    function createNewShare(
        string memory _shareName,
        address _bondingCurve,
        string memory _metadataURI
    ) external onlyShareCreator returns (uint256 id) {
        require(whitelistedBondingCurves[_bondingCurve], "Bonding curve not whitelisted");
        require(shareIDs[_shareName] == 0, "Share already exists");
        id = ++shareCount;//@Q ID inkrementalne (polygon reorg)
        shareIDs[_shareName] = id;
        shareData[id].bondingCurve = _bondingCurve;
        shareData[id].creator = msg.sender;
        shareData[id].metadataURI = _metadataURI;
        emit ShareCreated(id, _shareName, _bondingCurve, msg.sender);
    }
```

### Recommendation:
Correct your comment to the following
```solidity
/// @notice If true, only the whitelisted addresses and owner can create shares
```

### Affected lines:
https://github.com/code-423n4/2023-11-canto/blob/main/1155tech-contracts/src/Market.sol#L60

### LOW: Possibility to bypass blocking/exclusion by the owner

### Description:
The `ShareCreator` can front-run transaction where the owner tries to disable bonding curve from use by calling the 'changeBondingCurveAllowed' function and use function `createNewShare` to create new share with the no yet blocked bonding curve.

A very similar case will be for functions: `restrictShareCreation` - where if there was no restriction and it is going to change, you can create a share just before blocking this possibility, `changeShareCreatorWhitelist` - where if a person was on the whitelist and the owner decided to exclude them, they can still create a share just before being excluded.

### Recommendation:
Enforce the usage of a private mempool service for all admin actions.

### Affected lines:
https://github.com/code-423n4/2023-11-canto/blob/main/1155tech-contracts/src/Market.sol#L104-L108
https://github.com/code-423n4/2023-11-canto/blob/main/1155tech-contracts/src/Market.sol#L114-L127
https://github.com/code-423n4/2023-11-canto/blob/main/1155tech-contracts/src/Market.sol#L300-L304
https://github.com/code-423n4/2023-11-canto/blob/main/1155tech-contracts/src/Market.sol#L309-L312

### Modifier onlyShareCreator() is misleading

### Description:
The name of the `onlyShareCreator()` modifier suggests that only whitelisted share creators will be allowed to call functions with this modifier. However, this is not the case. The owner() can also call such a function, and any other user can too, if share creation is not restricted by `shareCreationRestricted`.

```solidity
    modifier onlyShareCreator() {
        require(
            !shareCreationRestricted || whitelistedShareCreators[msg.sender] || msg.sender == owner(),
            "Not allowed"
        );
        _;
    }
```

### Recommendation:
Instead of using a modifier, use an `if` statements in the `createNewShare` function and custom errors, to clearly express your intention.

### Affected lines:
https://github.com/code-423n4/2023-11-canto/blob/main/1155tech-contracts/src/Market.sol#L80-L86