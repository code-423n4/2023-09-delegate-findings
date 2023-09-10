## Gas Optimizations

## 1. Replace OZ ERC721 with solady ERC721

[https://github.com/code-423n4/2023-09-delegate/blob/main/src/PrincipalToken.sol#L6](https://github.com/code-423n4/2023-09-delegate/blob/main/src/PrincipalToken.sol#L6)

Solady's ERC721 implementation is more gas efficient than OZ's version. We recommend to switch to that library instead.

Reference:
[https://github.com/Vectorized/solady/blob/main/src/tokens/ERC721.sol](https://github.com/Vectorized/solady/blob/main/src/tokens/ERC721.sol)