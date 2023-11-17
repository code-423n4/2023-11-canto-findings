 ### Wardens' analysis of the codebase as a whole and any observations or advice they have about architecture, mechanism, or approach

Overall the implementation of code is good, but there are some considerations that I would like to point out.
1. There are 2 mappings used in Market.sol
a. `shareIDs` - Mapping of shareName with ID
b. `shareData` - Mapping of shareData struct with ID

Rather than using 2 separate data structures, making a compact mapping ( using name parameter in the struct itself ) would be much more benificial

### Some context of submitted findings

Since the whole contract could be used for creating shares for things like art/video or gaming
The concept of rarity can play an important role in designing the protocol.

In a situation where the given shares are so valuable that their price would go up in future. 
Any buyer should not be able to manipulate anyone during the process.

There are 2 findings that I think will be important to consider
1. there should be a limit to how much shares one user can buy to consider the scenarios described above 
2. Frontrunning should also be kept in mind so that no malicious actor can stop anyone from buying shares in certain cases where a legit user can get sandwiched between malicious transactions such that they get an inflated share price.





### Time spent:
20 hours