## LOW FINDINGS

##

## [L-1] Missing validation mechanisms for unique ``_name`` and ``_symbol`` in token deployment

### Impact
Without checks, users can create multiple tokens with identical names and symbols. This can be problematic as it could lead to confusion or exploitation, where malicious actors might create tokens that mimic established tokens to deceive users 

```solidity
FILE: Breadcrumbs2023-11-canto/asD/src/asDFactory.sol

function create(string memory _name, string memory _symbol) external returns (address) {
        asD createdToken = new asD(_name, _symbol, msg.sender, cNote, owner());
        isAsD[address(createdToken)] = true;
        emit CreatedToken(address(createdToken), _symbol, _name, msg.sender);
        return address(createdToken);
    }

```
https://github.com/code-423n4/2023-11-canto/blob/335930cd53cf9a137504a57f1215be52c6d67cb3/asD/src/asDFactory.sol#L33-L38

### Recommended Mitigation
You can implement a system to ensure that each token with a unique name or symbol can only be created once.

##

## [L-2] Addressing the Oversight of Zero ``totalSupply()`` in ``withdrawCarry`` function logic

### Impact

The ``withdrawCarry`` function does not seem to consider the scenario where ``totalSupply()`` is zero. This oversight can lead to ``unexpected behavior`` or ``inefficiencies``.

If totalSupply() is zero, the calculation for maximumWithdrawable might not behave as intended.

will evaluate to the full scaled balance of cNote tokens held by the contract when totalSupply() is zero. In a situation where the contract is expected to have no withdrawable funds (because it has no underlying token supply), this could lead to incorrect calculations and potentially allow the withdrawal of more tokens than should be allowed.

### POC

```diff
FILE: 2023-11-canto/asD/src/asD.sol

 uint256 maximumWithdrawable = (CTokenInterface(cNote).balanceOf(address(this)) * exchangeRate) /
            1e28 -
            totalSupply();

```
https://github.com/code-423n4/2023-11-canto/blob/335930cd53cf9a137504a57f1215be52c6d67cb3/asD/src/asD.sol#L75-L77

### Recommended Mitigation
Add the the check ``totalSupply() != 0``

##

## [L-3] ``shareCreationRestricted`` is breaks its creation purpose when any other checks true 

### Impact
The ``onlyShareCreator`` modifier is designed to restrict certain functionalities based on the condition of ``shareCreationRestricted`` and other checks. However, as you pointed out, the purpose of ``shareCreationRestricted`` seems to be undermined or bypassed when the other conditions ``(whitelistedShareCreators[msg.sender]`` or ``msg.sender == owner())`` are true.

If the intention was to make ``shareCreationRestricted`` an absolute rule (i.e., no share creation allowed when it is true, regardless of who the sender is), then the current logic doesn't align with this intention. Instead, it allows for exceptions.

### POC

```solidity
FILE: 2023-11-canto/1155tech-contracts/src/Market.sol

modifier onlyShareCreator() {
        require(
            !shareCreationRestricted || whitelistedShareCreators[msg.sender] || msg.sender == owner(),
            "Not allowed"
        );
        _;
    }

```

### Recommended Mitigation
If you want ``shareCreationRestricted`` to be an overriding condition that cannot be bypassed, the logic should be adjusted.

```solidity

modifier onlyShareCreator() {
    require(
        !shareCreationRestricted,
        "Share creation is restricted"
    );
    require(
        whitelistedShareCreators[msg.sender] || msg.sender == owner(),
        "Not allowed"
    );
    _;
}

```








