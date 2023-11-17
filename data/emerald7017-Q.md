No access controls on [mint() and burn()](https://github.com/code-423n4/2023-11-canto/blob/b78bfdbf329ba9055ba24bd710c7e1c60251039a/asD/src/asD.sol#L44-L67). Anyone can mint/burn tokens if they have NOTE approval. This is likely by design but should be called out.

```solidity
    /// @notice Mint amount of asD tokens by providing NOTE. The NOTE:asD exchange rate is always 1:1
    /// @param _amount Amount of tokens to mint
    /// @dev User needs to approve the asD contract for _amount of NOTE
    function mint(uint256 _amount) external {
        CErc20Interface cNoteToken = CErc20Interface(cNote);
        IERC20 note = IERC20(cNoteToken.underlying());
        SafeERC20.safeTransferFrom(note, msg.sender, address(this), _amount);
        SafeERC20.safeApprove(note, cNote, _amount);
        uint256 returnCode = cNoteToken.mint(_amount);
        // Mint returns 0 on success: https://docs.compound.finance/v2/ctokens/#mint
        require(returnCode == 0, "Error when minting");
        _mint(msg.sender, _amount);
    }


    /// @notice Burn amount of asD tokens to get back NOTE. Like when minting, the NOTE:asD exchange rate is always 1:1
    /// @param _amount Amount of tokens to burn
    function burn(uint256 _amount) external {
        CErc20Interface cNoteToken = CErc20Interface(cNote);
        IERC20 note = IERC20(cNoteToken.underlying());
        uint256 returnCode = cNoteToken.redeemUnderlying(_amount); // Request _amount of NOTE (the underlying of cNOTE)
        require(returnCode == 0, "Error when redeeming"); // 0 on success: https://docs.compound.finance/v2/ctokens/#redeem-underlying
        _burn(msg.sender, _amount);
        SafeERC20.safeTransfer(note, msg.sender, _amount);
    }
```

The asD contract is implementing a very simple 1:1 pegged token model. This means 1 asD can always be redeemed for 1 NOTE from the reserve. 

The part of the code is in the `mint` and `burn` functions.

```solidity
// mint
SafeERC20.safeTransferFrom(note, msg.sender, address(this), _amount); 
// Transfer NOTE from user to contract

cNoteToken.mint(_amount);  
// Mint cNote tokens 

_mint(msg.sender, _amount);
// Mint asD tokens

// burn 
cNoteToken.redeemUnderlying(_amount);
// Redeem cNote for NOTE 

_burn(msg.sender, _amount);
// Burn asD tokens

SafeERC20.safeTransfer(note, msg.sender, _amount);
// Transfer NOTE to user
```

When minting asD, the user transfers NOTE into the contract, which mints the same amount of cNote. It then mints the same amount of asD to the user. 

When burning, it redeems cNote for NOTE, burns the asD, and sends the NOTE back to the user.

This keeps the 1:1 peg by directly linking asD minting/burning to the NOTE reserves.

The lack of access control is because **anyone who has NOTE approved can mint and burn**. This provides flexibility and transparency, aligning with the simple 1:1 model.

However, the downside is that the contract owner has no ability to stop malicious minting/burning. Access could be abused if NOTE is readily available.

Summary:

- The open access appears to be an intentional design decision to increase transparency and liquidity. 

- It does mean the owner cannot stop malicious actors from manipulating the token supply if they access NOTE.

- Additional access controls would deviation from the simplistic 1:1 model.

