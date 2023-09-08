# Missing NatSpec for Descriptions

Some contract missing NatSpec in description, add **@notice/@dev** notation to describe the function of the contract to improve code documentation. 

*There is 9 instance of this issue.*

```solidity
File : src/libraries/CreateOffererLib.sol

76 : struct Receivers {
```

[[76](https://github.com/code-423n4/2023-09-delegate/blob/a6dbac8068760ee4fc5bababb57e3fe79e5eeb2e/src/libraries/CreateOffererLib.sol#L76)]

```solidity
File : src/PrincipalToken.sol

27 : function _checkDelegateTokenCaller() internal view {

50 : function isApprovedOrOwner(address account, uint256 id) external view returns (bool) {

54 : function tokenURI(uint256 id) public view override returns (string memory) {
```

[[27](https://github.com/code-423n4/2023-09-delegate/blob/a6dbac8068760ee4fc5bababb57e3fe79e5eeb2e/src/PrincipalToken.sol#L27), [50](https://github.com/code-423n4/2023-09-delegate/blob/a6dbac8068760ee4fc5bababb57e3fe79e5eeb2e/src/PrincipalToken.sol#L50), [54](https://github.com/code-423n4/2023-09-delegate/blob/a6dbac8068760ee4fc5bababb57e3fe79e5eeb2e/src/PrincipalToken.sol#L54)]

```solidity
File : src/libraries/DelegateTokenLib.sol

90 : library DelegateTokenHelpers {

91 : function revertOnCallingInvalidFlashloan(DelegateTokenStructs.FlashInfo calldata info) internal {

96 : function revertOnInvalidERC721ReceiverCallback(address from, address to, uint256 delegateTokenId, bytes calldata data) internal {

101 : function revertOnInvalidERC721ReceiverCallback(address from, address to, uint256 delegateTokenId) internal {

113 : function delegateIdNoRevert(address caller, uint256 salt) internal pure returns (uint256) {
```

[[90](https://github.com/code-423n4/2023-09-delegate/blob/a6dbac8068760ee4fc5bababb57e3fe79e5eeb2e/src/libraries/DelegateTokenLib.sol#L90), [91](https://github.com/code-423n4/2023-09-delegate/blob/a6dbac8068760ee4fc5bababb57e3fe79e5eeb2e/src/libraries/DelegateTokenLib.sol#L91), [96](https://github.com/code-423n4/2023-09-delegate/blob/a6dbac8068760ee4fc5bababb57e3fe79e5eeb2e/src/libraries/DelegateTokenLib.sol#L96), [101](https://github.com/code-423n4/2023-09-delegate/blob/a6dbac8068760ee4fc5bababb57e3fe79e5eeb2e/src/libraries/DelegateTokenLib.sol#L101), [113](https://github.com/code-423n4/2023-09-delegate/blob/a6dbac8068760ee4fc5bababb57e3fe79e5eeb2e/src/libraries/DelegateTokenLib.sol#L113)]