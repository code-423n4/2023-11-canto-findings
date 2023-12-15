---
sponsor: "Canto"
slug: "2023-11-canto"
date: "2023-12-15"
title: "Canto Application Specific Dollars and Bonding Curves for 1155s"
findings: "https://github.com/code-423n4/2023-11-canto-findings/issues"
contest: 306
---

# Overview

## About C4

Code4rena (C4) is an open organization consisting of security researchers, auditors, developers, and individuals with domain expertise in smart contracts.

A C4 audit is an event in which community participants, referred to as Wardens, review, audit, or analyze smart contract logic in exchange for a bounty provided by sponsoring projects.

During the audit outlined in this document, C4 conducted an analysis of the Canto Application Specific Dollars and Bonding Curves for 1155s smart contract system written in Solidity. The audit took place between November 13 - November 17, 2023.

## Wardens

125 Wardens contributed reports to Canto Application Specific Dollars and Bonding Curves for 1155s:

  1. [100su](https://code4rena.com/@100su)
  2. [bin2chen](https://code4rena.com/@bin2chen)
  3. [0xpiken](https://code4rena.com/@0xpiken)
  4. [SpicyMeatball](https://code4rena.com/@SpicyMeatball)
  5. [immeas](https://code4rena.com/@immeas)
  6. [ether\_sky](https://code4rena.com/@ether_sky)
  7. [pontifex](https://code4rena.com/@pontifex)
  8. [adriro](https://code4rena.com/@adriro)
  9. [bart1e](https://code4rena.com/@bart1e)
  10. [osmanozdemir1](https://code4rena.com/@osmanozdemir1)
  11. [T1MOH](https://code4rena.com/@T1MOH)
  12. [ast3ros](https://code4rena.com/@ast3ros)
  13. [ustas](https://code4rena.com/@ustas)
  14. [nazirite](https://code4rena.com/@nazirite)
  15. [Soul22](https://code4rena.com/@Soul22)
  16. [Oxsadeeq](https://code4rena.com/@Oxsadeeq)
  17. [wangxx2026](https://code4rena.com/@wangxx2026)
  18. [0xluckhu](https://code4rena.com/@0xluckhu)
  19. [0xSmartContract](https://code4rena.com/@0xSmartContract)
  20. [MrPotatoMagic](https://code4rena.com/@MrPotatoMagic)
  21. [Bauchibred](https://code4rena.com/@Bauchibred)
  22. [Krace](https://code4rena.com/@Krace)
  23. [d3e4](https://code4rena.com/@d3e4)
  24. [mojito\_auditor](https://code4rena.com/@mojito_auditor)
  25. [rvierdiiev](https://code4rena.com/@rvierdiiev)
  26. [Yanchuan](https://code4rena.com/@Yanchuan)
  27. [PENGUN](https://code4rena.com/@PENGUN)
  28. [tnquanghuy0512](https://code4rena.com/@tnquanghuy0512)
  29. [glcanvas](https://code4rena.com/@glcanvas)
  30. [0xAadi](https://code4rena.com/@0xAadi)
  31. [D1r3Wolf](https://code4rena.com/@D1r3Wolf)
  32. [AS](https://code4rena.com/@AS)
  33. [leegh](https://code4rena.com/@leegh)
  34. [lanrebayode77](https://code4rena.com/@lanrebayode77)
  35. [0xVolcano](https://code4rena.com/@0xVolcano)
  36. [hunter\_w3b](https://code4rena.com/@hunter_w3b)
  37. [sivanesh\_808](https://code4rena.com/@sivanesh_808)
  38. [mgf15](https://code4rena.com/@mgf15)
  39. [chaduke](https://code4rena.com/@chaduke)
  40. [lsaudit](https://code4rena.com/@lsaudit)
  41. [kaveyjoe](https://code4rena.com/@kaveyjoe)
  42. [K42](https://code4rena.com/@K42)
  43. [Kose](https://code4rena.com/@Kose)
  44. [clara](https://code4rena.com/@clara)
  45. [aariiif](https://code4rena.com/@aariiif)
  46. [Sathish9098](https://code4rena.com/@Sathish9098)
  47. [0xepley](https://code4rena.com/@0xepley)
  48. [unique](https://code4rena.com/@unique)
  49. [emerald7017](https://code4rena.com/@emerald7017)
  50. [cats](https://code4rena.com/@cats)
  51. [fouzantanveer](https://code4rena.com/@fouzantanveer)
  52. [0xbrett8571](https://code4rena.com/@0xbrett8571)
  53. [invitedtea](https://code4rena.com/@invitedtea)
  54. [Myd](https://code4rena.com/@Myd)
  55. [cheatc0d3](https://code4rena.com/@cheatc0d3)
  56. [tala7985](https://code4rena.com/@tala7985)
  57. [JCK](https://code4rena.com/@JCK)
  58. [0xAnah](https://code4rena.com/@0xAnah)
  59. [0xhex](https://code4rena.com/@0xhex)
  60. [0xta](https://code4rena.com/@0xta)
  61. [tabriz](https://code4rena.com/@tabriz)
  62. [parlayan\_yildizlar\_takimi](https://code4rena.com/@parlayan_yildizlar_takimi) ([ata](https://code4rena.com/@ata), [caglankaan](https://code4rena.com/@caglankaan) and [ulas](https://code4rena.com/@ulas))
  63. [peanuts](https://code4rena.com/@peanuts)
  64. [pep7siup](https://code4rena.com/@pep7siup)
  65. [aslanbek](https://code4rena.com/@aslanbek)
  66. [jasonxiale](https://code4rena.com/@jasonxiale)
  67. [Pheonix](https://code4rena.com/@Pheonix)
  68. [zhaojie](https://code4rena.com/@zhaojie)
  69. [max10afternoon](https://code4rena.com/@max10afternoon)
  70. [t0x1c](https://code4rena.com/@t0x1c)
  71. [OMEN](https://code4rena.com/@OMEN)
  72. [erebus](https://code4rena.com/@erebus)
  73. [ksk2345](https://code4rena.com/@ksk2345)
  74. [ZanyBonzy](https://code4rena.com/@ZanyBonzy)
  75. [tourist](https://code4rena.com/@tourist)
  76. [young](https://code4rena.com/@young)
  77. [btk](https://code4rena.com/@btk)
  78. [bareli](https://code4rena.com/@bareli)
  79. [Topmark](https://code4rena.com/@Topmark)
  80. [sl1](https://code4rena.com/@sl1)
  81. [SandNallani](https://code4rena.com/@SandNallani)
  82. [MohammedRizwan](https://code4rena.com/@MohammedRizwan)
  83. [wisdomn\_](https://code4rena.com/@wisdomn_)
  84. [Matin](https://code4rena.com/@Matin)
  85. [ayden](https://code4rena.com/@ayden)
  86. [shenwilly](https://code4rena.com/@shenwilly)
  87. [codynhat](https://code4rena.com/@codynhat)
  88. [critical-or-high](https://code4rena.com/@critical-or-high)
  89. [sbaudh6](https://code4rena.com/@sbaudh6)
  90. [nailkhalimov](https://code4rena.com/@nailkhalimov)
  91. [merlinboii](https://code4rena.com/@merlinboii)
  92. [firmanregar](https://code4rena.com/@firmanregar)
  93. [0xMango](https://code4rena.com/@0xMango)
  94. [turvy\_fuzz](https://code4rena.com/@turvy_fuzz)
  95. [Udsen](https://code4rena.com/@Udsen)
  96. [jnforja](https://code4rena.com/@jnforja)
  97. [Madalad](https://code4rena.com/@Madalad)
  98. [peritoflores](https://code4rena.com/@peritoflores)
  99. [inzinko](https://code4rena.com/@inzinko)
  100. [deepkin](https://code4rena.com/@deepkin)
  101. [DarkTower](https://code4rena.com/@DarkTower) ([Gelato\_ST](https://code4rena.com/@Gelato_ST), [Maroutis](https://code4rena.com/@Maroutis), [OxTenma](https://code4rena.com/@OxTenma) and [0xrex](https://code4rena.com/@0xrex))
  102. [0xarno](https://code4rena.com/@0xarno)
  103. [vangrim](https://code4rena.com/@vangrim)
  104. [mahyar](https://code4rena.com/@mahyar)
  105. [nmirchev8](https://code4rena.com/@nmirchev8)
  106. [0x175](https://code4rena.com/@0x175)
  107. [KupiaSec](https://code4rena.com/@KupiaSec)
  108. [rice\_cooker](https://code4rena.com/@rice_cooker)
  109. [neocrao](https://code4rena.com/@neocrao)
  110. [twcctop](https://code4rena.com/@twcctop)
  111. [openwide](https://code4rena.com/@openwide)
  112. [Tricko](https://code4rena.com/@Tricko)
  113. [0x3b](https://code4rena.com/@0x3b)
  114. [HChang26](https://code4rena.com/@HChang26)
  115. [RaoulSchaffranek](https://code4rena.com/@RaoulSchaffranek)
  116. [developerjordy](https://code4rena.com/@developerjordy)
  117. [Giorgio](https://code4rena.com/@Giorgio)
  118. [ElCid](https://code4rena.com/@ElCid)
  119. [rouhsamad](https://code4rena.com/@rouhsamad)
  120. [zhaojohnson](https://code4rena.com/@zhaojohnson)

This audit was judged by [0xTheC0der](https://code4rena.com/@0xTheC0der).

Final report assembled by [thebrittfactor](https://twitter.com/brittfactorC4).

# Summary

The C4 analysis yielded an aggregated total of 3 unique vulnerabilities. Of these vulnerabilities, 1 received a risk rating in the category of HIGH severity and 2 received a risk rating in the category of MEDIUM severity.

Additionally, C4 analysis included 44 reports detailing issues with a risk rating of LOW severity or non-critical. There were also 17 reports recommending gas optimizations.

All of the issues presented here are linked back to their original finding.

# Scope

The code under review can be found within the [C4 Canto Application Specific Dollars and Bonding Curves for 1155s repository](https://github.com/code-423n4/2023-11-canto), and is composed of 4 smart contracts written in the Solidity programming language and includes 316 lines of Solidity code.

In addition to the known issues identified by the project team, a Code4rena bot race was conducted at the start of the audit. The winning bot, **henry** from warden hihen, generated the [Automated Findings report](https://gist.github.com/code423n4/8152d0e300fc048c8aac9da328fa8475) and all findings therein were classified as out of scope.

# Severity Criteria

C4 assesses the severity of disclosed vulnerabilities based on three primary risk categories: high, medium, and low/non-critical.

High-level considerations for vulnerabilities span the following key areas when conducting assessments:

- Malicious Input Handling
- Escalation of privileges
- Arithmetic
- Gas use

For more information regarding the severity criteria referenced throughout the submission review process, please refer to the documentation provided on [the C4 website](https://code4rena.com), specifically our section on [Severity Categorization](https://docs.code4rena.com/awarding/judging-criteria/severity-categorization).

# High Risk Findings (1)

## [[H-01] Owner cannot withdraw all interest due to wrong calculation of accrued interest in `WithdrawCarry`](https://github.com/code-423n4/2023-11-canto-findings/issues/181)
*Submitted by [100su](https://github.com/code-423n4/2023-11-canto-findings/issues/181), also found by [pontifex](https://github.com/code-423n4/2023-11-canto-findings/issues/448), [immeas](https://github.com/code-423n4/2023-11-canto-findings/issues/392), [nazirite](https://github.com/code-423n4/2023-11-canto-findings/issues/391), [Soul22](https://github.com/code-423n4/2023-11-canto-findings/issues/381), [bart1e](https://github.com/code-423n4/2023-11-canto-findings/issues/369), [adriro](https://github.com/code-423n4/2023-11-canto-findings/issues/360), [Oxsadeeq](https://github.com/code-423n4/2023-11-canto-findings/issues/345), [ast3ros](https://github.com/code-423n4/2023-11-canto-findings/issues/264), [osmanozdemir1](https://github.com/code-423n4/2023-11-canto-findings/issues/259), [ether\_sky](https://github.com/code-423n4/2023-11-canto-findings/issues/256), [T1MOH](https://github.com/code-423n4/2023-11-canto-findings/issues/227), [bin2chen](https://github.com/code-423n4/2023-11-canto-findings/issues/126), [0xpiken](https://github.com/code-423n4/2023-11-canto-findings/issues/113), [ustas](https://github.com/code-423n4/2023-11-canto-findings/issues/74), [SpicyMeatball](https://github.com/code-423n4/2023-11-canto-findings/issues/57), [wangxx2026](https://github.com/code-423n4/2023-11-canto-findings/issues/50), and [0xluckhu](https://github.com/code-423n4/2023-11-canto-findings/issues/11)*

The current `withdrawCarry` function in the contract underestimates the accrued interest due to a miscalculation. This error prevents the rightful owner from withdrawing their accrued interest, effectively locking the assets. The primary issue lies in the calculation of `maximumWithdrawable` within `withdrawCarry`.

### Flawed Calculation

The following code segment is used to determine `maximumWithdrawable`:

    uint256 exchangeRate = CTokenInterface(cNote).exchangeRateCurrent(); // Scaled by 1 * 10^(18 - 8 + Underlying Token Decimals), i.e. 10^(28) in our case
    // The amount of cNOTE the contract has to hold (based on the current exchange rate which is always increasing) such that it is always possible to receive 1 NOTE when burning 1 asD
    uint256 maximumWithdrawable = (CTokenInterface(cNote).balanceOf(address(this)) * exchangeRate) / 1e28 - totalSupply();

The problem arises with the scaling of `exchangeRate`, which is assumed to be scaled by `10^28`. However, for `CNOTE` in the Canto network, the `exchangeRate` is actually scaled by `10^18`. This discrepancy causes the owner to withdraw only `10^(-10)` times the actual interest amount. 

Consequently, when a non-zero `_amount` is specified, `withdrawCarry` often fails due to the `require(_amount <= maximumWithdrawable, "Too many tokens requested");` condition.

### Proof of Concept

**CNOTE Scaling Verification**

An essential aspect of this audit involves verifying the scaling factor of the `CNOTE` exchange rate. The `exchangeRate` scale for `CNOTE` can be verified in the [Canto Network's GitHub repository](https://github.com/Canto-Network/clm/blob/298377c8711a067a2a49d75a8bf2f90bbbe3de9e/src/CErc20.sol#L15).  Evidence confirming that the exponent of the `CNOTE` exchange rate is indeed `18` can be found through [this link to the token tracker](https://tuber.build/token/0xEe602429Ef7eCe0a13e4FfE8dBC16e101049504C?tab=read_proxy). The data provided there shows the current value of the stored exchange rate (`exchangeRateStored`) as approximately `1004161485744613000`. This value corresponds to `1.00416 * 1e18`, reaffirming the `10^18` scaling factor.

This information is critical for accurately understanding the mechanics of CNOTE and ensuring the smart contract's calculations align with the actual scaling used in the token's implementation. The verification of this scaling factor supports the recommendations for adjusting the main contract's calculations and the associated test cases, as previously outlined.

**Testing with Solidity Codes**

Testing with the following Solidity code illustrates the actual `CNOTE` values:

```solidity
function updateBalance() external {
    updatedUnderlyingBalance = ICNoteSimple(cnote).balanceOfUnderlying(msg.sender);
    updatedExchangeRate = ICNoteSimple(cnote).exchangeRateCurrent();

    uint256 balance = IERC20(cnote).balanceOf(msg.sender);
    calculatedUnderlying = balance * updatedExchangeRate / 1e28;
}
```

The corresponding TypeScript logs show a clear discrepancy between the expected and calculated underlying balances:

    console.log("balanceCnote: ", (Number(balanceCnote) / 1e18).toString());
    console.log("exchangeRate: ", Number(exchangeRate).toString());
    console.log("underlyingBalance: ", Number(underlyingBalance).toString());
    console.log("calculatedUnderlying: ", Number(calculatedUnderlying).toString());

With the logs:

    balanceCnote:  400100.9100006097
    exchangeRate:  1004122567006264000
    underlyingBalance:  4.017503528113544e+23
    calculatedUnderlying:  40175035281135

### Tools Used

- Solidity for interacting with the Canto mainnet.
- TypeScript for testing and validation.

### Recommended Mitigation Steps

**Using `balanceOfUnderlying` Function** - Replace the flawed calculation with the `balanceOfUnderlying` function. This function accurately calculates the underlying `NOTE` balance and is defined in `CToken.sol` ([source](https://github.com/Canto-Network/clm/blob/298377c8711a067a2a49d75a8bf2f90bbbe3de9e/src/CToken.sol#L175)).

**Proposed Code Modifications** - Two alternative implementations are suggested:

1. Without `balanceOfUnderlying`: Modify the scaling factor in the existing calculation from `1e28` to `1e18`.

```

    function withdrawCarry(uint256 _amount) external onlyOwner {
        uint256 exchangeRate = CTokenInterface(cNote).exchangeRateCurrent(); // Scaled by 10^18
        // The amount of cNOTE the contract has to hold (based on the current exchange rate which is always increasing) such that it is always possible to receive 1 NOTE when burning 1 asD
        uint256 maximumWithdrawable = (CTokenInterface(cNote).balanceOf(address(this)) * exchangeRate) /
            1e18 -
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

2. With `balanceOfUnderlying` (Recommended): Utilize the `balanceOfUnderlying` function for a simpler calculation of `maximumWithdrawable`.

```

    function withdrawCarry(uint256 _amount) external onlyOwner {
        // The amount of cNOTE the contract has to hold (based on the current exchange rate which is always increasing) such that it is always possible to receive 1 NOTE when burning 1 asD
        uint256 maximumWithdrawable = CTokenInterface(cNote).balanceOfUnderlying(address(this)) - totalSupply();
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

The second option is highly recommended for its accuracy and simplicity.

### Modification of Related Test Codes

For post-modifications to the main contract code, it's essential to update the associated test cases. In the [`MockCNOTE`](https://github.com/code-423n4/2023-11-canto/blob/main/asD/src/test/asD.t.sol#L16-L49) test contract, all scaling is currently set to `10^28`. To align with the main contract changes, the following modifications are recommended for `MockCNOTE`:

    contract MockCNOTE is MockERC20 {
        address public underlying;
        uint256 public constant SCALE = 1e18;
        uint256 public exchangeRateCurrent = SCALE;

        constructor(string memory symbol, string memory name, address _underlying) MockERC20(symbol, name) {
            underlying = _underlying;
        }

        function mint(uint256 amount) public returns (uint256 statusCode) {
            SafeERC20.safeTransferFrom(IERC20(underlying), msg.sender, address(this), amount);
            _mint(msg.sender, (amount * SCALE) / exchangeRateCurrent);
            statusCode = 0;
        }

        function redeemUnderlying(uint256 amount) public returns (uint256 statusCode) {
            SafeERC20.safeTransfer(IERC20(underlying), msg.sender, amount);
            _burn(msg.sender, (amount * exchangeRateCurrent) / SCALE);
            statusCode = 0;
        }

        function redeem(uint256 amount) public returns (uint256 statusCode) {
            SafeERC20.safeTransfer(IERC20(underlying), msg.sender, (amount * exchangeRateCurrent) / SCALE);
            _burn(msg.sender, amount);
            statusCode = 0;
        }

        function setExchangeRate(uint256 _exchangeRate) public {
            exchangeRateCurrent = _exchangeRate;
        }
    }

This revised `MockCNOTE` contract reflects the updated scale factor and aligns with the main contract's logic.

### Scenario Testing with Mainnet Forking

For comprehensive validation, scenario testing using a fork of the mainnet is highly recommended. This approach allows for real-world testing conditions by simulating interactions with existing contracts on the mainnet. It provides a robust environment to verify the correctness and reliability of the contract modifications in real-world scenarios, ensuring that the contract behaves as expected when interfacing with other mainnet contracts.

This step is crucial to identify potential issues that might not be apparent in isolated or simulated environments, enhancing the overall reliability of the contract before deployment.

### Assessed type

Math

**[OpenCoreCH (Canto) confirmed and commented via duplicate issue #227](https://github.com/code-423n4/2023-11-canto-findings/issues/227#issuecomment-1827562804):**
> True, `cNOTE` only has 8 decimals unlike all other cTokens that have 18 decimals, causing this discrepancy. Will be fixed.

**[OpenCoreCH (Canto) commented](https://github.com/code-423n4/2023-11-canto-findings/issues/181#issuecomment-1838419120):**
 > Fixed.

***
 
# Medium Risk Findings (2)

## [[M-01] No slippage protection for Market functions](https://github.com/code-423n4/2023-11-canto-findings/issues/12)
*Submitted by [rvierdiiev](https://github.com/code-423n4/2023-11-canto-findings/issues/12), also found by [0xMango](https://github.com/code-423n4/2023-11-canto-findings/issues/505), [d3e4](https://github.com/code-423n4/2023-11-canto-findings/issues/496), [turvy\_fuzz](https://github.com/code-423n4/2023-11-canto-findings/issues/494), [Udsen](https://github.com/code-423n4/2023-11-canto-findings/issues/460), [pontifex](https://github.com/code-423n4/2023-11-canto-findings/issues/456), [jnforja](https://github.com/code-423n4/2023-11-canto-findings/issues/452), [Madalad](https://github.com/code-423n4/2023-11-canto-findings/issues/446), [peritoflores](https://github.com/code-423n4/2023-11-canto-findings/issues/441), [inzinko](https://github.com/code-423n4/2023-11-canto-findings/issues/440), [deepkin](https://github.com/code-423n4/2023-11-canto-findings/issues/438), [DarkTower](https://github.com/code-423n4/2023-11-canto-findings/issues/425), [peanuts](https://github.com/code-423n4/2023-11-canto-findings/issues/416), [pep7siup](https://github.com/code-423n4/2023-11-canto-findings/issues/411), [0xarno](https://github.com/code-423n4/2023-11-canto-findings/issues/395), [vangrim](https://github.com/code-423n4/2023-11-canto-findings/issues/379), [aslanbek](https://github.com/code-423n4/2023-11-canto-findings/issues/340), mojito\_auditor ([1](https://github.com/code-423n4/2023-11-canto-findings/issues/339), [2](https://github.com/code-423n4/2023-11-canto-findings/issues/338)), [mahyar](https://github.com/code-423n4/2023-11-canto-findings/issues/330), [nmirchev8](https://github.com/code-423n4/2023-11-canto-findings/issues/326), [Yanchuan](https://github.com/code-423n4/2023-11-canto-findings/issues/318), [PENGUN](https://github.com/code-423n4/2023-11-canto-findings/issues/316), [0x175](https://github.com/code-423n4/2023-11-canto-findings/issues/308), [KupiaSec](https://github.com/code-423n4/2023-11-canto-findings/issues/304), [jasonxiale](https://github.com/code-423n4/2023-11-canto-findings/issues/301), [Pheonix](https://github.com/code-423n4/2023-11-canto-findings/issues/296), [rice\_cooker](https://github.com/code-423n4/2023-11-canto-findings/issues/284), [osmanozdemir1](https://github.com/code-423n4/2023-11-canto-findings/issues/283), [neocrao](https://github.com/code-423n4/2023-11-canto-findings/issues/281), [twcctop](https://github.com/code-423n4/2023-11-canto-findings/issues/275), [T1MOH](https://github.com/code-423n4/2023-11-canto-findings/issues/269), [ast3ros](https://github.com/code-423n4/2023-11-canto-findings/issues/265), [Kose](https://github.com/code-423n4/2023-11-canto-findings/issues/260), [bart1e](https://github.com/code-423n4/2023-11-canto-findings/issues/242), [tnquanghuy0512](https://github.com/code-423n4/2023-11-canto-findings/issues/226), [openwide](https://github.com/code-423n4/2023-11-canto-findings/issues/225), t0x1c ([1](https://github.com/code-423n4/2023-11-canto-findings/issues/218), [2](https://github.com/code-423n4/2023-11-canto-findings/issues/176), [3](https://github.com/code-423n4/2023-11-canto-findings/issues/150)), [zhaojie](https://github.com/code-423n4/2023-11-canto-findings/issues/210), [Tricko](https://github.com/code-423n4/2023-11-canto-findings/issues/200), [0x3b](https://github.com/code-423n4/2023-11-canto-findings/issues/188), [Bauchibred](https://github.com/code-423n4/2023-11-canto-findings/issues/183), chaduke ([1](https://github.com/code-423n4/2023-11-canto-findings/issues/174), [2](https://github.com/code-423n4/2023-11-canto-findings/issues/170)), [HChang26](https://github.com/code-423n4/2023-11-canto-findings/issues/165), [RaoulSchaffranek](https://github.com/code-423n4/2023-11-canto-findings/issues/162), [max10afternoon](https://github.com/code-423n4/2023-11-canto-findings/issues/157), [ustas](https://github.com/code-423n4/2023-11-canto-findings/issues/148), [developerjordy](https://github.com/code-423n4/2023-11-canto-findings/issues/144), [Giorgio](https://github.com/code-423n4/2023-11-canto-findings/issues/128), [bin2chen](https://github.com/code-423n4/2023-11-canto-findings/issues/123), [SpicyMeatball](https://github.com/code-423n4/2023-11-canto-findings/issues/114), [0xpiken](https://github.com/code-423n4/2023-11-canto-findings/issues/112), [glcanvas](https://github.com/code-423n4/2023-11-canto-findings/issues/77), [ElCid](https://github.com/code-423n4/2023-11-canto-findings/issues/73), [rouhsamad](https://github.com/code-423n4/2023-11-canto-findings/issues/70), and [zhaojohnson](https://github.com/code-423n4/2023-11-canto-findings/issues/65)*

### Proof of Concept

Price for shares inside `Market` is calculated using bonding curve. Currently, `LinearBondingCurve` is supported. This bonging curve [increases each next shares with fixed amount](https://github.com/code-423n4/2023-11-canto/blob/main/1155tech-contracts/src/bonding_curve/LinearBondingCurve.sol#L21) and also [uses `10% / log2(shareIndex)` to calculate fee](https://github.com/code-423n4/2023-11-canto/blob/main/1155tech-contracts/src/bonding_curve/LinearBondingCurve.sol#L35) for the share.

In order to calculate price to [buy shares `getBuyPrice` is used](https://github.com/code-423n4/2023-11-canto/blob/main/1155tech-contracts/src/Market.sol#L152) and to calculate price to [`sell` shares `getSellPrice` is used](https://github.com/code-423n4/2023-11-canto/blob/main/1155tech-contracts/src/Market.sol#L175).

<https://github.com/code-423n4/2023-11-canto/blob/main/1155tech-contracts/src/Market.sol#L132-L145>

```solidity
    function getBuyPrice(uint256 _id, uint256 _amount) public view returns (uint256 price, uint256 fee) {
        // If id does not exist, this will return address(0), causing a revert in the next line
        address bondingCurve = shareData[_id].bondingCurve;
        (price, fee) = IBondingCurve(bondingCurve).getPriceAndFee(shareData[_id].tokenCount + 1, _amount);
    }

    function getSellPrice(uint256 _id, uint256 _amount) public view returns (uint256 price, uint256 fee) {
        // If id does not exist, this will return address(0), causing a revert in the next line
        address bondingCurve = shareData[_id].bondingCurve;
        (price, fee) = IBondingCurve(bondingCurve).getPriceAndFee(shareData[_id].tokenCount - _amount + 1, _amount);
    }
```

Both functions use `IBondingCurve(bondingCurve).getPriceAndFee`, but `getBuyPrice` provides `current token index` as start index for curve, while `getSellPrice` provides `current token index - amount to sell` as start index for curve.

In the case when someone wants to buy shares, the price depends on current circulation supply. If this supply is increased right after a user's `buy` tx, then they will pay more for the shares. If someone sells shares right before the `buy` tx, then a user will pay a lesser amount (which is good of course).

The same we can say about `sell` function. If someone sells shares right before a user's `sell` execution, then the user receives a smaller amount. If someone buys shares, then the user gets a bigger amount.

As the price at the moment when user initiates `buy` or `sell` function can be changed before execution (with frontrunning or just innocent), this means that slippage protection should be introduced, so a user can be sure that they will not pay more than expected or receive less than expected.

When a user expects to `buy` shares for 5 USD, then the attacker can sandwich a user's tx shares in order to make profit.

- Attacker first frontruns and `buy`s shares that cost 5 USD.
- Then, let a user's `buy` tx to be executed so a user buys shares more expensive (10 USD). 
- Then the attacker backruns with `sell` shares that cost now 10 USD and earn on this (5 USD).

The same thing can be done with `sell` sandwiching. When a user expects to `sell` their share for 20 USD:

- Attacker can `sell` own share (and get 20 USD).
- Then, let a user `sell` their share cheaper (and get 15 USD).
- Then the attacker buys share cheaper (for 10 USD) and earn on this (10 USD).

While `buy` sandwiching needs a user to give the full approval to `Market` as this price is then sent from user, `sell` sandwiching doesn't need that because it only sends tokens to the victim.

### Impact

User can lose funds.

### Tools Used

VsCode

### Recommended Mitigation Steps

Make a user provide slippage to `buy`, `sell`, `burnNFT` and `mintNFT` functions.

### Assessed type

Error

**[OpenCoreCH (Canto) confirmed](https://github.com/code-423n4/2023-11-canto-findings/issues/12#issuecomment-1827462251)**

**[0xTheC0der (judge) decreased severity to Medium and commented](https://github.com/code-423n4/2023-11-canto-findings/issues/12#issuecomment-1830907592):**
 > On the one hand, according to our [guidelines](https://docs.code4rena.com/roles/judges/how-to-judge-a-contest):
> > Unless there is something uniquely novel created by combining vectors, most submissions regarding vulnerabilities that are inherent to a particular system or the Ethereum network as a whole **should be considered QA**. Examples of such vulnerabilities include front running, **sandwich attacks**, and MEV.
> 
> But on the other hand, a slippage percentage parameter or expected amount has become state-of-the-art which also makes this a shortcoming of the protocol. 
> 
> Consequently, it seems appropriate to move forward with `Medium` severity.

**[OpenCoreCH (Canto) commented](https://github.com/code-423n4/2023-11-canto-findings/issues/12#issuecomment-1833719090):**
 > Introduced params.

***

## [[M-02] Users will lose rewards when buying new tokens if they already own some tokens](https://github.com/code-423n4/2023-11-canto-findings/issues/9)
*Submitted by [Krace](https://github.com/code-423n4/2023-11-canto-findings/issues/9), also found by [d3e4](https://github.com/code-423n4/2023-11-canto-findings/issues/500), [immeas](https://github.com/code-423n4/2023-11-canto-findings/issues/394), [0xAadi](https://github.com/code-423n4/2023-11-canto-findings/issues/373), [mojito\_auditor](https://github.com/code-423n4/2023-11-canto-findings/issues/337), [PENGUN](https://github.com/code-423n4/2023-11-canto-findings/issues/306), [Yanchuan](https://github.com/code-423n4/2023-11-canto-findings/issues/302), [D1r3Wolf](https://github.com/code-423n4/2023-11-canto-findings/issues/244), [ether\_sky](https://github.com/code-423n4/2023-11-canto-findings/issues/223), [tnquanghuy0512](https://github.com/code-423n4/2023-11-canto-findings/issues/221), [AS](https://github.com/code-423n4/2023-11-canto-findings/issues/168), [leegh](https://github.com/code-423n4/2023-11-canto-findings/issues/138), [lanrebayode77](https://github.com/code-423n4/2023-11-canto-findings/issues/130), [bin2chen](https://github.com/code-423n4/2023-11-canto-findings/issues/125), [0xpiken](https://github.com/code-423n4/2023-11-canto-findings/issues/110), [glcanvas](https://github.com/code-423n4/2023-11-canto-findings/issues/71), [SpicyMeatball](https://github.com/code-423n4/2023-11-canto-findings/issues/66), and [rvierdiiev](https://github.com/code-423n4/2023-11-canto-findings/issues/10)*

When users purchase new tokens, the old tokens they already own do not receive the fees from this purchase, resulting in a loss of funds for the users.

### Proof of Concept

According to the explanation by the sponsor (Roman), when users buy new tokens, the recently purchased token is not eligible for a fee. However, the "old shares" that users already own should be eligible.

Roman:

> Hey, just a design decision, the idea behind it is that the user does not own the new shares yet when he buys, so he should not get any fees for it. Whereas he still owns it when he sells (it is basically the last sale).

Krace:

> So the new shares should not get fee. If the user owns some "old shares", is it eligible for a fee when the user buying?

Roman:

> Yes, in that case, these shares are in the "owner pool" at the time of purchase and should be eligible

However, the [`buy`](https://github.com/code-423n4/2023-11-canto/blob/b78bfdbf329ba9055ba24bd710c7e1c60251039a/1155tech-contracts/src/Market.sol#L150) function will completely disregard the rewards associated with the buyer's "old shares" for these fees.

Let's analyze the `buy` function:

- **Step 1:** It retrieves the rewards for the buyer before the purchase. These rewards are not linked to the purchase because they use the old token amounts, and the fees from this purchase are not factored in.
- **Step 2:** It splits the fees for this purchase. `_splitFees` increases the value of `shareHolderRewardsPerTokenScaled` based on the fees from this purchase.
- **Step 3:** It updates the `rewardsLastClaimedValue` of the buyer to the latest `shareHolderRewardsPerTokenScaled`, which was increased in `_splitFees` just now.

The question arises: the fees from this purchase are not rewarded to the buyer's old tokens between Step 2 and Step 3, yet the buyer's `rewardsLastClaimedValue` is updated to the newest value. This results in the permanent loss of these rewards for the buyer.

```solidity
    function buy(uint256 _id, uint256 _amount) external {
        require(shareData[_id].creator != msg.sender, "Creator cannot buy");
        (uint256 price, uint256 fee) = getBuyPrice(_id, _amount); // Reverts for non-existing ID
        SafeERC20.safeTransferFrom(token, msg.sender, address(this), price + fee);
        // The reward calculation has to use the old rewards value (pre fee-split) to not include the fees of this buy
        // The rewardsLastClaimedValue then needs to be updated with the new value such that the user cannot claim fees of this buy
//@audit it will get the rewards first        
        uint256 rewardsSinceLastClaim = _getRewardsSinceLastClaim(_id);
        // Split the fee among holder, creator and platform
//@audit then split the fees of this buying        
        _splitFees(_id, fee, shareData[_id].tokensInCirculation);
//@audit and update sender's rewardsLastClaimedValue to the newest one, which is updated by `splitFees`
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

```solidity
    function _splitFees(
        uint256 _id,
        uint256 _fee,
        uint256 _tokenCount
    ) internal {
        uint256 shareHolderFee = (_fee * HOLDER_CUT_BPS) / 10_000;
        uint256 shareCreatorFee = (_fee * CREATOR_CUT_BPS) / 10_000;
        uint256 platformFee = _fee - shareHolderFee - shareCreatorFee;
        shareData[_id].shareCreatorPool += shareCreatorFee;
        if (_tokenCount > 0) {
//@audit shareHolderRewardsPerTokenScaled will be increased if tokenCount>0
            shareData[_id].shareHolderRewardsPerTokenScaled += (shareHolderFee * 1e18) / _tokenCount;
        } else {
            // If there are no tokens in circulation, the fee goes to the platform
            platformFee += shareHolderFee;
        }
        platformPool += platformFee;
    }
```

### Test

The following POC shows the case mentioned. Add it to `1155tech-contracts/src/Market.t.sol` and run it with `forge test --match-test testBuyNoReward -vv`.

        function testBuyNoReward() public {
            // Bob create share with id 1
            market.changeBondingCurveAllowed(address(bondingCurve), true);
            market.restrictShareCreation(false);
            vm.prank(bob);
            market.createNewShare("Test Share", address(bondingCurve), "metadataURI");
            assertEq(market.shareIDs("Test Share"), 1);

            token.transfer(alice, 1e17);

            // alice buy 1 token in share1 ==> token1
            vm.startPrank(alice);
            uint256 aliceBalanceBefore = token.balanceOf(alice);
            token.approve(address(market), 1e18);
            console.log(" alice balance ", aliceBalanceBefore);
            market.buy(1, 1);
            uint256 aliceBalanceAfter = token.balanceOf(alice);
            console.log(" alice balance after buy ",  aliceBalanceAfter);
            console.log(" alice cost after buy ",  aliceBalanceBefore - aliceBalanceAfter);

            // alice buy another 1 token in share1 ==> token2
            market.buy(1,1);
            uint256 aliceBalanceAfterSecondBuy = token.balanceOf(alice);
            console.log(" alice balance after second buy ",  aliceBalanceAfterSecondBuy);
            // alice get no reward for her token1
            console.log(" alice cost after second buy ",  aliceBalanceAfter - aliceBalanceAfterSecondBuy);

            vm.stopPrank();
        }

The result is shown below; Alice gets no rewards for `token1`:

    [PASS] testBuyNoReward() (gas: 443199)
    Logs:
       alice balance  100000000000000000
       alice balance after buy  98900000000000000
       alice cost after buy  1100000000000000
       alice balance after second buy  96700000000000000
       alice cost after second buy  2200000000000000

After my suggested patch, Alice can get the rewards for `token1`: `2200000000000000` - `2134000000000000` = `66000000000000`.

    [PASS] testBuyNoReward() (gas: 447144)
    Logs:
       alice balance  100000000000000000
       alice balance after buy  98900000000000000
       alice cost after buy  1100000000000000
       alice balance after second buy  96766000000000000
       alice cost after second buy  2134000000000000

### Tools Used

Foundry

### Recommended Mitigation Steps

Split the fees before getting the rewards in `buy`:

```diff
diff --git a/1155tech-contracts/src/Market.sol b/1155tech-contracts/src/Market.sol
index 59c5c96..85d91a5 100644
--- a/1155tech-contracts/src/Market.sol
+++ b/1155tech-contracts/src/Market.sol
@@ -151,11 +151,11 @@ contract Market is ERC1155, Ownable2Step {
         require(shareData[_id].creator != msg.sender, "Creator cannot buy");
         (uint256 price, uint256 fee) = getBuyPrice(_id, _amount); // Reverts for non-existing ID
         SafeERC20.safeTransferFrom(token, msg.sender, address(this), price + fee);
+        // Split the fee among holder, creator and platform
+        _splitFees(_id, fee, shareData[_id].tokensInCirculation);
         // The reward calculation has to use the old rewards value (pre fee-split) to not include the fees of this buy
         // The rewardsLastClaimedValue then needs to be updated with the new value such that the user cannot claim fees of this buy
         uint256 rewardsSinceLastClaim = _getRewardsSinceLastClaim(_id);
-        // Split the fee among holder, creator and platform
-        _splitFees(_id, fee, shareData[_id].tokensInCirculation);
         rewardsLastClaimedValue[_id][msg.sender] = shareData[_id].shareHolderRewardsPerTokenScaled;

         shareData[_id].tokenCount += _amount;
```

### Assessed type

Context

**[OpenCoreCH (Canto) confirmed, but disagreed with severity and commented](https://github.com/code-423n4/2023-11-canto-findings/issues/9#issuecomment-1828482474):**
 > These fees are not lost, the impact is just that for this particular `buys`; the other users get more rewards and the user that buys less (but the same also applies for other users, so it might even cancel out for long-term holders). So I think high is definitely over-inflated for that.
> 
> However, I agree that the current logic is a bit weird, but changing it such that they just receive rewards for the other shares would also be a bit weird (would incentivize to split up a buy into multiple ones). We discussed this internally and will change the logic such that users always receive fees for their `buys`, no matter if it is the first or subsequent ones.

**[0xTheC0der (judge) decreased severity to Medium](https://github.com/code-423n4/2023-11-canto-findings/issues/9#issuecomment-1830951178)**

**[OpenCoreCH (Canto) commented](https://github.com/code-423n4/2023-11-canto-findings/issues/9#issuecomment-1833716107):**
 > Fixed.

*Note: For full discussion, see [here](https://github.com/code-423n4/2023-11-canto-findings/issues/9).*

***

# Low Risk and Non-Critical Issues

For this audit, 44 reports were submitted by wardens detailing low risk and non-critical issues. The [report highlighted below](https://github.com/code-423n4/2023-11-canto-findings/issues/106) by **chaduke** received the top score from the judge.

*The following wardens also submitted reports: [pontifex](https://github.com/code-423n4/2023-11-canto-findings/issues/451), [MrPotatoMagic](https://github.com/code-423n4/2023-11-canto-findings/issues/383), [adriro](https://github.com/code-423n4/2023-11-canto-findings/issues/362), [Bauchibred](https://github.com/code-423n4/2023-11-canto-findings/issues/356), [kaveyjoe](https://github.com/code-423n4/2023-11-canto-findings/issues/333), [lsaudit](https://github.com/code-423n4/2023-11-canto-findings/issues/175), [d3e4](https://github.com/code-423n4/2023-11-canto-findings/issues/498), [OMEN](https://github.com/code-423n4/2023-11-canto-findings/issues/493), [hunter\_w3b](https://github.com/code-423n4/2023-11-canto-findings/issues/470), [erebus](https://github.com/code-423n4/2023-11-canto-findings/issues/466), [peanuts](https://github.com/code-423n4/2023-11-canto-findings/issues/435), [cheatc0d3](https://github.com/code-423n4/2023-11-canto-findings/issues/429), [ksk2345](https://github.com/code-423n4/2023-11-canto-findings/issues/423), [pep7siup](https://github.com/code-423n4/2023-11-canto-findings/issues/405), [ZanyBonzy](https://github.com/code-423n4/2023-11-canto-findings/issues/375), [jasonxiale](https://github.com/code-423n4/2023-11-canto-findings/issues/347), [tourist](https://github.com/code-423n4/2023-11-canto-findings/issues/335), [osmanozdemir1](https://github.com/code-423n4/2023-11-canto-findings/issues/327), [young](https://github.com/code-423n4/2023-11-canto-findings/issues/315), [aslanbek](https://github.com/code-423n4/2023-11-canto-findings/issues/313), [Pheonix](https://github.com/code-423n4/2023-11-canto-findings/issues/303), [btk](https://github.com/code-423n4/2023-11-canto-findings/issues/293), [bareli](https://github.com/code-423n4/2023-11-canto-findings/issues/289), [Topmark](https://github.com/code-423n4/2023-11-canto-findings/issues/274), [bart1e](https://github.com/code-423n4/2023-11-canto-findings/issues/270), [sl1](https://github.com/code-423n4/2023-11-canto-findings/issues/261), [T1MOH](https://github.com/code-423n4/2023-11-canto-findings/issues/243), [SandNallani](https://github.com/code-423n4/2023-11-canto-findings/issues/220), [zhaojie](https://github.com/code-423n4/2023-11-canto-findings/issues/217), [MohammedRizwan](https://github.com/code-423n4/2023-11-canto-findings/issues/216), [wisdomn\_](https://github.com/code-423n4/2023-11-canto-findings/issues/196), [0xpiken](https://github.com/code-423n4/2023-11-canto-findings/issues/191), [Matin](https://github.com/code-423n4/2023-11-canto-findings/issues/160), [max10afternoon](https://github.com/code-423n4/2023-11-canto-findings/issues/153), [ayden](https://github.com/code-423n4/2023-11-canto-findings/issues/132), [bin2chen](https://github.com/code-423n4/2023-11-canto-findings/issues/124), [shenwilly](https://github.com/code-423n4/2023-11-canto-findings/issues/111), [codynhat](https://github.com/code-423n4/2023-11-canto-findings/issues/103), [critical-or-high](https://github.com/code-423n4/2023-11-canto-findings/issues/60), [sbaudh6](https://github.com/code-423n4/2023-11-canto-findings/issues/56), [nailkhalimov](https://github.com/code-423n4/2023-11-canto-findings/issues/45), [merlinboii](https://github.com/code-423n4/2023-11-canto-findings/issues/22), and [firmanregar](https://github.com/code-423n4/2023-11-canto-findings/issues/19).*

## [01] Check whether `rewardsSinceLastClaim + price - fee == 0` in function `sell()`

The function should fail when `rewardsSinceLastClaim + price - fee == 0` to avoid not worthy sell.

[Market.sol#L187](https://github.com/code-423n4/2023-11-canto/blob/335930cd53cf9a137504a57f1215be52c6d67cb3/1155tech-contracts/src/Market.sol#L187)

Mitigation:

```diff
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
+       uint amt = rewardsSinceLastClaim + price - fee;

-        SafeERC20.safeTransfer(token, msg.sender, rewardsSinceLastClaim + price - fee);

+        if (amt > 0) SafeERC20.safeTransfer(token, msg.sender, amt);
+        else revert("no zero sell.");

        emit SharesSold(_id, msg.sender, _amount, price, fee);
    }
```

## [02] `asDFactory.create()` does not prevent that two `asD` contracts might have the same name and symbol.

[asDFactory.sol#L33-L38](https://github.com/code-423n4/2023-11-canto/blob/335930cd53cf9a137504a57f1215be52c6d67cb3/asD/src/asDFactory.sol#L33-L38)

## [03] `getPriceAndFee()` has the issue of division precision loss issue 

[LinearBondingCurve.sol#L14-L25](https://github.com/code-423n4/2023-11-canto/blob/335930cd53cf9a137504a57f1215be52c6d67cb3/1155tech-contracts/src/bonding_curve/LinearBondingCurve.sol#L14-L25)

The problem is in the following line: 

```javascipt
fee += (getFee(i) * tokenPrice) / 1e18;
```

The division of `1e18` should be performed after the loop is completed to avoid early division rounding error. 

Mitigation: The division of `1e19` is performed outside:

```diff
 function getPriceAndFee(uint256 shareCount, uint256 amount)
        external
        view
        override
        returns (uint256 price, uint256 fee)
    {
        for (uint256 i = shareCount; i < shareCount + amount; i++) {
            uint256 tokenPrice = priceIncrease * i;
            price += tokenPrice;
-            fee += (getFee(i) * tokenPrice) / 1e18;
+            fee += (getFee(i) * tokenPrice);
        }
+            fee = fee / 1e18;
    }
```

## [04] It is better to check `amount > 0` to avoid false event emit

[Market.sol#L244-L249](https://github.com/code-423n4/2023-11-canto/blob/335930cd53cf9a137504a57f1215be52c6d67cb3/1155tech-contracts/src/Market.sol#L244-L249)

## [05] The event emitting should be inside the if statement to avoid false event emitting

[Market.sol#L263-L270](https://github.com/code-423n4/2023-11-canto/blob/335930cd53cf9a137504a57f1215be52c6d67cb3/1155tech-contracts/src/Market.sol#L263-L270)

## [06] We need to emit an event for this function

[Market.sol#L309-L312](https://github.com/code-423n4/2023-11-canto/blob/335930cd53cf9a137504a57f1215be52c6d67cb3/1155tech-contracts/src/Market.sol#L309-L312)

## [07] Both `mintNFT()` and `burnNFT()` lacks slippage control for the fee

Note that the fee is not a fixed value, its value depends on `shareData[_id].tokenCount`.

[Market.sol#L194-L198](https://github.com/code-423n4/2023-11-canto/blob/335930cd53cf9a137504a57f1215be52c6d67cb3/1155tech-contracts/src/Market.sol#L194-L198)

Therefore, if the function is front-run by another `buy()` and `mintNFT()` function, then `shareData[_id].tokenCount` will increase, and as a result, the fee will increase. A user might pay more fee than they expected due to such front-runs. 

Mitigation: Add a slippage control so that the user can control the maximum amount of fee they are willing to pay. 

## [08] The function `changeBondingCurveAllowed()` will not be able to disable previously boundcurves that have been used in some share creation

It can only used to prevent future share creation to use the curve again. 

[Market.sol#L104-L108](https://github.com/code-423n4/2023-11-canto/blob/335930cd53cf9a137504a57f1215be52c6d67cb3/1155tech-contracts/src/Market.sol#L104-L108)

## [09] `changeShareCreatorWhitelist()` cannot disable existing creators for their already created shares

It can only prevent the creator to create more shares in the future. 

[Market.sol#L309-L312](https://github.com/code-423n4/2023-11-canto/blob/335930cd53cf9a137504a57f1215be52c6d67cb3/1155tech-contracts/src/Market.sol#L309-L312)

## [10] All the four functions `buy()`, `sell()`, `mintNFT()` and `burnNFT()` call `_splitFees()` to split the fees, but when `shareData[_id].tokensInCirculation ==0`, `shareHolderFee` will be added to `plantformFee`. 

[Market.sol#L280-L296](https://github.com/code-423n4/2023-11-canto/blob/335930cd53cf9a137504a57f1215be52c6d67cb3/1155tech-contracts/src/Market.sol#L280-L296)

In this case, a user can front-run these transactions with a `buy()` to buy a share so that `shareData[_id].tokensInCirculation > 0` and therefore, effectively be able to steal all such `shareHolderFee` in these situations.

Mitigation: Implement a snapshot mechanism for market so that a user can not just suddenly buy shares and enjoy rewards immediately. See ERC20Snapshot for an example:

[ERC20Snapshot](https://docs.openzeppelin.com/contracts/3.x/api/token/erc20#ERC20Snapshot)


**[0xTheC0der (judge) commented](https://github.com/code-423n4/2023-11-canto-findings/issues/106#issuecomment-1832881518):**
 > Although this report lacks formatting and therefore probably didn't get a high quality tag, its QA findings provide great value. Therefore, it's selected for report.  
> 
> 01: Low. <br>
> 02: Non-Critical.<br>
> 03: Low.<br>
> 04: Non-Critical.<br>
> 05: Non-Critical.<br>
> 06: Low.<br>
> 07: Low.<br>
> 08: Non-Critical (possibly undesired, but valid to mention).<br>
> 09: Non-Critical (possibly undesired, but valid to mention).<br>
> 10: Low.

***

# Gas Optimizations

For this audit, 17 reports were submitted by wardens detailing gas optimizations. The [report highlighted below](https://github.com/code-423n4/2023-11-canto-findings/issues/224) by **0xVolcano** received the top score from the judge.

*The following wardens also submitted reports: [sivanesh\_808](https://github.com/code-423n4/2023-11-canto-findings/issues/413), [hunter\_w3b](https://github.com/code-423n4/2023-11-canto-findings/issues/314), [mgf15](https://github.com/code-423n4/2023-11-canto-findings/issues/202), [tala7985](https://github.com/code-423n4/2023-11-canto-findings/issues/512), [cheatc0d3](https://github.com/code-423n4/2023-11-canto-findings/issues/424), [JCK](https://github.com/code-423n4/2023-11-canto-findings/issues/418), [0xAnah](https://github.com/code-423n4/2023-11-canto-findings/issues/409), [lsaudit](https://github.com/code-423n4/2023-11-canto-findings/issues/403), [MrPotatoMagic](https://github.com/code-423n4/2023-11-canto-findings/issues/402), [K42](https://github.com/code-423n4/2023-11-canto-findings/issues/372), [0xhex](https://github.com/code-423n4/2023-11-canto-findings/issues/358), [0xta](https://github.com/code-423n4/2023-11-canto-findings/issues/355), [tabriz](https://github.com/code-423n4/2023-11-canto-findings/issues/299), [chaduke](https://github.com/code-423n4/2023-11-canto-findings/issues/146), [100su](https://github.com/code-423n4/2023-11-canto-findings/issues/20), and [parlayan\_yildizlar\_takimi](https://github.com/code-423n4/2023-11-canto-findings/issues/6).*

# Gas Report

**Note: The following findings were not found by the bot. For max savings, please implement the changes found by bot.**

## [01] We can optimize the function `buy()`: Saves 287 Gas from tests included

https://github.com/code-423n4/2023-11-canto/blob/335930cd53cf9a137504a57f1215be52c6d67cb3/1155tech-contracts/src/Market.sol#L150-L169
|        | Min    | Average | Median   | Max   |
| ------ | --- | ------- | ----- | ----- |
| Before | 157773 | 157773 | 157773 | 157773 |
| After  | 157486 | 157486 | 157486 | 157486 |

```solidity
File: /src/Market.sol
150:    function buy(uint256 _id, uint256 _amount) external {
151:        require(shareData[_id].creator != msg.sender, "Creator cannot buy");
152:        (uint256 price, uint256 fee) = getBuyPrice(_id, _amount); // Reverts for non-existing ID
153:        SafeERC20.safeTransferFrom(token, msg.sender, address(this), price + fee);
156:        uint256 rewardsSinceLastClaim = _getRewardsSinceLastClaim(_id);
157:        // Split the fee among holder, creator and platform
158:        _splitFees(_id, fee, shareData[_id].tokensInCirculation);
159:        rewardsLastClaimedValue[_id][msg.sender] = shareData[_id].shareHolderRewardsPerTokenScaled;

161:        shareData[_id].tokenCount += _amount;
162:        shareData[_id].tokensInCirculation += _amount;
163:        tokensByAddress[_id][msg.sender] += _amount;
```

To get the `rewardsSinceLastClaim` we call an internal function `_getRewardsSinceLastClaim()` which has the following implementation:

```solidity
    function _getRewardsSinceLastClaim(uint256 _id) internal view returns (uint256 amount) {
        uint256 lastClaimedValue = rewardsLastClaimedValue[_id][msg.sender];
        amount =
            ((shareData[_id].shareHolderRewardsPerTokenScaled - lastClaimedValue) * tokensByAddress[_id][msg.sender]) /
            1e18;
    }
```

The internal function reads `shareData[_id].shareHolderRewardsPerTokenScaled`  and `tokensByAddress[_id][msg.sender]` which are state variablea and are also being read in the main function. We can avoid making this state read twice by inlining the internal function and caching one call:

```diff
diff --git a/1155tech-contracts/src/Market.sol b/1155tech-contracts/src/Market.sol
index 59c5c96..233d0ce 100644
--- a/1155tech-contracts/src/Market.sol
+++ b/1155tech-contracts/src/Market.sol
@@ -153,14 +153,19 @@ contract Market is ERC1155, Ownable2Step {
         SafeERC20.safeTransferFrom(token, msg.sender, address(this), price + fee);
         // The reward calculation has to use the old rewards value (pre fee-split) to not include the fees of this buy
         // The rewardsLastClaimedValue then needs to be updated with the new value such that the user cannot claim fees of this buy
-        uint256 rewardsSinceLastClaim = _getRewardsSinceLastClaim(_id);
+        uint256 lastClaimedValue = rewardsLastClaimedValue[_id][msg.sender];
+        uint256 _tokensByAddress =  tokensByAddress[_id][msg.sender];
+        uint256 _shareHolderRewardsPerTokenScaled = shareData[_id].shareHolderRewardsPerTokenScaled;
+        uint256 rewardsSinceLastClaim = ((_shareHolderRewardsPerTokenScaled - lastClaimedValue) * _tokensByAddress) / 1e18;
+
         // Split the fee among holder, creator and platform
-        _splitFees(_id, fee, shareData[_id].tokensInCirculation);
-        rewardsLastClaimedValue[_id][msg.sender] = shareData[_id].shareHolderRewardsPerTokenScaled;
+        uint256 _tokensInCirculation = shareData[_id].tokensInCirculation;
+        _splitFees(_id, fee, _tokensInCirculation);
+        rewardsLastClaimedValue[_id][msg.sender] = _shareHolderRewardsPerTokenScaled;

         shareData[_id].tokenCount += _amount;
-        shareData[_id].tokensInCirculation += _amount;
-        tokensByAddress[_id][msg.sender] += _amount;
+        shareData[_id].tokensInCirculation = _tokensInCirculation + _amount;
+        tokensByAddress[_id][msg.sender] = _tokensByAddress + _amount;
```

## [02] We can optimize the function `sell()`: Saves 234 Gas from tests included

https://github.com/code-423n4/2023-11-canto/blob/335930cd53cf9a137504a57f1215be52c6d67cb3/1155tech-contracts/src/Market.sol#L174-L189

|        | Min    | Average | Median   | Max   |
| ------ | --- | ------- | ----- | ----- |
| Before | 42142 | 42142 | 42142 | 42142 |
| After  | 41908 | 41908 | 41908 | 41908 |

```solidity
File: /src/Market.sol
174:    function sell(uint256 _id, uint256 _amount) external {
175:        (uint256 price, uint256 fee) = getSellPrice(_id, _amount);
176:        // Split the fee among holder, creator and platform
177:        _splitFees(_id, fee, shareData[_id].tokensInCirculation);

179:        uint256 rewardsSinceLastClaim = _getRewardsSinceLastClaim(_id);
180:        rewardsLastClaimedValue[_id][msg.sender] = shareData[_id].shareHolderRewardsPerTokenScaled;

182:        shareData[_id].tokenCount -= _amount;
183:        shareData[_id].tokensInCirculation -= _amount;
184:        tokensByAddress[_id][msg.sender] -= _amount; // Would underflow if user did not have enough tokens
```

To get the `rewardsSinceLastClaim` we call an internal function `_getRewardsSinceLastClaim()` which has the following implementation:

```solidity
    function _getRewardsSinceLastClaim(uint256 _id) internal view returns (uint256 amount) {
        uint256 lastClaimedValue = rewardsLastClaimedValue[_id][msg.sender];
        amount =
            ((shareData[_id].shareHolderRewardsPerTokenScaled - lastClaimedValue) * tokensByAddress[_id][msg.sender]) /
            1e18;
    }
```

The internal function reads `shareData[_id].shareHolderRewardsPerTokenScaled`  and `tokensByAddress[_id][msg.sender]` which are state variables and are also being read in the main function. We can avoid making this state read twice by inlining the internal function and caching one call:

```diff
diff --git a/1155tech-contracts/src/Market.sol b/1155tech-contracts/src/Market.sol
index 59c5c96..a73f406 100644
--- a/1155tech-contracts/src/Market.sol
+++ b/1155tech-contracts/src/Market.sol
@@ -176,12 +176,17 @@ contract Market is ERC1155, Ownable2Step {
         // Split the fee among holder, creator and platform
         _splitFees(_id, fee, shareData[_id].tokensInCirculation);
         // The user also gets the rewards of his own sale (which is not the case for buys)
-        uint256 rewardsSinceLastClaim = _getRewardsSinceLastClaim(_id);
-        rewardsLastClaimedValue[_id][msg.sender] = shareData[_id].shareHolderRewardsPerTokenScaled;
+
+        uint256 lastClaimedValue = rewardsLastClaimedValue[_id][msg.sender];
+        uint256 _tokensByAddress = tokensByAddress[_id][msg.sender];
+        uint256 _shareHolderRewardsPerTokenScaled = shareData[_id].shareHolderRewardsPerTokenScaled;
+        uint256 rewardsSinceLastClaim = ((_shareHolderRewardsPerTokenScaled - lastClaimedValue) * _tokensByAddress) / 1e18;
+
+        rewardsLastClaimedValue[_id][msg.sender] = _shareHolderRewardsPerTokenScaled;

         shareData[_id].tokenCount -= _amount;
         shareData[_id].tokensInCirculation -= _amount;
-        tokensByAddress[_id][msg.sender] -= _amount; // Would underflow if user did not have enough tokens
+        tokensByAddress[_id][msg.sender] = _tokensByAddress - _amount; // Would underflow if user did not have enough tokens

         // Send the funds to the user
         SafeERC20.safeTransfer(token, msg.sender, rewardsSinceLastClaim + price - fee);
```

## [03] We can optimize the function `mintNFT()`: Saves 215 Gas from tests included

https://github.com/code-423n4/2023-11-canto/blob/335930cd53cf9a137504a57f1215be52c6d67cb3/1155tech-contracts/src/Market.sol#L203-L221

|        | Min    | Average | Median   | Max   |
| ------ | --- | ------- | ----- | ----- |
| Before | 68280 | 68280 | 68280 | 68280 |
| After  | 68065 | 68065 | 68065 | 68065 |

```solidity
File: /src/Market.sol
203:    function mintNFT(uint256 _id, uint256 _amount) external {
204:        uint256 fee = getNFTMintingPrice(_id, _amount);

206:        SafeERC20.safeTransferFrom(token, msg.sender, address(this), fee);
207:        _splitFees(_id, fee, shareData[_id].tokensInCirculation);
208:        // The user also gets the proportional rewards for the minting
209:        uint256 rewardsSinceLastClaim = _getRewardsSinceLastClaim(_id);
210:        rewardsLastClaimedValue[_id][msg.sender] = shareData[_id].shareHolderRewardsPerTokenScaled;
211:        tokensByAddress[_id][msg.sender] -= _amount;
212:        shareData[_id].tokensInCirculation -= _amount;
```

To get the `rewardsSinceLastClaim` we call an internal function `_getRewardsSinceLastClaim()` which has the following implementation:

```solidity
    function _getRewardsSinceLastClaim(uint256 _id) internal view returns (uint256 amount) {
        uint256 lastClaimedValue = rewardsLastClaimedValue[_id][msg.sender];
        amount =
            ((shareData[_id].shareHolderRewardsPerTokenScaled - lastClaimedValue) * tokensByAddress[_id][msg.sender]) /
            1e18;
    }
```

The internal function reads `shareData[_id].shareHolderRewardsPerTokenScaled` and `tokensByAddress[_id][msg.sender])` which are state variables which are also being read in the main function. We can avoid making this state reads twice by inlining the internal function and caching one call:

```diff
diff --git a/1155tech-contracts/src/Market.sol b/1155tech-contracts/src/Market.sol
index 59c5c96..fc85039 100644
--- a/1155tech-contracts/src/Market.sol
+++ b/1155tech-contracts/src/Market.sol
@@ -206,9 +206,14 @@ contract Market is ERC1155, Ownable2Step {
         SafeERC20.safeTransferFrom(token, msg.sender, address(this), fee);
         _splitFees(_id, fee, shareData[_id].tokensInCirculation);
         // The user also gets the proportional rewards for the minting
-        uint256 rewardsSinceLastClaim = _getRewardsSinceLastClaim(_id);
-        rewardsLastClaimedValue[_id][msg.sender] = shareData[_id].shareHolderRewardsPerTokenScaled;
-        tokensByAddress[_id][msg.sender] -= _amount;
+
+        uint256 lastClaimedValue = rewardsLastClaimedValue[_id][msg.sender];
+        uint256 _tokensByAddress = tokensByAddress[_id][msg.sender];
+        uint256 _shareHolderRewardsPerTokenScaled = shareData[_id].shareHolderRewardsPerTokenScaled;
+        uint256 rewardsSinceLastClaim =((_shareHolderRewardsPerTokenScaled - lastClaimedValue) * _tokensByAddress) /1e18;
+
+        rewardsLastClaimedValue[_id][msg.sender] = _shareHolderRewardsPerTokenScaled;
+        tokensByAddress[_id][msg.sender] = _tokensByAddress - _amount;
         shareData[_id].tokensInCirculation -= _amount;

         _mint(msg.sender, _id, _amount, "");
```

## [04] We can optimize the function `burnNFT()`: Saves 230 Gas from tests included

https://github.com/code-423n4/2023-11-canto/blob/335930cd53cf9a137504a57f1215be52c6d67cb3/1155tech-contracts/src/Market.sol#L226-L241

|        | Min    | Average | Median   | Max   |
| ------ | --- | ------- | ----- | ----- |
| Before | 49135 | 49135 | 49135 | 49135 |
| After  | 48905 | 48905 | 48905 | 48905 |

```solidity
File: /src/Market.sol
226:    function burnNFT(uint256 _id, uint256 _amount) external {
227:        uint256 fee = getNFTMintingPrice(_id, _amount);

229:        SafeERC20.safeTransferFrom(token, msg.sender, address(this), fee);
230:        _splitFees(_id, fee, shareData[_id].tokensInCirculation);

232:        uint256 rewardsSinceLastClaim = _getRewardsSinceLastClaim(_id);
233:        rewardsLastClaimedValue[_id][msg.sender] = shareData[_id].shareHolderRewardsPerTokenScaled;
234:        tokensByAddress[_id][msg.sender] += _amount;
235:        shareData[_id].tokensInCirculation += _amount;
236:        _burn(msg.sender, _id, _amount);

238:        SafeERC20.safeTransfer(token, msg.sender, rewardsSinceLastClaim);
239:        // ERC1155 already logs, but we add this to have the price information
240:        emit NFTsBurned(_id, msg.sender, _amount, fee);
241:    }
```

We can call the internal function  `_getRewardsSinceLastClaim()` which has the following implementation:

```solidity
    function _getRewardsSinceLastClaim(uint256 _id) internal view returns (uint256 amount) {
        uint256 lastClaimedValue = rewardsLastClaimedValue[_id][msg.sender];
        amount =
            ((shareData[_id].shareHolderRewardsPerTokenScaled - lastClaimedValue) * tokensByAddress[_id][msg.sender]) /
            1e18;
    }
```

The internal function reads `shareData[_id].shareHolderRewardsPerTokenScaled`, which is a state variable which is also being read in the main function. We also read `tokensByAddress[_id][msg.sender]` in the main function as well as in the internal function:

```diff
diff --git a/1155tech-contracts/src/Market.sol b/1155tech-contracts/src/Market.sol
index 59c5c96..7f3c5e1 100644
--- a/1155tech-contracts/src/Market.sol
+++ b/1155tech-contracts/src/Market.sol
@@ -229,10 +229,18 @@ contract Market is ERC1155, Ownable2Step {
         SafeERC20.safeTransferFrom(token, msg.sender, address(this), fee);
         _splitFees(_id, fee, shareData[_id].tokensInCirculation);
         // The user does not get the proportional rewards for the burning (unless they have additional tokens that are not in the NFT)
-        uint256 rewardsSinceLastClaim = _getRewardsSinceLastClaim(_id);
-        rewardsLastClaimedValue[_id][msg.sender] = shareData[_id].shareHolderRewardsPerTokenScaled;
-        tokensByAddress[_id][msg.sender] += _amount;
-        shareData[_id].tokensInCirculation += _amount;
+
+        uint256 lastClaimedValue = rewardsLastClaimedValue[_id][msg.sender];
+
+        uint256 _tokensByAddress = tokensByAddress[_id][msg.sender];
+        uint256 _shareHolderRewardsPerTokenScaled = shareData[_id].shareHolderRewardsPerTokenScaled;
+        uint256 rewardsSinceLastClaim =
+            ((_shareHolderRewardsPerTokenScaled - lastClaimedValue) * _tokensByAddress) /
+            1e18;
+
+        rewardsLastClaimedValue[_id][msg.sender] = _shareHolderRewardsPerTokenScaled;
+        tokensByAddress[_id][msg.sender] = _tokensByAddress + _amount;
+        shareData[_id].tokensInCirculation +=  _amount;
         _burn(msg.sender, _id, _amount);

         SafeERC20.safeTransfer(token, msg.sender, rewardsSinceLastClaim);
```

## [05] We can optimize the function `claimHolderFee()`

**This is not covered by tests, so no gas benchmarks.**

https://github.com/code-423n4/2023-11-canto/blob/335930cd53cf9a137504a57f1215be52c6d67cb3/1155tech-contracts/src/Market.sol#L263-L270

```solidity
File: /src/Market.sol
263:    function claimHolderFee(uint256 _id) external {
264:        uint256 amount = _getRewardsSinceLastClaim(_id);
265:        rewardsLastClaimedValue[_id][msg.sender] = shareData[_id].shareHolderRewardsPerTokenScaled;
266:        if (amount > 0) {
267:            SafeERC20.safeTransfer(token, msg.sender, amount);
268:        }
269:        emit HolderFeeClaimed(msg.sender, _id, amount);
270:    }
```

To get the `amount` we call an internal function `_getRewardsSinceLastClaim()` which has the following implementation:

```solidity
    function _getRewardsSinceLastClaim(uint256 _id) internal view returns (uint256 amount) {
        uint256 lastClaimedValue = rewardsLastClaimedValue[_id][msg.sender];
        amount =
            ((shareData[_id].shareHolderRewardsPerTokenScaled - lastClaimedValue) * tokensByAddress[_id][msg.sender]) /
            1e18;
    }
```

The internal function reads `shareData[_id].shareHolderRewardsPerTokenScaled`, which is a state variable which is also being read in the main function. We can avoid making this state read twice by inlining the internal function and caching one call:

```diff
     function claimHolderFee(uint256 _id) external {
-        uint256 amount = _getRewardsSinceLastClaim(_id);
-        rewardsLastClaimedValue[_id][msg.sender] = shareData[_id].shareHolderRewardsPerTokenScaled;
+        uint256 lastClaimedValue = rewardsLastClaimedValue[_id][msg.sender];
+        uint256 _shareHolderRewardsPerTokenScaled = shareData[_id].shareHolderRewardsPerTokenScaled;
+        uint256 amount = ((_shareHolderRewardsPerTokenScaled - lastClaimedValue) * tokensByAddress[_id][msg.sender]) /1e18;
+        rewardsLastClaimedValue[_id][msg.sender] = _shareHolderRewardsPerTokenS
caled;
         if (amount > 0) {
             SafeERC20.safeTransfer(token, msg.sender, amount);
         }
```

**[OpenCoreCH (Canto) confirmed](https://github.com/code-423n4/2023-11-canto-findings/issues/224#issuecomment-1827454889)**

**[0xTheC0der (judge) commented](https://github.com/code-423n4/2023-11-canto-findings/issues/224#issuecomment-1832601386):**
 > Chosen for report due to extensive and well-presented optimization on a per-method level, while providing valuable before/after comparisons.

***

# Audit Analysis

For this audit, 17 analysis reports were submitted by wardens. An analysis report examines the codebase as a whole, providing observations and advice on such topics as architecture, mechanism, or approach. The [report highlighted below](https://github.com/code-423n4/2023-11-canto-findings/issues/514) by **0xSmartContract** received the top score from the judge.

*The following wardens also submitted reports: [MrPotatoMagic](https://github.com/code-423n4/2023-11-canto-findings/issues/461), [Bauchibred](https://github.com/code-423n4/2023-11-canto-findings/issues/357), [clara](https://github.com/code-423n4/2023-11-canto-findings/issues/502), [aariiif](https://github.com/code-423n4/2023-11-canto-findings/issues/476), [hunter\_w3b](https://github.com/code-423n4/2023-11-canto-findings/issues/463), [Sathish9098](https://github.com/code-423n4/2023-11-canto-findings/issues/439), [0xepley](https://github.com/code-423n4/2023-11-canto-findings/issues/437), [Kose](https://github.com/code-423n4/2023-11-canto-findings/issues/415), [unique](https://github.com/code-423n4/2023-11-canto-findings/issues/404), [K42](https://github.com/code-423n4/2023-11-canto-findings/issues/393), [emerald7017](https://github.com/code-423n4/2023-11-canto-findings/issues/364), [cats](https://github.com/code-423n4/2023-11-canto-findings/issues/359), [fouzantanveer](https://github.com/code-423n4/2023-11-canto-findings/issues/346), [0xbrett8571](https://github.com/code-423n4/2023-11-canto-findings/issues/291), [invitedtea](https://github.com/code-423n4/2023-11-canto-findings/issues/81), and [Myd](https://github.com/code-423n4/2023-11-canto-findings/issues/63).*

## Analysis - Canto Protocol

## Summary

| Title | Details |
|:----------------|:------|
| The approach I followed when reviewing the code | Stages in my code review and analysis. |
| Analysis of the code base | What is unique? How are the existing patterns used? "Solidity-metrics" were used.  |
| Test analysis | Test scope of the project and quality of tests. |
| Security Approach of the Project | Audit approach of the Project. |
| Other Audit Reports and Automated Findings | What are the previous Audit reports and their analysis. |
| Packages and Dependencies Analysis | Details about the project Packages. |
| Gas Optimization | Gas usage approach of the project and alternative solutions to it. |
| Other recommendations | What is unique? How are the existing patterns used? |
| New insights and learning from this audit | Things learned from the project. |

## The approach I followed when reviewing the code

First, by examining the scope of the code, I determined my code review and analysis strategy. Accordingly, I analyzed and audited the subject in the following steps:

|Stage|Details|Information|
|:----------------|:------|:------|
|Compile and Run Test|[Installation](https://github.com/code-423n4/2023-11-canto#tests)|Test and installation structure is simple, cleanly designed.|
|Architecture Review| [Canto](https://docs.canto.io/free-public-infrastructure-fpi/note) |Provides a basic architectural teaching for General Architecture.|
|Graphical Analysis  |Graphical Analysis with [Solidity-metrics](https://github.com/ConsenSys/solidity-metrics)|A visual view has been made to dominate the general structure of the codes of the project.|
|Slither Analysis  | [Slither Report](https://github.com/crytic/slither)| The project does not currently have a slither result, a slither control was created from scratch. |
|Test Suits|[Tests](https://github.com/code-423n4/2023-11-canto#tests)|In this section, the scope and content of the tests of the project are analyzed.|
|Manual Code Review|[Scope](https://github.com/code-423n4/2023-11-canto#scoping-details)|N/A|
|Infographic|[Figma](https://www.figma.com/)|I made Visual drawings to understand the hard-to-understand mechanisms.|


## Analysis of the code base

The most important summary in analyzing the code base is the stacking of codes to be analyzed.

In this way, many predictions can be made; including the difficulty levels of the contracts, which one is more important for the auditor, the features they contain that are important for security (payable functions, uses assembly, etc.), the audit cost of the project, and the time to be allocated to the audit.

Uses Consensys Solidity Metrics:
-  **Lines:** total lines of the source unit.
-  **nLines:** normalized lines of the source unit (e.g. normalizes functions spanning multiple lines).
-  **nSLOC:** normalized source lines of code (only source-code lines; no comments, no blank lines).
-  **Comment Lines:** lines containing single or block comments.
-  **Complexity Score:** a custom complexity score derived from code statements that are known to introduce code complexity (branches, loops, calls, external interfaces, ...).

![image](https://github.com/code-423n4/2023-11-canto/assets/104318932/673a9361-66d8-4349-b570-5754d2b322cc)

### Market.sol

**function createNewShare()`** - Creates a new share.

![image](https://github.com/code-423n4/2023-11-canto/assets/104318932/ccb08eb4-91a1-451c-985a-43f5a348a37b)

**function `buy()`** - Buy amount of tokens for a given share ID.

![image](https://github.com/code-423n4/2023-11-canto/assets/104318932/132b98f1-780a-452b-9703-d6d5c55a9df8)

**function `sell()`** - Sell amount of tokens for a given share ID.

![image](https://github.com/code-423n4/2023-11-canto/assets/104318932/6240af7d-bcca-492d-9d66-e4a590567d47)

**function `mint()` - function `burn()`**

![image](https://github.com/code-423n4/2023-11-canto/assets/104318932/98bde68d-a067-4de6-b968-a45b89666450)

## Test analysis

What did the project do differently? It can be said that the developers of the project did a quality job, there is a test structure consisting of tests with quality content. Especially the test coverage of Market.sol is very successful.

Market.sol smart contract test file:

1. Bonding Curve Operations:
    - `testChangeBondingCurveAllowed`: This test checks if a bonding curve can be whitelisted or removed correctly. It verifies the correct listing and delisting using `assertTrue` and `assertFalse`.
    - `testFailChangeBondingCurveAllowedNonOwner`: This test confirms that only the contract owner can change the bonding curve. A non-owner account is used with `vm.prank`, and the expected revert is checked.
    - `testFailCreateNewShareWhenBondingCurveNotWhitelisted`: It verifies that a new share cannot be created when a bonding curve is not whitelisted.

2. Share Creation and Pricing:
    - `testCreateNewShare`: Tests whether a new share can be created with a whitelisted bonding curve.
    - `testGetBuyPrice`: Checks if the buy price and fee for the created share are calculated correctly.

3. Buying and Selling Operations:
    - `testBuy`: Tests the purchase of a share and verifies the related processes (token approval, balance updates).
    - `testSell`: Tests the sale of a purchased share and its related transactions.

4. NFT Minting and Burning:
    - `testMint`: Tests the minting of an NFT and subsequent balance updates.
    - `testBurn`: Verifies the burning of the minted NFT and associated balance updates.

5. Fee Claims:
    - `claimCreatorFeeNonCreator`: Tests the rejection of a fee claim attempt by a non-creator account.
    - `claimCreator`: Confirms that the creator can claim their deserved fee.
    - `claimPlatform`: Tests the accurate calculation and claiming of platform fees.

## Security Approach of the Project

### Successful current security understanding of the project

First they managed the 1st audit process with Code4rena. In addition, the Canto platform has had continuous audits at every stage by an auditing company such as Code4rena, which shows how much importance they attach to auditing.

### What the project should add in the understanding of Security?

1. By distributing the project to testnets, this ensures that the audits are carried out in onchain audit (this will increase coverage).

2. After the project is published on the mainnet, there should be emergency action plans (not found in the documents).

3. Add On-Chain Monitoring System; If On-Chain Monitoring systems, such as Forta, are added to the project, security will increase.

    For example, this [bot](https://app.forta.network/bot/0x7f9afc392329ed5a473bcf304565adf9c2588ba4bc060f7d215519005b8303e3) tracks any DEFI transactions in which wrapping, unwrapping, swapping, depositing, or withdrawals occur over a threshold amount. If transactions occur with unusually high token amounts, the bot sends out an alert. 

4. After the Code4rena audit is completed and the project is live, I recommend the audit process to continue. 

## Packages and Dependencies Analysis

| Package | Version | Usage in the project | Audit Recommendation |
| ------- | ------ | ---------------- | ----- |
| [`openzeppelin`](https://www.npmjs.com/package/@openzeppelin/contracts) | [npm v5.0.1](https://www.npmjs.com/package/@openzeppelin/contracts) | Ownable2Step.sol, SafeERC20.sol, ERC1155.sol | Version `4.9.0` is used by the project. It is recommended to use the newest version `5.0.0` |

## Other recommendations

1. The use of assembly in project codes is very low. I especially recommend using such useful and gas-optimized [code patterns](https://github.com/dragonfly-xyz/useful-solidity-patterns/tree/main/patterns/assembly-tricks-1).

2. It is seen that the latest versions of imported important libraries, such as Openzeppelin, are not used in the project codes, it should be noted. See [here](https://security.snyk.io/package/npm/@openzeppelin%2Fcontracts/).

3. A good model can be used to systematically assess the risk of the project, for example [this modeling](https://www.notion.so/Smart-Contract-Risk-Assessment-3b067bc099ce4c31a35ef28b011b92ff#7770b3b385444779bf11e677f16e101e) is recommended.

## New insights and learning from this audit

1. **Overview of Application Specific Dollar (asD)**: asD is an innovative protocol allowing anyone to create stablecoins pegged to 1 NOTE. The unique aspect is that all yield generated goes to the creator of the stablecoin. This feature distinguishes asD from other stablecoin protocols, offering a novel way for creators to benefit from their creations.

2. **`asDFactory`**: The `asDFactory` is a specialized tool for creating new asD tokens. It uniquely allows the creation of tokens with non-unique names and symbols, broadening accessibility. The `isAsD` mapping is a critical feature for integrating contracts to recognize legitimate asD tokens. This inclusivity in token creation and verification is a distinct learning point.

3. **Minting and Burning in asD**: Minting in asD requires an equivalent amount of NOTE, which is then deposited into the Canto Lending Market (CLM). This process of converting to cNOTE is a crucial learning aspect. Burning asD tokens allows users to retrieve the corresponding NOTE, providing a clear 1:1 exchange mechanism. The withdrawal of accrued interest by the creator without affecting the exchange rate presents a unique financial model.

4. **1155tech Protocol**: 1155tech introduces a flexible system for creating shares with various bonding curves, currently supporting a linear model. The division of sale fees among creators, platform, and holders introduces a balanced economic model. The ability to mint and burn ERC1155 tokens for a fee based on current prices adds a layer of complexity and opportunity for users, distinguishing it from other token models.

5. **Creating, Buying, Selling, and Claiming in 1155tech**: Share creation in 1155tech can be either permissionless or restricted, offering a versatile approach to share management. The automatic claiming of token holder rewards during buy/sell actions ensures fair distribution and incentivization. The selling process includes fee deduction, highlighting the need for sustainable fee structures. The specific functions for claiming various fees (platform, holder, creator) demonstrate a detailed and equitable fee distribution system.

### Time spent
14 hours

**[OpenCoreCH (Canto) acknowledged](https://github.com/code-423n4/2023-11-canto-findings/issues/514#issuecomment-1827459939)**

**[0xTheC0der (judge) commented](https://github.com/code-423n4/2023-11-canto-findings/issues/514#issuecomment-1832690599):**
 > Selected for report due to graphs showing the underlying calls of the protocol's main entry points.

***


# Disclosures

C4 is an open organization governed by participants in the community.

C4 Audits incentivize the discovery of exploits, vulnerabilities, and bugs in smart contracts. Security researchers are rewarded at an increasing rate for finding higher-risk issues. Audit submissions are judged by a knowledgeable security researcher and solidity developer and disclosed to sponsoring developers. C4 does not conduct formal verification regarding the provided code but instead provides final verification.

C4 does not provide any guarantee or warranty regarding the security of this project. All smart contract software should be used at the sole risk and responsibility of users.
