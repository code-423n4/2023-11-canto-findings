# [QA-01] It's possible to create multiple of asD tokens with the same name and symbol

[File: asDFactory.sol](https://github.com/code-423n4/2023-11-canto/blob/main/asD/src/asDFactory.sol#L33)
```
function create(string memory _name, string memory _symbol) external returns (address) {
        asD createdToken = new asD(_name, _symbol, msg.sender, cNote, owner());
        isAsD[address(createdToken)] = true;
        emit CreatedToken(address(createdToken), _symbol, _name, msg.sender);
        return address(createdToken);
    }
```

Function `create()` does not verify if token with the same name and symbol already exist. Since function is `external`, anyone can call it and create own token.
Allowing creating multiple of different tokens with the same name and symbol may be misleading to the end user and may increase number of scams (malicious actors will be creating the same tokens with different addresses).

Our recommendation is to verify if asD token with the same name and symbol already exists.

 

 # [QA-02] Lack of 0-check in `LinearBondingCurve.sol` in `constructor()`

 Whenever contract is deployed with `priceIncrease`, function `getPriceAndFee()` will always return 0.

[File: LinearBondingCurve.sol](https://github.com/code-423n4/2023-11-canto/blob/335930cd53cf9a137504a57f1215be52c6d67cb3/1155tech-contracts/src/bonding_curve/LinearBondingCurve.sol#L20)
```
 for (uint256 i = shareCount; i < shareCount + amount; i++) {
            uint256 tokenPrice = priceIncrease * i;
            price += tokenPrice;
            fee += (getFee(i) * tokenPrice) / 1e18;
        }
```

Since `priceIncrease` defines `how much the price increases per share`, whenever this value is set to 0, both `tokenPrice` and `fee` will be `0` (since in both cases, we multiply by `priceIncrease`).



# [QA-03] Inconsistency in documentation about share creation

According to documentation, share creation can be: ["open for all or only for some whitelisted addresses"](https://github.com/code-423n4/2023-11-canto/blob/335930cd53cf9a137504a57f1215be52c6d67cb3/README.md?plain=1#L100).

Whenever share creation is restricted, only whitelisted addresses should be able to create new shares.
However, `onlyShareCreator()` modifier demonstrates, that when share creation is restricted, both whitelisted addresses AND the owner can create new shares:

[File: Market.sol](https://github.com/code-423n4/2023-11-canto/blob/335930cd53cf9a137504a57f1215be52c6d67cb3/1155tech-contracts/src/Market.sol#L82)
```
 modifier onlyShareCreator() {
        require(
            !shareCreationRestricted || whitelistedShareCreators[msg.sender] || msg.sender == owner(),
            "Not allowed"
        );
        _;
    }
```

Update the documentation, to let the users know, that when share creation is disabled, both whitelisted addresses and the owner can create new shares.


# [QA-04] `changeShareCreatorWhitelist()` does not emit an event

Function `changeShareCreatorWhitelist` adds or removes an address from the whitelist of share creator. However, it does not emit an event, whenever address is being added/removed from the whitelist

[File: Market.sol](https://github.com/code-423n4/2023-11-canto/blob/335930cd53cf9a137504a57f1215be52c6d67cb3/1155tech-contracts/src/Market.sol#L309C14-L309C41)
```
    function changeShareCreatorWhitelist(address _address, bool _isWhitelisted) external onlyOwner {
        require(whitelistedShareCreators[_address] != _isWhitelisted, "State already set");
        whitelistedShareCreators[_address] = _isWhitelisted;
    }
```


# [QA-05] Unfair fee split in `Market.sol`

[File: Market.sol](https://github.com/code-423n4/2023-11-canto/blob/335930cd53cf9a137504a57f1215be52c6d67cb3/1155tech-contracts/src/Market.sol#L289)
```
 if (_tokenCount > 0) {
            shareData[_id].shareHolderRewardsPerTokenScaled += (shareHolderFee * 1e18) / _tokenCount;
        } else {
            // If there are no tokens in circulation, the fee goes to the platform
            platformFee += shareHolderFee;
        }
```

Whenever there's no tokens in circulation, the whole fee goes to the platform. This favors platform over share creators, especially, for the first buys (when there's no tokens in circulations yet).
To balance the fee - our recommendation is to split the remaining fee (shareHolderFee) between share creator and platform:

```
        if (_tokenCount > 0) {
            shareData[_id].shareHolderRewardsPerTokenScaled += (shareHolderFee * 1e18) / _tokenCount;
        } else {
            
            uint feeToShare = shareHolderFee / 2;
            platformFee += feeToShare + feeToShare % 2; // % 2 to get the remaining 1, when feeToShare is not even
            shareData[_id].shareCreatorPool += feeToShare;
        }
```


# [N-01] Use constant for CSR address

To improve the code readability - it would be better to define CSR address as constant variable.

[File: asD.sol](https://github.com/code-423n4/2023-11-canto/blob/335930cd53cf9a137504a57f1215be52c6d67cb3/asD/src/asD.sol#L39)
```
Turnstile turnstile = Turnstile(0xEcf044C5B4b867CFda001101c617eCd347095B44);
```

[File: asDFactory.sol](https://github.com/code-423n4/2023-11-canto/blob/335930cd53cf9a137504a57f1215be52c6d67cb3/asD/src/asDFactory.sol#L28)
```
Turnstile turnstile = Turnstile(0xEcf044C5B4b867CFda001101c617eCd347095B44);
```

[File: Market.sol](https://github.com/code-423n4/2023-11-canto/blob/335930cd53cf9a137504a57f1215be52c6d67cb3/1155tech-contracts/src/Market.sol#L95)
```
Turnstile turnstile = Turnstile(0xEcf044C5B4b867CFda001101c617eCd347095B44);
```

# [N-02] Do not repeat the same block of code in multiple of files

```
        if (block.chainid == 7700 || block.chainid == 7701) {
            // Register CSR on Canto main- and testnet
            Turnstile turnstile = Turnstile(0xEcf044C5B4b867CFda001101c617eCd347095B44);
            turnstile.register(_csrRecipient);
        }
```

The code-block responsible for registering CSR on Canto main-net or testnet is used in multiple of files:
`asD.sol`, `asDFactory.sol`, `Market.sol`.


It will improve the code readability - when you implement this functionality as a function and call that function in above files.


# [N-03] Redundant `returnCode` variable in `asD.sol`

```
52:        uint256 returnCode = cNoteToken.mint(_amount);
53:        // Mint returns 0 on success: https://docs.compound.finance/v2/ctokens/#mint

63:        uint256 returnCode = cNoteToken.redeemUnderlying(_amount); // Request _amount of NOTE (the underlying of cNOTE)
64:        require(returnCode == 0, "Error when redeeming"); // 0 on success: https://docs.compound.finance/v2/ctokens/#redeem-underlying

85:        uint256 returnCode = CErc20Interface(cNote).redeemUnderlying(_amount);
86:        require(returnCode == 0, "Error when redeeming"); // 0 on success: https://docs.compound.finance/v2/ctokens/#redeem
```

Below code-snippets quote Compound documentation. In the documentation, the returnCode is used straightforwardly, e.g.:

```
require(cToken.redeemUnderlying(50) == 0, "something went wrong");
```

This implies, that declaration of `returnCode` is redundant.

While this issue should be reported in Gas report (redundant variable declaration saves gas) - I put this issue in QA Report.
Code-base straightforwardly points out to the Compound docs. Those docs provides straightforward examples of calling and checking return code for `mint()` and `redeemUnderlying()`.
For the code readability - the return code should be checked in the same way as it's done in the docs - without declaring additional variable.


# [N-04] Interpunction in `asD.sol` comments

[File: asD,sol](https://github.com/code-423n4/2023-11-canto/blob/335930cd53cf9a137504a57f1215be52c6d67cb3/asD/src/asD.sol#L71)
```
/// @dev The function checks that the owner does not withdraw too much NOTE, i.e. that a 1:1 NOTE:asD exchange rate can be maintained after the withdrawal
```


There should be comma after `i.e.` - `i.e.,` - [reference](https://english.stackexchange.com/a/16215)

# [N-05] Use constants instead of hard-coded numbers


Using constant instead of hard-coded numbers (such as: 1e28, 1e17, 1e18, 10_000), will improve code readability.


[File: asD.sol](https://github.com/code-423n4/2023-11-canto/blob/335930cd53cf9a137504a57f1215be52c6d67cb3/asD/src/asD.sol#L76)
```
1e28 - 
```

[File: LinearBondingCurve.sol](https://github.com/code-423n4/2023-11-canto/blob/335930cd53cf9a137504a57f1215be52c6d67cb3/1155tech-contracts/src/bonding_curve/LinearBondingCurve.sol#L23)
```
fee += (getFee(i) * tokenPrice) / 1e18;
```

[File: LinearBondingCurve.sol](https://github.com/code-423n4/2023-11-canto/blob/335930cd53cf9a137504a57f1215be52c6d67cb3/1155tech-contracts/src/bonding_curve/LinearBondingCurve.sol#L35)
```
 return 1e17 / divisor;
```

[File: Market.sol](https://github.com/code-423n4/2023-11-canto/blob/335930cd53cf9a137504a57f1215be52c6d67cb3/1155tech-contracts/src/Market.sol#L276)
```
1e18;
```

[File: Market.sol](https://github.com/code-423n4/2023-11-canto/blob/335930cd53cf9a137504a57f1215be52c6d67cb3/1155tech-contracts/src/Market.sol#L290)
```
shareData[_id].shareHolderRewardsPerTokenScaled += (shareHolderFee * 1e18) / _tokenCount;
```

[File: Market.sol](https://github.com/code-423n4/2023-11-canto/blob/335930cd53cf9a137504a57f1215be52c6d67cb3/1155tech-contracts/src/Market.sol#L197)
```
fee = (priceForOne * _amount * NFT_FEE_BPS) / 10_000;
```

[File: Market.sol](https://github.com/code-423n4/2023-11-canto/blob/335930cd53cf9a137504a57f1215be52c6d67cb3/1155tech-contracts/src/Market.sol#L285-L286)
```
        uint256 shareHolderFee = (_fee * HOLDER_CUT_BPS) / 10_000;
        uint256 shareCreatorFee = (_fee * CREATOR_CUT_BPS) / 10_000;
```