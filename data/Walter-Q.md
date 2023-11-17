## Market
1) it is useless to check if _tokenCount_ is greater than 0,because if not,the txn have already been reverted since the transferFrom of the tokens (line 289 to 294)