## 1. Low. Prevent specifying `_amount = 0` in functions `mintNFT()` and `burnNFT()`
If user specifies `amount = 0` in burn or mint, events `NFTsCreated` and `NFTsBurned` will be emitted with zero prices. These events are utilized off-chain "to have the price information".
However 0 values can introduce a bug on backend, therefore prevent supplying `amount = 0`

## 2. R. Remove unused variable `shareBondingCurves` in Market.sol