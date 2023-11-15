[/1155tech-contracts/src/Market.sol#L114](https://github.com/code-423n4/2023-11-canto/blob/main/1155tech-contracts/src/Market.sol#L114): createNewShare()
arguments of createNewShare could be calldata, instead of memory to save gas.
Before: 161187
After: 160342
Total saving: 845

[/1155tech-contracts/src/Market.sol#L150](https://github.com/code-423n4/2023-11-canto/blob/main/1155tech-contracts/src/Market.sol#L150): buy()
Property reads 2 times: `shareData[_id].tokensInCirculation` could be assigned to a variable to save 100 gas
Before: 350555
After: 350452
Total saving: 103

[/1155tech-contracts/src/Market.sol#L174](https://github.com/code-423n4/2023-11-canto/blob/main/1155tech-contracts/src/Market.sol#L174): sell()
Assign `shareData[_id].tokensInCirculation` to variable to save 106 gas

Before: 346371
After: 346265
Total saving: 106

[/1155tech-contracts/src/Market.sol#L203](https://github.com/code-423n4/2023-11-canto/blob/main/1155tech-contracts/src/Market.sol#L203): mintNft()

Assign `shareData[_id].tokensInCirculation` to variable to save 106 gas

Before: 397993
After: 397893
Total saving: 106



