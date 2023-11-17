[L-01] Centralization risk only authorised wallets can create new shares

### Severity

**Impact: Medium**, as only Canto allowed addresses can create shares
**Likelihood**: Low, as it requires malicious owner

### Description

At deployment the `Market::shareCreationRestricted` is set to `true` and we have to trust Canto to change this after deployment. 

### Recommendations

It would be better to set this value to false, or a setter  during deployment. 
It is also recommended to change the `bool` to `uint256`, this reduce gas for storing and changing the value. [Source](https://github.com/OpenZeppelin/openzeppelin-contracts/blob/58f635312aa21f947cae5f8578638a85aa2519f5/contracts/security/ReentrancyGuard.sol#L23-L27)

[L-02] Using ultra large strings can lead to high gas or transaction failure

### Severity

**Impact: Low**
**Likelihood**: Low, as share creators probably won't use ultra large strings

### Description
Using an ultra-large string for the `_shareName` in the `Market::createNewShare`  could lead to several issues, primarily related to gas costs and transaction execution

### Recommendations
**Limiting String Length**: Implement a check to enforce a maximum length for the `_shareName`.

[L-03] No checks on addresses in `asD::constructor` 
It is recommended to check the addresses that they are not  `address(0)` 

[L-04] No checks on addresses in `asDFactory::constructor` 
It is recommended to check the addresses that they are not  `address(0)` 

[N-01] Missing event after state changes in `Market::changeShareCreatorWhitelist` 

[N-02] return value of `safeTransfer` is ignored in `asD::mint` 

[N-03] Using of safeApprove is deprecated in `asD::mint` 
Use of safeIncreaseAllowance is recommended.