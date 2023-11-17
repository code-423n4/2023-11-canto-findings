# Abstract and Contents

The codebase consists of two distinct but related systems: **asD**, which includes a token and factory, and **market**, which includes a market and bonding curve. 

**asD** is a token pegged to NOTE and accrues rewards via the CNote token (to the owner of the token contract). It is a simple system with very few functionalities. However, the system lacks testing, and there is a possibility of missing problematic scenarios, considering its significant integration with CNote. This part of the analysis will mainly focus on the lack of testing suites and argue that the audit scope might not be enough to cover integration failures.

**market** is an ERC1155 implementation that allows the creation of shares, buying/selling shares, and minting/burning NFTs with these shares. For this part of the system, I will analyze weird incentives, testing suites, centralization risks, and potential single points of failure that may occur in the future if not addressed properly.

Although these two systems are related to each other (asD tokens are expected to be used within the market for the underlying token), I will just briefly analyze their interaction and possible failures.

## Application Spesific Dollars (asD)

This ERC20 implementation is a very basic system that exchanges the underlying (NOTE token) with asD token in a 1:1 exchange rate without any calculation requirement. Whenever minting occurs, NOTE will be taken from the user, and the contract will mint CNote tokens owned by the asD contract, which accrue interest. The contract will also mint asD tokens for the user. When burning, it will redeem CNote tokens and return the user's NOTE tokens to them, while burning the same amount (NOTE) of asD tokens from the user. CNote tokens' accrued interests are withdrawable by the contract owner at any time, so interest won't affect users; it will be only important for the owner.

One of the main problems for this system that I came across is the lack of a testing suite. Although it seems like a basic system, it is always interacting with CNote tokens, which is much more complex and has many intricacies. When we look at the tests that are written for this part of the system, unfortunately, what we see is complete trust in the manual calculations done by developers. The "**withdrawCarry()**" function is a crucial function in this system because, in the case of failure of this function, the main invariant of the system, which is:
>  It should always be possible to redeem 1 asD for 1 NOTE.
can be broken.
can be broken. In order for this invariant to hold, the contract calculates "**maximumWithdrawable**" as:
```solidity
uint256 exchangeRate = CTokenInterface(cNote).exchangeRateCurrent();
uint256 maximumWithdrawable = (CTokenInterface(cNote).balanceOf(address(this)) * exchangeRate) /
            1e28 -
            totalSupply();
```
As we can easily see, this calculation totally relies on cNote's "**exchangeRateCurrent()**" function. Unfortunately, cNote is not in scope for this 4-day-long audit, and testing is done via manual calculations such as:
```solidity
    function testWithdrawCarry() public {
        testMint();
        uint256 newExchangeRate = 1.1e28;
        cNOTE.setExchangeRate(newExchangeRate);
        uint256 initialBalance = NOTE.balanceOf(owner);
        uint256 asdSupply = asdToken.totalSupply();
        // Should be able to withdraw 10%
        uint256 withdrawAmount = asdSupply / 10;
        vm.prank(owner);
        asdToken.withdrawCarry(withdrawAmount);
        assertEq(NOTE.balanceOf(owner), initialBalance + withdrawAmount);
    }
```
The problem with this approach is that it is completely dependent on trust in developers if we do not check for cNote calculations. Considering provided information about c tokens with a compound docs reference, in order to deal with this integration, one must dive into the compound docs, then find cNote contract that is live in Canto and must try to build the integration from zero. Considering the limited amount of time for the contest and scope requirements, unfortunately, this audit might fall short within possible integration vulnerabilities.

## Market and Bonding Curve

### 1- Fee Mechanism and Strange Incentives

The approach for fees is quite complex and can cause weird incentives. In the buying shares process, the buyer does not get a fee share for their current buy process, while in selling and minting NFTs, the buyer does get a fee share for their buy process. In the burn process, they get a share from fees if they have token share in circulation. This complex process can create strange incentives for users who want to pay as few fees as possible.

One such example is splitting the buy process into multiple steps. While the user does not get a fee share from their current buy process, they get shares from that buy if they have tokens in circulation. Hence, instead of minting 10 tokens, the user is incentivized to mint them one by one between other users' function calls to decrease the fee they are paying. This is not incentivized in the long term for a share (considering the share is live and liquid enough) because more tokens in circulation mean a higher token price and fewer fees. Hence, it is more problematic for the early stages of a given share.

Considering this example, it is important to notice that users might find other ways that are not in this analysis that could potentially profit from the mechanism. This brings us to our next topic.

### 2-Lack of Invariant and Fuzz Tests

The market contract is completely path-dependent. Nearly every function call matters because every buy/sell, mint/burn changes the price of the share, fee amount of the share, price of minting, etc. For this type of codebase, one of the most important testing requirements is invariant and fuzz testing. It is important to check for different function call orders with different variables. Unfortunately, the tests that are written for the contract are weak. Only tests that are available are unit tests, which show a function "alone" working as intended for a "given" value. In the process of auditing, I had to write a test case for very different scenarios, and the tests that are written in a couple of hours, of course, are also not enough to see all possible scenarios that can occur in the protocol. I highly suggest creating a test suite that includes invariant tests with different function call orders by different users, fuzz tests with a wide range of variables, and also tests that include more than one active shareID. Although it is mentioned that the line coverage of tests is 80%, it is important to know that line coverage is not a very good variable for testing suite quality.

### 3-Centralization Risks and Potential Problems with Bonding Curves

In the discord, sponsor said that:
>  **One asD tokens can have none associated markets or it could also have multiple ones (although that is rather unusual)**

Considering this comment, we can assume that there will be multiple markets. Although what I will analyze next is not completely dependent on market count, multiple markets, multiple deployers, multiple owners can increase the risks of it. The bonding curve is an important contract that deals with all the calculations. All share flow and money flow are completely in the hands of the related Bonding Curve contract. So it is important to be sure that the bonding curve is secure so that the main invariant/purpose of the protocol holds.

My concerns with the current approach are:

a) The owner of the protocol can create a new bonding curve with an ability to change its parameters (shiftable curve after deployment) and use this curve to profit. I believe this sentence is enough to explain how catastrophic this situation can be. It is important to validate the bonding curve that is whitelisted, which I will explain next with a more friendly touch to the deployer.

b) Bonding curves should always be audited before they become whitelisted. In the above scenario, I explained a problem related to centralization. But it is not the only problem that can occur with bonding curves. Even if the owner of the market does not behave maliciously, he/she has the ability to whitelist any contract as a bonding curve. In this audit, LinearBondingCurve is audited and hopefully will be more secure after possible mitigations. But the problem is adding a new bonding curve to the protocol is as easy as one function call, and it is expected as mentioned by docs. I believe this is a very dangerous approach because bonding curves should always be audited before adding them. Since all shareIDs will be related to the mentioned market, and the market will hold all funds from different shareIDs, one tiny mistake in just one bonding curve can break all shares that are not even using the mentioned curve. So I suggest a different approach for bonding curve whitelisting. Whenever there is a new bonding curve implementation, be sure to audit that contract and also the market contract with integration and deploy new market contracts within new bonding curves. Hence, it would be best to remove capability of adding new bonding curves to the market to prevent potential single points of failure in the future.

c) Early users of the market are the ones that will profit. This is the nature of bonding curve. In the beginning, the price is much less than mid-late eras of the given shareID. Although the share creator cannot buy shares from the shareID that he/she created, as mentioned by docs, it is possible for him/her to buy shares from a different account easily. Right now, it is very profitable for the share creator to immediately buy shares for his/her own shareID. Considering it is possible in the future to add new bonding curves, for quadratic curves, it might be even more profitable. I suggest for the protocol to smooth the curve such that the first depositors' unfair advantage will diminish.

### 4- On Integration
Because I mentioned the lack of testing suites for both invariant tests and integration tests, I will try not to repeat myself in this part. Although two systems that are in the scope are related, they are completely in different folders, and there are no integration tests in between. I suggest providing them. Also, both systems have no check protection for reentrancy attacks. Check Effect Interactions (CEI) pattern is mostly followed, but considering both of these systems are not systems that will live by themselves, they are systems that will be integrated into the Canto Ecosystem and interact with many different contracts, I suggest adding reentrancy guards to both systems.

## Final Remarks, Approach in the Audit, Some Self Reflections

It was fun to audit such a protocol, but it was also hard. Lack of testing and the need for understanding more contracts that are not in scope (Scoping Details part says False to this but unfortunately not), made my auditing process harder, and I had to deal with creating test suites for different scenarios, diving into different contracts that are not in scope, hence giving less time to the contracts that are in scope. I believe it was also the case for so many other auditors. I wish good luck and best wishes to the team and suggest them to consider my humble suggestions for further contests so that we as wardens can use our time more effectively.

### Time spent:
11 hours