- Lack of Function to rescue tokens

The **Market** contract deals with user funds, but there is no rescue function. Any funds that are sent to the contract by mistake are forever locked and lost.

My advice is to add an `onlyOwner` method to support rescuing tokens that are sent mistakenly.