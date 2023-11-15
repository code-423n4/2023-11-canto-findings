[1] Old version of OpenZeppelin Contracts used
Using old versions of 3rd-party libraries in the build process can introduce vulnerabilities to the protocol code.

Latest is 5.0.0 (as of Oct 5, 2023), version used in project is 4.9.0. Considering that the latest release is a major, if the team doesn't consider that they want to change, atleast an upgrade of the library to 4.9.3 should be considered

[2] Storage variable never used
In the Market.sol at line 46
```
    /// @notice Stores the bonding curve per share
    mapping(uint256 => address) public shareBondingCurves;
```

[3] Missing event for whitelisting Creators
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

[4] Using hardcoded values instead of constants
The denominator for calculating fees is hardcoded in multiple places(lines 197, 285, 286). Instead of hardcoding the value of `10_000` a constant variable should be used:
```
uint256 public constant FEE_DENOMINATOR = 10_000;  
```