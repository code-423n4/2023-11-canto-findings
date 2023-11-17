# getPrice() is expensive
getPrice() is very expensive for big buyers. The potential high gas consumption could DoS big buyers from interacting with the contract which could incur missed rewards for the protocol, creator and holders.

Indeed the price's calculation looks like the sum of an arithmetic series which could be moved out of the loop:
https://chat.openai.com/share/f0464741-d5f7-4cb5-9ae5-5be657cf04a0

And if getFee() could avoid the log2 and if/else (for eg. as being a simple percent), it could also be simplified.

# events should not be emitted for 0 rewards
claimPlatformFee(), claimCreatorFee() and claimHolderFee() should not do transfers and emit events when the amount is 0.

# useless duplicate
claimPlatformFee():

```
    function claimPlatformFee() external onlyOwner {
        uint256 amount = platformPool; // useless
        platformPool = 0; 
        SafeERC20.safeTransfer(token, msg.sender, amount);
        emit PlatformFeeClaimed(msg.sender, amount);
    }
```

can be re-written as:
```
    function claimPlatformFee() external onlyOwner {
        SafeERC20.safeTransfer(token, msg.sender, platformPool);
        emit PlatformFeeClaimed(msg.sender, platformPool);
        platformPool = 0; 
    }
```

which makes it clearer and save gas.