LinearBondingCurve.sol
Gas Issue:
function getPriceAndFee(uint256 shareCount, uint256 amount)
   calls getFee looped on amount
        function getFee(uint256 shareCount)
           calls log2 each time
    log2 is computationally expensive
https://github.com/code-423n4/2023-11-canto/blob/335930cd53cf9a137504a57f1215be52c6d67cb3/1155tech-contracts/src/bonding_curve/LinearBondingCurve.sol#L20-L24
Once larger numbers of shares have been achieved,
For users looking to purchase multiple shares, the gas consumption rises for just computing the fee for each additional share.
Based on the definition of log2, when a fee computed for share (N) is not equal to the previous fee, the next (N-1) shares will have the same fee

Assuming shares will be issued in non-trivial numbers with multiple users.
One way to reduce the gas consumption for computing fees is to reduce the calls to log2

Concept:
   if log2(shareCount) == log2(shareCount+amount) all fees per share are the same
   the fee for one share can be multiplied by amount, eliminating multiple calls to log2
https://github.com/craig-iam-smith/2023-11-canto/blob/1bfa3056db5edec3d9a205e3f0e1586c57086567/1155tech-contracts/src/bonding_curve/LinearBondingCurve.sol#L20-L37

as the shareCount increases the gas savings will increase for large share purchases
https://github.com/craig-iam-smith/2023-11-canto/blob/1bfa3056db5edec3d9a205e3f0e1586c57086567/1155tech-contracts/src/test/LinearBondingCurve.t.sol#L29-L43

The tests outlined above here are the gas optimization results:
testGetPrice100() (gas: 72278) -> (gas:41877)
testGetPrice1000() (gas: 669724) -> (gas: 359423) 

so savings will vary, but can be greater than 50% for getPriceAndFee