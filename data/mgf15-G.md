|ID|Title|Category|Severity|Instances|
|-|:-|:-:|:-:|:-:|
| [[G-1](#g-1--Possible-Optimizations-in-getFee)] | Possible Optimizations in getFee  | Gas Optimization | Informational | 1 |
| [[G-2](#g-2--Possible-Optimizations-in-getPriceAndFee)] | Possible Optimizations in getPriceAndFee | Gas Optimization | Informational | 1 |
| [[G-3](#g-3--Possible-Optimizations-in-_splitFees)] | Possible Optimizations in _splitFees  | Gas Optimization | Informational | 1 |
| [[G-4](#g-4--Use-custom-error)] | Use custom error  | Gas Optimization | Informational | 3 |


## **G-1** | Possible Optimizations in getFee 

[LinearBondingCurve.sol#L27-L36](https://github.com/code-423n4/2023-11-canto/blob/335930cd53cf9a137504a57f1215be52c6d67cb3/1155tech-contracts/src/bonding_curve/LinearBondingCurve.sol#L27-L36)

```diff
diff --git a/LinearBondingCurve.sol b/aLinearBondingCurve.sol
index f0c6115..7c53347 100644
--- a/LinearBondingCurve.sol
+++ b/aLinearBondingCurve.sol
@@ -29,10 +29,19 @@ contract LinearBondingCurve is IBondingCurve {
         if (shareCount > 1) {
             divisor = log2(shareCount);
         } else {
-            divisor = 1;
+            return 1e17;
         }
+        uint256 result;
+        uint256 num = 1e17;
+        assembly{
+
+            result := div(num, divisor)
+
+        }
+        return result;
     }


```

> Gas diff

```diff
 | src/bonding_curve/LinearBondingCurve.sol:LinearBondingCurve contract |                 |      |        |      |         |
 |----------------------------------------------------------------------|-----------------|------|--------|------|---------|
 | Deployment Cost                                                      | Deployment Size |      |        |      |         |
-| 132774                                                               | 817             |      |        |      |         |
+| 127968                                                               | 793             |      |        |      |         |
 | Function Name                                                        | min             | avg  | median | max  | # calls |
-| getPriceAndFee                                                       | 860             | 1081 | 860    | 2852 | 12      |
+| getPriceAndFee                                                       | 821             | 1035 | 821    | 2753 | 12      |
 

```

## **G-2** | Possible Optimizations in getPriceAndFee

[LinearBondingCurve.sol#L14-L25](https://github.com/code-423n4/2023-11-canto/blob/335930cd53cf9a137504a57f1215be52c6d67cb3/1155tech-contracts/src/bonding_curve/LinearBondingCurve.sol#L14-L25)

```diff
diff --git a/LinearBondingCurve.sol b/aLinearBondingCurve.sol
index f0c6115..a94839e 100644
--- a/LinearBondingCurve.sol
+++ b/aLinearBondingCurve.sol
@@ -17,11 +17,29 @@ contract LinearBondingCurve is IBondingCurve {
         override
         returns (uint256 price, uint256 fee)
     {
-        for (uint256 i = shareCount; i < shareCount + amount; i++) {
-            uint256 tokenPrice = priceIncrease * i;
-            price += tokenPrice;
-            fee += (getFee(i) * tokenPrice) / 1e18;
-        }
+        uint256 i = shareCount;
+        uint256 len = i + amount;
+        uint256 tokenPrice;
+        uint256 _price = priceIncrease;
+        uint256 _fee;
+        uint256 num = 1e18;
+        do {
+            _fee = getFee(i);
+            assembly{ 
+                
+                tokenPrice := mul(_price,i) /// tokenPrice = priceIncrease * i
+                price := add(price,tokenPrice) /// price += tokenPrice
+                let f := mul(_fee,tokenPrice) /// getFee(i) * tokenPrice
+                let x := div(f,num) /// (getFee(i) * tokenPrice) / 1e18
+                fee := add(fee,x) /// fee += (getFee(i) * tokenPrice) / 1e18
+
+            }
+            unchecked {
+                ++i;
+            }
+        } while(i < len);
     }

```

> Gas diff

```diff
 | src/bonding_curve/LinearBondingCurve.sol:LinearBondingCurve contract |                 |      |        |      |         |
 |----------------------------------------------------------------------|-----------------|------|--------|------|---------|
 | Deployment Cost                                                      | Deployment Size |      |        |      |         |
 | 132774                                                               | 817             |      |        |      |         |
 | Function Name                                                        | min             | avg  | median | max  | # calls |
-| getPriceAndFee                                                       | 860             | 1081 | 860    | 2852 | 12      |
+| getPriceAndFee                                                       | 533             | 651  | 533    | 1601 | 12      |

```

## **G-3** | Possible Optimizations in _splitFees 

[Market.sol#L280-L296](https://github.com/code-423n4/2023-11-canto/blob/335930cd53cf9a137504a57f1215be52c6d67cb3/1155tech-contracts/src/Market.sol#L280-L296)
```diff
diff --git a/Market.sol b/aMarket.sol
index 59c5c96..6ee776f 100644
--- a/Market.sol
+++ b/aMarket.sol
@@ -282,10 +282,25 @@ contract Market is ERC1155, Ownable2Step {
         uint256 _fee,
         uint256 _tokenCount
     ) internal {
-        uint256 shareHolderFee = (_fee * HOLDER_CUT_BPS) / 10_000;
-        uint256 shareCreatorFee = (_fee * CREATOR_CUT_BPS) / 10_000;
+        //@audit use assembly for math
+        uint256 shareHolderFee ;
+        uint256 shareCreatorFee ;
+        assembly {
+
+            // Calculate (_fee * HOLDER_CUT_BPS)
+            let mulResult := mul(_fee, HOLDER_CUT_BPS)
+            // Calculate (_fee * CREATOR_CUT_BPS)
+            let mulResult2 := mul(_fee,CREATOR_CUT_BPS)
+
+            // Calculate (_fee * HOLDER_CUT_BPS) / 10_000
+            shareHolderFee := div(mulResult, 10000)
+            // Calculate (_fee * CREATOR_CUT_BPS) / 10_000
+            shareCreatorFee := div(mulResult2, 10000)
+
+        }

```

> Gas diff

*after change in G1 - G2*

```diff
diff --git a/1.txt b/3.txt
index a3d36f2..6351b57 100644
--- a/1.txt
+++ b/3.txt
@@ -1,32 +1,32 @@
 | src/Market.sol:Market contract |                 |        |        |        |         |
 |--------------------------------|-----------------|--------|--------|--------|---------|
 | Deployment Cost                | Deployment Size |        |        |        |         |
-| 2499761                        | 13158           |        |        |        |         |
+| 2496754                        | 13143           |        |        |        |         |
 | Function Name                  | min             | avg    | median | max    | # calls |
-| burnNFT                        | 49135           | 49135  | 49135  | 49135  | 1       |
-| buy                            | 157773          | 157773 | 157773 | 157773 | 4       |
+| burnNFT                        | 48832           | 48832  | 48832  | 48832  | 1       |
+| buy                            | 157395          | 157395 | 157395 | 157395 | 4       |
 | changeBondingCurveAllowed      | 2068            | 21128  | 26484  | 26484  | 9       |
 | createNewShare                 | 10479           | 101373 | 116522 | 116522 | 7       |
-| getBuyPrice                    | 3194            | 5112   | 5112   | 7030   | 2       |
-| mintNFT                        | 68280           | 68280  | 68280  | 68280  | 2       |
+| getBuyPrice                    | 2585            | 4642   | 4642   | 6700   | 2       |
+| mintNFT                        | 67978           | 67978  | 67978  | 67978  | 2       |
 | restrictShareCreation          | 5784            | 5784   | 5784   | 5784   | 6       |
-| sell                           | 42142           | 42142  | 42142  | 42142  | 1       |
+| sell                           | 41840           | 41840  | 41840  | 41840  | 1       |
 | shareIDs                       | 1543            | 1543   | 1543   | 1543   | 6       |
 | whitelistedBondingCurves       | 721             | 721    | 721    | 721    | 2       |
 
 
-| src/bonding_curve/LinearBondingCurve.sol:LinearBondingCurve contract |                 |      |        |      |         |
-|----------------------------------------------------------------------|-----------------|------|--------|------|---------|
-| Deployment Cost                                                      | Deployment Size |      |        |      |         |
-| 132774                                                               | 817             |      |        |      |         |
-| Function Name                                                        | min             | avg  | median | max  | # calls |
-| getPriceAndFee                                                       | 860             | 1081 | 860    | 2852 | 12      |
+| src/bonding_curve/LinearBondingCurve.sol:LinearBondingCurve contract |                 |     |        |      |         |
+|----------------------------------------------------------------------|-----------------|-----|--------|------|---------|
+| Deployment Cost                                                      | Deployment Size |     |        |      |         |
+| 123168                                                               | 769             |     |        |      |         |
+| Function Name                                                        | min             | avg | median | max  | # calls |
+| getPriceAndFee                                                       | 530             | 658 | 530    | 1685 | 12      |
 
 
 | src/test/Market.t.sol:MarketTest contract |                 |     |        |     |         |
 |-------------------------------------------|-----------------|-----|--------|-----|---------|
 | Deployment Cost                           | Deployment Size |     |        |     |         |
-| 5773044                                   | 28660           |     |        |     |         |
+| 5760425                                   | 28597           |     |        |     |         |
 | Function Name                             | min             | avg | median | max | # calls |
 | onERC1155Received                         | 977             | 977 | 977    | 977 | 2       |

```

## **G-4** | Use custom error 


[asD.sol#L54](https://github.com/code-423n4/2023-11-canto/blob/335930cd53cf9a137504a57f1215be52c6d67cb3/asD/src/asD.sol#L54)

[asD.sol#L64](https://github.com/code-423n4/2023-11-canto/blob/335930cd53cf9a137504a57f1215be52c6d67cb3/asD/src/asD.sol#L64)

[asD.sol#L86](https://github.com/code-423n4/2023-11-canto/blob/335930cd53cf9a137504a57f1215be52c6d67cb3/asD/src/asD.sol#L86)

```diff
diff --git a/asD.sol b/aasD.sol
index ce486fa..d233ae6 100644
--- a/asD.sol
+++ b/aasD.sol
@@ -18,7 +18,9 @@ contract asD is ERC20, Ownable2Step {
                                  EVENTS
     //////////////////////////////////////////////////////////////*/
     event CarryWithdrawal(uint256 amount);
-
+    
+    error Errorwhenredeeming();
+    error Errorwhenminting();
     /// @notice Initiates CSR on main- and testnet
     /// @param _name Name of the token
     /// @param _symbol Symbol of the token
@@ -51,17 +53,18 @@ contract asD is ERC20, Ownable2Step {
         SafeERC20.safeApprove(note, cNote, _amount);
         uint256 returnCode = cNoteToken.mint(_amount);
         // Mint returns 0 on success: https://docs.compound.finance/v2/ctokens/#mint
-        require(returnCode == 0, "Error when minting");
+        if(returnCode != 0) revert Errorwhenminting(); //require(returnCode == 0, "Error when minting");
         _mint(msg.sender, _amount);
     }
 
     /// @notice Burn amount of asD tokens to get back NOTE. Like when minting, the NOTE:asD exchange rate is always 1:1
     /// @param _amount Amount of tokens to burn
     function burn(uint256 _amount) external {
+        //@audit use custom error 
         CErc20Interface cNoteToken = CErc20Interface(cNote);
         IERC20 note = IERC20(cNoteToken.underlying());
         uint256 returnCode = cNoteToken.redeemUnderlying(_amount); // Request _amount of NOTE (the underlying of cNOTE)
-        require(returnCode == 0, "Error when redeeming"); // 0 on success: https://docs.compound.finance/v2/ctokens/#redeem-underlying
+        if(returnCode != 0) revert Errorwhenredeeming(); //require(returnCode == 0, "Error when redeeming"); // 0 on success: https://docs.compound.finance/v2/ctokens/#redeem-underlying
         _burn(msg.sender, _amount);
         SafeERC20.safeTransfer(note, msg.sender, _amount);
     }
@@ -83,9 +86,10 @@ contract asD is ERC20, Ownable2Step {
         // Technically, _amount can still be 0 at this point, which would make the following two calls unnecessary.
         // But we do not handle this case specifically, as the only consequence is that the owner wastes a bit of gas when there is nothing to withdraw
         uint256 returnCode = CErc20Interface(cNote).redeemUnderlying(_amount);
-        require(returnCode == 0, "Error when redeeming"); // 0 on success: https://docs.compound.finance/v2/ctokens/#redeem
+        if(returnCode != 0 ) revert Errorwhenredeeming();//require(returnCode == 0, "Error when redeeming"); // 0 on success: https://docs.compound.finance/v2/ctokens/#redeem
         IERC20 note = IERC20(CErc20Interface(cNote).underlying());
         SafeERC20.safeTransfer(note, msg.sender, _amount);
         emit CarryWithdrawal(_amount);
     }
 }
+

```
```diff
diff --git a/1.txt b/2.txt
index d8594dc..1b19605 100644
--- a/1.txt
+++ b/2.txt
@@ -1,24 +1,24 @@
 | src/asD.sol:asD contract |                 |        |        |        |         |
 |--------------------------|-----------------|--------|--------|--------|---------|
 | Deployment Cost          | Deployment Size |        |        |        |         |
-| 1309446                  | 7800            |        |        |        |         |
+| 1291833                  | 7712            |        |        |        |         |
 | Function Name            | min             | avg    | median | max    | # calls |
 | balanceOf                | 587             | 587    | 587    | 587    | 7       |
-| burn                     | 43908           | 43908  | 43908  | 43908  | 1       |
+| burn                     | 43884           | 43884  | 43884  | 43884  | 1       |
 | mint                     | 147403          | 147403 | 147403 | 147403 | 5       |
 | name                     | 859             | 859    | 859    | 859    | 1       |
 | owner                    | 523             | 523    | 523    | 523    | 1       |
 | symbol                   | 1126            | 1126   | 1126   | 1126   | 1       |
 | totalSupply              | 329             | 329    | 329    | 329    | 3       |
-| withdrawCarry            | 2675            | 25082  | 25629  | 46396  | 4       |
+| withdrawCarry            | 2675            | 25064  | 25611  | 46361  | 4       |
 
 
 | src/asDFactory.sol:asDFactory contract |                 |         |         |         |         |
 |----------------------------------------|-----------------|---------|---------|---------|---------|
 | Deployment Cost                        | Deployment Size |         |         |         |         |
-| 1841943                                | 9472            |         |         |         |         |
+| 1824321                                | 9384            |         |         |         |         |
 | Function Name                          | min             | avg     | median  | max     | # calls |
-| create                                 | 1372101         | 1372101 | 1372101 | 1372101 | 4       |
+| create                                 | 1354471         | 1354471 | 1354471 | 1354471 | 4       |
 | isAsD                                  | 438             | 438     | 438     | 438     | 1       |

```