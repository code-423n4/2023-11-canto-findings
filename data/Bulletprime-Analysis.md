Brief Overview of the prtotocol 
This protocol allows anyone to create stablecoins (pegged to 1 NOTE), with all yield going to the creator. The protocol is used by the 1155tech art protocol, which allows users to pay a fee to mint ERC1155 tokens based on their shares. The minting fee goes to the creator of the share. The ERC1155 tokens can be used on the secondary market and wherever ERC1155 tokens are supported. The asDFactory is used to create new asD tokens, and it keeps track of the created tokens in the is `asD` mapping, which allows integrating contracts to query if a given address is a legit asD.

Executive summary 
My analysis focuses on some of the potential and additional risk that comes from the composability within the contract in scope. The aim of this analysis is to show my adversarial thoghts and help suggest a wide security process thats going to help the mainatainability of the codebase making it safe for n Users. I begin by outlining the code audit approach employed for the in-scope contracts, followed by sharing my perspective on the architecture, and concluding with observations about the implementation’s code.

Audit Approach
I paid attention to my successful steps that has proven to fetch me low hanging fruits.
1.Read the Audit Readme.md and took necessary notes.
2.Retentively created a visual picture of the contracts intent.
3.Gained knowledge on bonding curves in relation to stablecoins their proper implementations in addition to the ERC1155 token functionality and limitations.
4.Although no much documentation on this particular project, much time was spent reading about similar projects and how to improve and better the project.
5.Analyzed the over all codebase one iteration at a time.
6.Then I read old audits and already known findings on similar projects, taking notes of things that could go wrong, at the same time going through the bot races.
7.Then setup my testing environment, Execute the tests to checks all test passed. 
8.Finally, I started with auditing the code base in-depth, that way I started understanding line by line code and took the necessary notes to ask some questions.

THE AUDIT PROCESS
The first stage of the audit
During the initial stage of the audit, the primary focus was on analyzing gas usage and quality assurance (QA) aspects. This phase of the audit aimed to ensure the efficiency of gas consumption. 
The second stage
The focus shifted towards understanding the protocol usage in more detail. This involved identifying and analyzing the important contracts and functions within the system. By examining these key components, what the codes is aimed to gain and if it corelates to the protocol’s functionality.
The third stage
the focus was on thoroughly examining and marking out any doubtful or vulnerable areas within the protocol. This stage involved conducting comprehensive vulnerability assessments and identifying potential weaknesses in the system both through testing and manual audits.
The fourth stage
Comprises of a comprehensive analysis and testing of the previously identified doubtful and vulnerable areas were conducted. This stage involved diving deeper into these areas, performing in-depth examinations, and subjecting them to rigorous testing.

LEARNINGS
Although I personally didn't find any critical issue, I believe the codebase being simple, direct and less complicated helped reduced critical issues and developers should take advantage of this, but that should not go without prioritizing security, as code security is an ever improving process and should not be left in the hands of just a team but the entire community for more eye lookout.

Architecture overview
The protocol is quite complex, involving multiple modules and functionalities. Some recommendations that should be looked into involve 
Simplify Governance: The governance process for enabling or disabling the conversion of tokens can be simplified. This could involve reducing the number of steps required for a decision to be made, or providing clearer guidelines on when such decisions should be made.

Improve Documentation: The information in the docs are quite technical and might be difficult for non-experts to understand. Improving the documentation, providing more context, and breaking down the information into simpler terms could make it more accessible.
User Experience: The protocol should aim to provide a seamless and user-friendly experience. This could involve simplifying the process of converting between tokens and native coins, and providing clearer instructions for users.
Overall, the architecture seems to have considered various security measures and roles to mitigate potential risks that i must commend. However, it's crucial to regularly review and update these measures, conduct security audits, and stay updated with the latest best practices in the industry to ensure the ongoing security of the protocol.

Code commentary
The quality of the code appears to be good, in terms of security practices and code structure. However, there are always areas where a codebase can be improved. Here are some of my suggestions;
Code Comments: While the code is clear and understandable, adding comments can make it even more accessible, and easy to read especially for developers who might need to work with this code in the future.

Code Reusability: The contract uses an interface `IBondingCurve` for interacting with bonding curves, which is a good practice for code reusability and modularity. This could be extended to other parts of the code, where common functionalities can be abstracted into separate modules or contracts.

Centralization risks 
Centralization risk for privileged functions,Contracts with privileged functions need owner/admin to be trusted not to perform malicious updates or drain funds. So as to not cause a single point failure.
the centralization risk for privileged functions lies in the fact that certain functions can only be called by the contract owner. This means that the owner has complete control over these functions, and if the owner's account is compromised, the entire protocol could be at risk.
For instance, the `restrictShareCreation` function can be called only by the owner and it restricts or unrestricts share creation. Similarly, the `changeShareCreatorWhitelist` function adds or removes an address from the whitelist of share creators and can only be called by the owner.

To mitigate this risk, it's important to implement multi-signature functionality, where crucial functions require approval from multiple authorized parties. This adds an extra layer of security, as compromising one account is not enough to perform a malicious update.

Mechanism Review 
A mechanism review would involve a thorough analysis of the contract's functionality, its use of security mechanisms like access control and reentrancy protection, and its potential vulnerabilities.
Access Control: The `changeBondingCurveAllowed` function has an access control mechanism. It ensures that only the contract owner can change the state of a bonding curve. The recommendation would be to ensure that only necessary functions are restricted and that the access control is granular enough.
Code Reusability: The code uses interfaces for code reusability. This makes the code more modular and easier to maintain. The recommendation would be to continue using interfaces for code reusability.

Security Best Practices: The code follows security best practices by whitelisting bonding curves and ensuring that share creation is restricted to whitelisted addresses. The recommendation would be to continue following these best practices.

Error Handling: The contract uses revert statements to handle errors and revert transactions when necessary. However, the error messages could be more informative to help users understand why a transaction failed.

Code Organization: The code could benefit from better organization and more comments to explain the purpose and functionality of each function.

Possible System Risk
Third-Party Dependency Risk: Contracts rely on external data source, such as @openzeppelin/contracts, and there is a risk that if any issues are found with these dependencies in your contracts, the protocol could also be affected. 
Test Coverage Risks: With only 80% test coverage, the protocol may have undiscovered vulnerabilities. Inadequate security audits could leave critical issues unnoticed, increasing the risk of attacks.
Remember, security is a continuous process and these are just some initial observations. A thorough security review and testing are recommended to identify and address any potential vulnerabilities.


### Time spent:
24 hours