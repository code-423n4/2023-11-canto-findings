https://github.com/code-423n4/2023-11-canto/blob/335930cd53cf9a137504a57f1215be52c6d67cb3/1155tech-contracts/src/Market.sol#L280
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
            shareData[_id].shareHolderRewardsPerTokenScaled += (shareHolderFee * 1e18) / _tokenCount;
        } else {
            // If there are no tokens in circulation, the fee goes to the platform
            platformFee += shareHolderFee;
        }
        platformPool += platformFee;
    }

#EXplanation 
uint256 shareHolderFee = (_fee * HOLDER_CUT_BPS) / 10_000;
uint256 shareCreatorFee = (_fee * CREATOR_CUT_BPS) / 10_000;
theses two calcualations provide the same output because the fee is same,
HOLDER_CUT_BPS and CREATOR_CUT_BPS is equal to 
https://github.com/code-423n4/2023-11-canto/blob/335930cd53cf9a137504a57f1215be52c6d67cb3/1155tech-contracts/src/Market.sol#L15
uint256 public constant HOLDER_CUT_BPS = 3_300; // 33%
https://github.com/code-423n4/2023-11-canto/blob/335930cd53cf9a137504a57f1215be52c6d67cb3/1155tech-contracts/src/Market.sol#L16
uint256 public constant CREATOR_CUT_BPS = 3_300; // 33%

#suggestion
calculate single value and assign to another reduce the computation.

