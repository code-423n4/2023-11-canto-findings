## Your tests are not being run in the right environment.

Test files after calling the `forge test` command pass correctly only because `chainId` is taken as default (31337), therefore some key functionality is excluded from tests, giving a false sense of correct operation of the protocol.

**Example of code snippet skipped by invalid chainId:**
https://github.com/code-423n4/2023-11-canto/blob/main/asD/src/asDFactory.sol#L26-L29
```solidity
        if (block.chainid == 7700 || block.chainid == 7701) {
            // Register CSR on Canto main- and testnet
            Turnstile turnstile = Turnstile(0xEcf044C5B4b867CFda001101c617eCd347095B44);
            turnstile.register(tx.origin);
        }
```
After adding the chainId allowing you to test contracts in the target environment via `vm.chainId(7700)` in the `setUp` function the tests will fail.

## Recommendation
Extend tests and run them reflecting the right environment. Use `vm.chainId(7700)` and/or `vm.chainId(7701)` for this purpose.

## The ability to indicate any contract address without restrictions on its implementation that participates in key business flow is dangerous

Bonding curves can be any contract specified by the owner. Even though this is a trusted role, it poses a threat to all users that can and should be minimized.

## Recommendation
Create internal criteria and strong limitations for what bonding curves may be. Among the criteria for evaluating the possibility of adding them, consider things like:
- can they be upgradeable? If so, on what terms?
- should they have a pre-imposed limit on maximum values?
- should `Market` check the values returned from bondingCurve?

Introduce restrictive access policies and use multi sig.

## Consider creating a vault for CSR rewards

Each registration via `turnstile` creates a separate NFT, which entitles you to receive rewards. Instead of storing them at the addresses of owner, they could be stored in a special vault managed by the owner. This way, if the owner changes, you will not have to remember to transfer the NFT authorizing you to collect a CSR reward.



### Time spent:
16 hours