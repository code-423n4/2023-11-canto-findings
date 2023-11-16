## Unused `isAsD()`

https://github.com/code-423n4/2023-11-canto/blob/335930cd53cf9a137504a57f1215be52c6d67cb3/asD/src/asDFactory.sol#L33-L38

As the function `isAsD()` serves to verify whether it is an `asD` Token, it has not been utilized in the `Market` or other contracts.

```
    function create(string memory _name, string memory _symbol) external returns (address) {
        asD createdToken = new asD(_name, _symbol, msg.sender, cNote, owner());
        isAsD[address(createdToken)] = true;
        emit CreatedToken(address(createdToken), _symbol, _name, msg.sender);
        return address(createdToken);
    }
```