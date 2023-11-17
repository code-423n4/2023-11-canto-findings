# [L-01] approve can revert if the current approval is not zero
Some tokens like USDT check for the current approval, and they revert if it's not zero. 
File:
https://github.com/code-423n4/2023-11-canto/blob/335930cd53cf9a137504a57f1215be52c6d67cb3/asD/src/asD.sol#L51

# [L-02] old version of openzeppelin is used
File:
https://github.com/code-423n4/2023-11-canto/blob/335930cd53cf9a137504a57f1215be52c6d67cb3/1155tech-contracts/lib/openzeppelin-contracts/package.json#L4
https://github.com/code-423n4/2023-11-canto/blob/335930cd53cf9a137504a57f1215be52c6d67cb3/1155tech-contracts/lib/openzeppelin-contracts/package.json#L4