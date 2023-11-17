### [Low-01] Even after a `bondingCurve contract` get `delisted` still it will be in used inside previously created `Shares`
May be due to some issue or bug in `bondingCurve` contract, protocol has functionality to remove/delist them from allowList. Currently it only using `Linnear bonding Curve` for calculating fee and price. But protocol states that in future it may support other types of bonding curve.

In that case, when a bondingCurve delisted it still used by Previous Shares which implemented those on time of creation. 

There is no way to change it by Share Owner. 

```solidity
    function changeBondingCurveAllowed(address _bondingCurve, bool _newState) external onlyOwner { 
        require(whitelistedBondingCurves[_bondingCurve] != _newState, "State already set");
        whitelistedBondingCurves[_bondingCurve] = _newState;
        emit BondingCurveStateChange(_bondingCurve, _newState);
    }

    /// @notice Creates a new share
    /// @param _shareName Name of the share
    /// @param _bondingCurve Address of the bonding curve, has to be whitelisted
    /// @param _metadataURI URI of the metadata
    function createNewShare(
        string memory _shareName,
        address _bondingCurve,
        string memory _metadataURI
    ) external onlyShareCreator returns (uint256 id) {
        require(whitelistedBondingCurves[_bondingCurve], "Bonding curve not whitelisted");
        require(shareIDs[_shareName] == 0, "Share already exists");
        id = ++shareCount;
        shareIDs[_shareName] = id;
        shareData[id].bondingCurve = _bondingCurve; // @audit
        shareData[id].creator = msg.sender;
        shareData[id].metadataURI = _metadataURI;
        emit ShareCreated(id, _shareName, _bondingCurve, msg.sender);
    }
```
```
https://github.com/code-423n4/2023-11-canto/tree/main/1155tech-contracts/src/Market.sol#L104-L108
https://github.com/code-423n4/2023-11-canto/tree/main/1155tech-contracts/src/Market.sol#L123
```

### Mitigation
Developer should re-consider it.

### [Low-02] `Share's` Creator can simply bypass `shareData[_id].creator != msg.sender` check in `buy()` by simply calling it from other EOA/SmartContract
```solidity
    function buy(uint256 _id, uint256 _amount) external { // @audit id check for gas save 8337Gas
        require(shareData[_id].creator != msg.sender, "Creator cannot buy"); // @audit can call from other contract
        (uint256 price, uint256 fee) = getBuyPrice(_id, _amount); // Reverts for non-existing ID
        SafeERC20.safeTransferFrom(token, msg.sender, address(this), price + fee);
....
....
```
```
https://github.com/code-423n4/2023-11-canto/tree/main/1155tech-contracts/src/Market.sol#L151
```

### [Low-03] Should implement `_safeMint()` while minting `NFT` to ensure if receiving address is a contract then it implemented `onReceiver` function or not
Token may get lost.
```solidity
        _mint(msg.sender, _id, _amount, "");
```
```
https://github.com/code-423n4/2023-11-canto/tree/main/1155tech-contracts/src/Market.sol#L214
```