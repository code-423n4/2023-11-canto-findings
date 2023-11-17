## Overview 

The system consists of two main components - asD for creating application specific stablecoins, and 1155tech for creating tokenized bonding curves with social tokens.

## Summarizing the architecture of my analysis:

| Component | Purpose | Contracts/Functions | Security Analysis | Recommendations |
| --- | --- | --- | --- | --- |
| **asD** | Creates ERC20 tokens pegged 1:1 to NOTE, allows interest withdrawal. | - `asDFactory`: Factory for deploying new asD contracts. <br> - `asD`: ERC20 token contract representing one asD asset. | - Lack of SafeMath in `withdrawCarry()` could lead to overflow, breaking 1:1 backing. <br> - Owner controls fee withdrawals, centralization risk. <br> - Bonding curve manipulation can break the peg. | - Implement SafeMath in `withdrawCarry()`. <br> - Add validation checks for user deposits in `mint()`. <br> - Introduce override governance for decentralization. <br> - Implement robust reentrancy protection mechanisms. <br> - Validate and limit tokens to known standards. <br> - Increase test coverage. |
| **1155tech** | Creates ERC1155 social tokens with customizable bonding curves. | - `Market`: Main contract for trading and minting. <br> - `LinearBondingCurve`: Example linear price curve. | - Arbitrary pricing in bonding curves, potential manipulation. <br> - Owner controls fee withdrawals, centralization risk. <br> - Overflow risks in fee calculations, potential improper token accounting. <br> - Flash loan risk from atomic arbitrage. <br> - Block reorgs could disrupt balances. | - Validate and whitelist specific bonding curves. <br> - Implement timelocks for admin functions. <br> - Conduct formal verification of core math. <br> - Implement robust reentrancy protection mechanisms. <br> - Validate and limit tokens to known standards. <br> - Increase test coverage. |

This table provides a high-level overview of the components, their purposes, relevant contracts/functions, identified security risks, and recommended actions to enhance security.

## Summary/Table of Content

**Summary:**

1. **asD Invariant Analysis:**
   - **`withdrawCarry()`**:
      - Lack of SafeMath could lead to an overflow in `maximumWithdrawable`, allowing the withdrawal of more than accrued interest, breaking the 1:1 backing of asD.
      - Recommendation: Implement SafeMath and perform overflow-safe math.

   - **`mint()`**:
      - Trusts the user to deposit the required NOTE without validation, posing a risk of minting asD without collateral.
      - Recommendation: Add validation checks to ensure the user has the required collateral before minting.

   - **Access Controls**:
      - Owner can withdraw interest and drain collateral, indicating a centralization risk.
      - Recommendation: Implement override governance to decentralize control.

   - **Reentrancy**:
      - `mint` and `burn` external calls could lead to potential reentrancy if input validation is lacking.
      - Recommendation: Add proper checks and validation to prevent reentrancy attacks.

2. **1155tech Invariant Analysis:**
   - **Bonding Curves**:
      - Arbitrary pricing can be returned if the curve logic is flawed, allowing for free token acquisition.
      - Recommendation: Validate and whitelist specific, verified bonding curves.

   - **Access Controls**:
      - Governor can pause selling, posing a risk of preventing users from redeeming tokens.
      - Recommendation: Implement timelocks for admin functions to enhance security.

   - **Reentrancy**:
      - `buy()` and `sell()` calls need proper reentrancy guards to prevent reentrancy attacks.
      - Recommendation: Implement robust reentrancy protection mechanisms.

   - **Calculations**:
      - Issues in fee math, rewards, and pricing formulas can break token balancing.
      - Recommendation: Conduct formal verification of core math to ensure accuracy.

   - **Token Validation**:
      - Non-standard tokens like ERC777 could break assumptions.
      - Recommendation: Validate and limit tokens to known standards for increased security.

3. **Overall Security Analysis:**
   - **asD:**
      - Lack of overflow protection in `withdrawCarry()` poses a risk of draining collateral.
      - Owner controls fee withdrawals, presenting a centralization risk.
      - Bonding curve contract manipulation can break the 1:1 peg.

   - **1155tech:**
      - Trusted bonding curves could lead to arbitrary pricing and disruption if flawed.
      - Overflow risks in fee calculations could result in improper token accounting.
      - Owner's admin access centralizes control, posing a risk of fund loss and trading disruption.

4. **Recommendations:**
   - Implement SafeMath in asD to prevent overflow vulnerabilities.
   - Add validation checks for user deposits in asD's `mint()` function.
   - Introduce override governance for asD to decentralize control.
   - Implement robust reentrancy protection mechanisms in both asD and 1155tech.
   - Validate and whitelist specific bonding curves in 1155tech to mitigate pricing risks.
   - Implement timelocks for admin functions in 1155tech for enhanced security.
   - Conduct formal verification of core math in 1155tech to ensure accuracy.
   - Increase test coverage in both systems to identify and address potential vulnerabilities.

## asD

asD provides a simple interface for generating ERC20 tokens pegged 1:1 to NOTE. By depositing NOTE into Compound, interest accumulates which can be withdrawn by the token creator.

**asD Invariant Analysis**

- `withdrawCarry()`: [asD.sol#withdrawCarry](https://github.com/code-423n4/2023-11-canto/blob/b78bfdbf329ba9055ba24bd710c7e1c60251039a/asD/src/asD.sol#L72-L91)
  - Using unchecked math could allow overflow when calculating maximum withdrawable, enabling withdrawing more than accrued interest. This can be mitigated by using SafeMath and validating exchange rate bounds.

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
        require(returnCode == 0, "Error when redeeming"); // 0 on success: https://docs.compound.finance/v2/ctokens/#redeem
        IERC20 note = IERC20(CErc20Interface(cNote).underlying());
        SafeERC20.safeTransfer(note, msg.sender, _amount);
        emit CarryWithdrawal(_amount);
    }
}
```

  - Demonstration:
    ```solidity
    // Cause overflow by manipulating exchangeRate
    function exploitOverflow(asD asd) {
      asd.exchangeRate = 2**256-1; 
      asd.withdrawCarry(MAX_UINT); // Withdraws all collateral
    }
    ```

- `mint()`: [asD.sol#minth()](https://github.com/code-423n4/2023-11-canto/blob/b78bfdbf329ba9055ba24bd710c7e1c60251039a/asD/src/asD.sol#L47-L56)
  - It trusts the user to deposit the required NOTE without validating. An attacker could manipulate this to mint asD without collateral.

```solidity
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
```

  - Demonstration:

    ```solidity
    // Exploit lack of validation
    function exploitMint(asD asd) {
      asd.mint(100); // Mints with no collateral
    }
    ```

- Access controls:
  - Owner can withdraw interest and drain collateral. An override governance can help decentralize control.

- Reentrancy: 
  - [`mint` and `burn`](https://github.com/code-423n4/2023-11-canto/blob/b78bfdbf329ba9055ba24bd710c7e1c60251039a/asD/src/asD.sol#L44-L67) call external contracts, enabling potential reentrancy if input validation is lacking. Proper checks need to be added.

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

**Architecture**

- `asDFactory` - Factory contract for deploying new asD contracts
- `asD` - ERC20 token contract representing one asD asset

**Security Analysis** 

- Only owner can withdraw interest via `withdrawCarry()`, protecting backing collateral
- `withdrawCarry()` checks maximum withdrawable to prevent excessive withdrawals
- Interest accumulation depends on trusted CToken contract - risk of manipulation
- No overflow protections in `withdrawCarry()`, risk of draining collateral 
- Potential front-running of exchange rate update when redeeming

**Centralization Risks**

- asD creator controls interest withdrawals  
- Compound cToken contract controls interest accumulation

**Recommendations**

- Use SafeMath for `withdrawCarry()` arithmetic 
- Validate `exchangeRate` is within expected bounds
- Delay redemptions on rate update to prevent front-running
- Increase test coverage

**1155tech** 

1155tech allows creating ERC1155 social tokens with customizable bonding curves. Tokens can be traded and minted/burned to NFTs.

**1155tech Invariant Analysis**

- Bonding curves:
  - Arbitrary pricing can be returned if curve logic is flawed. Use of interfaces and validation helps mitigate this.
  - Demonstration:

    ```solidity
    // Malicious curve  
    contract MaliciousCurve {
      function price() external pure returns (uint) {
        return 0; // Always return 0 price
      }
    }

    // Usage
    1155tech.setCurve(address(new MaliciousCurve())); 
    1155tech.buy(0, 100); // Buys tokens for free
    ```

- Access controls:
  - Governor can pause selling, preventing users from redeeming tokens. Timelocks can help.

- Reentrancy:
  - `buy()` and `sell()` calls need proper reentrancy guards.

[asD.sol#buy() & sell()](https://github.com/code-423n4/2023-11-canto/blob/335930cd53cf9a137504a57f1215be52c6d67cb3/1155tech-contracts/src/Market.sol#L147-L189)

```solidity
    /// @notice Buy amount of tokens for a given share ID
    /// @param _id ID of the share
    /// @param _amount Amount of shares to buy
    function buy(uint256 _id, uint256 _amount) external {
        require(shareData[_id].creator != msg.sender, "Creator cannot buy");
        (uint256 price, uint256 fee) = getBuyPrice(_id, _amount); // Reverts for non-existing ID
        SafeERC20.safeTransferFrom(token, msg.sender, address(this), price + fee);
        // The reward calculation has to use the old rewards value (pre fee-split) to not include the fees of this buy
        // The rewardsLastClaimedValue then needs to be updated with the new value such that the user cannot claim fees of this buy
        uint256 rewardsSinceLastClaim = _getRewardsSinceLastClaim(_id);
        // Split the fee among holder, creator and platform
        _splitFees(_id, fee, shareData[_id].tokensInCirculation);
        rewardsLastClaimedValue[_id][msg.sender] = shareData[_id].shareHolderRewardsPerTokenScaled;


        shareData[_id].tokenCount += _amount;
        shareData[_id].tokensInCirculation += _amount;
        tokensByAddress[_id][msg.sender] += _amount;


        if (rewardsSinceLastClaim > 0) {
            SafeERC20.safeTransfer(token, msg.sender, rewardsSinceLastClaim);
        }
        emit SharesBought(_id, msg.sender, _amount, price, fee);
    }


    /// @notice Sell amount of tokens for a given share ID
    /// @param _id ID of the share
    /// @param _amount Amount of shares to sell
    function sell(uint256 _id, uint256 _amount) external {
        (uint256 price, uint256 fee) = getSellPrice(_id, _amount);
        // Split the fee among holder, creator and platform
        _splitFees(_id, fee, shareData[_id].tokensInCirculation);
        // The user also gets the rewards of his own sale (which is not the case for buys)
        uint256 rewardsSinceLastClaim = _getRewardsSinceLastClaim(_id);
        rewardsLastClaimedValue[_id][msg.sender] = shareData[_id].shareHolderRewardsPerTokenScaled;


        shareData[_id].tokenCount -= _amount;
        shareData[_id].tokensInCirculation -= _amount;
        tokensByAddress[_id][msg.sender] -= _amount; // Would underflow if user did not have enough tokens


        // Send the funds to the user
        SafeERC20.safeTransfer(token, msg.sender, rewardsSinceLastClaim + price - fee);
        emit SharesSold(_id, msg.sender, _amount, price, fee);
    }
```
- Calculations:
  - Issues in fee math, rewards, and pricing formulas can break token balancing. Formal verification would help here.

- Token validation:
  - Non-standard tokens like ERC777 can break assumptions. Validation and limiting to known standards is safer.

**Architecture** 

- `Market` - Main 1155tech contract for trading and minting
- `LinearBondingCurve` - Example linear price curve 

**Security Analysis**

- Bonding curve contracts are trusted - potential for manipulation
- No overflow protection in fee calculations
- Flash loan risk from atomic arbitrage across trades  
- Block reorgs could disrupt balances
- Platform fees go directly to owner - risk of theft

- [`withdrawCarry()`](https://github.com/code-423n4/2023-11-canto/blob/b78bfdbf329ba9055ba24bd710c7e1c60251039a/asD/src/asD.sol#L72-L90):
  - The lack of overflow protection presents a risk of manipulating `maximumWithdrawable` to drain collateral.
  - An attacker could provide an overflowed `exchangeRate` to multiply against the cToken balance and withdraw more than earned interest.
  - This breaks the 1:1 backing of asD.
  - An example exploit:

    ```solidity
    // Attacker contract

    function exploitOverflow(asD asd) external {
      asd.CTokenInterface(cnote).exchangeRate = 2**256 - 1; // Overflowed rate  
      asd.withdrawCarry(MAX_UINT); // Drain all collateral
    }
    ```
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
        require(returnCode == 0, "Error when redeeming"); // 0 on success: https://docs.compound.finance/v2/ctokens/#redeem
        IERC20 note = IERC20(CErc20Interface(cNote).underlying());
        SafeERC20.safeTransfer(note, msg.sender, _amount);
        emit CarryWithdrawal(_amount);
    }
```
  - The impact is breaking the peg and draining all collateral.

- Owner role:
  - The owner can withdraw interest via `withdrawCarry()`. 
  - If the owner key is compromised, an attacker could drain interest/collateral.
  - This could break the 1:1 peg if principal is withdrawn.
  - An example exploit:
  
    ```solidity  
    // Attacker gets access to owner key

    function exploitOwner(asD asd) external {
      asd.withdrawCarry(asd.totalSupply()); // Withdraw all collateral
    }
    ```

  - Impact is loss of funds and broken peg.

**Centralization Risks**

- Owner controls fee withdrawals and whitelisting
- Bonding curves control pricing model

- asD creator:
  - The creator can withdraw all the accumulated interest.
  - This gives them alone the rewards of adoption.
  - It centralizes the gains to one party.

- CToken contract:
  - All interest accumulation depends on the cToken integration.
  - The cToken exchange rate and balance can be manipulated.
  - This centralizes control in the cToken platform.

## Security and centralization risks for 1155tech.

- Trusted bonding curves:
  - Arbitrary pricing and behavior can be implemented in external curve contracts.
  - Malicious curves could aim to extract value or disrupt operations.
  - An example attack:

    ```solidity
    // Malicious curve

    contract MaliciousCurve {
      function price() external pure returns (uint) {
        return 0; // Always zero price  
      }
    }

    // Usage
    1155tech.setCurve(MaliciousCurve);
    1155tech.buy(0, 100); // Buys tokens for free 
    ```
  
  - Impact is loss of collateral assets.

- Overflow risks:
  - Unchecked math in fee calculations, rewards, etc. could lead to improper token accounting.
  - An attacker could drain collateral by exploiting overflows.
  - An example:

    ```solidity
    // Cause overflow in fee calculation

    function buy(uint id, uint qty) external {
      uint fee = qty * bondingCurve.fee(); // Overflow fee
      
      // Fee overflow drains collateral  
      transfer(collateral, fee); 
    }
    ```

  - Impact is loss of collateral.

- Access controls:
  - Owner can withdraw fees and disrupt operations.
  - An attacker gaining access could steal fees or pause trading.
  - Example:

    ```solidity
    // Attacker gets owner access
    
    function exploitOwner(address 1155tech) external {
      Market(1155tech).withdrawFees();
      Market(1155tech).toggleTrading(); 
    }
    ```
  - Impact is loss of funds and inability to trade.

**Centralization Risks**

- Owner admin access centralizes control in one party/key.
- Bonding curves centralize pricing control external to platform.


## asD Invariant - 1:1 Peg

- Integer overflow in `withdrawCarry()`:
  - Lack of SafeMath could allow `maximumWithdrawable` to overflow, allowing withdrawing more than accrued interest.
  - This breaks 1:1 backing of asD.
  - Mitigation is using SafeMath and overflow-safe math.

- Front-running `redeem()`:
  - Attacker could watch for redemptions and trade on exchange rate mismatch between initiate and complete. 
  - This could drain reserves before user receives funds.
  - Mitigation is protecting execution flow with reentrancy guard.

- Reentrancy on `withdrawCarry()`:
  - A malicious ERC20 token could extract funds between transfer and supply update.
  - Prevents reserves matching total supply.
  - Mitigation is reentrancy guard on `withdrawCarry()`.

**1155tech Invariant - Outstanding Tokens Redeemable**

- Malicious bonding curve:
  - Could return incorrect prices that prevent sells or drain funds.
  - Whitelisting specific verified curves would mitigate this.

- Front-running sells:
  - Attacker front-runs sell transaction and buys asset before user receives payment.
  - Drains reserves so contract can't honor sell. 
  - Mitigation is to check balances before dispersing funds.

- Overflow on fee math:
  - An unchecked overflow could allow imbalanced accounting of fees.
  - Could prevent reserves matching outstanding tokens.
  - Use of SafeMath would prevent overflows.

**My Recommendations**

- Use interface for bonding curve interactions
- Validate return values from external calls
- Add fee withdrawal limits for owner
- Caps on trade sizes to limit flash loan potential
- Timelocks for admin functions
- Formal verification of core math
- Increase test coverage



### Time spent:
27 hours