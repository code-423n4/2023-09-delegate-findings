## TITLE
use _safemint instead of _mint

## Relevant GitHub Links
https://github.com/code-423n4/2023-09-delegate/blob/a6dbac8068760ee4fc5bababb57e3fe79e5eeb2e/src/PrincipalToken.sol#L35

## Summary
Use safeMint instead of mint for ERC721 

## Tools Used
Manual code analysis

## Recommended mitigation steps
According to OpenZeppelin usage of _mintis discouraged,use _safeMint whenever possible.