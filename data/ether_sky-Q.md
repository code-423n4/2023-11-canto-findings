[L-1] The market is not compatible with rebase tokens.
If the balance decreases in a rebase token, the market lacks sufficient funds to sell all outstanding tokens.

[L-2] There is an inconsistency in registering to the `Turnstile` in the `asDFactory`.
In the `asDFactory` constructor, we registered `tx.origin` but passed `owner()` as `csrRecipient` to the `asD` contract. 
To receive `CSR` rewards, the contract should have a function to call the `withdraw()` function in the `Turnstile` contract. 
Therefore, when registering the contract (owner of `asDFactory` in our case) into the `Turnstile`, this consideration should be taken into account.

[L-3] Unused variable.
The variable `shareBondingCurves` is defined but not used in the `Market`.