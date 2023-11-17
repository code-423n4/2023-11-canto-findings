## [1] LACK OF EVENTS IN THE CRITICAL OPERATIONS OF THE PROTOCOL

The 1155tech-contracts ```Market.sol``` smart contract performs critical operations such as burning and minting NFTs and buying and selling tokens for shares. However, the ```changeShareCreatorWhitelist(...)``` which adds or removes an address from the whitelist of share creators lack events as shown below.

```solidity
function changeShareCreatorWhitelist(address _address, bool _isWhitelisted) external onlyOwner {
        require(whitelistedShareCreators[_address] != _isWhitelisted, "State already set");
        whitelistedShareCreators[_address] = _isWhitelisted;
    }
```

it is recommended to emit events when creators are successfully added or removed for off-chain analysis.

## [2] DISCREPENCY BETWEEN NATSPEC COMMENT AND LOGIC IMPLEMENTATION OF THE getNFTMintingPrice(...) FUNCTION
The NATSPEC comment of Market.getNFTMintingPrice(...) function states that it _Returns the price and fee for minting a given number of NFTs_. But the logic implementation shows that it returns only the fee for minting a given number of NFTs as as shown below
```solidity
function getNFTMintingPrice(uint256 _id, uint256 _amount) public view returns (uint256 fee)
```

it is recommended to update the NATSPEC comments of the Market.getNFTMintingPrice(...) to state that it _Returns the fee for minting a given number of NFTs_.