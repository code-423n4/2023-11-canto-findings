[1] Storage variable never used
In the Market.sol at line 46
```
    /// @notice Stores the bonding curve per share
    mapping(uint256 => address) public shareBondingCurves;
```

[2] Missing event for whitelisting Creators
Function `changeShareCreatorWhitelist` doesn't emit an event even if the change is important for the protocol.
```
    function changeShareCreatorWhitelist(address _address, bool _isWhitelisted) external onlyOwner {
        require(whitelistedShareCreators[_address] != _isWhitelisted, "State already set");
        whitelistedShareCreators[_address] = _isWhitelisted;
    }
```
Recommandation:
```
    function changeShareCreatorWhitelist(address _address, bool _isWhitelisted) external onlyOwner {
        require(whitelistedShareCreators[_address] != _isWhitelisted, "State already set");
        whitelistedShareCreators[_address] = _isWhitelisted;
        emit ShareCreatorWhitelistChanged(_address, _isWhitelisted);
    }
```

[3] Using hardcoded values instead of constants
The denominator for calculating fees is hardcoded in multiple places(lines 197, 285, 286). Instead of hardcoding the value of `10_000` a constant variable should be used:
```
uint256 public constant FEE_DENOMINATOR = 10_000;  
```