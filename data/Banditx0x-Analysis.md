## Protocol Overview

Delegate is a contract which stores a token and then gives out an NFT which reperesents the delegation rights and a principal token which represents the right to claim the underlying. It can take in ERC721, ERC20 and ERC115 tokens.

Each creation of a delegation split involves entering an expiry date. After the expiry date is reached, the delegateToken is burned and the underlying token can be withdrawn from the vault by the principal token holder.

The seperation of delegation rights from ownership of the token is extremely useful. Here are examples of why you may use it.

## Codebase Quality Analysis

However here are some constructive feedback from an auditor's perspective on the codebase:

The function names can be confusing

For example, "delegate" is used in function and variable names to mean "the person doing the delegating". However it is also used as the verb "the act of delegating". To add further confusion, there are other codebases which use "delegate" to mean the person being delegated to. For example, the Livepeer contest which took place just before used delegate to mean the opposite - the person that received the delegation. I propose a more differentiated naming convention:

- Delegate: the act of delegating
- Delegator: The person who delegates to somebody else
- Delegatee: The person who recieves the delegation

Another instance of naming was for example `BURN_NOT_AUTHORIZED` being used for access control. It was hard to understand the logic between the changing of this variable and what conditions burn was authorized. It turns out that it was a form of re-entrancy protection, but it was difficult to figure out.

The codebase was extremely well written. The use of composition over inheritance diverged from the usual coding sytle of solidity contracts. This made it difficult to pattern match with previous codebases.

- More code comments describing what functions intend to do
- More seperation/spaces between lines within functions to denote different chunks of logic
- Function names which better represent what the code actually does.

## Architecture

The contract uses alot of techniques to reduce gas. One example is that it stores 3 addresses, which is 160 bits each, or 480 bits in total, into 2 storage slots, which is 512 bits.

It does this by shifting the bits representing the data of 2 addresses to the right, and then appending data from the 3rd address to the end.

From a gas perspective this saves gas by reducing calls to storage.
From an auditing perspective this made the code both hard to understand.

## Auditing Approach

My most successful approach in this audit was thinking of all the different types of tokens that could interact with this protocol. In the code4rena specification, all ERC20's were accepted. This opens up potential issues with ERC777 re-entrancies, rebasing tokens, blacklisted tokens etc. I went over all the types of tokens that I could think of and wondered what would happen if they were used as the underlying for the delegate contarct.

However, most of my time was spent trying to gain a thorough understanding of the protocol. However, I didn't uncover many issues from the understanding I gained during this timeboxed audit. I attribute this to two reasons:

- The codebase was well thought out from a security perspective
- There wasn't much documentation or code comments
- The codebase was hard to understand due to its complexity
- Delegate is a unique codebase. I haven't seen the separation of ownership and delegation rights implemented via a smart contract before.





### Time spent:
40 hours