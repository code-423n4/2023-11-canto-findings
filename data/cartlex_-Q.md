| ID | Description | Severity |
| :-: | - | :-: |
| [NC-01](#nc-01-function-can-be-used-with-zero-values)| Function can be used with zero values. | Non-critical |


# [NC-01] Function can be used with zero values.

## Description
The next functions from `Market.sol` contract can be used with zero values.
- `sell`
- `buy`
- `mintNFT`
- `burnNFT`

## Recommended Mitigation Steps
Consider to implement checks in `sell`, `buy`, `mintNFT` and `burnNFT` functions that input amount is not equal to zero.


