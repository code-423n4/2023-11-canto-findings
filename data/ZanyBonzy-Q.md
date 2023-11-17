### 1. Reviews of the input files to solpp are not the same as reviews of its output
From the bot report, there seems to be an [issue](https://github.com/code-423n4/2023-11-canto/blob/main/bot-report.md#D-12) where `SPDX-License-Identifier` appears to not be on the first line even though the audit files had it on the first line. This is because of a [bug](https://github.com/merklejerk/solpp/issues/21) in the deprecated solpp tool in use by the project. It seems that the audit is done on the solpp input, rather than its outputs. This is not a recommended good practice, as there might be  other lurking bugs.

### 2. Use of a single step ownership transfer
Consider implementing a two step variant, where the acceptor has to call a separate function in order for the transfer to take effect.

asD.sol [[L35]](https://github.com/code-423n4/2023-11-canto/blob/335930cd53cf9a137504a57f1215be52c6d67cb3/asD/src/asD.sol#L35)
```
_transferOwnership(_owner);
```

### 3. Possible loss of precision 
LinearBondingCurve.sol [23](https://github.com/code-423n4/2023-11-canto/blob/516099801101950ac9e1117a70e095b06f9bf6a1/1155tech-contracts/src/bonding_curve/LinearBondingCurve.sol#L23), [35](https://github.com/code-423n4/2023-11-canto/blob/516099801101950ac9e1117a70e095b06f9bf6a1/1155tech-contracts/src/bonding_curve/LinearBondingCurve.sol#L35)

```
            fee += (getFee(i) * tokenPrice) / 1e18;
```
```
        return 1e17 / divisor;
```
### 4. Consider requiring that strings lenght != 0
Values like name, symbol, or uri can be passed with no entries. IT'd be odd to have a token with no name, no symbol and haveing an empty uri can affect the functionality of the token uri.

Market.sol [L91](https://github.com/code-423n4/2023-11-canto/blob/335930cd53cf9a137504a57f1215be52c6d67cb3/1155tech-contracts/src/Market.sol#L91C1-L91C85), [L114](https://github.com/code-423n4/2023-11-canto/blob/335930cd53cf9a137504a57f1215be52c6d67cb3/1155tech-contracts/src/Market.sol#L114C1-L118C8)
```
    constructor(string memory _uri, address _paymentToken) ERC1155(_uri) Ownable() {
```
```
    function createNewShare(
        string memory _shareName,
        address _bondingCurve,
        string memory _metadataURI
    ) 
  ```

asDFactory.sol [L33](https://github.com/code-423n4/2023-11-canto/blob/335930cd53cf9a137504a57f1215be52c6d67cb3/asD/src/asDFactory.sol#L32C1-L33C93)
```
    function create(string memory _name, string memory _symbol) external returns (address) {
```
