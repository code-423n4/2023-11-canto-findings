# Report

## Non Critical Issues

### [NC-1] buy(), sell(), mintNFT(), and burnNFT() in Market.sol can successfully run with 0 as amount

One is able to pass 0 as the amount into `buy()`, `sell()`, `mintNFT()`, and `burnNFT()`, and the functions will run successfully without reverting. I couldn't find a case where this may cause a weird state change but do consider enforcing a minimum amount of 1.

https://github.com/code-423n4/2023-11-canto/blob/main/1155tech-contracts/src/Market.sol#L150
https://github.com/code-423n4/2023-11-canto/blob/main/1155tech-contracts/src/Market.sol#L174
https://github.com/code-423n4/2023-11-canto/blob/main/1155tech-contracts/src/Market.sol#L203
https://github.com/code-423n4/2023-11-canto/blob/main/1155tech-contracts/src/Market.sol#L226

### [NC-2] claimPlatformFee(), claimCreatorFee(), and claimHolderFee() in Market.sol do not check for 0 amount in Market.sol

`claimPlatformFee()`, `claimCreatorFee()`, and `claimHolderFee()` allow you to call them without having any fees accrued. If that's the case, they basically do an empty transfer and emit an event. I would suggest requiring there are rewards before executing.

https://github.com/code-423n4/2023-11-canto/blob/main/1155tech-contracts/src/Market.sol#L244
https://github.com/code-423n4/2023-11-canto/blob/main/1155tech-contracts/src/Market.sol#L253
https://github.com/code-423n4/2023-11-canto/blob/main/1155tech-contracts/src/Market.sol#L263

### [NC-3] mint() and burn() in asD.sol can be run with 0 amount

`mint()` and `burn()` can be run with 0 amount. Consider enforcing a minimum of 1.

https://github.com/code-423n4/2023-11-canto/blob/main/asD/src/asD.sol#L47
https://github.com/code-423n4/2023-11-canto/blob/main/asD/src/asD.sol#L60

### [NC-4] Consider using descriptive names for magic numbers

The following magic numbers can be found in multiple contracts: `7700`, `7701`, `0xEcf044C5B4b867CFda001101c617eCd347095B44`. While the amount is kept to a minimum, I would recommend storing those into descriptive constants and potentially sharing them across all contracts through a util library contract.
