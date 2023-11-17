# Canto Application Specific Dollars and Bonding Curves for 1155s Analysis

# Security Assessment Summary

### Scope

The following smart contracts were in scope of the audit:

- `1155tech-contracts/src/Market.sol`
- `1155tech-contracts/src/bonding_curve/LinearBondingCurve.sol`
- `asD/src/asDFactory.sol`
- `asD/src/asD.sol`

The following number of issues were found, categorized by their severity:

- Critical & High: 0 issues
- Medium: 0 issues
- Low: 4 issues
- Info: 1 issue

---

# Findings Summary

| ID     | Title                                                    | Severity |
| ------ | -------------------------------------------------------- | -------- |
| [L-01] | A mint can happen with an amount of 0                    | Low      |
| [L-02] | Shares from a blacklisted creator can still be bought    | Low      |
| [L-03] | A blacklisted creator can claim his creator fee          | Low      |
| [L-04] | A burn can be done with an amount of 0                   | Low      |
| [N-01] | Unused mapping                                           | Info     |


# Findings Details

# [L-01] A mint can happen with an amount of 0

In market.sol for the following method:
```
function mintNFT(uint256 _id, uint256 _amount) external
```
If the input amount is 0, an NFT will still be minted and no tokens will be used from users to create the NFT, an `NFTCreated` event will also be triggered. The following test shows it.

```
    event NFTsCreated(uint256 indexed id, address indexed creator, uint256 amount, uint256 fee);

    function testMintWithAmountO() public {
        testBuy();    
        vm.expectEmit(true, true, false, true);
        emit NFTsCreated(1, address(this), 0, 0);
        market.mintNFT(1, 0);
    }
```

## Recommendations

Add a check on the amount to be strictly positive.

# [L-02] Shares from blacklisted creator can still be bought

In the `buy` method of `Market.sol`:

```
function buy(uint256 _id, uint256 _amount) external
```

There is no check done on the `_id` if the share. It can be from a blacklisted creator, it will still be possible to buy it.

## Recommendations

Add a check on the share ID using the available mappings to avoid the shares of blacklisted creators to be bought:
```
whitelistedShareCreators[shareData[id].creator]
```

# [L-03] A blacklisted creator can claim his creator fee

In a similar idea with [L-02], a creator that is not whitelisted is still able to claim its creator fee since there is no check performed on the creator in the claim fee method:

```
function claimCreatorFee(uint256 _id) external
```

## Recommendations

A check on the creator should be added in the method as follow:
```
    function claimCreatorFee(uint256 _id) external {
        require(shareData[_id].creator == msg.sender, "Not creator");                   // validate that the sender is the creator
        require(whitelistedShareCreators[msg.sender], "Creator is not whitelisted");    // validate that the creator is whitelisted
        [...]
    }
```

# [L-04] A burn can be done with an amount of 0

There is no check on the amount in the following method:
```
function burnNFT(uint256 _id, uint256 _amount) external
```
So the following test will pass and a `NFTsBurned` event will be triggered:
```
    event NFTsBurned(uint256 indexed id, address indexed burner, uint256 amount, uint256 fee);
    
    function testBurnWithAmount0() public {
        testBuy();
        market.mintNFT(1, 0);
        vm.expectEmit(true, true, false, true);
        emit NFTsBurned(1, address(this), 0, 0);
    }
```

## Recommendations

An additional check on the amount should be done in the method.
```
require(amount > 0, "The amount should be strictly positive");
```

# [N-01] Unused mapping

The following object in the `Market.sol` contract is not used anywhere in the contract:
```
mapping(uint256 => address) public shareBondingCurves;
```

## Recommendations
It should be removed from the contract.

### Time spent:
3 hours