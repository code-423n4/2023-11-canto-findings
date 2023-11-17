### 1. `x = x + y` is cheaper than `x += y;`

Market.sol [L293](https://github.com/code-423n4/2023-11-canto/blob/335930cd53cf9a137504a57f1215be52c6d67cb3/1155tech-contracts/src/Market.sol#L293)
```
           platformFee += shareHolderFee;
```

### 2. Failure to check for 0x0 address in the constructor could case the contract to be deployed  by mistakenly entering a zero address. As a result, the contract will need to be redeployed. This means extra gas consumption as contract deployment costs are high.

Market.sol [L91](https://github.com/code-423n4/2023-11-canto/blob/516099801101950ac9e1117a70e095b06f9bf6a1/1155tech-contracts/src/Market.sol#L91)
```
constructor(string memory _uri, address _paymentToken) ERC1155(_uri) Ownable() {
```
asD.sol [L28](https://github.com/code-423n4/2023-11-canto/blob/516099801101950ac9e1117a70e095b06f9bf6a1/asD/src/asD.sol#L28-L34)
```
     constructor(
         string memory _name,
         string memory _symbol,
         address _owner,
         address _cNote,
         address _csrRecipient
     ) ERC20(_name, _symbol) {
```
asDFactory.sol [L24](https://github.com/code-423n4/2023-11-canto/blob/516099801101950ac9e1117a70e095b06f9bf6a1/asD/src/asDFactory.sol#L24)
```
constructor(address _cNote) {
```
### 3. Upgrade solidity's optimizer
Make sure Solidity’s optimizer is enabled. It reduces gas costs. If you want to gas optimize for contract deployment then set the optimizer to a lower number. If you want to optimize for run-time gas costs then set the optimizer to a higher number.

[1155tech foundry.toml](https://github.com/code-423n4/2023-11-canto/blob/335930cd53cf9a137504a57f1215be52c6d67cb3/1155tech-contracts/foundry.toml#L5C1-L6C23), [asD foundry.toml](https://github.com/code-423n4/2023-11-canto/blob/335930cd53cf9a137504a57f1215be52c6d67cb3/asD/foundry.toml#L5C1-L6C23)
```
optimizer = true
optimizer_runs = 1_000
```
