# [L1] - Ownable2Step Unused

# Summary
The ```asDFactory``` contract inherits from ```Ownable2Step``` but it doesn't use any of its functionality. If the ```owner()``` function or other features of ```Ownable2Stepv is not used, it is be unnecessary to include it.

# Vulnerability
```solidity
@> import {Ownable2Step} from "@openzeppelin/contracts/access/Ownable2Step.sol"; //line 5

@> contract asDFactory is Ownable2Step {. //line 8
```

#Impact
The ```asDFactory``` contract inherits from ```Ownable2Step``` but it doesn't use any of its functionality. If the ```owner()``` function or other features of ```Ownable2Stepv is not used, it is unnecessary to include it.

#Tool Used
Manual review

#Recommendations
Remove unused import.
```solidity
- import {Ownable2Step} from "@openzeppelin/contracts/access/Ownable2Step.sol"; 

- contract asDFactory is Ownable2Step {
+ contract asDFactory {
```


# [L2] - Hardcoded Chain IDs
# Summary
The ```asDFactory::constructor``` checks if the current chain ID is 7700 or 7701. Hardcoding chain IDs can lead to issues if the contract is deployed on a different network (or it would be in the future). It would be better to pass the chain ID as a parameter to the constructor or to have a separate function to set it.

# Vulnerability
```solidity
constructor(address _cNote) {
        cNote = _cNote;
@>        if (block.chainid == 7700 || block.chainid == 7701) {
            // Register CSR on Canto main- and testnet
            Turnstile turnstile = Turnstile(0xEcf044C5B4b867CFda001101c617eCd347095B44);
            turnstile.register(tx.origin);
        }
    }
```

#Impact
Hardcoding chain IDs can lead to issues if the contract is deployed on a different network (or it would be in the future). It would be better to pass the chain ID as a parameter to the constructor or to have a separate function to set it.

#Tool Used
Manual review

#Recommendations
Pass the chain ID as a parameter in the constructor.
```diff
+ uint256 chainIdMainNet;
+ uint256 chainIdTestNet;

- constructor(address _cNote) {
+ constructor(address _cNote, uint256 _chainIdMainNet, uint256 _chainIdTestNet) {
        cNote = _cNote;
        chainIdMainNet = _chainIdMainNet;
        chainIdTestNet = _chainIdTestNet;
        if (block.chainid == chainIdMainNet || block.chainid == chainIdTestNet) {
            // Register CSR on Canto main- and testnet
            Turnstile turnstile = Turnstile(0xEcf044C5B4b867CFda001101c617eCd347095B44);
            turnstile.register(tx.origin);
        }
    }
```