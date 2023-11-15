[Don't support deflationary token as payment token in `Market`]

Take `Market::buy()` function as example, if `token` is deflationary token (has fee in the `transfer()/transferFrom()`), the market contract can't receive exactly the expected amount of payment from the `SafeERC20.safeTransferFrom(...)`.

    function buy(uint256 _id, uint256 _amount) external {
        require(shareData[_id].creator != msg.sender, "Creator cannot buy");
        (uint256 price, uint256 fee) = getBuyPrice(_id, _amount); // Reverts for non-existing ID
        SafeERC20.safeTransferFrom(token, msg.sender, address(this), price + fee);
        ...
    }

A mitigation is to add a whitelist for the supported payment tokens.
Another option is to check the balance changes before and after the `SafeERC20.safeTransferFrom(...)`, and use the increased balance in the market contract as the received payment.