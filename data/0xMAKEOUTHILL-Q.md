1[LOW] - In `Market.sol`, there is a mapping `shareBondingCurves` which **"Stores the bonding curve per share ID"**, but it is not used anywhere in the code.

Either use this mapping whenever creating a new `share`: 
```
function createNewShare(
        string memory _shareName,
        address _bondingCurve,
        string memory _metadataURI
    ) external onlyShareCreator returns (uint256 id) {
        
        require(whitelistedBondingCurves[_bondingCurve], "Bonding curve not whitelisted");
    
        require(shareIDs[_shareName] == 0, "Share already exists");
    
        id = ++shareCount;
    
        shareIDs[_shareName] = id;
         
        //Make use of the mapping this way.
@>>     shareBondingCurves[id][_bondingCurve];

        shareData[id].bondingCurve = _bondingCurve;
        shareData[id].creator = msg.sender;
        shareData[id].metadataURI = _metadataURI;

        emit ShareCreated(id, _shareName, _bondingCurve, msg.sender);

    }
```

or remove the mapping from the contract.


