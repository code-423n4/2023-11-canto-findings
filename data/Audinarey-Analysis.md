## Overview of Canto Application Specific Dollar and Bonding Curves

Application Specific Dollar (asD) is a protocol that allows anyone to create stablecoins (pegged to 1 NOTE), with all yield going to the creator.

1155tech is an art protocol that will use asD as its currency. In contrast to existing bonding curve protocols, users can pay a fee to mint ERC1155 tokens based on their shares. While they do not earn any trading fees for those ERC1155 tokens, they can be traded on the secondary market and used wherever ERC1155 tokens are supported (for instance as a profile picture).

## Audit approach
I followed the steps below while analyzing and auditing the code base:
- Read the audit Readme.md and took the required notes.
- Analyzed the overall codebase on iterations very fast (Since there were 4 contracts in scope).
- Study of documentation provided in the attached links to understand how each contract interacts with the other contracts.
- Since the codebase has not been previously audited I went through the bot race findings


## Codebase quality analysis
Both the ```1155tech-contracts``` and the ```asD``` are typical of a security by design codebase.
Typical demonstration of Modularity, object Calisthenics and reusability. Most worthy of note that I'll like to mention is the ```LinearBondingCurve.getPriceAndFee(...)``` function. which at first glance seems incapable of handling ```buy```s and ```sell```s. However, with proper input parameter management, the ```LinearBondingCurve.getPriceAndFee(...)``` was put to good use severally in the ```Market``` contract.

State changes and updates were properly performed as at when due in different parts of all the contracts in scope.


## Time spent on analysis
21 Hours


### Time spent:
21 hours