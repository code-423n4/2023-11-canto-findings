# QA Report

## Table of Contents

| Issue ID                                                                                             | Description                                                                          |
| ---------------------------------------------------------------------------------------------------- | ------------------------------------------------------------------------------------ |
| [QA-01](#qa-01-the-return-value-of-the-turnstileregister-should-be-checked)                          | The return value of the `turnstile.register()` should be checked                     |
| [QA-02](#qa-02-asdwithdrawcarry-should-direct-to-compounds-redeeemunderlying-instead)                | `asD::withdrawCarry()` should direct to Compound's `redeemUnderlying()` instead      |
| [QA-03](#qa-03-a-limit-should-be-attached-to-sharecount)                                             | A limit should be attached to `shareCount`                                           |
| [QA-04](#qa-04-getnftmintingprice-should-align-more-with-its-documentation)                          | `getNFTMintingPrice()` should align more with its documentation                      |
| [QA-05](#qa-05-assembly-code-should-always-be-accompanied-with-comments-due-to-their-complex-nature) | Assembly code should always be accompanied with comments due to their complex nature |
| [QA-06](#qa-06-introduce-emergency-functionalities)                                                  | Introduce emergency functionalities                                                  |
| [QA-07](#qa-06-fee-calculation-should-be-better-documented)                                          | Fee calculation should be better documented                                          |

## QA-01 The return value of the `turnstile.register()` should be checked

### Impact

In the constructor for some contracts in scope, e.g `aSD.sol`, the code attempts to register the Turnstile contract if the chain ID is the `Canto` mainnet or testnet, i.e 7700 / 7701. However, it doesn't check the return value of the `turnstile.register()`function call. Failing to check the return value can lead to unexpected behaviour if the registration fails or if the function reverts under certain conditions.

### Proof of Concept

Using `aSD.sol` as a case syidy
Take a look at []()

```solidity
    constructor(
        string memory _name,
        string memory _symbol,
        address _owner,
        address _cNote,
        address _csrRecipient
    ) ERC20(_name, _symbol) {
        _transferOwnership(_owner);
        cNote = _cNote;
        if (block.chainid == 7700 || block.chainid == 7701) {
            // Register CSR on Canto main- and testnet
            Turnstile turnstile = Turnstile(0xEcf044C5B4b867CFda001101c617eCd347095B44);
            //@audit
            turnstile.register(_csrRecipient);
        }
    }
```

Use this search command: `https://github.com/search?q=repo%3Acode-423n4%2F2023-11-canto%20turnstile.register(&type=code` in order to find all instances of this in scope

### Recommended Mitigation Steps

Create a custom error and revert with this if the attempt to register does not go through.

> Additionally, consider storing the Turnstile contract address in a configuration file that can be updated externally.

## QA-02 `asD::withdrawCarry()` should direct to Compound's `redeeemUnderlying()` instead

### Impact

Info, incorrect integration of external codebase

### Proof of Concept

Take a look at [withdrawCarry()]()

```solidity
    function withdrawCarry(uint256 _amount) external onlyOwner {
        uint256 exchangeRate = CTokenInterface(cNote).exchangeRateCurrent(); // Scaled by 1 * 10^(18 - 8 + Underlying Token Decimals), i.e. 10^(28) in our case
        // The amount of cNOTE the contract has to hold (based on the current exchange rate which is always increasing) such that it is always possible to receive 1 NOTE when burning 1 asD
        uint256 maximumWithdrawable = (CTokenInterface(cNote).balanceOf(address(this)) * exchangeRate) /
            1e28 -
            totalSupply();
        if (_amount == 0) {
            _amount = maximumWithdrawable;
        } else {
            require(_amount <= maximumWithdrawable, "Too many tokens requested");
        }
        // Technically, _amount can still be 0 at this point, which would make the following two calls unnecessary.
        // But we do not handle this case specifically, as the only consequence is that the owner wastes a bit of gas when there is nothing to withdraw
        uint256 returnCode = CErc20Interface(cNote).redeemUnderlying(_amount);
        //@audit
        require(returnCode == 0, "Error when redeeming"); // 0 on success: https://docs.compound.finance/v2/ctokens/#redeem
        IERC20 note = IERC20(CErc20Interface(cNote).underlying());
        SafeERC20.safeTransfer(note, msg.sender, _amount);
        emit CarryWithdrawal(_amount);
    }
}
```

As seen the function directs to the Compound docs for `#redeem` where as it should direct it to `#redeemUnderlying` just like the `burn()` function

### Recommended Mitigation Steps

Correctly integrate external codebase and also make this changes to the `withdrawCarry()` function

```diff
    function withdrawCarry(uint256 _amount) external onlyOwner {
        uint256 exchangeRate = CTokenInterface(cNote).exchangeRateCurrent(); // Scaled by 1 * 10^(18 - 8 + Underlying Token Decimals), i.e. 10^(28) in our case
        // The amount of cNOTE the contract has to hold (based on the current exchange rate which is always increasing) such that it is always possible to receive 1 NOTE when burning 1 asD
        uint256 maximumWithdrawable = (CTokenInterface(cNote).balanceOf(address(this)) * exchangeRate) /
            1e28 -
            totalSupply();
        if (_amount == 0) {
            _amount = maximumWithdrawable;
        } else {
            require(_amount <= maximumWithdrawable, "Too many tokens requested");
        }
        // Technically, _amount can still be 0 at this point, which would make the following two calls unnecessary.
        // But we do not handle this case specifically, as the only consequence is that the owner wastes a bit of gas when there is nothing to withdraw
        uint256 returnCode = CErc20Interface(cNote).redeemUnderlying(_amount);
-        require(returnCode == 0, "Error when redeeming"); // 0 on success: https://docs.compound.finance/v2/ctokens/#redeem
+        require(returnCode == 0, "Error when redeeming"); // 0 on success: https://docs.compound.finance/v2/ctokens/#redeemUnderlying
        IERC20 note = IERC20(CErc20Interface(cNote).underlying());
        SafeERC20.safeTransfer(note, msg.sender, _amount);
        emit CarryWithdrawal(_amount);
    }
}
```

## QA-03 A limit should be attacehd to `shareCount`

### Impact

Low, info on better code structure

### Proof of Concept

Take a look at [getPriceAndFee()]()

```solidity
    function getPriceAndFee(uint256 shareCount, uint256 amount)
        external
        view
        override
        returns (uint256 price, uint256 fee)
    {
        for (uint256 i = shareCount; i < shareCount + amount; i++) {
            uint256 tokenPrice = priceIncrease * i;
            price += tokenPrice;
            fee += (getFee(i) * tokenPrice) / 1e18;
        }
    }
```

As seen this function queries and returns the price and fee, one thing to note is that the function does not limit the amount of `shareCount` it could work with which could lead to it getting error'd due to an OOG.

### Recommended Mitigation Steps

Introduce a max share count to protocol

## QA-04 `getNFTMintingPrice()` should align more with it's documentation

### Impact

Low/info... confusing codes.

### Proof of Concept

Take a look at [getNFTMintingPrice()]()

```solidity
    /// @notice Returns the price and fee for minting a given number of NFTs. @audit this should return both the price and fee as suggested by the comments, but it instead returns only the fee
    /// @param _id The ID of the share
    /// @param _amount The number of NFTs to mint.
    function getNFTMintingPrice(uint256 _id, uint256 _amount) public view returns (uint256 fee) {
        address bondingCurve = shareData[_id].bondingCurve;
        (uint256 priceForOne, ) = IBondingCurve(bondingCurve).getPriceAndFee(shareData[_id].tokenCount, 1);
        fee = (priceForOne * _amount * NFT_FEE_BPS) / 10_000;
    }
```

As seen, the function's name and its comments project this to return both the price to mint an NFT and it's fee, but the function only returns the fee even in all the instances where it's been queried

### Recommended Mitigation Steps

Either make the function more aligned to the documentation or better still correct the documentation if this is intended.

## QA-05 Assembly code should always be accompanied with comments due to their complex nature

### Impact

Info

### Proof of Concept

Take a look at [LinearBondingCurve::log2()]()

```solidity
    function log2(uint256 x) internal pure returns (uint256 r) {
        /// @solidity memory-safe-assembly
        assembly {
            r := shl(7, lt(0xffffffffffffffffffffffffffffffff, x))
            r := or(r, shl(6, lt(0xffffffffffffffff, shr(r, x))))
            r := or(r, shl(5, lt(0xffffffff, shr(r, x))))
            r := or(r, shl(4, lt(0xffff, shr(r, x))))
            r := or(r, shl(3, lt(0xff, shr(r, x))))
            // forgefmt: disable-next-item
            r := or(
                r,
                byte(
                    and(0x1f, shr(shr(r, x), 0x8421084210842108cc6318c6db6d54be)),
                    0x0706060506020504060203020504030106050205030304010505030400000000
                )
            )
        }
    }
}
```

As seen, this function is used to get the log2 of any value x, issue is that this code block uses assembly code, which is a way lower level than normal solidity code and is highly error prone

### Recommended Mitigation Steps

As popularly recommended in the web 3 space, whenever assembly code is used, it should always be accompanied with comments, this should also be implemented with `Canto` too.

## QA-06 Introduce emergency functionalities

### Impact

Info

### Proof of Concept

Using this[ search command: ](https://github.com/search?q=repo%3Acode-423n4%2F2023-11-canto%20pause%20unpause&type=code) `https://github.com/search?q=repo%3Acode-423n4%2F2023-11-canto%20pause%20unpause&type=code` we can see that all in-scope contracts lack any sort of emergency functionality, whereas adding this could lead to additional centralization, it's advisable cause in the case of a black swan event protocol can easily protect itself from too much harm

### Recommended Mitigation Steps

Add emergency functionalities to protocol.

## QA-07 Fee calculation should be better documented

### Impact

Info, non-understandable piece of code

### Proof of Concept

In `Market.sol` it can be seen that the process for fee calculation is not transparently documented in the contract. Clear comments or documentation regarding how fees are calculated and distributed would be beneficial for both users/developers and auditors.

### Recommended Mitigation Steps

Improve fee calculation documentation.
