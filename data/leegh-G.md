## [G-01] Multiplication and division in `getPriceAndFee` can be moved out of the loop.
Multiplication and division can be moved out of the loop, which saves gas and prevents trivial loss of precision.
```solidity
File: 1155tech-contracts/src/bonding_curve/LinearBondingCurve.sol
origin:
14:    function getPriceAndFee(uint256 shareCount, uint256 amount)
15:        external
16:        view
17:        override
18:        returns (uint256 price, uint256 fee)
19:    {
20:        for (uint256 i = shareCount; i < shareCount + amount; i++) {
21:            uint256 tokenPrice = priceIncrease * i;
22:            price += tokenPrice;
23:            fee += (getFee(i) * tokenPrice) / 1e18;
24:        }
25:    }

optimized:
14:    function getPriceAndFeeNew2(uint256 shareCount, uint256 amount)
15:        external
16:        view
17:        returns (uint256 price, uint256 fee)
18:    {
19:        uint256 endShareCount = shareCount + amount;
20:        for (uint256 i = shareCount; i < endShareCount; i++) {
21:            price += i;
22:            fee += getFee(i) * i;
23:        }
24:        price = price * priceIncrease;
25:        fee = fee * priceIncrease / 1e18;
26:    }
```
https://github.com/code-423n4/2023-11-canto/blob/main/1155tech-contracts/src/bonding_curve/LinearBondingCurve.sol#L14-L25

## [G-02] Add a new function to get price only in bonding curve contract for `getNFTMintingPrice`.
The `fee` returned by `getPriceAndFee` is not used in `getNFTMintingPrice` (L196). It's better to add a new function named `getPrice` to only calculate the price, to save the gas for calculating fee in `getPriceAndFee`.
```solidity
File: 1155tech-contracts/src/Market.sol

194:    function getNFTMintingPrice(uint256 _id, uint256 _amount) public view returns (uint256 fee) {
195:        address bondingCurve = shareData[_id].bondingCurve;
196:        (uint256 priceForOne, ) = IBondingCurve(bondingCurve).getPriceAndFee(shareData[_id].tokenCount, 1);
197:        fee = (priceForOne * _amount * NFT_FEE_BPS) / 10_000;
198:    }
```
https://github.com/code-423n4/2023-11-canto/blob/main/1155tech-contracts/src/Market.sol#L194-L198
```solidity
File: 1155tech-contracts/src/bonding_curve/LinearBondingCurve.sol

14:    function getPriceAndFee(uint256 shareCount, uint256 amount)
15:        external
16:        view
17:        override
18:        returns (uint256 price, uint256 fee)
19:    {
20:        for (uint256 i = shareCount; i < shareCount + amount; i++) {
21:            uint256 tokenPrice = priceIncrease * i;
22:            price += tokenPrice;
23:            fee += (getFee(i) * tokenPrice) / 1e18;
24:        }
25:    }
```
https://github.com/code-423n4/2023-11-canto/blob/main/1155tech-contracts/src/bonding_curve/LinearBondingCurve.sol#L14-L25