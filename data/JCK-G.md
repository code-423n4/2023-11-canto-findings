

## Gas Optimizations

| Number | Issue | Instances |
|--------|-------|-----------|
|[G-01]| Possible Optimizations in Market.sol | 2 | 
|[G-02]| Use calldata instead of memory for immutable arguments  | 7 | 
|[G-03]| Transfer of 0 should be checked  | 6 | 
|[G-04]| Caching a variable that is used once just wastes Gas  | 1 | 
|[G-05]| Don’t cache calls that are only used once  | 4 | 
|[G-06]| Avoid Unnecessary Public Variables  | 10 | 
|[G-07]| Use hardcoded address instead of address(this)  | 5 | 
|[G-08]| Use assembly for loops  | 1 | 
|[G-09]| Combine events to save Glogtopic (375 gas)  | 3 | 
|[G-10]| For same condition checks use modifiers  | 2 | 
|[G-11]| Declare the variables outside the loop  | 1 | 
|[G-12]| Use uint256(1)/uint256(2) instead for true and false boolean states  | 6 | 
|[G-13]| require()/revert() strings longer than 32 bytes cost extra gas  | 10 | 


## [G-01] Possible Optimizations in Market.sol

General Optimization
Reducing the number of state variable updates can save gas. In Ethereum, every storage operation costs a significant amount of gas. Therefore, optimizing the contract to minimize storage operations can lead to substantial gas savings.

### Possible Optimization in function createNewShare()

In the createNewShare functions, the shareIDs[_shareName] mapping is updated twice and also the shareData[id] mapping is updated 3 time . This can be optimized to a single update, which would save gas.


```solidity
file: blob/main/1155tech-contracts/src/Market.sol

114   function createNewShare(
        string memory _shareName,
        address _bondingCurve,
        string memory _metadataURI
    ) external onlyShareCreator returns (uint256 id) {
        require(whitelistedBondingCurves[_bondingCurve], "Bonding curve not whitelisted");
        require(shareIDs[_shareName] == 0, "Share already exists");
        id = ++shareCount;
        shareIDs[_shareName] = id;
        shareData[id].bondingCurve = _bondingCurve;
        shareData[id].creator = msg.sender;
        shareData[id].metadataURI = _metadataURI;
        emit ShareCreated(id, _shareName, _bondingCurve, msg.sender);
    }
```
https://github.com/code-423n4/2023-11-canto/blob/main/1155tech-contracts/src/Market.sol#L114-L127


Before Gas value:
[PASS] testCreateNewShare() (gas: 161187)

After gas value:
[PASS] testCreateNewShare() (gas: 101098)


### Possible Optimization in function buy()

In the buy functions, the shareData[_id] mapping is updated 5 time . This can be optimized to a single update, which would save gas.


```solidity
file: blob/main/1155tech-contracts/src/Market.sol

150   function buy(uint256 _id, uint256 _amount) external {
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

```
https://github.com/code-423n4/2023-11-canto/blob/main/1155tech-contracts/src/Market.sol#L150-L169

Before Gas value:
[PASS] testBuy() (gas: 351400)  

After Gas value:
[PASS] testBuy() (gas: 312029)


## [G‑02] Use calldata instead of memory for immutable arguments

Mark data types as calldata instead of memory where possible. This makes it so that the data is not automatically loaded into memory. If the data passed into the function does not need to be changed (like updating values in an array), it can be passed in as calldata. The one exception to this is if the argument must later be passed into another function that takes an argument that specifies memory storage.

```solidity
file: blob/main/1155tech-contracts/src/Market.sol

114  function createNewShare(
        string memory _shareName,
        address _bondingCurve,
        string memory _metadataURI
    ) external onlyShareCreator returns (uint256 id) {

```
https://github.com/code-423n4/2023-11-canto/blob/main/1155tech-contracts/src/Market.sol#L114-L118

Before Gas value:
[PASS] testCreateNewShare() (gas: 161187)

After gas value:
[PASS] testCreateNewShare() (gas: 160342)

```solidity
file: blob/main/asD/src/asDFactory.sol

33  function create(string memory _name, string memory _symbol) external returns (address) {

```
https://github.com/code-423n4/2023-11-canto/blob/main/asD/src/asDFactory.sol#L33

Before Gas value:
[PASS] test_create_asD() (gas: 1380297)

After Gas value:
[PASS] test_create_asD() (gas: 1379611)


```solidity
file: main/1155tech-contracts/src/Market.sol

91    constructor(string memory _uri, address _paymentToken) ERC1155(_uri) Ownable() {

```
https://github.com/code-423n4/2023-11-canto/blob/main/1155tech-contracts/src/Market.sol#L91

```solidity
file: blob/main/asD/src/asD.sol

28    constructor(
        string memory _name,
        string memory _symbol,
        address _owner,
        address _cNote,
        address _csrRecipient
    ) ERC20(_name, _symbol) {

```
https://github.com/code-423n4/2023-11-canto/blob/main/asD/src/asD.sol#L28-L34

### Possible Optimization in function burnNFT()

In the burnNFT functions, the shareData[_id] mapping is updated 3 time . This can be optimized to a single update, which would save gas.

```solidity
file: blob/main/1155tech-contracts/src/Market.sol

226  function burnNFT(uint256 _id, uint256 _amount) external {
        uint256 fee = getNFTMintingPrice(_id, _amount);

        SafeERC20.safeTransferFrom(token, msg.sender, address(this), fee);
        _splitFees(_id, fee, shareData[_id].tokensInCirculation);
        // The user does not get the proportional rewards for the burning (unless they have additional tokens that are not in the NFT)
        uint256 rewardsSinceLastClaim = _getRewardsSinceLastClaim(_id);
        rewardsLastClaimedValue[_id][msg.sender] = shareData[_id].shareHolderRewardsPerTokenScaled;
        tokensByAddress[_id][msg.sender] += _amount;
        shareData[_id].tokensInCirculation += _amount;
        _burn(msg.sender, _id, _amount);

        SafeERC20.safeTransfer(token, msg.sender, rewardsSinceLastClaim);
        // ERC1155 already logs, but we add this to have the price information
        emit NFTsBurned(_id, msg.sender, _amount, fee);
    }


```
https://github.com/code-423n4/2023-11-canto/blob/main/1155tech-contracts/src/Market.sol#L226-L241


Before Gas value:
[PASS] testBurn() (gas: 442355)

After GAs value:
[PASS] testBurn() (gas: 423296)


## [G-03] Transfer of 0 should be checked

When you transfer 0 ether, the transaction still costs gas, even though no ether is actually transferred. The gas cost of a transfer transaction is determined by the amount of ether being transferred, plus a base gas cost. So, even though you're transferring 0 ether, you still have to pay the base gas cost

```solidity
file: blob/main/asD/src/asD.sol

50    SafeERC20.safeTransferFrom(note, msg.sender, address(this), _amount);

```

https://github.com/code-423n4/2023-11-canto/blob/main/asD/src/asD.sol#L50

### Foundry Test only for mint function

Before Gas value:
[PASS] testMint() (gas: 214044)

After Gas value:
[PASS] testMint() (gas: 83511)


```solidity
file: blob/main/asD/src/asD.sol

66    SafeERC20.safeTransfer(note, msg.sender, _amount);

88    SafeERC20.safeTransfer(note, msg.sender, _amount);

```
https://github.com/code-423n4/2023-11-canto/blob/main/asD/src/asD.sol#L66

```solidity
file: blob/main/1155tech-contracts/src/Market.sol

247    SafeERC20.safeTransfer(token, msg.sender, amount);

257    SafeERC20.safeTransfer(token, msg.sender, amount);

267    SafeERC20.safeTransfer(token, msg.sender, amount);

```
https://github.com/code-423n4/2023-11-canto/blob/main/1155tech-contracts/src/Market.sol#L247

## [G-04] Caching a variable that is used once just wastes Gas

No need to cache rewardsLastClaimedValue[_id][msg.sender]; as it’s being used once to save gas

```solidity
file: blob/main/1155tech-contracts/src/Market.sol

272   function _getRewardsSinceLastClaim(uint256 _id) internal view returns (uint256 amount) {
        uint256 lastClaimedValue = rewardsLastClaimedValue[_id][msg.sender];
        amount =
            ((shareData[_id].shareHolderRewardsPerTokenScaled - lastClaimedValue) * tokensByAddress[_id][msg.sender]) /
            1e18;
    }

```
https://github.com/code-423n4/2023-11-canto/blob/main/1155tech-contracts/src/Market.sol#L272-L277


Before Gase value:
[PASS] testBurn() (gas: 442355)
[PASS] testBuy() (gas: 351400)
[PASS] testGetBuyPrice() (gas: 173036)
[PASS] testMint() (gas: 398941)
[PASS] testSell() (gas: 347319)

Aftere Gas vlaue:
[PASS] testBurn() (gas: 442337)
[PASS] testBuy() (gas: 351394)
[PASS] testGetBuyPrice() (gas: 173036)
[PASS] testMint() (gas: 398929)
[PASS] testSell() (gas: 347307)


## [G-05] Don’t cache calls that are only used once


```solidity
file: blob/main/asD/src/asD.sol

52   uint256 returnCode = cNoteToken.mint(_amount);

63   uint256 returnCode = cNoteToken.redeemUnderlying(_amount); 

73   uint256 exchangeRate = CTokenInterface(cNote).exchangeRateCurrent()

85   uint256 returnCode = CErc20Interface(cNote).redeemUnderlying(_amount);

```
https://github.com/code-423n4/2023-11-canto/blob/main/asD/src/asD.sol#L52



## [G-06] Avoid Unnecessary Public Variables

Public storage variables increase the contract's size due to the implicit generation of public getter functions. This makes the contract larger and could increase deployment and interaction costs.

If you do not require other contracts to read these variables, consider making them private or internal.

```solidity
file: blob/main/1155tech-contracts/src/Market.sol

27    uint256 public shareCount;

30    mapping(string => uint256) public shareIDs;

43    mapping(uint256 => ShareData) public shareData;

46    mapping(uint256 => address) public shareBondingCurves;

49    mapping(address => bool) public whitelistedBondingCurves;

52    mapping(uint256 => mapping(address => uint256)) public tokensByAddress;

55    mapping(uint256 => mapping(address => uint256)) public rewardsLastClaimedValue;

58    uint256 public platformPool;

61    bool public shareCreationRestricted = true;

64    mapping(address => bool) public whitelistedShareCreators;

```
https://github.com/code-423n4/2023-11-canto/blob/main/1155tech-contracts/src/Market.sol#L27

## [G-07] Use hardcoded address instead of address(this)

Instead of using address(this), it is more gas-efficient to pre-calculate and use the hardcoded address. Foundry’s script.sol and solmate’s LibRlp.sol contracts can help achieve this. Refrences


```solidity
file:  blob/main/1155tech-contracts/src/Market.sol

153    SafeERC20.safeTransferFrom(token, msg.sender, address(this), price + fee);

206    SafeERC20.safeTransferFrom(token, msg.sender, address(this), fee);

229    SafeERC20.safeTransferFrom(token, msg.sender, address(this), fee);

```
https://github.com/code-423n4/2023-11-canto/blob/main/1155tech-contracts/src/Market.sol#L153


```solidity
file: blob/main/asD/src/asD.sol

50   SafeERC20.safeTransferFrom(note, msg.sender, address(this), _amount);

75   uint256 maximumWithdrawable = (CTokenInterface(cNote).balanceOf(address(this)) * exchangeRate) /

```
https://github.com/code-423n4/2023-11-canto/blob/main/asD/src/asD.sol#L50

## [G-08] Use assembly for loops

In the following instances, assembly is used for more gas efficient loops. The only memory slots that are manually used in the loops are scratch space (0x00-0x20), the free memory pointer (0x40), and the zero slot (0x60). This allows us to avoid using the free memory pointer to allocate new memory, which may result in memory expansion costs.

Note that in order to do this optimization safely we will need to cache and restore the free memory pointer after the loop. We will also set the zero slot (0x60) back to 0.


```solidity
file:  blob/main/1155tech-contracts/src/bonding_curve/LinearBondingCurve.sol

20   for (uint256 i = shareCount; i < shareCount + amount; i++) {
            uint256 tokenPrice = priceIncrease * i;
            price += tokenPrice;
            fee += (getFee(i) * tokenPrice) / 1e18;
        }

```
https://github.com/code-423n4/2023-11-canto/blob/main/1155tech-contracts/src/bonding_curve/LinearBondingCurve.sol#L20-L24



## [G-09] Combine events to save Glogtopic (375 gas)

Saves 2450 GAS
We can combine the events into one singular event to save two Glogtopic (375 gas) that would otherwise be paid for the additional events.

```solidity
file:  blob/main/1155tech-contracts/src/Market.sol

168    emit SharesBought(_id, msg.sender, _amount, price, fee);

188    emit SharesSold(_id, msg.sender, _amount, price, fee);
```
https://github.com/code-423n4/2023-11-canto/blob/main/1155tech-contracts/src/Market.sol#L168


```solidity
file: blob/main/1155tech-contracts/src/Market.sol

220   emit NFTsCreated(_id, msg.sender, _amount, fee);

240   emit NFTsBurned(_id, msg.sender, _amount, fee);

```
https://github.com/code-423n4/2023-11-canto/blob/main/1155tech-contracts/src/Market.sol#L220


```solidity
file: blob/main/1155tech-contracts/src/Market.sol

248   emit PlatformFeeClaimed(msg.sender, amount);

258   emit CreatorFeeClaimed(msg.sender, _id, amount);

269   emit HolderFeeClaimed(msg.sender, _id, amount);

```
https://github.com/code-423n4/2023-11-canto/blob/main/1155tech-contracts/src/Market.sol#L248


## [G-10] For same condition checks use modifiers

```solidity
file: blob/main/1155tech-contracts/src/Market.sol

165   if (rewardsSinceLastClaim > 0) {

216   if (rewardsSinceLastClaim > 0) {

```
https://github.com/code-423n4/2023-11-canto/blob/main/1155tech-contracts/src/Market.sol#L165

```solidity
file: blob/main/asD/src/asD.sol

54   require(returnCode == 0, "Error when minting");

64   require(returnCode == 0, "Error when redeeming"); // 0 on success: https://docs.compound.finance/v2/ctokens/#redeem-underlying

86   require(returnCode == 0, "Error when redeeming"); // 0 on success: https://docs.compound.finance/v2/ctokens/#redeem

```
https://github.com/code-423n4/2023-11-canto/blob/main/asD/src/asD.sol#L54

## [G-11] Declare the variables outside the loop

Per iterations saves 26 GAS

```solidity
file: src/bonding_curve/LinearBondingCurve.sol

20   for (uint256 i = shareCount; i < shareCount + amount; i++) {
            uint256 tokenPrice = priceIncrease * i;
            price += tokenPrice;
            fee += (getFee(i) * tokenPrice) / 1e18;
        }

```
https://github.com/code-423n4/2023-11-canto/blob/main/1155tech-contracts/src/bonding_curve/LinearBondingCurve.sol#L20-L24

## [G-12] Use uint256(1)/uint256(2) instead for true and false boolean states

Use uint256(1) and uint256(2) for true/false to avoid a Gwarmaccess (100 gas), and to avoid Gsset (20000 gas) when changing from ‘false’ to ‘true’, after having been ‘true’ in the past. see source:


```solidity
file: blob/main/asD/src/asDFactory.sol

35   isAsD[address(createdToken)] = true;

```
https://github.com/code-423n4/2023-11-canto/blob/main/asD/src/asDFactory.sol#L35

After Gas value:
[PASS] test_create_asD() (gas: 1380297)

Before gas Value:
[PASS] test_create_asD() (gas: 1360391)

```solidity
file: blob/main/asD/src/asDFactory.sol

15   mapping(address => bool) public isAsD;

```
https://github.com/code-423n4/2023-11-canto/blob/main/asD/src/asDFactory.sol#L15

```solidity
file: blob/main/1155tech-contracts/src/Market.sol

49  mapping(address => bool) public whitelistedBondingCurves;

61  bool public shareCreationRestricted = true;

64  mapping(address => bool) public whitelistedShareCreators;

78  event ShareCreationRestricted(bool isRestricted);

```
https://github.com/code-423n4/2023-11-canto/blob/main/1155tech-contracts/src/Market.sol#L49

## [G‑13] require()/revert() strings longer than 32 bytes cost extra gas

```solidity
file:  blob/main/1155tech-contracts/src/Market.sol

105    require(whitelistedBondingCurves[_bondingCurve] != _newState, "State already set");

119    require(whitelistedBondingCurves[_bondingCurve], "Bonding curve not whitelisted");

120    require(shareIDs[_shareName] == 0, "Share already exists");

151    require(shareData[_id].creator != msg.sender, "Creator cannot buy");

301    require(shareCreationRestricted != _isRestricted, "State already set");

310    require(whitelistedShareCreators[_address] != _isWhitelisted, "State already set");

```
https://github.com/code-423n4/2023-11-canto/blob/main/1155tech-contracts/src/Market.sol#L105 

```solidity
file: blob/main/asD/src/asD.sol

54    require(returnCode == 0, "Error when minting");

64    require(returnCode == 0, "Error when redeeming"); // 0 on success: https://docs.compound.finance/v2/ctokens/#redeem-underlying

81    require(_amount <= maximumWithdrawable, "Too many tokens requested");

86    require(returnCode == 0, "Error when redeeming"); // 0 on success: https://docs.compound.finance/v2/ctokens/#redeem

```
https://github.com/code-423n4/2023-11-canto/blob/main/asD/src/asD.sol#L54


