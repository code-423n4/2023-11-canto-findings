## Analysis Report

---

### Submitted By

Aamir

---

### preface

This audit report should be approached with the following points in mind:

- The report does not include repetitive documentation that the team is already aware of.
- The report is crafted towards providing the sponsors with value such as unknown edge case scenarios, faulty developer assumptions and unnoticed architecture-level weak spots.
- If there exists repetitive documentation, it is to provide the judge with more context on a specific high-level or in-depth scenario for ease of understandability.

---

### Findings Summary

| Severity | Instances |
| -------- | :-------: |
| High     |     1     |
| Medium   |     0     |
| Low      |     1     |

---

### For judges

I have submitted 1 High and 1 low findings. The low finding is very basic and doesn't require anything to be explained. I would like to point-out few things on 1 High that i submitted:

High severity Issue: `asd::mint(...)` and `asd::burn(...)` functions are assumes that the exchange rate for the `cNOTE` token will always be `1e28`. Even the tests written by sponsors haven't tested for that. This could be very big problem for the users and protocol as this will lead to loss of tokens to the user. And the protocol might loose their trust. So it is necessary to do updates in the codebase to make sure this never happen. I have submitted few things that can help in doing this. But still proper care should be taken by team to handle this throughout the codebase. Special care should be given to the reading of `cToken` docs by Compound Protocol.

Also for the sake of making test looks good and not lengthy, I have created some functions that is used by the PoC test itself. Just wanted to point out that you can find those below the test itself.

---

### Approach taken in evaluating the codebase

#### Time Spent: ~2 days (14 hours approx)

Couldn't spent much time on codebase due to some problem. Whatever amount of time that i could spent can be summarize like this

1. Day 1

   - Started Reading the docs from the code4rena audit page for the contest
   - Started gaining more knowledge about Compound Protocol's `cTokens`. Read complete integration guidelines for token by Compound Team. Had Look at the some of the original `cNOTE` contracts function like `mint`, `exchangeRateCurrent`, `redeem`, and `redeemUnderlying` etc.
   - Then started reading the codebase. Initiated with `asD` contracts. This is where i spent most of my time. The 1 High that I submitted came from doing this for whole day. Read about it again and again. Had look at some exchangeRate. Did some rough calculations. This was the whole summary of Day 1.

2. Day 2
   - Started with `asD` contracts again. Started gaining more knowledge about `cTokens`. Finally came up with the exact exploit and then started creating good PoC for the same. Spent almost half day on this.
   - Started reading other part of codebase where I was able to find other low. Read other contracts as well. But couldn't come up with anything.

---

### Architecture recommendations

The whole architecture looks good to me. The contracts are created in a very clever way. Only few recommendations i have is for `asd.sol` contract:

`asd.sol`: The number of SLOC is 58. Although it was a very small contract but figuring out different things it was connected with was very difficult task. The codebase looks very solid when we have first look at it. And there is no severity per-se if we look at the code. The actual severity was lying with the integration of `cNOTE`. This one thing made 2/3 functions vulnerable. The only thing i would recommend is to have a proper look at all of the functions from the `cNOTE` contract and then implement those functions as it is very crucial part for the protocol.

#### how this codebase compare to others I'm familiar with

The codebase is very similar to one of the contest I joined for `kelp-DAO`. The only difference was it was using `LP` tokens and `ETH` for the deposit and was minting `rsETH` in terms of that. One thing I liked about this one though is that it is not using any kind of oracle or something like that to calculate the prices and other stuff like that. Adding these things can cause some additional level of problems. In that codebase there was a lot vulnerability present just because of this.

---

### Codebase quality analysis

I think codebase was solid. Only one or two bad decisions were there that I have talked about before. One thing i didn't like was the testing of the codebase. Here are some issues that was present:

- Codebase didn't have fuzzing test
- Didn't have integration test
- Was only tested with hardcoded values
- Mocks were not implemented or used properly

Although I couldn't come out with a lot of issues, but bots had found few dozens and this could be little bit problematic as the bots are build mostly to scratch out all of the basic issues and there were many. Even 4naly3er report had few interesting finds. If this type of basic things has not been implemented properly then there are very high chances that there are some good amount of hidden and dangerous bugs as well.

---

### Centralization risks

Centralization risks in contracts have been handled well through the use of a role-based access control system. There are some specific scenarios (mentioned in the below section) I would like to point out that can allow certain entities to control specific parts of the system.

    - Use Time based access control system because that it will make the protocol more secure. It will give users sometime to do proper research and do whatever they can in the grace time period before update. This also generates better trust within the users.

---

### Resources used to gain the deeper knowledge of codebase

1. C4 contest page: [Link](https://code4rena.com/contests/2023-11-canto-application-specific-dollars-and-bonding-curves-for-1155s#top)
2. Compound Protocol's token integration docs: [Link](https://docs.compound.finance/v2/ctokens/#exchange-rate)
3. Canto Docs: [Link](https://docs.canto.io/)
4. Solodit: [Link](https://solodit.xyz/)
5. Tuber block explorer for Canto blockchain: [Link](https://tuber.build/)

### Week Spot

There are few of them in the codebase. One that i have talked about already above is `cNOTE` contract. The token is very different than any normal ERC20 or LP token. Important functions of it like `mint`, `redeem`, `transfer` depends on the exchange rate for the token. Also this exchange rate is volatile. That mean it will rise in the future and that isn't handled at all in the contracts. This is the special area that team should pay more attention to.


### Time spent:
13 hours