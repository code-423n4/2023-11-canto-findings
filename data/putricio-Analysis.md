haven't found any issue in the codebase yet, but i see you are using a compoundV2 fork for cToken. there is a known first deposit bug that maybe you already know about, but generally for any compoundV2 fork when launching new markets is recommended to mint some cTokens and burn them to make sure the total supply never goes to zero because of how rounding is handled. 

theres some interesting articles about this 
https://www.akshaysrivastav.com/articles/first-deposit-bug-in-compound-v2
https://twitter.com/hexagate_/status/1650177762336944137

### Time spent:
10 hours