## [GAS-01] Use shift Right/Left instead of division/multiplication if possible

Instances (4):

```
 fee = (priceForOne * _amount * NFT_FEE_BPS) / 10_000;
```
https://github.com/code-423n4/2023-11-canto/blob/main/1155tech-contracts/src/Market.sol#L197C8-L197C62

```
 uint256 shareHolderFee = (_fee * HOLDER_CUT_BPS) / 10_000;
```
https://github.com/code-423n4/2023-11-canto/blob/main/1155tech-contracts/src/Market.sol#L285C8-L285C67

```
 uint256 shareCreatorFee = (_fee * CREATOR_CUT_BPS) / 10_000;
```
https://github.com/code-423n4/2023-11-canto/blob/main/1155tech-contracts/src/Market.sol#L286C8-L286C69

```
 shareData[_id].shareHolderRewardsPerTokenScaled += (shareHolderFee * 1e18) / _tokenCount;
```
https://github.com/code-423n4/2023-11-canto/blob/main/1155tech-contracts/src/Market.sol#L290C12-L290C102

## [GAS-02] internal functions not called by the contract should be removed
If the functions are required by an interface, the contract should inherit from that interface and use the override keyword

Instances (1):

```
 function _getRewardsSinceLastClaim(uint256 _id) internal view returns (uint256 amount) {
```
https://github.com/code-423n4/2023-11-canto/blob/main/1155tech-contracts/src/Market.sol#L272C4-L272C93

## [GAS-03] Don't initialize variables with default value

Instances (2):

```
 platformPool = 0;
```
https://github.com/code-423n4/2023-11-canto/blob/main/1155tech-contracts/src/Market.sol#L246C8-L246C26

```
 shareData[_id].shareCreatorPool = 0;
```
https://github.com/code-423n4/2023-11-canto/blob/main/1155tech-contracts/src/Market.sol#L256C8-L256C45

## [GAS-04] Long revert strings

Instances (2):

```
 require(whitelistedBondingCurves[_bondingCurve], "Bonding curve not whitelisted");
```
https://github.com/code-423n4/2023-11-canto/blob/main/1155tech-contracts/src/Market.sol#L119C8-L119C91

```
 require(_amount <= maximumWithdrawable, "Too many tokens requested");
```
https://github.com/code-423n4/2023-11-canto/blob/main/asD/src/asD.sol#L81C12-L81C82

## [G-05] IF’s/require() statements that check input arguments should be at the top of the function

Instances (3):

```
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
https://github.com/code-423n4/2023-11-canto/blob/main/asD/src/asD.sol#L47C4-L56C6

```
  function burn(uint256 _amount) external {
        CErc20Interface cNoteToken = CErc20Interface(cNote);
        IERC20 note = IERC20(cNoteToken.underlying());
        uint256 returnCode = cNoteToken.redeemUnderlying(_amount); // Request _amount of NOTE (the underlying of cNOTE)
        require(returnCode == 0, "Error when redeeming"); // 0 on success: https://docs.compound.finance/v2/ctokens/#redeem-underlying
        _burn(msg.sender, _amount);
        SafeERC20.safeTransfer(note, msg.sender, _amount);
    }
```
https://github.com/code-423n4/2023-11-canto/blob/main/asD/src/asD.sol#L60C3-L67C6

```
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
https://github.com/code-423n4/2023-11-canto/blob/main/asD/src/asD.sol#L72C4-L90C6

## [G-06] Use bytes32 rather than string/bytes.
If you can fit your data in 32 bytes, then you should use bytes32 datatype rather than bytes or strings
as it is much cheaper in solidity. Basically, Any fixed size variable in solidity is cheaper than variable
size.

Instances (4):

```
 string metadataURI; // URI of the metadata
```
https://github.com/code-423n4/2023-11-canto/blob/main/1155tech-contracts/src/Market.sol#L39C8-L39C51

```
 event ShareCreated(uint256 indexed id, string name, address indexed bondingCurve, address indexed creator);
```
https://github.com/code-423n4/2023-11-canto/blob/main/1155tech-contracts/src/Market.sol#L70C4-L70C112

```
 function create(string memory _name, string memory _symbol) external returns (address) {
```
https://github.com/code-423n4/2023-11-canto/blob/main/asD/src/asDFactory.sol#L33C4-L33C93

```
string memory _name,
string memory _symbol,
```
https://github.com/code-423n4/2023-11-canto/blob/main/asD/src/asD.sol#L29C9-L30C31

## [G-07] Gas optimizations by using external over public
Using public over external has an impact on execution cost.
If we run the following methods on Remix, we can see the difference
// transaction cost 21448 gas
// execution cost 176 gas
function tt() external returns(uint256) {
return 0;
}
// transaction cost 21558 gas
// execution cost 286 gas
function tt_public() public returns(uint256) {
return 0;
}

Instances (3):

```
  function getBuyPrice(uint256 _id, uint256 _amount) public view returns (uint256 price, uint256 fee) {
```
https://github.com/code-423n4/2023-11-canto/blob/main/1155tech-contracts/src/Market.sol#L132C3-L132C106

```
function getSellPrice(uint256 _id, uint256 _amount) public view returns (uint256 price, uint256 fee) {
```
https://github.com/code-423n4/2023-11-canto/blob/main/1155tech-contracts/src/Market.sol#L141C5-L141C107

```
 function getNFTMintingPrice(uint256 _id, uint256 _amount) public view returns (uint256 fee) {
```
https://github.com/code-423n4/2023-11-canto/blob/main/1155tech-contracts/src/Market.sol#L194C4-L194C98