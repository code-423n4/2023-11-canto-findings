# L-1 ~ createNewShare() only allows anyone to create new shares.
In the docs it is stated that
> Market.createNewShare is used to create a new share. Share creation can be completely permissionless or it can be restricted to whitelisted addresses only. No fee is charged for the creation of new shares.

Link [here](https://code4rena.com/contests/2023-11-canto-application-specific-dollars-and-bonding-curves-for-1155s#top)

in the [createNewShare()](https://github.com/code-423n4/2023-11-canto/blob/335930cd53cf9a137504a57f1215be52c6d67cb3/1155tech-contracts/src/Market.sol#L110C1-L127C6) function this is not implemented in its logic.
## Mitigation
create a boolean state variable to check if anyone should be allowed to create new shares or not, and in a situation where not everyone is allowed the msg.sender(caller of the function) address should be checked to ensure it is whitelisted

# L-2 ~ No reentrancy guard on critical functions that are prone to reentrancy.
In market.sol, most of the functions minting nfts can be reentered, as with the use of safeTransferFrom low-level calls are made for certain checks, these functions are not properly protected from reentrancy as they like the non-reentrant modifier.
## Mitigation
make use of OZ reentrancy guard to ensure against reentrancy.

# L-3 ~ getPriceAndFee() can return an incorrect fee.
in [getPriceAndFee()](https://github.com/code-423n4/2023-11-canto/blob/335930cd53cf9a137504a57f1215be52c6d67cb3/1155tech-contracts/src/bonding_curve/LinearBondingCurve.sol#L14C1-L25C6) when calculating the fee to be returned under the hood division is done before multiplication on the call to getFee(), 
**POC**
```
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

    function getFee(uint256 shareCount) public pure override returns (uint256) {
        uint256 divisor;
        if (shareCount > 1) {
            divisor = log2(shareCount);
        } else {
            divisor = 1;
        }
        // 0.1 / log2(shareCount)
        return 1e17 / divisor;
    }
```
This will lead to an incorrect amount of fee returned for certain `amount`s/`shareCount`s.
## Mitigation
Ensure multiplication is done before division, in this instance
# L-4 ~ Not enough docs on the contracts and main functions flow of Canto ASD & Bonding Curves for 1155s
The docs for the audit lack functions logic supposed implementation and more things that would help give a better understanding to users and auditors outside of the developers of the protocol.
## Mitigation
add more documentation that would put a lot of supposed flows and intrinsic logics to be better understood.
# NC-5 ~ `priceIncrease` should not be immutable.
[priceIncrease](https://github.com/code-423n4/2023-11-canto/blob/335930cd53cf9a137504a57f1215be52c6d67cb3/1155tech-contracts/src/bonding_curve/LinearBondingCurve.sol#L8C30-L8C43) in LinearBondingCurve.sol can never be changed after deployment, this can lead to issues if the owner wants to change the amount of fees charged from users of the protocol.
## Mitigation
create a function to be able to change `priceIncrease` but put a min and max range to protect users from the owners.