### Summary

* \[L-01\] `Market::createNewShare()` function should update the `shareBondingCurves` mapping
    
* \[L-02\] `Market::_splitFess()` should split the shareholder fee to the protocol and creator when there are no tokens in circulation
    
* \[L-03\] `Market::changeShareCreatorWhitelist` should emit an event
    

---

### \[L-01\] `Market::createNewShare()` function should update the `shareBondingCurves` mapping

[https://github.com/code-423n4/2023-11-canto/blob/335930cd53cf9a137504a57f1215be52c6d67cb3/1155tech-contracts/src/Market.sol#L46](https://github.com/code-423n4/2023-11-canto/blob/335930cd53cf9a137504a57f1215be52c6d67cb3/1155tech-contracts/src/Market.sol#L46)  
[https://github.com/code-423n4/2023-11-canto/blob/335930cd53cf9a137504a57f1215be52c6d67cb3/1155tech-contracts/src/Market.sol#L122C1-L125C50](https://github.com/code-423n4/2023-11-canto/blob/335930cd53cf9a137504a57f1215be52c6d67cb3/1155tech-contracts/src/Market.sol#L122C1-L125C50)

The Market contract has a [`shareData`](https://github.com/code-423n4/2023-11-canto/blob/335930cd53cf9a137504a57f1215be52c6d67cb3/1155tech-contracts/src/Market.sol#L43) mapping from `uint256` to `ShareData` struct (uint256 =&gt; ShareData), and it also has a separate mapping called [`shareBondingCurves`](https://github.com/code-423n4/2023-11-canto/blob/335930cd53cf9a137504a57f1215be52c6d67cb3/1155tech-contracts/src/Market.sol#L46C40-L46C58)

ShareData struct already includes the `bondingCurve` element in itself and this is updated when creating a new share. However, the `shareBondingCurves` mapping is not updated and this situation creates a mismatch between these two mappings.

```solidity
    function createNewShare(
        string memory _shareName,
        address _bondingCurve,
        string memory _metadataURI
    ) external onlyShareCreator returns (uint256 id) {
        require(whitelistedBondingCurves[_bondingCurve], "Bonding curve not whitelisted");
        require(shareIDs[_shareName] == 0, "Share already exists");
        id = ++shareCount;
        shareIDs[_shareName] = id;
        shareData[id].bondingCurve = _bondingCurve;
        shareData[id].creator = msg.sender;
        shareData[id].metadataURI = _metadataURI;
        emit ShareCreated(id, _shareName, _bondingCurve, msg.sender);
    } //@audit this function doesn't update shareBondingCurves mapping. It should update it by doing this --> shareBondingCurves[id] = _bondingCurve;
```

This function should update the shareBondingCurves mapping or that mapping should not be created in the first place.

---

### \[N-01\] `Market::_splitFess()` should split the shareholder fee to the protocol and creator when there are no tokens in circulation

[https://github.com/code-423n4/2023-11-canto/blob/335930cd53cf9a137504a57f1215be52c6d67cb3/1155tech-contracts/src/Market.sol#L289C9-L294C10](https://github.com/code-423n4/2023-11-canto/blob/335930cd53cf9a137504a57f1215be52c6d67cb3/1155tech-contracts/src/Market.sol#L289C9-L294C10)

```solidity
    function _splitFees(
        uint256 _id,
        uint256 _fee,
        uint256 _tokenCount
    ) internal {
        uint256 shareHolderFee = (_fee * HOLDER_CUT_BPS) / 10_000;
        uint256 shareCreatorFee = (_fee * CREATOR_CUT_BPS) / 10_000;
        uint256 platformFee = _fee - shareHolderFee - shareCreatorFee;
        shareData[_id].shareCreatorPool += shareCreatorFee;
        if (_tokenCount > 0) {
            shareData[_id].shareHolderRewardsPerTokenScaled += (shareHolderFee * 1e18) / _tokenCount;
        } else {
            // If there are no tokens in circulation, the fee goes to the platform
            platformFee += shareHolderFee; //@audit QA - it should be split between the creator and the platform
        }
        platformPool += platformFee;
    }
```

Normally, the fees gained are split between the protocol, the creator and the shareholders.  
But when there are no tokens in circulation, all of the shareholder fees are transferred to the protocol. However, this is unfair to the token creator.

The token creator is the reason why those fees are generated. If there are no shareholders, the shareholder fee should be split between the protocol and the token creator.

---

### \[N-02\] `Market::changeShareCreatorWhitelist`

### should emit an event

[https://github.com/code-423n4/2023-11-canto/blob/335930cd53cf9a137504a57f1215be52c6d67cb3/1155tech-contracts/src/Market.sol#L309C5-L312C6](https://github.com/code-423n4/2023-11-canto/blob/335930cd53cf9a137504a57f1215be52c6d67cb3/1155tech-contracts/src/Market.sol#L309C5-L312C6)

```solidity
    function changeShareCreatorWhitelist(address _address, bool _isWhitelisted) external onlyOwner {
        require(whitelistedShareCreators[_address] != _isWhitelisted, "State already set");
        whitelistedShareCreators[_address] = _isWhitelisted;
    }
```

`changeShareCreatorWhitelist` function does not emit an event at the moment. Important actions should emit events. Adding an event called `CreatorWhitelistStatusChanged` or something similar would be better for this function.