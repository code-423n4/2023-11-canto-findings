# QA Report

## Summary

| id | title |
| --- | --- |
| [L-01](#l-01-share-creation-can-be-front-run) | Share creation can be front run |
| [R-01](#r-01-avoid-negations) | Avoid negations |
| [NC-01](#nc-01-_tokencount-passed-to-_splitfees-is-always-sharedata_idtokensincirculation) | `_tokenCount` passed to `_splitFees` is always `shareData[_id].tokensInCirculation` |
| [NC-02](#nc-02-testfail-todo-explained) | `testFail` TODO explained |

## Low

### L-01 Share creation can be front run

[`Market::createNewShare`](https://github.com/code-423n4/2023-11-canto/blob/main/1155tech-contracts/src/Market.sol#L120):
```solidity
File: 1155tech-contracts/src/Market.sol

120:        require(shareIDs[_shareName] == 0, "Share already exists");
```

This can allow for a malicious user to front run someones creation of a share with the same name. If this is a SocialFi username or a rally or something similar the name can be very important.

It is possible the victim might not notice the `creator` address is different.

#### Recommendations
Consider using `msg.sender` as a part of the uniqueness for the names as well. That way the account is tied to the name.

## Recommendations

### R-01 Avoid negations

[`Market::onlyShareCreator`](https://github.com/code-423n4/2023-11-canto/blob/main/1155tech-contracts/src/Market.sol#L81-L84):
```solidity
File: 1155tech-contracts/src/Market.sol

81:        require(
82:            !shareCreationRestricted || whitelistedShareCreators[msg.sender] || msg.sender == owner(),
83:            "Not allowed"
84:        );
```

Negations are harder to grasp and easier to misunderstand.

Consider flipping `shareCreationRestricted` to `shareCreationOpen`:
```solidity
        require(
            shareCreationOpen || whitelistedShareCreators[msg.sender] || msg.sender == owner(),
            "Not allowed"
        );
```

This makes it much faster to understand the logic.

## Non-critical / Informational

### NC-01 `_tokenCount` passed to `_splitFees` is always `shareData[_id].tokensInCirculation`

[`Market::buy`](https://github.com/code-423n4/2023-11-canto/blob/main/1155tech-contracts/src/Market.sol#L158), [`sell`](https://github.com/code-423n4/2023-11-canto/blob/main/1155tech-contracts/src/Market.sol#L177), [`mintNFT`](https://github.com/code-423n4/2023-11-canto/blob/main/1155tech-contracts/src/Market.sol#L207) and [`burnNFT`](https://github.com/code-423n4/2023-11-canto/blob/main/1155tech-contracts/src/Market.sol#L230):
```solidity

158:        _splitFees(_id, fee, shareData[_id].tokensInCirculation);

177:        _splitFees(_id, fee, shareData[_id].tokensInCirculation);

207:        _splitFees(_id, fee, shareData[_id].tokensInCirculation);

230:        _splitFees(_id, fee, shareData[_id].tokensInCirculation);
```

Consider, instead of passing `shareData[_id].tokensInCirculation` just reading it directly in `Market::_splitFees`:
```diff
        if (_tokenCount > 0) {
-           shareData[_id].shareHolderRewardsPerTokenScaled += (shareHolderFee * 1e18) / _tokenCount;
+           shareData[_id].shareHolderRewardsPerTokenScaled += (shareHolderFee * 1e18) / shareData[_id].tokensInCirculation;
        } else {
```

### NC-02 `testFail` TODO explained

[`testFailChangeBondingCurveAllowedNonOwner` and `testFailChangeBondingCurveAllowedNonOwner`](https://github.com/code-423n4/2023-11-canto/blob/main/1155tech-contracts/src/test/Market.t.sol#L42-L53):
```solidity
File: 1155tech-contracts/src/test/Market.t.sol

42:    function testFailChangeBondingCurveAllowedNonOwner() public {
43:        // TODO: For some reason, assertion fails, but call reverts as expected
44:        // vm.expectRevert("Ownable: caller is not the owner");
45:        vm.prank(bob);
46:        market.changeBondingCurveAllowed(address(bondingCurve), true);
47:    }
48:
49:    function testFailCreateNewShareWhenBondingCurveNotWhitelisted() public {
50:        // TODO: For some reason, assertion fails, but call reverts as expected
51:        // vm.expectRevert("Bonding curve not whitelisted");
52:        market.createNewShare("Test Share", address(bondingCurve), "metadataURI");
53:    }
```

From forge [docs](https://book.getfoundry.sh/forge/writing-tests):
> `testFail`: The inverse of the `test` prefix - if the function does not revert, the test fails.

Hence, these tests are now passing since the calls done reverts. And if the `vm.expectRevert` are applied, it will cause the test to not revert, hence the test will fail. I hope that was not too confusing.

They will work, as in verify the specific revert reason, if the names are changed from `Fail` to `Revert`:
```solidity
    function testRevertChangeBondingCurveAllowedNonOwner() public {
        vm.expectRevert("Ownable: caller is not the owner");
        vm.prank(bob);
        market.changeBondingCurveAllowed(address(bondingCurve), true);
    }

    function testRevertCreateNewShareWhenBondingCurveNotWhitelisted() public {
        vm.expectRevert("Bonding curve not whitelisted");
        market.createNewShare("Test Share", address(bondingCurve), "metadataURI");
    }
```
