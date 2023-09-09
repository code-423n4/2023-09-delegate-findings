**Low-1: No check for maximal extension time**
In [function extend](https://github.com/code-423n4/2023-09-delegate/blob/main/src/DelegateToken.sol#L325-L336) an user can supply arbitrary extension period. This period is not limited and cannot be changed later. It may happen that due to an user error, someone will introduce an extremely large date in the future, due to which the NFT will be stuck forever.
It is high impact, but very low probability, so rating as low.
Recommendation: Introduce maximal extension period e.g. 1 year from current date.

**Info-1: Payable multicall**
In [DelegateRegistry](https://github.com/delegatexyz/delegate-registry/blob/6d1254de793ccc40134f9bec0b7cb3d9c3632bc1/src/DelegateRegistry.sol) there is a `multicall` function being payable that could be used by users. The risk associated with it is that if its used with functions that work with msg.value, it could be accounted multiple times due to the delegatecall being performed in a loop. It is a [known issue](https://github.com/Uniswap/v3-periphery/issues/52). Since so far there is no function within the contract that could be abused, it is reported as info, for future reference and increasing awareness.