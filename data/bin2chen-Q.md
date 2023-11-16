## Findings Summary

| Label | Description |
| - | - |
| [L-01](#) | Lack of `shareBondingCurves` setting|


## [L-01] Lack of `shareBondingCurves` setting
Currently, `shareBondingCurves` is defined, but it is not set in `createNewShare()`, which results in this read always being address(0)

```diff
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

+       shareBondingCurves[id]=_bondingCurve;
        emit ShareCreated(id, _shareName, _bondingCurve, msg.sender);
    }
```