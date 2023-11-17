## \[G‑01\] Save gas by preventing zero amount in mint() and burn()

```
_mint(msg.sender, _id, _amount, "");
```

https://github.com/code-423n4/2023-11-canto/blob/main/1155tech-contracts/src/Market.sol#L214

```
_burn(msg.sender, _id, _amount);
```

https://github.com/code-423n4/2023-11-canto/blob/main/1155tech-contracts/src/Market.sol#L236

https://github.com/code-423n4/2023-11-canto/blob/main/asD/src/asD.sol#L65C8-L66C59

https://github.com/code-423n4/2023-11-canto/blob/main/asD/src/asD.sol#L55

## \[G-02\] Amounts should be checked for 0 before calling a transfer

It is considered a best practice to verify for zero values before executing any transfers within smart contract functions. This approach helps prevent unnecessary external calls and can effectively reduce gas costs.

This practice is particularly crucial when dealing with the transfer of tokens or ether, as sending these assets to an address with a zero value will result in the loss of those assets.

In Solidity, you can determine whether a value is zero by utilizing the == operator. Here's an example demonstrating how to check for a zero value before performing a transfer:

```
//example 
function transfer(address payable recipient, uint256 amount) public {
    require(amount > 0, "Amount must be greater than zero");
    recipient.transfer(amount);
}
```

```
SafeERC20.safeTransfer(token, msg.sender, amount);
```

https://github.com/code-423n4/2023-11-canto/blob/main/1155tech-contracts/src/Market.sol#L247

https://github.com/code-423n4/2023-11-canto/blob/main/1155tech-contracts/src/Market.sol#L257

https://github.com/code-423n4/2023-11-canto/blob/main/1155tech-contracts/src/Market.sol#L267

https://github.com/code-423n4/2023-11-canto/blob/main/asD/src/asD.sol#L88

https://github.com/code-423n4/2023-11-canto/blob/main/asD/src/asD.sol#L65C8-L66C59

https://github.com/code-423n4/2023-11-canto/blob/main/asD/src/asD.sol#L50

## \[G-03\] Use hardcode address instead `address(this)`

Instead of using `address(this)`, it is more gas-efficient to pre-calculate and use the hardcoded `address`. Foundry’s script.sol and solmate’s `LibRlp.sol` contracts can help achieve this.

References: <ins>https://book.getfoundry.sh/reference/forge-std/compute-create-address</ins>

<ins>https://twitter.com/transmissions11/status/1518507047943245824</ins>

```
SafeERC20.safeTransferFrom(token, msg.sender, address(this), price + fee);
```

https://github.com/code-423n4/2023-11-canto/blob/main/1155tech-contracts/src/Market.sol#L153

https://github.com/code-423n4/2023-11-canto/blob/main/1155tech-contracts/src/Market.sol#L206

https://github.com/code-423n4/2023-11-canto/blob/main/asD/src/asD.sol#L75

&nbsp;

## \[G-04\] Use uint256(1)/uint256(2) instead for true and false boolean states

https://github.com/code-423n4/2023-11-canto/blob/main/1155tech-contracts/src/Market.sol#L61

## \[G-05\] Expensive operation inside a for loop

```
for (uint256 i = shareCount; i < shareCount + amount; i++) {
            uint256 tokenPrice = priceIncrease * i;
            price += tokenPrice;
            fee += (getFee(i) * tokenPrice) / 1e18;
        }
```

[https://github.com/code-423n4/2023-11-canto/blob/main/1155tech-contracts/src/bonding\\\_curve/LinearBondingCurve.sol#L20C9-L24C10](https://github.com/code-423n4/2023-11-canto/blob/main/1155tech-contracts/src/bonding%5C_curve/LinearBondingCurve.sol#L20C9-L24C10)

## \[G-06\] Make for loop unchecked

The risk of for loops getting overflowed is extremely low. Because it always increments by 1 and is limited to the arrays length. Even if the arrays are extremely long, it will take a massive amount of time and gas to let the for loop overflow.

```
for (uint256 i = shareCount; i < shareCount + amount; i++) {
            uint256 tokenPrice = priceIncrease * i;
            price += tokenPrice;
            fee += (getFee(i) * tokenPrice) / 1e18;
        }
```

[https://github.com/code-423n4/2023-11-canto/blob/main/1155tech-contracts/src/bonding\\\_curve/LinearBondingCurve.sol#L20C9-L24C10](https://github.com/code-423n4/2023-11-canto/blob/main/1155tech-contracts/src/bonding%5C_curve/LinearBondingCurve.sol#L20C9-L24C10)

&nbsp;

&nbsp;